---
type: concept
tags: [agile, scrum, estimation, story-points, planning-poker, velocity]
created: 2026-03-18
modified: 2026-03-18
status: draft
---

# Estimation & Story Points

## Overview
Story points are a unit of *relative effort* used to estimate the size of user stories. Unlike hours, they encode complexity, uncertainty, and risk — not just time. Teams use them to forecast how much work can fit in a sprint.

## Key Points
- Story points are **relative**, not absolute — a 5-point story is roughly twice as hard as a 2-point story
- They are a team-level measurement; comparing points across teams is meaningless
- Points capture effort + complexity + uncertainty, not just duration
- Velocity (points per sprint) emerges naturally and improves forecasting over time

## The Fibonacci Scale

Teams typically use a modified Fibonacci sequence to force deliberate distinctions:

```
1 — trivial (change a label, update a config value)
2 — small (simple CRUD endpoint with no edge cases)
3 — medium (feature with some complexity, known path)
5 — larger (multiple components, some uncertainty)
8 — large (significant unknowns, may need to split)
13 — very large (should be split before committing)
? — can't estimate (needs a spike first)
```

> Items estimated at 8+ should be considered for splitting into smaller stories.

## Planning Poker

A consensus-based estimation technique that prevents anchoring bias.

### How It Works
1. PO reads a user story and answers clarifying questions
2. Each developer privately selects a card (story point value)
3. Everyone reveals simultaneously
4. If consensus → that's the estimate
5. If spread (e.g., 2 and 8) → high and low estimators explain reasoning briefly → re-vote
6. Repeat until consensus

### Reference Story
Before estimating a batch of stories, agree on a **reference story** — a previously done story everyone remembers — and peg it to a specific point value. All new stories are estimated relative to it.

## Velocity

```
Velocity = story points completed per sprint (rolling average)
```

- Measure over 3–5 sprints before trusting it
- Use for **planning** (how many stories fit next sprint), not for performance evaluation
- Velocity naturally accounts for holidays, team changes, and tech debt paydown

### Capacity Planning
```
Sprint capacity ≈ velocity (average of last 3 sprints)
Adjust for: holidays, team member absence, known large meetings
```

## T-Shirt Sizing (Alternative)

For early-stage roadmap planning when stories aren't fully refined:

| Size | Rough equivalent |
|------|-----------------|
| XS | 1–2 points |
| S | 3 points |
| M | 5 points |
| L | 8 points |
| XL | 13+ (split before sprint) |

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Estimating in hours | Switch to points; hours invite false precision |
| One person dominates estimation | Use planning poker with simultaneous reveal |
| Points used as performance metric | Separate velocity (team planning tool) from output measurement |
| Never re-estimating as team learns | Recalibrate reference stories after major team or tech changes |
| Estimating stories that aren't ready | Refuse to estimate until acceptance criteria are clear |

## When to Use
- Sprint Planning for stories that are refined and have acceptance criteria
- Quarterly / PI planning for roadmap sizing (use T-shirt sizes here)

## When NOT to Use
- Kanban teams — cycle time and throughput replace estimation
- Spikes — spikes are time-boxed, not point-estimated

## Related Concepts
- [[Scrum Ceremonies]] — estimation happens in Sprint Planning
- [[User Stories & Backlog]] — stories must be refined before estimating
- [[Scrum Framework]] — velocity feeds sprint capacity

## References
- *Agile Estimating and Planning* — Mike Cohn
- [Mountain Goat Software — Story Points](https://www.mountaingoatsoftware.com/blog/what-are-story-points)
- [Planning Poker](https://www.planningpoker.com/) — free online tool
