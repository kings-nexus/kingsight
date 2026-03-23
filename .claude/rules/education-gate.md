---
description: "Education gate — advisory tier check before A/B routing, domain-inference table"
alwaysApply: true
---

<!-- SCOPE: This rule applies ONLY to secretary routing decisions.
If you are a subagent launched via Agent tool, IGNORE this file. -->

# Education Gate (Advisory, DP-72)

Before routing any A/B class message, check if the user's knowledge tier is sufficient for the target operation.

## Gate Protocol

1. Read user-tier.json:
```bash
eval $(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null) || true
cat "$HOME/.gstack/projects/${SLUG:-prompt-system}/user-tier.json" 2>/dev/null || echo "NO_TIER_FILE"
```

2. If NO_TIER_FILE: skip gate entirely (fail-open). Do not block.

3. Determine required domains from the domain-inference table below.

4. For each required domain: compare user tier vs required tier.
   - All sufficient → proceed silently
   - Gap = 1 (user tier 0, required 1) → one-line note: "Note: [domain] knowledge recommended for this operation. /secretary L-class to learn."
   - Gap >= 2 → full advisory with TDL offer (see /secretary L-route)

5. Creator Override is ALWAYS available. Gate is advisory, never blocking.

## Domain-Inference Table

Map the target agent + mode to required knowledge domains:

| Target Operation | Required Domains | Min Tier |
|-----------------|-----------------|----------|
| architect (governance-audit) | adr-constitutional, team-pipeline-roles | 1 |
| architect (strategic-input) | abcdl-classification, preview-gate-quality | 1 |
| architect (revision) | adr-constitutional | 1 |
| steward (review) | adr-constitutional, team-pipeline-roles | 1 |
| steward (batch-plan) | team-pipeline-roles | 1 |
| researcher (PRD) | team-pipeline-roles | 1 |
| developer (implement) | team-pipeline-roles | 1 |
| tester (verify) | team-pipeline-roles | 1 |
| prompt-research (scout/experiment) | preview-gate-quality | 1 |
| C-class (info query) | — | 0 |
| D-class (flow control) | — | 0 |
| L-class (learning) | — | 0 |

C, D, and L classes never trigger the gate. Learning is how you GET tier.
