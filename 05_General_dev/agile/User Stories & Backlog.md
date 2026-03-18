---
type: concept
tags: [agile, user-stories, backlog, requirements]
created: 2026-03-18
modified: 2026-03-18
status: draft
---

# User Stories & Backlog

## Overview
A user story is a short, user-centered description of a feature told from the perspective of the person who needs it. The product backlog is the ordered collection of all such stories (and other work items) for a product.

## The User Story Format

```
As a <type of user>,
I want <some goal>
so that <some reason/benefit>.
```

### Examples

```
As a registered user,
I want to reset my password via email
so that I can regain access if I forget my credentials.

As an admin,
I want to export the user list as CSV
so that I can import it into our reporting tool.
```

## Acceptance Criteria

Each story must have explicit, testable conditions that define when the story is "done."

**Format — Given/When/Then (BDD style):**
```
Given I am on the login page and I click "Forgot Password"
When I enter my registered email and submit
Then I receive a reset link within 2 minutes

Given the reset link is older than 24 hours
When I click it
Then I see an "expired link" error and am prompted to request a new one
```

**Or as a simple checklist:**
```
- [ ] Password reset email sent within 2 minutes
- [ ] Reset link expires after 24 hours
- [ ] User is redirected to login after successful reset
- [ ] Invalid/expired links show a clear error message
```

## The INVEST Criteria

Good user stories are:

| Letter | Meaning | Check |
|--------|---------|-------|
| **I** | Independent | Can be built without depending on another story |
| **N** | Negotiable | Details can be discussed and refined |
| **V** | Valuable | Delivers value to a user or the business |
| **E** | Estimable | Team can size it reasonably |
| **S** | Small | Fits within a single sprint |
| **T** | Testable | Has clear acceptance criteria |

## The Product Backlog

### Key Properties
- **Ordered** (not just prioritized) — item 1 is more important than item 2
- **Never complete** — it evolves as the product evolves
- **Single source of truth** — only the Product Owner owns ordering

### Backlog Item Types
- **User Stories** — feature work from a user's perspective
- **Bugs** — defects to fix
- **Technical Debt / Enablers** — infrastructure, refactoring, upgrades
- **Spikes** — time-boxed research/investigation tasks

### Backlog Refinement (Grooming)
A regular session (not a Scrum event but a recommended practice) where the team:
1. Adds detail / splits large stories (Epics → Stories → Tasks)
2. Estimates stories near the top of the backlog
3. Removes or reorders items based on new information

**Recommended**: spend ~10% of sprint capacity on refinement

## Story Sizing & Splitting

### Epics → Stories → Tasks
```
Epic: User Authentication
  ├── Story: Register with email/password
  │     ├── Task: Design registration form
  │     ├── Task: Implement API endpoint
  │     └── Task: Write unit tests
  ├── Story: Login with email/password
  └── Story: Password reset flow
```

### Splitting Techniques
- **By workflow step**: Register / Verify email / Activate account
- **By data variation**: Simple case first, edge cases as follow-up stories
- **By user role**: Admin view / Regular user view
- **Happy path first**: Core flow → error handling → edge cases

## Key Points
- Stories are placeholders for a conversation, not a spec document
- Acceptance criteria belong on the story, not in a separate document
- "Tasks" are how developers break down a story internally — not visible to the PO
- A backlog with 200+ items is unmanageable — keep only the top ~2 sprints refined

## When to Use
- Any project with evolving requirements and regular stakeholder feedback
- Teams using Scrum or Kanban

## When NOT to Use
- Pure R&D/research work with no user-facing output — use spikes instead
- Maintenance tasks with no user impact — bugs or tech debt items are more appropriate

## Related Concepts
- [[Scrum Framework]]
- [[Estimation & Story Points]]
- [[Definition of Done]]
- [[Scrum Ceremonies]]

## References
- *User Story Mapping* — Jeff Patton
- *Writing Effective Use Cases* — Alistair Cockburn
- [Mountain Goat Software — User Stories](https://www.mountaingoatsoftware.com/agile/user-stories)
