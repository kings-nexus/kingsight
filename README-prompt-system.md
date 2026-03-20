# Prompt Engineering System

A systematic approach to prompt engineering — research, education, and tooling built on top of [gstack](https://github.com/garrytan/gstack)'s agent team architecture and inspired by [everything-claude-code](https://github.com/affaan-m/everything-claude-code)'s self-evolution patterns.

**This is not a prompt library.** It's a research-driven system where every technique is validated through controlled experiments, graded through a knowledge hierarchy (L1 Tactic → L2 Principle → L3 Constitution), and deployed with measured impact.

## What's built

### /prompt-research — Systematic Prompt Research Agent

A new agent role that discovers which prompt techniques work, why they work, and when they fail.

**Four modes:**

| Mode | Command | What it does |
|------|---------|-------------|
| **Scout** | `/prompt-research scout` | Generate testable hypotheses from literature gaps and existing findings |
| **Experiment** | `/prompt-research experiment` | Design and execute controlled A/B experiments with cross-model validation |
| **Distill** | `/prompt-research distill` | Extract findings from experiments, evaluate level upgrades (L1→L2→L3) |
| **Deploy** | `/prompt-research deploy` | Inject validated findings into production skill templates, measure eval delta |

**Founded on 22 literature methods (35+ papers), 4 completed experiments, and 5 validated findings:**

- **F-003 (L2 Principle):** Poetic/metaphor prompt re-encoding improves creative output +52% (cross-model validated on Claude + Gemini)
- **F-001 (L1 Tactic):** Positive mission framing stabilizes reasoning quality
- **C-001:** Style as Vector — use poetic style for creative prompts
- **C-002:** Mission Micro-Injection — append ≤40 token mission statement
- **C-003:** Inverted-U Awareness — more is not always better
- **C-004:** Task-Technique Matching — no universal optimal prompt

**14 cognitive patterns** guide the researcher's instincts: confound paranoia, effect size discipline, mechanism hunger, cross-model suspicion, inverted-U vigilance, baseline discipline, evaluator independence, and more.

**Three core mechanisms** classify all prompt techniques:
1. **Anchoring** — examples/style/structure anchor output standards
2. **Motivation Activation** — emotion/identity/mission activate higher investment
3. **Representation Shift** — style/structure/framework shift output patterns

### /architect — Four-Phase Dialectical Protocol

Strategic thinker that discovers problems, proposes solutions, then attacks them until only survivors remain.

```
Phase 1 AUDIT     → evidence-forced findings, root-cause chains
Phase 2 ENVISION  → paradigm detection, LLM reliability check, proposal templates
Phase 3a          → 5-angle self-challenge (ADR/Occam/worst-case/coupling/generalizability)
Phase 3b          → Gemini CLI cross-model adversarial (1 round, degradation to subagent)
Phase 4 META      → self-evolution + KC promotion (constrained by 7 structural invariants)
```

Three modes: governance audit (full 4-phase), strategic input (skip Phase 1), proposal retrospective.
Architect reads and proposes — never modifies files directly. Proposals go through ADR-006 review.

### /tester — Prompt Fidelity Auditor

Last line of defense. Three-tier verification: structural (bun test gate), semantic (6-dimension audit including DESIGN-DRIFT and DOC-DRIFT detection), behavioral (E2E via subagent scenarios in `test/scenarios/*.yaml`). Reports findings to developer — never self-repairs. Explicit 3-round escalation to architect.

### /developer — Faithful PRD Implementation

Implements researcher PRDs by modifying skill templates, governance docs, scripts, and rules. Five output categories with independent constraint sets. Three-tier deviation reporting — prompt template files use binary exact/deviation (no structural match: every wording change is semantic).

Does not write tests (tester exclusive). CLAUDE.md modification requires separate dedicated PRD.

### /researcher — Proposal-to-PRD Translation Layer

Translates APPROVED architect proposals into executable PRDs with section-path modification specs, before/after content blocks, and tiered acceptance criteria (static/validation/E2E).

Single-mode design: one proposal in, one PRD out. Section paths replace line numbers (gen-skill-docs invalidates line refs). No external research — gaps noted as dependencies for architect/prompt-research.

### /steward — Governance Reviewer

Reviews architect proposals for conflicts, dependencies, and priorities. Five verdicts: approve, reject, defer, needs-split, or needs-revision. Can challenge architect proposals via C-A/C-B design challenges (shared format in `docs/governance/challenge-format.md`).

Three modes: proposal review, batch plan (for coupled proposals), status report.
Advisory role — human always has final authority. MAX_REVISION_ROUNDS=1, then escalate to human.

### /secretary — Routing Hub (Entry Point)

The secretary is the main entry point. It classifies every user message and routes to the correct agent. It does NOT make decisions — only forwards and fact-finds.

**ABCDL classification:**

| Category | Intent | Routes to |
|----------|--------|-----------|
| **A** | Exploration/thinking ("I think...", "what if...") | prompt-research scout |
| **B** | Execution ("run experiment", "write a prompt") | prompt-research or prompt-writer |
| **C** | Query ("what is F-003", "show status") | Secretary answers directly |
| **D** | Continue ("next step", "approve") | Resume active pipeline |
| **L** | Learning ("teach me...", "I don't understand...") | Education system |

- Fuzzy/ambiguous messages default to **A** (think before act)
- Preview gate on A/B/L routes — shows routing plan, waits for confirmation
- Session journaling in JSONL for cross-session continuity
- TDL tier gate with fail-open (never blocks the user)

### /stack — Call Stack Visibility

See where you are in the project, what's done, what's next. Persistent across sessions at `~/.gstack/projects/$SLUG/callstack.md`.

```
/stack          # full view — grouped by theme, dependency indicators
/stack brief    # compact — fold completed themes, show frontier + blocked
/stack next     # show tasks whose prerequisites are all met
/stack update   # mark done + deviation detection + auto-unblock
/stack init     # create new graph (or migrate from old callstack.md)
/stack add      # add task with dependencies
/stack graph    # ASCII dependency visualization (experimental)
```

State is a YAML DAG (`taskgraph.yaml`) with dependency tracking:

```yaml
tasks:
  "2.14":
    title: "验证秘书分类"
    status: pending
    blockedBy: ["2.13"]    # dependency
    theme: agents           # grouping
```

- **HEAD** = last completed task. **NEXT** = computed frontier (all deps met)
- **Deviation detection**: skip a prerequisite → system asks why (ADR-005 advisory, not blocking)
- **Themes**: tasks grouped by domain (infrastructure, agents, governance, education)
- Shares infrastructure with future education system via `{{TASKGRAPH_CORE}}`

## Architecture decisions

### Independent project, not a fork

This project takes **design ideas** from gstack and ECC, not runtime dependencies:
- From gstack: cognitive patterns methodology, SKILL.md.tmpl template system, interactive workflows
- From ECC: continuous-learning philosophy, confidence scoring, knowledge leveling
- Neither codebase is a dependency — we extract patterns and build our own

### Three AI invocation paths

| Path | When | Auth |
|------|------|------|
| **Subagent** (Agent tool) | Agent team development, testing, all dev workflows | Inherits session — **no API key needed** |
| **CLI subprocess** (`claude -p`) | Multi-model, stateful sessions, content production | CLI's own OAuth |
| **Hybrid** | Subagent orchestrates + CLI generates | Both |

**Rule: Agent team work always uses subagent.** Never require API keys for development or testing.

### ABCDL message classification

Every user message is classified before routing (ADR-004):
- **A** (exploration) and **B** (execution) are the key distinction — "thinking vs doing"
- **C** (query) and **D** (continue) are lightweight — no agent dispatch needed
- **L** (learning) routes to the education system — the project's core entry point
- Fuzzy defaults to A: over-think rather than over-execute

### One agent, one domain (revised from "one task")

Each agent covers one cohesive domain, not one output type. The secretary handles routing + routing prompt construction (these are inseparable). The researcher handles technique validation + template injection. No separate prompt-writer agent — that capability is split between secretary (routing prompts) and prompt-research deploy (template optimization). See ADR-003 revision.

### Flat management, not dictatorship (ADR-005)

Agents can challenge humans. Subagents can push back on the secretary. The relationship is collaborative, not hierarchical. Challenges must be fact-based with a round limit — exceed it and escalate to human.

### Constitutional protection (ADR-006)

ADRs and L3 design principles are "constitutional" — no single party (human or agent) can modify them unilaterally. Changes require: proposal → architect evaluation → human review → approval. Unapproved changes are invalid and should be rolled back.

### Knowledge governance

Architecture decisions recorded as ADRs. Design principles accumulated via KC-1~4 criteria. Deferred items tracked with observable trigger conditions. See `docs/governance/`.

## Roadmap

| Phase | What | Status |
|-------|------|--------|
| 1 | /secretary (ABCDL routing hub) | Done |
| 2 | /prompt-research (research tool) | Done |
| 3 | /stack (call stack visibility) | Done |
| 4 | Knowledge governance (ADR + DP + DF + KC) | Done |
| 5 | Secretary as default entry point (L2 identity in CLAUDE.md) | Done |
| 6 | Education system (core entry point — learn before you use) | Planned |
| 7 | Continuous learning (self-evolution from practice) | Deferred (DF-001) |
| 8 | Multi-session coordination (5-layer system) | Designed, not built |
| 9 | Independent project scaffold | Planned |

## For contributors

### Commands

```bash
bun install              # install dependencies
bun test                 # run free tests (skill validation + snapshot)
bun run gen:skill-docs   # regenerate SKILL.md from templates
bun run skill:check      # health dashboard
```

### Project structure (new skills)

```
secretary/               # /secretary routing hub (entry point)
  SKILL.md.tmpl
  SKILL.md
prompt-research/         # /prompt-research agent
  SKILL.md.tmpl          # template source (edit this)
  SKILL.md               # generated (don't edit)
stack/                   # /stack visibility tool
  SKILL.md.tmpl
  SKILL.md
docs/governance/         # ADR, DP, DF, KC frameworks
  ADR.md
  DP.md
  DF.md
subproject/              # reference repos (.gitignored)
  everything-claude-code/
  prompt_alchemy_reference.md
```

### Commit style

Bisect commits — each commit is one logical change. Infrastructure separate from features, templates separate from generated files.

## License

MIT — same as gstack.
