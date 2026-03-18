---
type: concept
tags: [agile, scrum, ceremonies, sprint, retrospective]
created: 2026-03-18
modified: 2026-03-18
status: draft
---

# Scrum Ceremonies

## Overview
Scrum defines five formal events (ceremonies) that create regularity and reduce the need for unplanned meetings. Each has a strict time-box and a clear purpose.

## 1. The Sprint

- **Length**: 1–4 weeks (consistent; don't vary mid-project)
- **Purpose**: Container for all other events; produces one Increment
- **Rules**:
  - Sprint Goal does not change once started
  - No changes that would endanger the Sprint Goal
  - Can be cancelled only by the Product Owner (rare)

## 2. Sprint Planning

- **Time-box**: ≤8 hours for a 4-week sprint (scale proportionally)
- **Attendees**: Full Scrum Team (PO + SM + Developers)
- **Output**: Sprint Goal + Sprint Backlog

### Agenda

**Part 1 — What? (with the PO)**
- PO presents top backlog items and their value
- Team asks clarifying questions
- Team selects items they can commit to (based on capacity and velocity)
- Agree on a **Sprint Goal** — one sentence: *"By end of this sprint, users can..."*

**Part 2 — How? (developers only)**
- Break selected stories into tasks
- Identify unknowns and risks
- Estimate task-level effort if needed

### Tips
- Backlog must be refined *before* Sprint Planning — don't do refinement in the meeting
- Pull, don't push — developers choose how much they can take on
- The Sprint Goal is the north star; individual stories may change, the goal shouldn't

## 3. Daily Scrum (Standup)

- **Time-box**: 15 minutes
- **Attendees**: Developers (SM and PO may attend but not required)
- **Purpose**: Inspect progress toward Sprint Goal; adapt today's plan

### Format (3 Questions)
1. What did I complete yesterday that contributed to the Sprint Goal?
2. What will I do today to contribute to the Sprint Goal?
3. Do I see any impediment that blocks me or the team?

### Rules
- Same time, same place, every day
- Blockers are raised, NOT solved — take solving offline
- Not a status report to the Scrum Master; it's for the developers, by the developers

## 4. Sprint Review

- **Time-box**: ≤4 hours for a 4-week sprint
- **Attendees**: Scrum Team + stakeholders
- **Purpose**: Inspect the Increment and adapt the Product Backlog

### Agenda
1. PO confirms what was Done vs. not Done
2. Developers **demo** completed work (live, not slides)
3. Stakeholders ask questions and give feedback
4. PO updates backlog based on feedback and market changes
5. Discuss what to do next (informal Sprint Planning input)

### Tips
- Demo only items that meet the [[Definition of Done]]
- Incomplete items go back to the backlog — never "carry over" into next sprint as if done
- This is a working session, not a presentation — encourage discussion

## 5. Sprint Retrospective

- **Time-box**: ≤3 hours for a 4-week sprint
- **Attendees**: Scrum Team only (no stakeholders)
- **Purpose**: Inspect *how the team works* and commit to improvements

### Format — Start / Stop / Continue
```
Start:   What should we begin doing that we aren't?
Stop:    What are we doing that isn't helping?
Continue: What's working well that we should keep?
```

### Other Formats
- **4Ls**: Liked / Learned / Lacked / Longed For
- **Mad / Sad / Glad**: Emotional temperature check + themes
- **Sailboat**: Wind (what helps) / Anchors (what slows us) / Rocks (risks) / Sun (goal)

### Rules
- Safe space — no blame, no managers unless they're part of the team
- Output: **1–3 actionable improvements with owners and due dates**
- Start next retro by reviewing previous action items

## Key Points
- All ceremonies have strict time-boxes — cut them short if done early, never run over
- The SM's job is to facilitate, not to lead or fill silence
- Skipping any ceremony is technical debt on the team's process
- Ceremonies are the heartbeat of Scrum; irregular heartbeats cause problems

## When to Use
- Any team running Scrum sprints

## When NOT to Use
- Kanban teams — ceremonies are optional; replace with flow reviews and replenishment meetings

## Related Concepts
- [[Scrum Framework]]
- [[Definition of Done]]
- [[User Stories & Backlog]]
- [[Estimation & Story Points]]

## References
- [Scrum Guide 2020](https://scrumguides.org/scrum-guide.html) — sections on Events
- *Agile Retrospectives* — Esther Derby & Diana Larsen
