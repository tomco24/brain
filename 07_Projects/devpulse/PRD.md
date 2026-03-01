This is finally a project that:

- Is real
    
- Is useful
    
- Has serious async challenges
    
- Forces actor design
    
- Justifies event bus
    
- Has CI applicability
    
- Could evolve into a real product
    

We’ll design this properly. Not fluffy. Not vague.

---

# 📘 PRD — 2 Week Plan

# DevPulse: Local Event Intelligence Engine

## 1. Vision

DevPulse is a **local, event-driven observability agent** that:

- Runs as a CLI process (locally or in CI)
    
- Collects OS-level developer-relevant signals
    
- Normalizes them into structured event envelopes
    
- Publishes them to an internal async event bus
    
- Streams them via WebSocket to a live dashboard
    
- Supports filtering, backpressure, and supervision
    
- Gracefully handles burst loads
    

It is:

- Local-first
    
- No cloud required
    
- No external data dependencies
    
- Cross-platform (Linux first-class)
    
- Fully async and actor-based
    

---

# 2. Core Functional Scope (2-Week MVP)

## Included Actors

### 🧵 ProcessMonitorActor

- Poll active processes (psutil)
    
- Detect:
    
    - New process
        
    - Process termination
        
    - CPU spike above configurable threshold
        
    - Memory spike above configurable threshold
        
- Maintain snapshot state
    
- Emit structured ProcessEvent
    

---

### 📁 FileSystemWatcherActor

- Watch configured directories
    
- Detect:
    
    - Created
        
    - Modified
        
    - Deleted
        
- Debounce rapid events
    
- Optionally compute small diffs for small text files (< 100KB)
    
- Emit FileEvent
    

---

### 🌐 NetworkMonitorActor

- Poll active connections
    
- Detect:
    
    - New outbound connection
        
    - New listening port
        
- Maintain last-seen snapshot
    
- Emit NetworkEvent
    

---

### 🐳 DockerMonitorActor (Optional but recommended)

- Detect container start/stop
    
- Poll container resource usage
    
- Emit DockerEvent
    

If Docker integration risks scope explosion, stub and mock in week 1 and complete in week 2.

---

# 3. Architecture

## 3.1 Actor Runtime

### Requirements

- Typed Actor base class
    
- Each actor owns:
    
    - Mailbox (asyncio.Queue)
        
    - Lifecycle TaskGroup
        
    - Internal state
        
- Supervisor manages:
    
    - Startup
        
    - Shutdown
        
    - Restart-on-failure
        
- ActorRef abstraction (no direct object coupling)
    

Structured concurrency:

- One root TaskGroup
    
- Each actor has its own TaskGroup
    
- Supervisor handles failure propagation
    

---

## 3.2 Event Bus

Interface:

class EventBus:  
    async def publish(topic: str, event: Envelope[Any]) -> None  
    def subscribe(topic: str) -> AsyncIterator[Envelope[Any]]

Requirements:

- Topic-based pub/sub
    
- Multiple subscribers per topic
    
- Bounded queue per subscriber
    
- Backpressure strategy:
    
    - Drop oldest OR
        
    - Drop subscriber if lagging
        
- Cancellation-safe AsyncIterator
    

Topics:

- process.lifecycle
    
- process.spike
    
- filesystem.change
    
- network.connection
    
- docker.lifecycle
    
- docker.resource
    

---

## 3.3 Event Envelope

@dataclass(frozen=True, slots=True)  
class Envelope[T]:  
    event_id: UUID  
    topic: str  
    source: str  
    timestamp: datetime  
    payload: T

Payloads:

- ProcessEvent
    
- FileEvent
    
- NetworkEvent
    
- DockerEvent
    

Strict typing.  
mypy --strict compatible.

---

## 3.4 CLI Runtime

Command:

devpulse run --config config.yaml

Responsibilities:

- Parse config
    
- Initialize actors
    
- Start supervisor
    
- Start WebSocket server
    
- Handle SIGINT
    
- Coordinated shutdown
    

---

## 3.5 WebSocket Interface

Endpoint:

ws://localhost:8765/ws

Features:

- Topic subscription filter
    
- Stream JSON-serialized envelopes
    
- Heartbeat
    
- Detect slow clients
    
- Drop clients exceeding backpressure threshold
    
- Clean disconnect handling
    

---

## 3.6 Dashboard

Simple HTML + JS:

- Live event stream
    
- Filter by topic
    
- Highlight spikes
    
- Show event rate per topic
    
- Show actor health status
    

No React.  
No framework.  
Minimal.

---

# 4. Non-Functional Requirements

## Performance

- Handle 5,000+ events/minute without crash
    
- No memory leaks under 30-minute load
    

## Reliability

- Actor crash does not kill runtime
    
- Supervisor restarts actor
    
- Shutdown within 3 seconds on SIGINT
    

## Code Quality

- Python 3.12
    
- asyncio TaskGroup
    
- No orphan tasks
    
- 80%+ test coverage
    
- Strict typing
    

---

# 5. CI Use Case

In CI environment:

- Monitor:
    
    - Process spikes during build
        
    - Unexpected network calls
        
    - Docker container instability
        
- Export summary JSON at end of run
    
- Optionally:
    
    - Fail pipeline if anomalies detected
        

Future extension, but foundation must allow it.

---

# 6. 2-Week Execution Plan

---

# Week 1 — Concurrency Core

## Day 1–2: Actor Runtime

- Implement Actor base
    
- Implement Supervisor
    
- Implement ActorRef
    
- Lifecycle control
    
- Structured concurrency tree
    
- Restart-on-failure
    
- Unit tests for cancellation + restart
    

Deliverable:  
Stable actor runtime.

---

## Day 3–4: Event Bus

- Topic-based routing
    
- AsyncIterator subscription
    
- Bounded subscriber queues
    
- Backpressure policy
    
- Cancellation tests
    
- Stress test with burst publisher
    

Deliverable:  
Reliable in-memory bus.

---

## Day 5: ProcessMonitorActor

- Snapshot modeling
    
- CPU/memory threshold logic
    
- Spike detection algorithm
    
- Event emission
    
- State consistency tests
    

---

## Day 6: FileSystemWatcherActor

- Integrate watchdog or similar
    
- Thread → async bridge
    
- Debounce logic
    
- Event normalization
    
- Burst simulation test
    

---

## Day 7: Integration Test

- Start 2 actors
    
- Publish events
    
- Subscribe via dummy WebSocket client
    
- Verify flow
    
- Verify graceful shutdown
    

---

# Week 2 — Interface + Hardening

---

## Day 8–9: NetworkMonitorActor

- Connection snapshot diffing
    
- Outbound detection logic
    
- Event emission
    
- Performance profiling
    

---

## Day 10–11: WebSocket Server

- Async server
    
- Subscription filter
    
- JSON serialization
    
- Slow client handling
    
- Backpressure enforcement
    

Test:  
Simulate 50 clients.

---

## Day 12: Dashboard

- Simple HTML page
    
- Connect via WebSocket
    
- Render events
    
- Basic filtering
    

---

## Day 13: Hardening

- Memory leak test
    
- Cancellation stress test
    
- Actor crash simulation
    
- Backpressure overflow test
    
- Logging refinement
    

---

## Day 14: Polish + Documentation

- Architecture diagram
    
- Concurrency diagram
    
- Cancellation tree explanation
    
- README with usage
    
- CI pipeline config
    
- Example config.yaml
    

---

# 7. Future Extensions (Not in 2 Weeks)

- Plugin system
    
- Prometheus metrics
    
- Persistent event log
    
- Remote monitoring mode
    
- Slack alerts
    
- Security anomaly scoring
    
- ML-based anomaly detection
    

---

# 8. Key Async Challenges You Will Solve

- Bridging blocking OS calls safely
    
- Designing backpressure strategy
    
- Handling cancellation correctly
    
- Avoiding event-loop starvation
    
- Managing burst loads
    
- Designing structured shutdown
    
- Preventing slow client collapse
    
- Actor failure isolation
    

This is real async maturity.

---

# 9. Honest Mentor Notes

You must avoid:

- Overengineering dashboard
    
- Adding persistence
    
- Adding authentication
    
- Making it cross-platform-perfect
    
- Adding Kafka
    
- Adding cloud mode
    

Two weeks = tight, sharp, disciplined.

Focus on:

Runtime correctness > UI beauty.