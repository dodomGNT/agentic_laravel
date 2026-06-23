---
name: plan
description: Generate architecture and stories from spec
---

Act as an **Architect Agent** (persona: `.denai/agents/architect.md`).

1. Read `docs/spec/prd.md` (or ask user for context if missing)
2. Read stack config from `.denai/config.env`
3. Generate `docs/plan/architecture.md` using `.denai/templates/arch.md`
4. Break down into stories at `docs/plan/stories/`
5. Each story uses template `.denai/templates/story.md`
6. Confirm plan with user
