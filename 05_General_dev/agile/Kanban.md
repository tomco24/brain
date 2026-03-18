---
type: framework
tags: [agile, kanban, flow, wip]
created: 2026-03-18
modified: 2026-03-18
status: draft
---

# Kanban

## What It Is
Kanban is a flow-based method for managing and improving work by visualizing work items on a board, limiting work-in-progress (WIP), and continuously optimizing flow — with no fixed iterations or prescribed roles.

## Core Concepts

### The Kanban Board
Work items move left to right through columns representing workflow states.

```
| Backlog | Ready | In Progress | Review | Done |
|---------|-------|-------------|--------|------|
| item 6  | item4 |   item 2    | item 1 |item A|
| item 7  | item5 |   item 3    |        |item B|
```

### WIP Limits
Each column has a maximum number of items allowed simultaneously.

- **Why**: Forces the team to finish before starting new work
- **Effect**: Surfaces bottlenecks immediately (column fills up = there's a problem)
- **Rule**: If WIP limit is reached, help pull blocked work through rather than starting new work

> "Stop starting, start finishing."

### The 4 Core Practices

1. **Visualize** — Make all work and its status visible on the board
2. **Limit WIP** — Set explicit limits per column/stage
3. **Manage Flow** — Track how work moves; identify and remove bottlenecks
4. **Make Policies Explicit** — Define entry/exit criteria for each column

### Key Metrics

| Metric | Definition | Use |
|--------|-----------|-----|
| **Cycle Time** | Time from "started" to "done" | Predict delivery dates |
| **Lead Time** | Time from "requested" to "done" | Customer-facing SLAs |
| **Throughput** | Items completed per time period | Capacity planning |
| **WIP** | Items currently in progress | Flow health indicator |

### Cumulative Flow Diagram (CFD)
A stacked area chart showing how many items are in each stage over time. Expanding bands = WIP growing = trouble.

## Common Patterns in Kanban

- [[User Stories & Backlog]] — Backlog column feeds the Kanban board
- Expedite swim lane — urgent items bypass WIP limits (use sparingly)
- Blocked column — explicit visibility for items waiting on external dependency

## Best Practices

### Column Design
- Start simple: `Backlog | In Progress | Done`
- Add columns only when they represent a real handoff or waiting state
- Avoid "In Review" just to feel organized — only add if it has a distinct owner

### WIP Limit Sizing
- Start with: `2 × number of team members` per active column
- Tighten limits over time as flow improves
- Don't set limits so tight that one blocker paralyzes the team

### Replenishment
- Pull new work into "Ready" only when capacity allows (do not push)
- Hold regular backlog refinement to keep the queue healthy

## Gotchas & Troubleshooting

- **WIP limits ignored**: Limits only work if the team enforces them as a social contract
- **Board not updated in real time**: Stale boards lose trust; automate or build a daily habit
- **Too many columns**: Each column adds cognitive load; only model real workflow states
- **No WIP limits at all**: Without limits it's just a task list with a visual skin
- **Kanban vs. Scrum confusion**: Kanban has no sprints, no velocity, no prescribed roles — it's pull-based, not time-boxed

## Integration Patterns

- With [[Scrum Framework]] — "Scrumban": Scrum ceremonies + Kanban-style WIP limits mid-sprint
- With CI/CD — deployment pipeline maps directly to board columns
- With support/ops teams — Kanban fits better than Scrum for interrupt-driven work

## Learning Resources

- [Official Kanban Guide](https://kanbanguides.org/) — free reference
- *Kanban: Successful Evolutionary Change for Your Technology Business* — David J. Anderson
- [Kanbanize Blog](https://kanbanize.com/kanban-resources/) — practical articles
