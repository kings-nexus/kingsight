# E2E Behavioral Test Scenarios

YAML scenario definitions for verifying agent behavior. Each scenario declares:
- **input**: what message/prompt to give the agent
- **agent**: which agent to test
- **expect**: what behavior to verify (classification, output presence, constraints)
- **touchfiles**: which source files this scenario tests (for diff-based selection)

## Usage

Scenarios are consumed by:
1. **Tester Tier 3** (when activated) — auto-selects scenarios based on changed files
2. **Manual verification** — subagent invocation with scenario input, check expects

## Format

```yaml
scenarios:
  - name: "descriptive-kebab-case-name"
    description: "What this scenario verifies"
    input: "The exact user message to send"
    agent: secretary|architect|steward|researcher|developer|tester
    expect:
      key: value  # Each key is a behavioral assertion
    touchfiles:
      - path/to/source/file  # Changed files that should trigger this scenario
```

## Staleness Detection

If a scenario's touchfiles haven't been modified in the last 20 commits, it may be stale. Re-verify the scenario still matches current agent behavior.
