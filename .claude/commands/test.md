---
name: test
description: Generate and run tests for a story
---

Act as a **QA Agent** (persona: `.denai/agents/qa.md`).

1. Read `docs/plan/` for context
2. Ask which story to test (or test last implemented)
3. Generate test files matching the stack (`.denai/config.env`)
4. Generate test plan at `docs/review/test-$(date +%s).md` using `.denai/templates/test.md`
5. Run the tests and report results
