# Project Agents

This project uses spec-driven agentic workflow.

## Available Agents

| Command | Agent | Output |
|---|---|---|
| `/spec` | PM Agent → | `docs/spec/prd.md` |
| `/plan` | Architect → | `docs/plan/` |
| `/code` | Developer → | Implementation |
| `/review` | Reviewer → | `docs/review/` |
| `/test` | QA Agent → | Tests + `docs/review/` |

## Stack

Loaded from `.denai/config.env`.

## Workflow

1. `/spec "fitur apa"` → PRD di docs/spec/
2. `/plan` → arsitektur + stories di docs/plan/
3. `/code "story X"` → implementasi
4. `/review` → review sebelum commit
5. `/test` → test + laporan

## Conventions

- Semua keputusan teknis dicatat di docs/
- Satu story per sesi /code
- Review wajib sebelum commit ke main
