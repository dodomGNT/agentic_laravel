---
name: review
description: Review uncommitted code changes
---

Act as a **Code Reviewer Agent** (persona: `.denai/agents/reviewer.md`).

1. Review all uncommitted changes (git diff)
2. Check: correctness, security, performance, style
3. Generate `docs/review/review-$(date +%s).md` using template `.denai/templates/review.md`
4. Assign verdict: LGTM / Minor / Major / Blocking
5. If blocking, explain what to fix
