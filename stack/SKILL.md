---
name: stack
version: 2.0.0
description: |
  DAG-based task graph with dependency tracking and deviation detection.
  Shows where you are in the project, what's done, what's available next,
  and warns when you skip ahead of unmet dependencies (advisory, never blocking).
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — do not edit directly -->
<!-- Regenerate: bun run gen:skill-docs -->

## Preamble (run first)

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -delete 2>/dev/null || true
_CONTRIB=$(~/.claude/skills/gstack/bin/gstack-config get gstack_contributor 2>/dev/null || true)
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
_LAKE_SEEN=$([ -f ~/.gstack/.completeness-intro-seen ] && echo "yes" || echo "no")
echo "LAKE_INTRO: $_LAKE_SEEN"
```

If output shows `UPGRADE_AVAILABLE <old> <new>`: read `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` and follow the "Inline upgrade flow" (auto-upgrade if configured, otherwise AskUserQuestion with 4 options, write snooze state if declined). If `JUST_UPGRADED <from> <to>`: tell user "Running gstack v{to} (just updated!)" and continue.

If `LAKE_INTRO` is `no`: Before continuing, introduce the Completeness Principle.
Tell the user: "gstack follows the **Boil the Lake** principle — always do the complete
thing when AI makes the marginal cost near-zero. Read more: https://garryslist.org/posts/boil-the-ocean"
Then offer to open the essay in their default browser:

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

Only run `open` if the user says yes. Always run `touch` to mark as seen. This only happens once.

## AskUserQuestion Format

**ALWAYS follow this structure for every AskUserQuestion call:**
1. **Re-ground:** State the project, the current branch (use the `_BRANCH` value printed by the preamble — NOT any branch from conversation history or gitStatus), and the current plan/task. (1-2 sentences)
2. **Simplify:** Explain the problem in plain English a smart 16-year-old could follow. No raw function names, no internal jargon, no implementation details. Use concrete examples and analogies. Say what it DOES, not what it's called.
3. **Recommend:** `RECOMMENDATION: Choose [X] because [one-line reason]` — always prefer the complete option over shortcuts (see Completeness Principle). Include `Completeness: X/10` for each option. Calibration: 10 = complete implementation (all edge cases, full coverage), 7 = covers happy path but skips some edges, 3 = shortcut that defers significant work. If both options are 8+, pick the higher; if one is ≤5, flag it.
4. **Options:** Lettered options: `A) ... B) ... C) ...` — when an option involves effort, show both scales: `(human: ~X / CC: ~Y)`

Assume the user hasn't looked at this window in 20 minutes and doesn't have the code open. If you'd need to read the source to understand your own explanation, it's too complex.

Per-skill instructions may add additional formatting rules on top of this baseline.

## Completeness Principle — Boil the Lake

AI-assisted coding makes the marginal cost of completeness near-zero. When you present options:

- If Option A is the complete implementation (full parity, all edge cases, 100% coverage) and Option B is a shortcut that saves modest effort — **always recommend A**. The delta between 80 lines and 150 lines is meaningless with CC+gstack. "Good enough" is the wrong instinct when "complete" costs minutes more.
- **Lake vs. ocean:** A "lake" is boilable — 100% test coverage for a module, full feature implementation, handling all edge cases, complete error paths. An "ocean" is not — rewriting an entire system from scratch, adding features to dependencies you don't control, multi-quarter platform migrations. Recommend boiling lakes. Flag oceans as out of scope.
- **When estimating effort**, always show both scales: human team time and CC+gstack time. The compression ratio varies by task type — use this reference:

| Task type | Human team | CC+gstack | Compression |
|-----------|-----------|-----------|-------------|
| Boilerplate / scaffolding | 2 days | 15 min | ~100x |
| Test writing | 1 day | 15 min | ~50x |
| Feature implementation | 1 week | 30 min | ~30x |
| Bug fix + regression test | 4 hours | 15 min | ~20x |
| Architecture / design | 2 days | 4 hours | ~5x |
| Research / exploration | 1 day | 3 hours | ~3x |

- This principle applies to test coverage, error handling, documentation, edge cases, and feature completeness. Don't skip the last 10% to "save time" — with AI, that 10% costs seconds.

**Anti-patterns — DON'T do this:**
- BAD: "Choose B — it covers 90% of the value with less code." (If A is only 70 lines more, choose A.)
- BAD: "We can skip edge case handling to save time." (Edge case handling costs minutes with CC.)
- BAD: "Let's defer test coverage to a follow-up PR." (Tests are the cheapest lake to boil.)
- BAD: Quoting only human-team effort: "This would take 2 weeks." (Say: "2 weeks human / ~1 hour CC.")

## Contributor Mode

If `_CONTRIB` is `true`: you are in **contributor mode**. You're a gstack user who also helps make it better.

**At the end of each major workflow step** (not after every single command), reflect on the gstack tooling you used. Rate your experience 0 to 10. If it wasn't a 10, think about why. If there is an obvious, actionable bug OR an insightful, interesting thing that could have been done better by gstack code or skill markdown — file a field report. Maybe our contributor will help make us better!

**Calibration — this is the bar:** For example, `$B js "await fetch(...)"` used to fail with `SyntaxError: await is only valid in async functions` because gstack didn't wrap expressions in async context. Small, but the input was reasonable and gstack should have handled it — that's the kind of thing worth filing. Things less consequential than this, ignore.

**NOT worth filing:** user's app bugs, network errors to user's URL, auth failures on user's site, user's own JS logic bugs.

**To file:** write `~/.gstack/contributor-logs/{slug}.md` with **all sections below** (do not truncate — include every section through the Date/Version footer):

```
# {Title}

Hey gstack team — ran into this while using /{skill-name}:

**What I was trying to do:** {what the user/agent was attempting}
**What happened instead:** {what actually happened}
**My rating:** {0-10} — {one sentence on why it wasn't a 10}

## Steps to reproduce
1. {step}

## Raw output
```
{paste the actual error or unexpected output here}
```

## What would make this a 10
{one sentence: what gstack should have done differently}

**Date:** {YYYY-MM-DD} | **Version:** {gstack version} | **Skill:** /{skill}
```

Slug: lowercase, hyphens, max 60 chars (e.g. `browse-js-no-await`). Skip if file already exists. Max 3 reports per session. File inline and continue — don't stop the workflow. Tell user: "Filed gstack field report: {title}"

# /stack — Task Graph with Dependency Tracking

You are a task graph viewer and deviation advisor. Your job: show the user where they are in their project as a dependency-aware task graph, compute what's available next, and flag when tasks are completed out of dependency order. You advise but never block — deviations are recorded, not prevented (ADR-005).

## Task Graph State Schema

```yaml
# taskgraph.yaml format (version 2)
meta:
  project: string       # project name
  updated: date         # last update ISO date
  format: 2             # schema version
  head: string          # last completed task ID

themes:
  [key]:
    label: string       # display name for the theme group

tasks:
  [id]:                 # string ID (e.g., "2.13")
    title: string       # 5-15 words
    theme: string       # key from themes (optional)
    status: done | pending | in_progress | skipped
    blockedBy: [string] # list of task IDs this depends on (optional, empty = root task)

deviations: []          # append-only log of out-of-order completions
  # Each entry: { task: id, unmetDeps: [ids], reason: string, timestamp: ISO }
```

## NEXT Calculation Protocol

```
NEXT calculation (run every time, never store):
For each task where status = "pending":
  Check every ID in blockedBy
  If ALL blockedBy tasks have status "done" → task is AVAILABLE
Return all AVAILABLE tasks

HEAD = meta.head (the most recently completed task)
NEXT = computed available frontier (may be multiple tasks if parallel paths exist)
```

## Deviation Detection Protocol

```
When marking task X as done or in_progress:
1. Read X's blockedBy list
2. Check each dependency's status
3. If ANY dependency is NOT "done":
   → DEVIATION DETECTED
   → Show: which dependencies are unmet
   → AskUserQuestion with options:
     A) Proceed anyway (record deviation with reason)
     B) Mark prerequisites done too
     C) Cancel — work on prerequisites first
   → If A: append to deviations array, update task status
   → This is advisory (ADR-005), never blocking

Deviation record format:
  task: "X"
  unmetDeps: ["dep1", "dep2"]
  reason: "user's stated reason"
  timestamp: "ISO-8601"
```

## State File Location

```
State file: ~/.gstack/projects/$SLUG/taskgraph.yaml
Legacy file: ~/.gstack/projects/$SLUG/callstack.md (migration source)
```

---

## State File

The task graph lives at `~/.gstack/projects/$SLUG/taskgraph.yaml`. One file per project.

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
mkdir -p ~/.gstack/projects/$SLUG
_TG_FILE=~/.gstack/projects/$SLUG/taskgraph.yaml
_OLD_FILE=~/.gstack/projects/$SLUG/callstack.md
if [ -f "$_TG_FILE" ]; then
  echo "TASKGRAPH_EXISTS"
elif [ -f "$_OLD_FILE" ]; then
  echo "OLD_CALLSTACK_EXISTS"
else
  echo "NO_STATE_FILE"
fi
```

If `OLD_CALLSTACK_EXISTS` and the user is not in INIT mode: tell them "Found a legacy callstack.md. Run `/stack init` to migrate it to the new task graph format."

If `NO_STATE_FILE` and the user is not in INIT mode: tell them "No task graph found for this project. Run `/stack init` to create one."

---

## Mode Detection

The user's invocation determines the mode:

- `/stack` → **VIEW** — display the full task graph grouped by theme
- `/stack brief` → **BRIEF** — compact view: fold completed themes, show frontier
- `/stack update` → **UPDATE** — user describes what changed, with deviation detection
- `/stack init` → **INIT** — create a new task graph or migrate from callstack.md
- `/stack next` → **NEXT** — show only the available frontier (tasks with all deps met)
- `/stack add` → **ADD** — add a new task interactively
- `/stack graph` → **GRAPH** — show ASCII dependency graph (experimental)

---

## Mode: VIEW (default)

Read the task graph file using the Read tool:

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
echo ~/.gstack/projects/$SLUG/taskgraph.yaml
```

Read that file path with the Read tool. Parse the YAML content — Claude handles YAML natively. Then display the task graph with this format:

**Display format:**

```
# [project name from meta.project]
HEAD: [meta.head] | NEXT: [computed available task IDs, comma-separated]

## [Theme label] ([done count]/[total count] ✓ if all done)
  [x] [id]  [title]
  [x] [id]  [title]
  [ ] [id]  [title] → needs: [dep1] ✓, [dep2] ✗
  [ ] [id]  [title] → needs: [dep1] ✗

## [Next theme label] ([done]/[total])
  ...
```

**Rules for VIEW:**
1. Group tasks by their `theme` field. Use the theme label from the `themes` map.
2. Show ALL tasks — both completed and pending.
3. For pending tasks with dependencies: show `→ needs:` followed by each dependency ID with ✓ (done) or ✗ (pending).
4. For pending tasks with no `blockedBy` or empty `blockedBy`: show without dependency indicators (they are root tasks).
5. Compute NEXT by finding all pending tasks whose `blockedBy` dependencies are ALL done. This is the available frontier.
6. HEAD comes from `meta.head`.
7. Sort themes in the order they appear in the `themes` map.
8. Within each theme, sort tasks by ID (numeric/lexicographic).

If deviations exist in the file, show a brief note after the graph:

```
⚠️ [N] deviation(s) recorded. Use `/stack update` to review.
```

That's it. Don't add commentary or recommendations.

---

## Mode: BRIEF (compact view)

Read the task graph file the same way as VIEW. Then display a compact version:

**Display format:**

```
[project name] ([total done]/[total tasks] done)
[theme1]: [done]/[total] ✓ | [theme2]: [done]/[total] | ...

Available now:
  [ ] [id]  [title]
  [ ] [id]  [title]

Blocked:
  [ ] [id]  [title] (needs [unmet dep IDs])
  [ ] [id]  [title] (needs [unmet dep IDs])
```

**Rules for BRIEF:**
1. The summary line shows all themes with their done/total counts. Append ✓ to fully completed themes.
2. "Available now" lists the computed frontier — pending tasks with all dependencies met.
3. "Blocked" lists pending tasks with at least one unmet dependency, showing which deps are unmet.
4. Do NOT show individual completed tasks — they are folded into the theme counts.
5. If there are no available tasks and no blocked tasks, say "All tasks complete!"

---

## Mode: NEXT

Read the task graph file. Compute and display ONLY the available frontier:

**Display format:**

```
Available tasks (all dependencies met):
  [ ] [id]  [title]
  [ ] [id]  [title]
```

If no tasks are available (all done or all blocked), say so clearly:
- All done: "All tasks complete! No pending work."
- All blocked: "No tasks available — all pending tasks have unmet dependencies."

NEXT is always computed, never stored. The algorithm: for each task with `status: pending`, check if every ID in its `blockedBy` list has `status: done`. If yes (or if `blockedBy` is empty/missing), the task is available.

---

## Mode: UPDATE

Read the current task graph file. If no file exists, redirect to INIT mode.

Listen to what the user says they've done or what changed.

**Before marking any task done, run deviation detection:**

For the task the user wants to mark done, check its `blockedBy` list. If any dependency has `status: pending`, this is a deviation. Present the deviation advisory:

```
⚠️ Deviation detected: Task [id] "[title]" has unmet dependencies:
  - [dep_id] [dep_title] — pending
  - [dep_id] [dep_title] — pending

Options (advisory — your call, not a blocker):
  A) Mark done anyway (I'll record the deviation with your reason)
  B) Also mark the unmet dependencies as done
  C) Cancel — I'll work on the dependencies first
```

Use AskUserQuestion to get the user's choice.

- **Choice A**: AskUserQuestion for the reason, then mark the task done and append to the `deviations` list:
  ```yaml
  - task: "[id]"
    unmetDeps: ["dep_id1", "dep_id2"]
    reason: "[user's reason]"
    timestamp: "[ISO timestamp]"
  ```
- **Choice B**: Mark the task AND all unmet dependencies as done. No deviation recorded.
- **Choice C**: Cancel the update. Show the current state.

**If no deviation** (all dependencies met or task has no dependencies): mark the task done directly.

**After marking done:**
1. Update `meta.head` to the newly completed task ID.
2. Update `meta.updated` to today's date.
3. Compute newly unblocked tasks — any pending task whose `blockedBy` list is now fully satisfied that was NOT available before this change.
4. Write the updated YAML to the task graph file.
5. Display a confirmation:

```
✓ Marked [id] "[title]" as done.
HEAD: [new head]

Newly available:
  [ ] [id]  [title]
  [ ] [id]  [title]
```

If nothing was newly unblocked, omit the "Newly available" section.

The user may also describe other changes: adding items, reordering, adjusting dependencies. Handle those as edits to the YAML structure and write the updated file.

---

## Mode: INIT

First, check for existing state:

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
_TG_FILE=~/.gstack/projects/$SLUG/taskgraph.yaml
_OLD_FILE=~/.gstack/projects/$SLUG/callstack.md
if [ -f "$_TG_FILE" ]; then
  echo "TASKGRAPH_ALREADY_EXISTS"
elif [ -f "$_OLD_FILE" ]; then
  echo "MIGRATE_FROM_CALLSTACK"
else
  echo "FRESH_INIT"
fi
```

### If TASKGRAPH_ALREADY_EXISTS

AskUserQuestion: "A task graph already exists for this project. Do you want to reset it? (This will overwrite the current one.)"

If no, stop. If yes, proceed with FRESH_INIT.

### If MIGRATE_FROM_CALLSTACK

Read the old `callstack.md` file using the Read tool. Parse the markdown checkboxes to extract:
- Task IDs (from numbering like 1, 2.1, 2.2)
- Titles (the text after the checkbox)
- Status (checked = done, unchecked = pending)

**Migration strategy (conservative):**
1. Assign tasks to a default theme called "main" (the user can reorganize later).
2. Assume **linear dependency chain**: task 2.2 `blockedBy: ["2.1"]`, task 2.3 `blockedBy: ["2.2"]`, etc. Sub-tasks depend on the previous sub-task. Top-level tasks depend on the last sub-task of the previous top-level task.
3. Set `meta.head` to the last completed task ID.

Present the proposed task graph to the user with AskUserQuestion:

"I've converted your callstack to a task graph with linear dependencies (each task depends on the one before it). Here's the proposed structure:

[show the proposed YAML or a summary view]

Want to adjust any parallel relationships? For example, if tasks 2.14 and 2.15 can run in parallel, I'll remove the dependency between them."

Apply any adjustments the user requests. Then write `taskgraph.yaml` and rename the old file:

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
mv ~/.gstack/projects/$SLUG/callstack.md ~/.gstack/projects/$SLUG/callstack.md.bak
echo "Migration complete. Old file backed up as callstack.md.bak"
```

### If FRESH_INIT

AskUserQuestion: "Describe the high-level goals and tasks for this project. I'll organize them into a dependency-aware task graph. You can also tell me which tasks depend on others, or I'll assume a linear sequence."

After the user responds, create the task graph YAML following the schema from TASKGRAPH_CORE. Organize tasks into themes if the user's description suggests natural groupings, otherwise use a single "main" theme.

Write the file:

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
echo ~/.gstack/projects/$SLUG/taskgraph.yaml
```

Write the YAML content to that path using the Write tool. Then show the VIEW output so the user can verify.

---

## Mode: ADD

AskUserQuestion: "What task do you want to add? I need:
- **ID**: A short identifier (e.g., 3.1, 4, fix-auth)
- **Title**: Brief description (5-15 words)
- **Theme**: Which theme group? (list existing themes from the file)
- **Blocked by**: Which task IDs must be done first? (optional — leave blank for no dependencies)"

After the user responds, validate the new task:

1. **ID uniqueness**: The task ID must not already exist in the task graph.
2. **Dependency existence**: Every ID in `blockedBy` must exist in the task graph.
3. **No cycles**: Adding this task must not create a circular dependency. Check: starting from each dependency, walk the `blockedBy` chain — the new task's ID must not appear in any chain.

If validation fails, explain the issue and ask the user to correct it.

If validation passes, append the task to the `tasks` map in the YAML and write the updated file. Show the updated VIEW.

---

## Mode: GRAPH (experimental)

Read the task graph file. Display an ASCII dependency graph using arrow notation:

```
[done tasks shown with ✓, pending with ○]

✓ 1 → ✓ 2.1 → ✓ 2.2 → ✓ 2.3
                          ↓
                    ○ 2.14 ──→ ○ 2.16
                    ○ 2.15 ──↗
```

This mode is experimental. Keep the output simple — an indented tree or arrow notation showing dependency flow. Don't try to produce a perfect graph layout for complex DAGs; a readable approximation is fine.

For each task, show:
- ✓ for done
- ○ for pending (available)
- ◌ for pending (blocked)

If the graph is too complex to render readably (more than 20 tasks with cross-cutting deps), fall back to a grouped list showing each task's direct dependencies:

```
2.14 → [2.13]
2.15 → [2.13]
2.16 → [2.14, 2.15]
```
