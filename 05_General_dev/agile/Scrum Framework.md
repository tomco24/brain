---
type: framework
tags: [agile, scrum, sprint, framework]
created: 2026-03-18
modified: 2026-03-18
status: draft
---

# Scrum Framework

## What It Is
Scrum is a lightweight framework for delivering value iteratively through fixed-length cycles called **Sprints** (1–4 weeks), using defined roles, events, and artifacts.

## The 3 Roles

| Role | Responsibility |
|------|---------------|
| **Product Owner (PO)** | Owns the Product Backlog; prioritizes what gets built and why |
| **Scrum Master (SM)** | Facilitates events, removes impediments, coaches the team on Scrum |
| **Developers** | Self-organizing team that builds and delivers the Increment |

> The Scrum Master is NOT a project manager. They serve the team, not direct it.

## The 3 Artifacts

| Artifact | Description |
|----------|-------------|
| **Product Backlog** | Ordered list of everything that might be done in the product |
| **Sprint Backlog** | The items selected for the current Sprint + a plan to deliver them |
| **Increment** | The sum of all completed work — must be usable at end of each Sprint |

## The 5 Events

See [[Scrum Ceremonies]] for detailed format of each.

| Event | Time-box | Purpose |
|-------|----------|---------|
| **Sprint** | 1–4 weeks | Container for all other events; delivers one Increment |
| **Sprint Planning** | ≤8h (4-week sprint) | Decide what and how to build this Sprint |
| **Daily Scrum** | 15 min | Inspect progress toward Sprint Goal; adapt the plan |
| **Sprint Review** | ≤4h | Inspect Increment; gather stakeholder feedback |
| **Retrospective** | ≤3h | Inspect team process; commit to one improvement |

## Core Concepts

### Sprint Goal
A single objective that gives the Sprint coherence. The team can adapt *how* they reach it, but the goal itself is fixed once the Sprint starts.

### Definition of Done (DoD)
A shared understanding of what "complete" means for any Increment. See [[Definition of Done]].

### Backlog Refinement
Not an official Scrum event but a recommended practice — the team regularly grooms backlog items (splitting, estimating, clarifying) so they're ready for Sprint Planning.

## Common Patterns in Scrum

- [[User Stories & Backlog]] — How to write and order backlog items
- [[Estimation & Story Points]] — Sizing stories for Sprint Planning
- [[Definition of Done]] — Quality gates applied to every Increment

## Best Practices

### Sprint Length
- Start with **2-week sprints** — short enough for feedback, long enough to build something meaningful
- Keep it consistent; changing length disrupts velocity tracking

### Sprint Planning
- Bring the top-priority refined backlog items
- Team pulls work (doesn't get assigned) based on capacity
- Output: a Sprint Goal + Sprint Backlog

### Daily Scrum
- Stand-up, same time, same place
- 3 questions: What did I do yesterday? What will I do today? Any blockers?
- Blockers get taken *offline* — don't solve in the standup

## Gotchas & Troubleshooting

- **Mini-waterfall per sprint**: Requirements → design → dev → test all in one sprint is fine; splitting phases across sprints is not
- **PO unavailable**: Sprint Planning and Review break down; PO must be actively engaged
- **Sprint commitment as a quota**: Velocity is a planning tool, not a performance metric
- **Skipping retros**: Teams that skip retros stop improving; protect the ceremony
- **Scrum Master = project manager**: SM removes obstacles, not assigns tasks or reports status upward

## Integration Patterns

- With [[GIT]] — one branch per story/task; merged on Sprint completion
- With [[Kanban]] — Scrumban hybrid for teams needing more flow flexibility
- With CI/CD — automated pipelines support "potentially shippable" at end of Sprint

## Learning Resources

- [Official Scrum Guide](https://scrumguides.org/) — authoritative and free (~13 pages)
- *Scrum: The Art of Doing Twice the Work in Half the Time* — Jeff Sutherland
- [Scrum.org](https://www.scrum.org/) — certifications and training
