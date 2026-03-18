---
type: moc
tags: [moc, agile, scrum, kanban, software-development]
created: 2026-03-18
updated: 2026-03-18
---

# Agile MOC

## Overview
Agile is a set of values and principles for iterative, collaborative software development that prioritizes delivering working software frequently and responding to change over following a fixed plan.

**Prerequisites**: Basic software development concepts, team collaboration

## Core Concepts

### Values & Principles
- [[Agile Values & Principles]] - The Agile Manifesto: 4 values and 12 principles

### Frameworks
- [[Scrum Framework]] - Roles, events, and artifacts for iterative delivery
- [[Kanban]] - Visualizing and limiting work-in-progress for flow-based delivery

### Practices
- [[User Stories & Backlog]] - Writing user-centered requirements and managing the product backlog
- [[Scrum Ceremonies]] - Sprint Planning, Daily Standup, Review, and Retrospective
- [[Definition of Done]] - Shared quality standards for completing work
- [[Estimation & Story Points]] - Relative effort estimation and planning poker

## Implementation Patterns

### Scrum Sprint Cycle
```
Sprint Planning → Daily Standups → Sprint Review → Retrospective
     ↑                                                     |
     └─────────────── next Sprint ────────────────────────┘
```

### Kanban Flow
```
Backlog → Ready → In Progress → Review → Done
          (limit WIP at each stage)
```

### Common Team Rhythms
- **Daily**: 15-min standup — what did I do, what will I do, any blockers?
- **Weekly/Bi-weekly**: Sprint planning + review + retro (Scrum)
- **Ongoing**: Backlog refinement, continuous deployment (Kanban)

## Advanced Topics

- [[Scaled Agile (SAFe)]] - Agile at enterprise/multi-team scale
- [[OKRs & Agile]] - Aligning objectives and key results with sprints
- [[Agile Metrics]] - Velocity, cycle time, lead time, burndown charts
- [[Continuous Delivery]] - Automating the path from commit to production

## Common Problems & Solutions

- **Scope creep mid-sprint**: Protect sprint commitment; add new items to backlog for next sprint
- **No real retrospective improvement**: Use action items with owners and deadlines, review them next retro
- **Story points inflation over time**: Re-calibrate with reference stories; focus on relative sizing
- **Ceremonies feel like overhead**: Time-box strictly; make outcomes visible (decisions, action items)
- **Team not self-organizing**: Define team norms in a working agreement; coach rather than direct

## Examples

- Sprint Planning board with capacity planning
- Kanban board column setup for a dev team
- Well-written user story with acceptance criteria
- Retrospective formats: Start/Stop/Continue, 4Ls, Mad/Sad/Glad

## Related Areas

- [[GIT]] - Feature branch workflow maps well to sprint stories
- [[CI/CD]] - Continuous integration supports iterative delivery
- [[Code Review]] - Pull request workflow fits Agile team collaboration

## Learning Path

1. Start with [[Agile Values & Principles]] to understand the "why"
2. Pick a framework: [[Scrum Framework]] (most common) or [[Kanban]]
3. Learn [[User Stories & Backlog]] to write good requirements
4. Understand [[Scrum Ceremonies]] to run effective team rituals
5. Establish [[Definition of Done]] before starting any sprint
6. Refine estimation with [[Estimation & Story Points]]

## References

- [Agile Manifesto](https://agilemanifesto.org/) - Original 2001 declaration
- [Scrum Guide](https://scrumguides.org/) - Official Scrum framework reference
- [Kanban Guide](https://kanbanguides.org/) - Official Kanban guide
- *Clean Agile* — Robert C. Martin
- *User Story Mapping* — Jeff Patton
