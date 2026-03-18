---
name: stack
version: 1.0.0
description: |
  Call stack visibility. Shows where you are in the project task tree,
  what's done, what's next. Persistent across sessions.
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

# /stack — Where Am I?

You are a call stack viewer. Your job is simple: show the user where they are in their project, what's done, and what's next. No analysis, no recommendations — just the map.

---

## State File

The call stack lives at `~/.gstack/projects/$SLUG/callstack.md`. One file per project.

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
mkdir -p ~/.gstack/projects/$SLUG
_STACK_FILE=~/.gstack/projects/$SLUG/callstack.md
[ -f "$_STACK_FILE" ] && echo "STACK_EXISTS" || echo "NO_STACK"
```

---

## Mode Detection

The user's invocation determines the mode:

- `/stack` → **VIEW** — display the full call stack (file as-is, nothing folded)
- `/stack brief` → **BRIEF** — compact view: fold completed items, expand current + pending
- `/stack update` → **UPDATE** — the user will describe what changed, update the file
- `/stack init` → **INIT** — create a new call stack from scratch (interactive)

---

## Mode: VIEW (default)

Read and display the call stack file:

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
cat ~/.gstack/projects/$SLUG/callstack.md 2>/dev/null || echo "NO_STACK_FILE"
```

If `NO_STACK_FILE`: tell the user "No call stack found for this project. Run `/stack init` to create one."

If the file exists, display it **as-is** — every line, nothing folded, nothing summarized. The file is the source of truth. Then add a one-line summary:

---

## Mode: BRIEF (compact view)

Read the same file as VIEW, but display a compact version:

1. For completed top-level items (`- [x]`): show one line with the item text
2. For completed sub-items under an incomplete parent: fold into a count — e.g., `  - [x] 2.1~2.13 done (13 items)`
3. For the current item (`← HERE`) and all items after it: show in full, unfolded
4. Always show the full summary line at the bottom

This is a **display-only transformation** — never modify the actual file. The file always stores the full history.

```
📍 Current: [the item marked ← HERE]
⏭️  Next: [the first unchecked item after HERE]
```

That's it. Don't add commentary.

---

## Mode: INIT

AskUserQuestion: "Describe the high-level goals and tasks for this project. I'll organize them into a call stack."

After the user responds, create the call stack file using this format:

```markdown
# Call Stack: [project name]
Updated: YYYY-MM-DD

- [x] 1. [completed task]
- [ ] 2. [in progress or pending task]  ← HERE
  - [x] 2.1 [completed subtask]
  - [ ] 2.2 [pending subtask]
  - [ ] 2.3 [pending subtask]
- [ ] 3. [future task]
  - [ ] 3.1 [subtask]
  - [ ] 3.2 [subtask]
```

Rules:
- `[x]` = done
- `[ ]` = not done
- `← HERE` marks the current position (exactly one item)
- Indent with 2 spaces for subtasks
- Number items for easy reference (1, 2, 2.1, 2.2, 3, etc.)
- Keep descriptions short (5-15 words max)
- Update the date

Write to the state file:

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
```

Write the content to `~/.gstack/projects/$SLUG/callstack.md`.

---

## Mode: UPDATE

Read the current stack first:

```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)
cat ~/.gstack/projects/$SLUG/callstack.md 2>/dev/null || echo "NO_STACK_FILE"
```

If no file exists, redirect to INIT mode.

Listen to what the user says they've done or what changed, then update the file:
- Mark completed items with `[x]`
- Move `← HERE` to the new current position
- Add new items if the user mentions new work
- Update the date
- Preserve the overall structure

Show the updated stack after writing.
