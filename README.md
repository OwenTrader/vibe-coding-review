# vibe-coding-review

> 中文说明见 [README.zh-CN.md](README.zh-CN.md)

A multi-dimension **code review / code audit** Agent Skill with **49 review dimensions**, producing a severity-graded (P0–P3) structured report with file:line evidence for every finding.

## Install

```bash
# Install the skill from this repo
npx skills add OwenTrader/vibe-coding-review --skill code-review

# List what's inside without installing
npx skills add OwenTrader/vibe-coding-review --list
```

## What it covers

| Group | Dimensions |
|-------|-----------|
| Presentation | UI/UX style consistency, shared style/component extraction |
| Correctness | Code quality, business logic correctness, state management, CRUD completeness, functional flow tracing, error handling |
| Tooling | Static-analysis & lint adoption (ruff for Python, ESLint for TS, etc.) — low-level errors must be caught by tools, not by human review alone |
| Security | Security boundaries, hardcoded key-parameter audit, permissions & authn/authz, secrets lifecycle |
| Data | Data consistency, caching, database design, data lifecycle, data migration, backup & recovery |
| Reliability | Logging (mandatory on critical paths **and** must not grow unbounded), observability, async/message queue, task scheduling, rate limiting, disaster recovery, resource lifecycle |
| Delivery | Feature flags / canary, deployment & release, rollback-ability, API versioning |
| Evolution | Architecture, maintainability, extensibility, technical debt, product gaps, API contracts, privacy, i18n & a11y, testability, SEO/web performance |

See `skills/code-review/SKILL.md` for the full dimension table and workflow, and `skills/code-review/references/` for per-dimension checklists.

## Usage

Just ask your coding agent to review code, e.g.:

- "审查这个模块的代码"
- "Review this PR before merge"
- "上线前审查这段代码"

The skill then walks the review dimensions, cross-verifies P0/P1 findings, and emits a report with a "must fix / can defer / needs your decision" closing checklist.

## Repository layout

```
.
├── README.md
├── README.zh-CN.md
├── LICENSE
└── skills/
    └── code-review/
        ├── SKILL.md
        └── references/
```

## License

Apache-2.0
