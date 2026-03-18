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

### /stack — Call Stack Visibility

See where you are in the project, what's done, what's next. Persistent across sessions.

```
/stack          # view current position
/stack init     # create a new call stack
/stack update   # mark progress, move position
```

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

### One agent, one task

Each agent role does exactly one thing. The prompt researcher researches — it doesn't write prompts (that's a future prompt-writer role). The stack viewer shows state — it doesn't manage tasks.

## Roadmap

| Phase | What | Status |
|-------|------|--------|
| 1 | Education system (core entry point — learn before you use) | Planned |
| 2 | /prompt-research (research tool) | Done |
| 3 | /prompt-writer (apply research findings) | Planned |
| 4 | Continuous learning (self-evolution from practice) | Planned |
| 5 | Multi-session coordination (5-layer system) | Designed, not built |
| 6 | Independent project scaffold | Planned |

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
prompt-research/         # /prompt-research agent
  SKILL.md.tmpl          # template source (edit this)
  SKILL.md               # generated (don't edit)
stack/                   # /stack visibility tool
  SKILL.md.tmpl
  SKILL.md
subproject/              # reference repos (.gitignored)
  everything-claude-code/
  prompt_alchemy_reference.md
```

### Commit style

Bisect commits — each commit is one logical change. Infrastructure separate from features, templates separate from generated files.

## License

MIT — same as gstack.
