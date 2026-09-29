# Contributing to OpsPilot

OpsPilot is being built as a production-style AI system, so contributions should keep changes testable, reviewable, and observable.

## Workflow

1. Pick or create an issue.
2. Create a focused branch.
3. Keep the PR scoped to the issue.
4. Add or update tests for behavior changes.
5. Update documentation when architecture/configuration changes.
6. Include benchmark evidence when changing performance-sensitive code.

## Branch names

Examples:

```text
feature/local-ingestion
feature/source-router
fix/cache-isolation
docs/security-model
```

## Pull requests

A PR should explain:

- what changed
- why
- how it was tested
- security/privacy implications
- performance implications when relevant
- screenshots/traces/benchmark output when useful

## Engineering rules

- Do not hard-code provider-specific behavior into core logic.
- Do not log private document content by default.
- Retrieval and caching must always be workspace-scoped.
- New AI behavior should have an evaluation story, not only manual examples.
- Keep local mode usable without AWS.
- Keep AWS deployment reproducible through Infrastructure as Code.

## Development setup

Concrete setup commands will be added as the first runnable scaffold lands.

Until then, see:

- [Architecture](docs/ARCHITECTURE.md)
- [Security](docs/SECURITY.md)
- [Evaluation](docs/EVALUATION.md)
- [Roadmap](docs/ROADMAP.md)
