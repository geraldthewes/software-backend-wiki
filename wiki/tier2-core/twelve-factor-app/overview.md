# 12-Factor App — Overview

> **Tier 2** | Source: Adam Wiggins / Heroku (2011) | Derives From: ka02-architecture, ka06-operations | Authority: established practice

## Summary

The 12-Factor App methodology is a set of twelve principles for building software-as-a-service applications that are portable, scalable, and maintainable. Originally documented by Adam Wiggins and colleagues at Heroku in 2011, it distills hard-won operational experience from running thousands of production applications. The methodology is language-agnostic but has particular impact on Python and Go backend services.

## Key Concepts

### The Twelve Factors at a Glance

| # | Factor | One-Line Summary |
|---|--------|-----------------|
| I | Codebase | One codebase tracked in version control; many deploys |
| II | Dependencies | Explicitly declare and isolate dependencies |
| III | Config | Store config in environment variables |
| IV | Backing Services | Treat backing services as attached resources |
| V | Build, Release, Run | Strictly separate build and run stages |
| VI | Processes | Execute the app as one or more stateless processes |
| VII | Port Binding | Export services via port binding |
| VIII | Concurrency | Scale out via the process model |
| IX | Disposability | Maximize robustness with fast startup and graceful shutdown |
| X | Dev/Prod Parity | Keep development, staging, and production as similar as possible |
| XI | Logs | Treat logs as event streams |
| XII | Admin Processes | Run admin/management tasks as one-off processes |

### Most Critical Factors for Agents

Four factors have the highest impact on agent-generated services:

- **III — Config**: Agents must never hardcode configuration values. All environment-specific values belong in environment variables. A `.env.template` file must always be committed; the `.env` file itself must never be committed.
- **VI — Processes**: Stateless processes are the prerequisite for horizontal scaling. Agents must not store session data, accumulated state, or user context in process memory.
- **IX — Disposability**: Services must start in under 30 seconds and handle `SIGTERM` gracefully. Agents must implement signal handlers.
- **XI — Logs**: Services must write structured logs to stdout only. Agents must never open log files or manage log rotation.

## Agent Guidance

### Do
- Read `factors.md` in full before designing a new service
- Apply Factor III (Config) on the first day — retrofitting is expensive
- Verify Factor VI (Processes) before any load-balancing or horizontal scaling discussion
- Design for Factor IX (Disposability) from the start: stateless, fast-starting, SIGTERM-aware

### Do Not
- Do not hardcode database hostnames, ports, or credentials — ever
- Do not store session or user state in process memory
- Do not write to log files; log to stdout
- Do not run database migrations inside application startup (use admin processes per Factor XII)

## Checklist
- [ ] All config values come from environment variables
- [ ] No in-process session or accumulated state
- [ ] Logs go to stdout in structured format (JSON preferred)
- [ ] Service handles SIGTERM with graceful shutdown
- [ ] Migrations are separate admin processes, not part of `app.run()`

## See Also
- `wiki/tier2-core/twelve-factor-app/factors.md`
- `wiki/tier1-sources/swebok-v4/ka06-operations.md`
- `wiki/tier2-core/distributed-systems/overview.md`

## Source

Adam Wiggins, *The Twelve-Factor App* (2011), https://12factor.net. Synthesized from *Software Development Best Practices for Agent* reference document.
