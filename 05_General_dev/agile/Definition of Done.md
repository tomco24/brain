---
type: concept
tags: [agile, scrum, quality, dod, done]
created: 2026-03-18
modified: 2026-03-18
status: draft
---

# Definition of Done

## Overview
The Definition of Done (DoD) is a shared, formal list of criteria that every Increment must meet before it can be considered complete. It creates a consistent quality standard across the team and eliminates ambiguity about "done."

## Key Points
- DoD applies to **every** story/Increment — it is not per-story acceptance criteria
- Per-story **acceptance criteria** define *what* was built; the DoD defines *how well* it was built
- If an item doesn't meet the DoD, it is **not done** — it cannot be presented in Sprint Review
- The DoD should evolve over time as the team improves

## DoD vs. Acceptance Criteria

| | Definition of Done | Acceptance Criteria |
|---|---|---|
| **Scope** | All stories / entire Increment | A single story |
| **Author** | Team (agreed collectively) | Product Owner / stakeholders |
| **Content** | Quality standards (tests, review, deploy) | Business rules and expected behavior |
| **Changes** | Rarely; when process improves | Per story |

## Example DoD for a Development Team

```
A story is Done when:
  Code
  [ ] Code reviewed and approved by at least 1 other developer
  [ ] No unresolved review comments
  [ ] Follows team coding standards and style guide

  Testing
  [ ] Unit tests written and passing (≥80% coverage on new code)
  [ ] Integration tests passing
  [ ] No regression failures in CI pipeline

  Documentation
  [ ] README / inline comments updated if behavior changed
  [ ] API docs updated (if public endpoint added/changed)

  Deployment
  [ ] Merged to main branch
  [ ] Deployed to staging environment
  [ ] Smoke test passed on staging

  Quality
  [ ] No known critical or high-severity bugs introduced
  [ ] Accessibility requirements met (if UI story)
  [ ] Performance within agreed thresholds
```

## Starting Your DoD

Build it collaboratively in a team session. Ask: *"What must be true for us to be confident shipping this?"*

**Starter checklist:**
- [ ] Code peer-reviewed
- [ ] Automated tests written and passing
- [ ] Merged to trunk / main branch
- [ ] Deployed to a non-production environment
- [ ] No new known defects

Add items as your process matures (security scanning, performance testing, documentation, etc.).

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| DoD is aspirational, not enforced | If the team regularly skips checklist items, remove them or automate them |
| "Done" means "dev complete" | Include testing and deployment in the DoD from day one |
| DoD never evolves | Review it in Retrospectives; add items when quality gaps emerge |
| DoD is per-team but org deploys together | Align DoD with the broader release pipeline requirements |

## DoD and Technical Debt
Items that "pass" per-story acceptance criteria but fail the DoD should **not** be shipped. Shipping them creates technical debt that compounds. The DoD is the last line of defense.

## When to Use
- Any Agile team; mandatory in Scrum

## When NOT to Use
- Pure research spikes — spikes have their own exit criteria (a decision or a prototype)

## Related Concepts
- [[Scrum Framework]]
- [[User Stories & Backlog]] — acceptance criteria live here
- [[Scrum Ceremonies]] — DoD is verified at Sprint Review

## References
- [Scrum Guide — Definition of Done](https://scrumguides.org/scrum-guide.html#artifact-transparency-done)
- [Mountain Goat Software — DoD](https://www.mountaingoatsoftware.com/blog/the-definition-of-done)
