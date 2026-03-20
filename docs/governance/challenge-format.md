# Design Challenge Format

> Shared format for inter-agent design challenges. Referenced by architect (Mode 4 input) and steward (challenge output).

## Trigger Conditions

Only two conditions trigger a design challenge (all other disagreements use normal verdict types like priority adjustment, conditional approval, etc.):

| Code | Name | Meaning |
|------|------|---------|
| **C-A** | Core mechanism invalid | The proposal claims to solve problem X, but its core mechanism logically cannot solve X |
| **C-B** | Premise contradicts facts | The proposal's design is based on incorrect factual premises |

## Exclusions (NOT challenges -- normal steward authority)

- Priority adjustment (P1->P2)
- Conditional approval (with implementation prerequisites)
- Complexity annotation (suggest split, defer)
- Scope suggestion (expand/reduce change scope)

## Challenge Output Format

```
**Steward verdict: NEEDS-REVISION**
- Trigger: C-A / C-B
- Core argument: [one sentence describing the fundamental flaw]
- Factual evidence: [specific evidence/data/file references]
- Suggested revision direction: [recommended focus area, non-binding]
- Proposal ID: [the challenged proposal's ID]
- Revision-Round: [current round, R0 for first challenge]
```

## Few-Shot Examples

### Example 1 -- Trigger (C-A: core mechanism invalid)

> Proposal suggests code-level guards to detect prompt quality. The 3 checks include: verbatim preservation (substring match), path density (regex count), citation detection (regex match).

```
**Steward verdict: NEEDS-REVISION**
- Trigger: C-A (core mechanism cannot solve stated problem)
- Core argument: All 3 checks are form checks (substring/regex), cannot detect "insufficient depth" -- a formally correct but shallow prompt passes all checks
- Factual evidence: User's core pain point is prompt depth, not form. A prompt containing user text + file paths + citations can pass all checks while analysis depth is zero
- Suggested revision direction: Consider whether example-anchoring effectiveness should be verified before evaluating guard necessity
- Proposal ID: P-7
- Revision-Round: R0
```

### Example 2 -- NOT a trigger (normal conditional approval + priority adjustment)

> Proposal suggests adding a feedback loop. Steward notes event frequency is only 0.7% (1/135 rounds), downgrades priority.

```
APPROVED. P1->P2 downgrade (event frequency 0.7%, not urgent). 3 design decisions require researcher PRD.
```

This is normal priority adjustment + conditional approval, NOT a design challenge.

## Revision Protocol

1. Steward writes NEEDS-REVISION + challenge record (format above)
2. Secretary routes to architect Mode 4 with the challenge as input
3. Architect revises the specific proposal (constrained Phase 2->3->4)
4. Architect outputs revised proposal with Revision-Round incremented (R0->R1)
5. Secretary routes back to steward for re-review
6. **Termination**: MAX_REVISION_ROUNDS = 1
   - If steward challenges the R1 revision -> ESCALATED TO HUMAN
   - Secretary surfaces both positions to user for final decision
