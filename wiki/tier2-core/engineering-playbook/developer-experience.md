# Developer Experience (DevEx) (Tier 2)

> **Tier 2** | Source: Microsoft ISE Engineering Fundamentals Playbook | Authority: established | Derives From: SWEBOK KA6, KA4

## Summary

Developer Experience (DevEx) measures how readily engineers can execute the four essential development tasks: build, test, start, and debug. Poor DevEx is a compounding tax on every engineer's productivity — every minute spent on setup, environment issues, or configuration is a minute not spent delivering value.

The playbook frames DevEx not as a comfort feature but as a team reliability concern. A slow F5 Contract degrades onboarding, increases context-switch cost, and erodes the feedback loop that makes iterative development effective. For a coding agent, DevEx standards define the expectations any contribution must meet: changes that break the build script, slow down the test suite, or require undocumented manual setup steps are DoD violations.

## Key Concepts

### The Four Essential Tasks

Every developer on every project must be able to execute these four operations with a single command or keystroke:

| Task | Definition | Default Shortcut |
|------|-----------|-----------------|
| **Build** | Compile source, verify syntax, generate artifacts | `Ctrl+Shift+B` (VS Code) |
| **Test** | Run the full test suite and report results | `Ctrl+Shift+T` or `npm test` / `pytest` / `go test ./...` |
| **Start** | Launch the full system end-to-end | `F5` (VS Code debugger) |
| **Debug** | Attach a debugger and inspect variables in a running system | `F5` with breakpoints |

Any solution component (microservice, library, CLI tool) that cannot be built, tested, started, or debugged with a documented single command has a DevEx deficit.

### DevEx Metrics

Two quantitative measures track DevEx health:

#### F5 Contract (Time to First E2E Result)

> "Measures the setup duration for an untouched machine to run the complete system end-to-end."

The F5 Contract is the time from `git clone` on a blank machine to having the full application running locally. Teams should measure and track this.

| F5 Contract Duration | Assessment |
|---------------------|------------|
| < 15 minutes | Excellent |
| 15–60 minutes | Acceptable |
| 1–4 hours | Needs improvement |
| > 4 hours | Critical DevEx debt |

#### Time to First Commit

> "Tracks how quickly developers can make locally-verified changes."

Time from `git clone` to a passing local build with a meaningful code change committed. Target: same day as onboarding, ideally within the first hour.

### Organizational Roles

DevEx is a shared responsibility with defined ownership:

| Role | Responsibility |
|------|---------------|
| **Dev Lead** | Defines development environments, source control structure, and baseline DevEx expectations; owns the F5 Contract |
| **DevEx Champion** | Identifies improvement opportunities, manages DevEx backlog items, serves as the subject-matter expert on tooling |
| **Team Members** | Uphold DevEx expectations during code reviews — reject PRs that worsen build times, break scripts, or remove automation |
| **New Team Members** | Document gaps discovered during onboarding; their friction is a signal, not a personal problem |

### Implementation Strategies

| Strategy | Detail |
|----------|--------|
| **Keyboard shortcut standards** | F5 = debug/start; Ctrl+Shift+B = build; Ctrl+Shift+T = test. Configure these in all IDE config files committed to the repo |
| **Comprehensive onboarding docs** | `CONTRIBUTING.md` or `docs/onboarding.md` must document every prerequisite and every step to reach a passing F5 |
| **Standardized task execution** | Every solution component uses the same build/test/start commands — no per-service snowflakes |
| **Minimize repository count** | Each additional repository multiplies setup friction; prefer monorepos or clear multi-repo tooling over sprawl |
| **Solution-level automation** | `Makefile`, `taskfile.yaml`, or `scripts/` that wrap all essential tasks at the project root |
| **Local mock implementations** | Use dependency injection + configuration toggles to substitute real cloud services with local mocks during development |
| **Emulators for cloud services** | Use official emulators (Azurite, DynamoDB Local, Pub/Sub emulator) so developers don't need cloud credentials to run locally |

### Dependency Injection for Local Development

The playbook specifically endorses dependency injection as the mechanism for swapping local mock implementations:

```python
# Interface defined once
class MessageQueue(Protocol):
    def publish(self, topic: str, message: dict) -> None: ...

# Production implementation (real cloud)
class PubSubQueue:
    def publish(self, topic: str, message: dict) -> None:
        ...  # Google Pub/Sub SDK call

# Local development implementation
class InMemoryQueue:
    def __init__(self) -> None:
        self._messages: list[dict] = []

    def publish(self, topic: str, message: dict) -> None:
        self._messages.append({"topic": topic, **message})

# Wiring via config toggle
def build_queue(config: Config) -> MessageQueue:
    if config.use_local_mock:
        return InMemoryQueue()
    return PubSubQueue(config.project_id)
```

This pattern ensures the F5 Contract works without cloud credentials and without running actual cloud services locally. See `wiki/tier3-working/worked-examples/dependency-injection.md` for a full worked example.

## Agent Guidance

### Do

- Ensure every change that affects build, test, start, or debug scripts is tested end-to-end before submitting.
- Document new prerequisites in `CONTRIBUTING.md` or onboarding docs as part of the PR — this is a DoD item.
- Use dependency injection to isolate external dependencies, enabling local mock substitution.
- Prefer adding to a `Makefile` / `taskfile.yaml` at the project root over instructions in prose.
- Flag DevEx regressions (longer build times, broken scripts, new undocumented setup steps) during code review.

### Do Not

- Do not introduce dependencies that require manual cloud-account setup to run tests or start the application locally.
- Do not commit IDE configuration files that hardcode personal machine paths.
- Do not add solution components that cannot be built and tested with the same single-command pattern as the rest of the project.
- Do not leave new environment variables undocumented — add them to `.env.template`.

## Checklist

- [ ] `make build` / `make test` / `make start` (or equivalent) all pass on a clean clone
- [ ] New prerequisites documented in onboarding docs
- [ ] New environment variables added to `.env.template`
- [ ] External service dependencies have local mock or emulator paths
- [ ] IDE configuration files (`.vscode/tasks.json`, `launch.json`) committed and functional

## See Also

- wiki/tier2-core/engineering-playbook/overview.md
- wiki/tier2-core/engineering-playbook/documentation-practices.md
- wiki/tier1-sources/swebok-v4/ka06-operations.md
- wiki/tier1-sources/swebok-v4/ka04-construction.md
- wiki/tier3-working/worked-examples/dependency-injection.md
- wiki/tier3-working/checklists/pre-commit.md
- wiki/tier2-core/twelve-factor-app/factors.md

## Source

Microsoft ISE Engineering Fundamentals Playbook — Developer Experience.
https://microsoft.github.io/code-with-engineering-playbook/developer-experience/
