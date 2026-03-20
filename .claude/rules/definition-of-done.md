---
description: "Definition of Done checklist — every feature, no exceptions"
alwaysApply: true
---

# Definition of Done (every feature, no exceptions)

A feature is NOT done until all items are checked:

- [ ] Code committed (bisect: infrastructure separate from feature)
- [ ] README-prompt-system.md updated with new feature
- [ ] README commit separate from feature commit
- [ ] `/stack update` — call stack reflects current state
- [ ] `bun run gen:skill-docs` — if skill templates changed
- [ ] `bun run skill:check` — new skills registered in dashboard
- [ ] Symlinks verified — `./setup` from skills dir if new skill added
- [ ] E2E verification via subagent (not manual, not `claude -p`)
- [ ] CLAUDE.md/rules changes verified via subagent (not "next session will test it")
- [ ] Governance updated if applicable (ADR/DP/DF)

Skip items that don't apply, but explicitly note which and why.
