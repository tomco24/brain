# DDD Development Process — Detailed Guide

---

## Phase 1 — PRD (Product Requirements Document)

**Goal:** Capture business intent without technical bias.

### What to include

- **Problem statement** — what pain or opportunity this addresses
- **User personas** — who uses it, their goals and constraints
- **Feature list** — what the system must do (user-facing capabilities)
- **Business rules** — invariants and policies from the business side
- **Non-functional requirements** — performance, compliance, SLA
- **Out of scope** — explicit exclusions to prevent scope creep
- **Success metrics** — how stakeholders will know it's done

### Rules

- No class names, database schemas, or API contracts here
- Use business language, not technical language
- Every feature must trace to a business goal

### Output

A document that a non-technical stakeholder can validate.

---

## Phase 2 — Event Storming

**Goal:** Discover the domain by exploring what happens, not what exists.

### Format

A collaborative workshop (domain experts + devs + QA). Use sticky notes on a timeline:

| Color | Represents | Example |
|---|---|---|
| Orange | Domain Event | `InvoiceSent`, `OrderPlaced` |
| Blue | Command | `SendInvoice`, `PlaceOrder` |
| Yellow | Actor | `Customer`, `Billing Clerk` |
| Purple | Policy | "When payment received, trigger shipment" |
| Pink | External System | Payment gateway, email provider |
| Yellow (large) | Aggregate | `Order`, `Invoice` |
| Red | Hotspot / question | Unresolved ambiguity |

### Process

1. **Chaotic exploration** — everyone dumps Domain Events on the timeline without order
2. **Enforce timeline** — sort events chronologically, remove duplicates
3. **Add commands** — what triggered each event? Who issued the command?
4. **Add policies** — automated reactions (when X happens, system does Y)
5. **Add aggregates** — what entity owns the state change?
6. **Mark hotspots** — unresolved conflicts, unclear ownership, language disagreements

### Output

- Raw domain map on a board (Miro, physical wall, etc.)
- Initial list of Domain Events in past-tense verb form
- First draft of ubiquitous language

---

## Phase 3 — Ubiquitous Language & Bounded Context Map

**Goal:** Formalize terminology and draw context boundaries.

### Ubiquitous Language Glossary

For each term used by domain experts, document:

```
Term: Invoice
Context: Billing
Definition: A financial document issued to a customer after order fulfillment requesting payment.
Distinct from: Order (pre-fulfillment), Receipt (post-payment)
```

Every term that means different things in different contexts belongs to **separate bounded contexts** — do not unify them with prefixes (`BillingInvoice` vs `ShippingInvoice`). Keep them separate.

### Bounded Context Identification

A bounded context is a boundary within which:
- The ubiquitous language is consistent
- A single team owns the model
- The model can evolve independently

Signs you need a new context:
- The same word means something different to two domain experts
- Two parts of the system have conflicting invariants around the same concept
- A model change in one area would break unrelated functionality

### Context Map Relationships

Document how contexts relate:

| Relationship | Meaning |
|---|---|
| **Upstream / Downstream** | One context feeds another |
| **Shared Kernel** | Two contexts share a small common model (use sparingly) |
| **Customer / Supplier** | Downstream dictates requirements to upstream |
| **Conformist** | Downstream conforms to upstream's model with no negotiation |
| **Anti-Corruption Layer (ACL)** | Downstream translates upstream's model to protect its own |
| **Open Host Service** | Upstream publishes a stable protocol for many consumers |
| **Published Language** | Shared interchange format (e.g. events on a message bus) |

### Output

- Ubiquitous language glossary (one per context)
- Context map diagram with named relationships
- Team ownership per context

---

## Phase 4 — BDD Feature Definitions (Gherkin)

**Goal:** Express acceptance criteria in domain language, per bounded context.

### Principles

- One `.feature` file per bounded context capability (not per UI screen)
- Use terms from the ubiquitous language glossary — enforce this in review
- Scenarios describe **observable domain behavior**, not UI interaction or DB state
- No "click button", "open page", "check database" — these are implementation details
- A scenario should be understandable by the domain expert who attended Event Storming

### Feature file structure

```gherkin
Feature: Invoice issuance
  As a Billing Clerk
  I want to issue invoices after order fulfillment
  So that customers are prompted to pay

  Background:
    Given a fulfilled order exists for customer "Acme Corp"
    And the billing configuration specifies NET30 terms

  Scenario: Invoice is issued for a fulfilled order
    When the billing clerk issues an invoice for the order
    Then an invoice is created with NET30 payment terms
    And the invoice status is "Pending Payment"
    And a domain event "InvoiceIssued" is raised

  Scenario: Invoice cannot be issued for an unfulfilled order
    Given the order has not been fulfilled
    When the billing clerk attempts to issue an invoice
    Then the system rejects the request
    And no invoice is created

  Scenario Outline: Invoice due date reflects configured payment terms
    Given billing terms are "<terms>"
    When an invoice is issued today
    Then the due date is <days> days from today

    Examples:
      | terms  | days |
      | NET15  | 15   |
      | NET30  | 30   |
      | NET60  | 60   |
```

### Writing rules

- **Given** — establish preconditions in domain state, not setup steps
- **When** — one action only; if you're tempted to write two, split the scenario
- **Then** — assert observable outcomes: state changes, events raised, rejections
- Avoid conjunctions in When: `When X and Y happens` → split into two scenarios or use Background
- Tag scenarios by context, feature area, and risk level: `@billing @invoice @critical`

### Review checklist

- [ ] All terms appear in the ubiquitous language glossary
- [ ] No UI or infrastructure references
- [ ] Domain expert can read and validate each scenario
- [ ] Each scenario tests exactly one behavior
- [ ] Sad paths (rejections, invariant violations) are covered
- [ ] Domain events are named in Then clauses where applicable

### Output

- `.feature` files organized by bounded context
- Scenarios signed off by domain expert

---

## Phase 5 — Domain Model Design

**Goal:** Translate the domain map into a precise object model per bounded context.

### Aggregate design

An aggregate is a cluster of objects treated as a unit for data changes. Rules:

- One **Aggregate Root** — the only entry point for external commands
- All invariants are enforced **within** the aggregate boundary
- Never hold a direct reference to another aggregate's internals — use ID references
- Keep aggregates small; if they grow large, extract a new aggregate

```
Aggregate: Order
Root: Order
Entities: OrderLine
Value Objects: Money, Address, Quantity
Invariants:
  - Total must equal sum of line amounts
  - Cannot add lines to a confirmed order
  - Quantity must be > 0
Domain Events emitted: OrderPlaced, OrderConfirmed, OrderCancelled
```

### Value Objects

Immutable, identity-less objects defined by their attributes. Model them as such:

- `Money(amount, currency)` — not a float
- `Address(street, city, zip, country)` — not five separate strings
- `DateRange(start, end)` — enforces start < end as an invariant

### Domain Services

Stateless operations that don't naturally belong to any aggregate:

- Cross-aggregate coordination
- Domain logic requiring external input (e.g. currency conversion using a rate)
- Named after domain verbs: `PaymentAllocator`, `InvoiceCalculator`

### Repository interfaces

Defined in the domain layer. No ORM imports, no SQL, no infrastructure knowledge:

```python
class OrderRepository(Protocol):
    def get(self, order_id: OrderId) -> Order: ...
    def save(self, order: Order) -> None: ...
    def find_by_customer(self, customer_id: CustomerId) -> list[Order]: ...
```

### Output

- Aggregate definitions with invariants documented
- Value Object catalog
- Domain Event list (formal, with payload schema)
- Repository interfaces
- Domain Service list

---

## Phase 6 — Application Layer Design

**Goal:** Define use cases that orchestrate the domain model.

### Application Services

Thin orchestrators. They:
- Load aggregates via repositories
- Call domain methods
- Persist results
- Publish domain events
- Do **not** contain business logic

```python
class IssueInvoiceUseCase:
    def execute(self, cmd: IssueInvoiceCommand) -> InvoiceId:
        order = self.order_repo.get(cmd.order_id)
        invoice = Invoice.issue_for(order, cmd.billing_terms)
        self.invoice_repo.save(invoice)
        self.event_bus.publish(invoice.pull_events())
        return invoice.id
```

### CQRS split (if applicable)

Separate the write model (Commands → Aggregates) from the read model (Queries → Projections):

- **Commands** mutate state, go through aggregates, enforce invariants
- **Queries** read from optimized read models (denormalized views, projections)
- Never query through the aggregate for read purposes

### Command / Query catalog

For each Gherkin `When` clause, there should be a corresponding Command or Query:

| Gherkin When | Application Command/Query |
|---|---|
| the clerk issues an invoice | `IssueInvoiceCommand` |
| the clerk views invoice history | `GetInvoicesByCustomerQuery` |

### Output

- Command and Query DTOs
- Application Service per use case
- Event publishing strategy (synchronous, outbox, message bus)

---

## Phase 7 — Infrastructure / Ports & Adapters

**Goal:** Implement the "outside world" without touching domain logic.

### Ports (defined in domain/application)

```python
class EmailNotifier(Protocol):
    def send_invoice(self, invoice: Invoice, recipient: Email) -> None: ...
```

### Adapters (implemented in infrastructure)

```python
class SmtpEmailNotifier:
    def send_invoice(self, invoice: Invoice, recipient: Email) -> None:
        # SMTP implementation here
```

### What belongs here

- ORM models and repository implementations
- HTTP clients for external APIs
- Message bus producers/consumers
- Database migrations
- Auth middleware
- Configuration loading

### Rules

- Infrastructure must depend on domain, never the reverse
- Every adapter implements a port defined in the inner layers
- Adapters are swappable without touching domain or application code
- Keep infrastructure out of unit tests — use fakes that implement the same ports

### Output

- Repository implementations
- External service adapters
- Infrastructure config (connection strings, credentials via env)
- Migration scripts

---

## Phase 8 — Test Implementation

**Goal:** Wire Gherkin scenarios to executable tests backed by the real domain model.

### Test pyramid for DDD

```
         [ BDD / Acceptance ]     ← Gherkin scenarios, full stack or near-full
        [  Integration Tests  ]   ← Application service + real repo + real DB
       [    Domain Unit Tests   ]  ← Aggregate + Value Object logic, no infra
```

### Domain unit tests

Test invariants and behavior on aggregates directly. No mocks, no DB:

```python
def test_invoice_cannot_be_issued_for_unfulfilled_order():
    order = Order.create(customer_id=..., lines=[...])
    # order is not fulfilled
    with pytest.raises(OrderNotFulfilledError):
        Invoice.issue_for(order, billing_terms=NET30)
```

### BDD step definitions

Map Gherkin steps to domain operations using fakes for infrastructure:

```python
@given("a fulfilled order exists for customer {customer_name}")
def step_fulfilled_order(ctx, customer_name):
    ctx.order = OrderBuilder().fulfilled().for_customer(customer_name).build()
    ctx.order_repo.save(ctx.order)

@when("the billing clerk issues an invoice for the order")
def step_issue_invoice(ctx):
    cmd = IssueInvoiceCommand(order_id=ctx.order.id, billing_terms=NET30)
    ctx.result = IssueInvoiceUseCase(ctx.order_repo, ctx.invoice_repo).execute(cmd)

@then('an invoice is created with NET30 payment terms')
def step_assert_invoice(ctx):
    invoice = ctx.invoice_repo.get(ctx.result)
    assert invoice.terms == NET30
```

### Fake vs Mock

- **Fakes** — lightweight in-memory implementations of ports (preferred in domain/application tests)
- **Mocks** — verify call behavior (use only when testing side effects like email sending)
- Never mock the domain model itself

### Output

- Domain unit test suite
- BDD step definitions wired to application layer
- Integration tests for repository implementations
- Test fixtures and builders (not raw object construction in every test)

---

## Phase 9 — Implementation (Red → Green → Refactor)

**Goal:** Make failing tests pass while keeping the model clean.

### Cycle

1. Pick a failing Gherkin scenario
2. Trace it to the failing domain unit test
3. Write minimum code to pass the domain test
4. Run the Gherkin scenario — if still failing, fix application/infrastructure wiring
5. Refactor: rename to match ubiquitous language, remove duplication, sharpen invariants
6. Repeat

### Code review checklist

- [ ] No business logic in application services or infrastructure
- [ ] All terms match the ubiquitous language glossary
- [ ] Aggregates enforce their own invariants (no validation in services)
- [ ] Domain Events are raised by aggregates, not by services
- [ ] No direct aggregate-to-aggregate references (use IDs)
- [ ] Repository interfaces untouched by infrastructure changes
- [ ] All new scenarios in Gherkin before implementation begins

---

## Traceability Matrix

Every piece of work should trace back to intent:

```
PRD feature
  └── Bounded Context
        └── Domain Event / Aggregate
              └── Gherkin Scenario
                    └── Application Command
                          └── Domain method
                                └── Unit test
```

If a unit test or class cannot be traced back to a Gherkin scenario, question whether it belongs.

---

## Common Anti-Patterns to Avoid

| Anti-Pattern | Symptom | Fix |
|---|---|---|
| Gherkin written from PRD | Steps reference UI elements, DB tables | Redo after Event Storming |
| Anemic domain model | Aggregates are data bags, logic in services | Move invariants into aggregates |
| God aggregate | One aggregate owns everything | Split on consistency boundaries |
| Repository leaking to domain | Domain imports ORM models | Define ports in domain, implement in infra |
| Ubiquitous language ignored | Class names differ from domain terms | Rename aggressively, enforce in review |
| Skipping sad paths in Gherkin | Only happy path scenarios exist | Model every invariant violation as a scenario |
| Integration tests as unit tests | Every test spins up a DB | Use fakes; reserve real DB for integration layer |
