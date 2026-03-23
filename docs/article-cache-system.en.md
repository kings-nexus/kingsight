[中文版](article-cache-system.md) · [← Back to project](../README.md)

# You've Been Building a Cache System. You Just Didn't Know It.

If you've spent any time with Claude Code, you've been doing one thing over and over: building a cache system.

Except you're not caching data. You're caching human decisions.

Each layer up, you're hardening a higher-order decision into reusable structure, so you intervene less often. From raw prompts, to skills, to pipelines, to agents, to agent teams — you've probably climbed most of this ladder already. But I found there's one more layer at the top that nobody's built yet.

This article maps out the full progression, then introduces the final layer: the Secretary Agent.

---

## Layer 0 — Raw Prompt (No Cache)

In the beginning, everyone drives tasks by hand. One prompt at a time. No reuse.

You are the scheduler inside the for loop, and you have to show up every iteration.

You want Claude Code to build a React component. You say "create the file." Then "add state." Then "write a useEffect." Then "add error handling." Each step requires you to push. Finish one component, start the next — same sequence all over again.

It works. It does not scale. You notice most of your conversations are the same conversation wearing different clothes.

## Layer 1 — Skill (Caches: Single-Task Execution Paths)

Someone noticed that prompting through a fixed-pattern task one round at a time is repetitive and boring.

So they packaged "a fixed task that used to take many prompts" into a single invocation. That's a skill — the first layer of cache.

In Claude Code terms, a skill is a SKILL.md file: steps, tools, output format, all pre-defined. You say "generate a report using the docx skill" and it runs the entire path without hand-holding. Anthropic officially shipped the skill system in October 2025. The community now has hundreds — PDF generation, Playwright testing, code review, you name it.

**What's cached: how to do a specific thing.**

But a large task doesn't fit in one skill. You can factor out pieces into smaller skills and compose them like Lego blocks. The problem: not every task decomposes cleanly into atomic skills, and manually orchestrating their order and data-passing is tedious.

Enter the pipeline — this is the problem Dify and n8n were originally built to solve.

## Layer 2 — Pipeline (Caches: Inter-Task Dependencies)

A pipeline is a DAG of skills, handling complex task flows where sub-tasks have ordering constraints and state dependencies.

Concrete example: you need a "competitive analysis report." Scrape data, clean formats, run comparisons, generate charts, typeset the document. Each step's output feeds the next. Skip one or reorder them and you get garbage. That's why pipelines exist.

Skills solve "how to do it." Pipelines solve "in what order, and whose output feeds whose input." Dify and n8n are the canonical visual pipeline builders — drag nodes, draw edges, define data flow.

**What's cached: the orchestration logic between tasks.**

But pipelines are static. You have to design all branches in advance. In practice, many tasks require runtime decisions: what to do next depends on what just happened, and you can't enumerate every path ahead of time. Every time a fork appears, a human has to decide which branch to take — which is the same manual-driving problem you thought you'd solved.

## Layer 3 — Agent (Caches: Runtime Decisions)

Agents solve this by encoding "when you see X, do Y" into a rules file.

In Claude Code, that's your CLAUDE.md. It specifies: which skill to call when, which pipeline to follow, what counts as "done," when to stop and ask the human. The agent reads these rules and makes decisions at runtime. You don't have to watch.

This is the core distinction Anthropic makes in *Building Effective Agents*: a workflow (pipeline) uses predefined code paths to orchestrate LLMs and tools; an agent lets the LLM dynamically direct its own process and tool usage. A pipeline is a map you drew in advance. An agent is a driver who can read traffic and navigate on its own.

The agent's judgment isn't magic — it comes from the rules you wrote plus the model's own reasoning.

**What's cached: the ability to choose tools and paths at runtime.**

But a single agent's rules file has limits. When the task spans multiple domains — research *and* implementation *and* documentation — cramming too many responsibilities into one agent makes it start hallucinating, rules conflict with each other, and the context window buckles. You're back to manually splitting "who handles this part."

## Layer 4 — Agent Team (Caches: Specialization)

Naturally, you split into multiple specialist agents: one for research, one for execution, one for teaching. Each has its own rules file, its own lane.

By 2026, this is the mainstream pattern in the Claude Code ecosystem.

GStack (open-sourced by Y Combinator CEO Garry Tan, 16,000+ stars) turns Claude Code into a virtual engineering team — CEO reviews product direction (`/plan-ceo-review`), engineering manager locks architecture (`/plan-eng-review`), QA opens real browsers for live testing (`/qa`), release engineer ships PRs (`/ship`). Garry Tan himself averages 10,000 lines of code and 100 PRs per week using this workflow, running 10–15 parallel sprints.

agency-agents goes further, packaging an entire "AI agency" — frontend specialist, security auditor, Reddit community manager — each agent with its own persona, process, and delivery standards, cross-compatible with Claude Code, Copilot, Gemini CLI, and Cursor.

OpenCrew uses Slack channels as job roles, each channel bound to a specialist agent (CoS, CTO, Builder, Knowledge Officer...), with threads as work orders. Its creator is a non-technical user with an economics/MBA background — he manages a 7-agent team through Slack without writing code.

**What's cached: domain specialization and collaboration structure.**

Sounds complete? Here's the problem.

**You're still the dispatcher.** In GStack, you decide whether to type `/plan-ceo-review` or `/qa` or `/ship`. In agency-agents, you say "Use the code-reviewer to check my changes" — which means you need to know the agent exists, what it's called, and when to call it. In OpenCrew, you need to know which channel to go to and which bot to @.

The team is powerful. But the cognitive overhead of *using* the team is entirely on you. Every time you want something done, you mentally index "whose job is this?", switch to the right agent, and issue the instruction in the right format.

You've become your own dispatch center — which is Layer 0 again, except now you're dispatching agents instead of prompts.

## Layer 5 — Secretary Agent (Caches: User Intent Understanding)

So you need a secretary.

Like an executive who doesn't need to memorize every employee's responsibilities, communication style, and workflow — the secretary handles that. You tell the secretary what you want. The secretary knows who to call and how to brief them.

**Nobody is building this layer right now.**

I searched the top 20 projects on the March 2026 star-history Coding AI rankings — OpenClaw (+6k), agency-agents (+5.8k), GStack (+3.6k), Ruflo, agent-teams-lite, Overstory — none of them build this layer. They all stop at Layer 4: increasingly powerful agent teams, but with the assumption that the user knows who to call and what to say.

The two closest approaches:

**OpenAI Swarm's triage agent pattern.** It classifies user messages by intent and routes to specialist agents — the right idea. But it classifies by *domain*: is this a billing question or a refund question? "I think the refund flow might have a bug" and "process this refund" both look like "refund domain" to it, so both get routed to the refund agent.

**The AgentOS paper from academia.** It proposes an Agent Kernel — a unified natural-language entry point (called "Single Port") that does intent parsing and dispatches to Skill Modules. Architecturally, it's almost isomorphic to my Secretary. But it's still a paper, and it hasn't solved the core problem of intent ambiguity.

### ABCDL: Classifying What You're *Doing*, Not What You're *Saying*

The Secretary Agent's first job isn't execution. It's classification.

This is the ABCDL system:

| Class | What you're doing | Secretary's action | Examples |
|-------|-------------------|-------------------|----------|
| A | Thinking, exploring, undecided | Route to research agent | "I think...", "What if we..." |
| B | Clear execution instruction | Route to execution agent | "Run the tests", "Deploy F-003" |
| C | Factual question | Secretary answers directly | "How many ADRs do we have?" |
| D | Continuing a previous flow | Resume pipeline state | "Continue", "Choose A", "Approved" |
| L | Learning request | Route to education system | "Teach me...", "I don't understand..." |

Compared to Swarm's triage agent, the key difference is this: **the same topic can be A or B.** "I think we should optimize the deployment pipeline" and "Optimize the deployment pipeline" — same domain, completely different system responses. The first should go to the research agent for feasibility discussion. The second should go straight to the execution agent. This distinction doesn't exist in any triage/router system today.

**Why not fewer than five classes?** Because every merge creates problems. Merge A and B, and "I think we should..." gets treated as an execution order — the stronger your team's execution capability, the higher the cost of that misclassification. Merge C and D, and "continue" gets treated as a new query.

**When ambiguous, default to A: better to overthink than to overact.** Excess deliberation is correctable. Premature execution may not be.

**What's cached: the translation process itself — turning "what the user said" into "what the system should do."**

### But the Secretary Isn't Just a Classifier

If the secretary only classified your message and forwarded it, it would just be a fancier version of Swarm. What makes it different: **it does its homework before forwarding.**

You say "I think the architect's Phase 3 has a problem." A router would toss that sentence straight to the architect. But think about how this works in real life — you tell an executive's assistant "set up a meeting with Director Zhang," and a good assistant doesn't just pick up the phone. She checks Director Zhang's recent calendar, confirms unresolved items from the last meeting, prepares relevant materials.

My secretary does the same thing: reads files, checks history, reviews governance documents, then packages those facts alongside your original message into a task briefing far denser than your one sentence.

Then — **it shows you the briefing before dispatching.**

Why? Because you said one sentence, but you didn't write the briefing. The secretary translated your intent into a format the system can act on — **and that translation is where miscommunication is most likely to happen.** One glance from you, and the misfire disappears.

There's an uncomfortable fact here: in a serial architecture, **one degree off upstream means ten degrees off downstream.** The quality of the secretary's briefing directly determines the architect's analysis quality, which determines the steward's review quality. All the way down. A small translation error or a lazy briefing from the secretary gets amplified at every layer.

So the secretary isn't the simplest role on the team. It's the most critical one.

### Then I Found an Even More Uncomfortable Problem

After building the Secretary, I thought I was done. Layer 0 through Layer 5, cache system complete.

Then I looked at what I'd actually been doing during development.

I had approved architecture proposals I didn't understand. I'd hit enter on agent outputs because I didn't know how to evaluate whether they were good or bad. I'd told the agent to "optimize the secretary's routing" — but I couldn't explain how the routing system actually worked. The agent obligingly went and executed. The direction was wrong. We spent a lot of time on rework.

I'm not a lazy user. I'm a **user who lacked the ability to evaluate agent output.** These two things look identical from the outside — both involve hitting enter — but the root causes are completely different. One is an attitude problem. The other is a structural problem.

**The better agents get, the less you need to understand to keep the project moving. The less you understand, the worse your guidance. The worse your guidance, the more the agents work around you. This is a positive feedback loop, and it self-accelerates.**

Where's the circuit breaker? Not by making agents weaker (that's regression). By making the human stronger. Not by writing documentation for people to read (nobody reads docs). By delivering understanding at the exact moment the person needs it, in the most efficient format possible.

So I added an education system to the Secretary.

### The Exam Is the Lesson

When you send an A-class message, the secretary first checks: is your understanding of the relevant knowledge domain sufficient? If yes — you don't notice anything. The message routes normally. If not:

```
"This operation requires understanding ABCDL classification.
  You haven't been assessed on this yet.
  A) Learn now (~8 min)
  B) Skip and continue
  C) 30-second overview"
```

If you choose A, you don't get a document. You get three scenarios:

> *"User sends: 'I think we should redesign Phase 3' — how should this be classified? Why isn't it B?"*

You answer. The system reveals the correct answer, the reasoning chain, and why your answer was wrong. **This process is the teaching.** Not "learn first, test later" — "test first, learn from the results." Same philosophy as TDD: write the failing test first, then learn just enough to make it pass.

There's a concept in education research called the testing effect (retrieval practice) — testing itself promotes long-term memory more effectively than re-reading. I didn't invent this. It's a basic finding from cognitive science. I just engineered it into the system.

The final step is self-assessment — the system doesn't tell you whether you passed. You check the box: "I believe I understand these concepts." The system respects your judgment (it never blocks you), but it remembers your diagnostic score. If you got 3 questions wrong but checked "I understand everything" — next time, the prompts come more frequently.

Someone will ask: can't this be gamed?

Yes. But every shortcut leads to learning: read the questions early? Reading them *is* learning. Skip everything? The system records it; prompts come back sooner next time. Fake the self-assessment? Diagnostic scores are tracked independently; advisory frequency adjusts automatically.

**It is impossible to game this system into a state where you've learned nothing.** This is by design, not by accident.

## Summary

Each layer of cache takes something that used to require a real-time human decision and hardens it into reusable system structure:

| Layer | What's cached | Humans no longer need to... |
|-------|--------------|----------------------------|
| Skill | Actions | Push forward one prompt at a time |
| Pipeline | Ordering | Manually sequence skills |
| Agent | Judgment | Decide which branch to take at runtime |
| Agent Team | Division of labor | Decide who should do it |
| Secretary | Intent | Decide what they're even doing |
| + Education | Understanding | Worry about falling behind the project |

From Layer 0 to Layer 5, human intervention granularity goes from "every sentence" to "one sentence." That sentence can be vague, casual, even a thought muttered to yourself — the Secretary figures out what you're actually doing and translates it into something the system can execute.

The education system makes sure that one sentence is a good one. Not by restricting what you can say, but by making sure you understand what the system is doing — and once you understand, you naturally say better things.

**This is the final layer of cache, because it caches the translation layer between you and the system. Above this layer, you're just a person with ideas — not an operator.**

---

## About This Article and This Project

Honest disclosure: this article and the entire system it describes were built through human-AI collaboration.

Specifically: the Layer 0–4 framework and the core insight behind ABCDL were written by me (the human). The detailed design of the Secretary and education system was done together with Claude Opus 4.6 — I set direction and constraints, it did research and solution design, I reviewed and revised, it implemented and tested. The second half of this article was AI-drafted and human-edited.

This isn't false modesty. It's the best possible illustration of the problem this system exists to solve. **I could not have designed a 6-agent collaboration system by myself.** But I could evaluate design proposals (most of the time), propose constraints the AI wouldn't think of (like "agents should be able to say no to the human"), and correct course when the AI went wrong (the education system exists because the AI kept making the same category of mistake).

The relationship between human and AI in this project isn't "I direct, it executes." It's closer to "we each handle what we're good at." The education system exists to make sure "what I'm good at" doesn't shrink to zero as the project grows more complex.

---

This system is open source: [GitHub](https://github.com/kings-nexus/kingsight) | [English README](../README.md) | [中文 README](../README.zh-CN.md)

Current status is v0.1 MVP — the full 6-agent team, education system, governance framework, and task tracking are all working. Everything described in this article is running code, not slides. 320 tests passing, 12 education scenarios ready, 9 architecture decisions protected by the constitution.

If you've ever had that uneasy feeling — "I don't know what the agent just did, but the tests pass" — give it a try. The first 30 minutes might change how you think about working with AI.

And if this direction interests you, perhaps the most valuable thing you could do is: **share your own Layer in the comments.** Where did you stop? What's keeping you from going higher? Maybe you're already building a Layer 6 I haven't thought of.
