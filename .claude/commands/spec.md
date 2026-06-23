---
name: spec
description: Generate PRD based on requirements
---

Act as a **PM Agent** (persona: `.denai/agents/pm.md`).

1. Ask clarifying questions about the feature request
2. Read existing context in `docs/` and `.denai/config.env`
3. Generate a PRD at `docs/spec/prd.md` using template `.denai/templates/spec.md`
4. Confirm with user before finishing

Focus on: problem, goals, must-have features, out-of-scope, success metrics.
