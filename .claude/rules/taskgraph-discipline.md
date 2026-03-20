---
description: "Taskgraph persistence and recommendation discipline"
alwaysApply: true
---

# Taskgraph Discipline

**Immediate persistence:** When user confirms a new task or direction in conversation, write it to taskgraph.yaml in the same turn. Do not accumulate and batch-update later.

**Recommendation accuracy:** When answering "what to do next":
1. Check taskgraph for available tasks
2. Check recent conversation for decisions not yet persisted to taskgraph
3. If conversation contains tasks not in taskgraph → persist first, then recommend
4. Only recommend tasks that are actually executable (user has provided enough info, dependencies met)
5. Mark one task as ⭐ recommended with a one-line reason (e.g., "critical path — unblocks N downstream")
