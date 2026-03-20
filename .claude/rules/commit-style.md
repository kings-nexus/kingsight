---
description: "Commit and CHANGELOG conventions"
alwaysApply: true
---

# Commit Style

**Always bisect commits.** Every commit should be a single logical change.

Examples of good bisection:
- Rename/move separate from behavior changes
- Test infrastructure separate from test implementations
- Template changes separate from generated file regeneration
- Mechanical refactors separate from new features

When the user says "bisect commit" or "bisect and push," split staged/unstaged changes into logical commits and push.

# CHANGELOG Style

CHANGELOG.md is **for users**, not contributors:
- Lead with what the user can now **do**. Sell the feature.
- Use plain language, not implementation details.
- Put contributor/internal changes in a separate "For contributors" section.
