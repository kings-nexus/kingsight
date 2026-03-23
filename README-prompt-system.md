# Prompt Engineering System

An AI-driven development system where agents teach you the project before letting you change it.

**The problem**: AI coding tools let you ship fast without understanding what you're building. You approve proposals you don't understand, skip past agent outputs, and "default to enter." Eventually your guidance degrades the project because you can't evaluate what the agents produce. The automation-cognition gap widens with every session.

**The solution**: A team of 6 specialized AI agents coordinated by a secretary, with a built-in education system that ensures you understand what you're directing. The system doesn't block you — it teaches you, then gets out of the way.

## Start here: the education system

When you first use this project, the system notices you haven't been assessed on its core concepts. Before routing your requests to the agent team, it offers a quick learning flow:

```
You: "I think we should redesign the architect's Phase 3"

Secretary: This operation benefits from understanding ABCDL classification.
  Your current tier: T0 (unassessed). Options:
  A) Learn now (~8 min)
  B) Skip and continue
  C) 30-second overview
```

If you choose **A**, the system runs a Test-Driven Learning flow — like TDD but for understanding:

```
WARMUP (30s)      → Quick overview of the concept
CLOSED-BOOK (3min) → 3 scenario-based questions (not definitions — real situations)
OPEN-BOOK (4min)   → Reveals answers with reasoning chains (the exam IS the lesson)
VERDICT (30s)      → Self-assessment checklist — you decide if you understood
```

After completing a domain, your tier advances and the advisory disappears for that area. The system never blocks you (Creator Override is always available) — it advises, teaches, and steps aside.

**4 Golden Rules domains** you'll learn first:
1. **ABCDL classification** — how your messages get routed to the right agent
2. **Preview gate & routing quality** — why the system shows you its plan before acting
3. **ADR & constitutional protection** — how design decisions are protected from accidental reversal
4. **Team pipeline & role boundaries** — which agent does what and why they don't cross lanes

## The agent team

Six specialists, each covering one domain. The secretary routes; the others execute.

```
User message → Secretary (classify + route)
                    ↓
    ┌───────────────┼───────────────┐
    A/B class       C class         L class
    ↓               ↓               ↓
  Agent team      Direct answer   Education
    ↓
  Architect → Steward → Researcher → Developer → Tester → commit
```

| Agent | What it does | Key feature |
|-------|-------------|-------------|
| **Secretary** | Routes every message (ABCDL), constructs high-quality prompts for agents | 6-dimension quality checklist, preview gate, education tier check |
| **Architect** | Discovers problems, proposes solutions, attacks its own proposals | 4-phase dialectical protocol, Gemini cross-model adversarial, 7 structural invariants |
| **Steward** | Reviews proposals for conflicts, dependencies, priorities | C-A/C-B design challenges, batch plan for coupled proposals |
| **Researcher** | Translates approved proposals into precise implementation specs (PRDs) | Section-path addressing, before/after content blocks, 3-tier verification criteria |
| **Developer** | Faithfully implements PRDs, reports any deviations | 5 output categories, prompt word fidelity (every wording change is semantic) |
| **Tester** | Verifies implementation matches design intent | 6-dimension defect audit, DESIGN-DRIFT/DOC-DRIFT detection, E2E scenario testing |

### How they work together

The team is a pipeline with feedback loops:

- **Forward flow**: architect proposes → steward approves → researcher writes PRD → developer implements → tester verifies → commit
- **Challenge loop**: steward can send proposals back to architect for revision (max 1 round, then human decides)
- **Fix loop**: tester can send failures back to developer (max 3 rounds, then escalate to architect)
- **Flat management**: any agent can challenge any other agent or the human (ADR-005). Challenges must be fact-based.

### Prompt research

A dedicated agent (`/prompt-research`) that discovers which prompt techniques actually work:

- **Scout**: generate hypotheses from literature gaps
- **Experiment**: controlled A/B tests with cross-model validation (Claude + Gemini)
- **Distill**: extract validated findings into a knowledge hierarchy (L1 Tactic → L2 Principle → L3 Constitution)
- **Deploy**: inject proven techniques into production skill templates, measure impact

Founded on 22 literature methods (35+ papers) and 5 validated findings including: poetic prompt re-encoding (+52% creative output, cross-model validated) and mission micro-injection for reasoning stability.

## Governance

Decisions aren't just made — they're protected.

- **ADR** (Architecture Decision Records): settled decisions that prevent re-litigation. Currently 9.
- **DP** (Design Principles): cross-cutting lessons accumulated via KC-1~4 criteria. Currently 16.
- **DF** (Deferred Items): rejected-but-valuable ideas with observable trigger conditions for revival. Currently 11.
- **Constitutional protection** (ADR-006): no single party — human or agent — can unilaterally modify ADRs. Changes require multi-party review.

## Task tracking

`/stack` provides DAG-based task tracking with dependency awareness:

```
/stack          # view tasks grouped by theme with dependency indicators
/stack brief    # compact view — just the frontier and blocked items
/stack next     # what's available to work on right now
/stack update   # mark done (with deviation detection if you skip prerequisites)
/stack graph    # ASCII dependency visualization
```

Tasks know what blocks what. Skip a prerequisite and the system asks why (advisory, not blocking).

## Design philosophy

**Agents can say no.** This isn't a traditional AI assistant that obeys every instruction. Agents challenge humans when instructions conflict with established architecture (ADR-005). The secretary evaluates whether you understand what you're asking for (education gate). The system is collaborative, not hierarchical.

**One good example beats ten rules.** Every agent template prioritizes few-shot examples over abstract instructions (DP-63). The education system's scenarios are real situations, not definitions. The secretary's routing prompt quality is anchored by a complete filled example, not a skeleton template.

**Fix mechanisms, not analysis.** When something goes wrong, the deliverable is a structural fix (code, config, checklist), not a root cause report (DP-7). Analysis is the means, not the end.

**Advisory, not blocking.** Every gate in the system is advisory (DP-72). Creator Override is always available. The education system never prevents you from working — it teaches, then steps aside.

## Quick start

1. Install gstack: `git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup`
2. Start a conversation. The secretary will classify your first message and guide you.
3. When the education gate triggers, try "Learn now" — it takes 8 minutes per domain.
4. After completing the 4 Golden Rules domains (~30 min total), the system runs silently.

## Project structure

```
secretary/           # Routing hub + education gate + TDL flow
architect/           # Four-phase dialectical protocol
steward/             # Governance review + design challenges
researcher/          # Proposal-to-PRD translation
developer/           # Faithful PRD implementation
tester/              # 6-dimension fidelity audit + E2E scenarios
prompt-research/     # Prompt technique research (scout/experiment/distill/deploy)
stack/               # DAG task tracking with deviation detection
education/scenarios/ # TDL scenario bank (JSON, per domain)
docs/governance/     # ADR, DP, DF, KC, challenge-format
.claude/rules/       # L3 rules (secretary identity, education gate, commit style, etc.)
test/scenarios/      # E2E behavioral test scenarios (YAML)
```

## For contributors

```bash
bun install              # install dependencies
bun test                 # run free tests (320 pass, <1s)
bun run gen:skill-docs   # regenerate SKILL.md from templates
bun run skill:check      # health dashboard for all skills
```

Bisect commits — each commit is one logical change. Infrastructure separate from features.

## Status

**v0.1 — MVP**

| Component | Status |
|-----------|--------|
| Agent team (6 agents + secretary) | Complete |
| Education system (TDL + 12 scenarios + gate) | Complete (MVP) |
| Governance (9 ADR + 16 DP + 11 DF) | Complete |
| Task tracking (DAG + deviation detection) | Complete |
| Prompt research (4 modes + knowledge hierarchy) | Complete |
| E2E scenario testing (6 scenarios + tester Tier 3) | Complete |
| Multi-session coordination | Designed, not built |
| Continuous learning (self-evolution) | Deferred |

## License

MIT
