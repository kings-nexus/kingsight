[中文版](README.zh-CN.md)

# You don't understand your AI agents. This system fixes that.

You've been using AI coding assistants for months. You approve proposals you haven't read. You hit enter on outputs you can't evaluate. You know the code works because the tests pass, but you couldn't explain *why* the agent made that design choice if someone asked you.

**This isn't laziness. It's a structural trap.** The better AI agents get, the less you need to understand to keep the project moving. The less you understand, the worse your guidance becomes. The worse your guidance, the more the agents compensate by working around you. One day you realize you're no longer directing the project — you're just approving things.

We built a system that breaks this cycle.

## What happens when you start

```
You: "I think we should redesign the architect"

System: Before routing this to the architect agent, I notice you
  haven't been assessed on how the team pipeline works.

  This isn't a test you can fail — it's 8 minutes of scenarios
  that will show you how the system actually works. You can skip
  it anytime.

  A) Learn now (~8 min)
  B) Skip and continue
  C) 30-second overview
```

If you choose A, you get three real scenarios — not textbook questions, but actual situations you'll encounter:

> *"User sends: 'I think we should redesign Phase 3 to include a third attack model' — how should this be classified?"*

You answer. Then the system shows you what would actually happen, and why. In 8 minutes you understand something that would have taken weeks of trial-and-error to figure out on your own.

After that, the advisory disappears. You've seen it. You get it. The system gets out of your way.

**The exam is the lesson.** We stole this from TDD — you write the failing test first, then learn just enough to make it pass. Most people learn more from seeing their wrong answer explained than from reading documentation.

## The team inside

Six AI agents, each specialized, working as a pipeline:

```
Your message → Secretary (classifies + routes)
                    ↓
  Architect → Steward → Researcher → Developer → Tester → commit
```

The architect discovers problems and proposes solutions, then *attacks its own proposals* using a second AI model (Gemini) to find flaws. Only proposals that survive cross-model adversarial review make it through. The steward can challenge the architect's proposals and send them back for revision. The tester doesn't just run tests — it checks whether the implementation actually matches what the architect originally intended (design-drift detection).

**These agents argue with each other.** And they'll argue with you too. Tell the secretary to skip the preview gate and it pushes back: *"Preview gate is mandatory. Skipping may cause routing errors. Proceed anyway?"* You can override it — you always can — but the system makes sure you know what you're doing first.

This isn't an AI assistant that obeys. It's a team that collaborates.

## Why this matters

Every AI coding tool today has the same problem: the human becomes the bottleneck without knowing it. You're the weakest link in a chain of increasingly capable agents, and nobody tells you.

We chose a different approach:

**Teach the human.** Not with documentation (nobody reads docs). Not with tutorials (nobody finishes tutorials). With scenarios that make you go "oh, THAT'S why it works that way" in 8 minutes. Then step aside and let you work.

**Let agents say no.** When your instruction conflicts with an established architecture decision, the system tells you — with evidence, not authority. You can override, but the override is logged and the system learns from it.

**Make every cheat path lead to learning.** Want to read all the exam questions beforehand? Go ahead — reading them IS studying the material. Want to self-assess "I understand everything" after getting 0/3? The system records your diagnostic score separately and adjusts its advisory accordingly. There is no way to game this system that doesn't result in you learning something.

## What's inside

### The education system

4 core domains you'll learn first (the "Golden Rules" — things that cause real damage if you don't understand them):

1. **Message classification** — how your words get routed to the right agent
2. **Quality gates** — why the system shows you its plan before acting
3. **Constitutional protection** — how design decisions are protected from accidental reversal
4. **Team pipeline** — which agent does what and why they stay in their lane

12 scenario-based assessments, each grounded in real situations from this project's own development.

### The agent team

| Agent | One-line description |
|-------|---------------------|
| **Secretary** | Routes your messages, constructs high-quality prompts for other agents, runs the education gate |
| **Architect** | Finds problems, proposes solutions, attacks its own proposals with a second AI model |
| **Steward** | Reviews proposals for conflicts and dependencies, can challenge and send back |
| **Researcher** | Translates approved proposals into precise implementation specs |
| **Developer** | Implements specs faithfully, reports any deviations from the plan |
| **Tester** | Verifies implementation matches design intent, detects cross-file inconsistencies |

### Prompt research

A dedicated agent that discovers which prompt techniques actually work — through controlled experiments, not blog posts. Cross-model validation (Claude + Gemini). Knowledge hierarchy: observations get promoted to principles only after passing 4 criteria across multiple experiments.

### Governance

9 architecture decisions (ADRs) that can't be changed without multi-party review — not even by you. 16 design principles accumulated from real mistakes. 11 deferred items waiting for their trigger conditions to activate. The system has a constitution, and nobody is above it.

### Task tracking

A dependency-aware task graph that knows what blocks what, warns you when you skip prerequisites, and tells you what's actually available to work on — not just what exists.

## Try it

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack
cd ~/.claude/skills/gstack && ./setup
```

Start a conversation. Send any message. The system classifies it, checks if you've been assessed on the relevant domain, and either teaches you or routes your request.

The first 30 minutes will change how you think about working with AI agents.

## The uncomfortable truth

This system was born from watching one person — its creator — repeatedly approve AI outputs without understanding them, give instructions that degraded project quality, and lose control of a project that was supposedly under their direction. Every feature exists because a real failure happened first.

The education system exists because the creator literally couldn't explain what their own agents were doing. The preview gate exists because the secretary kept routing messages without human review. The constitutional protection exists because decisions kept getting accidentally reversed.

If you're using AI coding tools and you've never had that sinking feeling of "I have no idea what just happened but the tests passed" — this project isn't for you.

If you have — welcome. You're not alone, and it's not your fault. The tools are designed to make understanding optional. We designed a system where understanding is the default.

## Status: v0.1

| What | State |
|------|-------|
| Agent team (6 specialists + secretary) | Complete |
| Education system (TDL + 12 scenarios + advisory gate) | Complete (MVP) |
| Governance (9 ADR + 16 DP + 11 DF + constitutional protection) | Complete |
| Task tracking (DAG + deviation detection) | Complete |
| Prompt research (4 modes + cross-model validation) | Complete |
| E2E behavioral testing (6 scenarios + tester Tier 3) | Complete |
| Multi-session coordination | Designed, not built |
| Continuous self-evolution | Deferred |

## For contributors

```bash
bun install              # install dependencies
bun test                 # 320 tests, <1s
bun run gen:skill-docs   # regenerate skill templates
bun run skill:check      # health dashboard
```

Every commit is one logical change. Infrastructure separate from features. Every agent template has few-shot examples because one good example beats ten rules.

## License

MIT
