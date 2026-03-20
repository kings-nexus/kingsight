---
description: "E2E eval failure blame protocol"
alwaysApply: true
---

# E2E Eval Failure Blame Protocol

When an E2E eval fails, **never claim "not related to our changes" without proving it.**

**Required before attributing a failure to "pre-existing":**
1. Run the same eval on main and show it fails there too
2. If it passes on main but fails on the branch — it IS your change. Trace the blame.
3. If you can't run on main, say "unverified — may or may not be related" and flag as risk

"Pre-existing" without receipts is a lazy claim. Prove it or don't say it.
