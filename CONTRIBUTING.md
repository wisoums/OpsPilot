# Contributing to OpsPilot

OpsPilot is an AWS-native production-style AI engineering project. Contributions should keep changes testable, reviewable, observable, secure, and cost-aware.

## Workflow

1. Pick or create an issue.
2. Create a focused branch.
3. Keep the PR scoped to the issue.
4. Add/update tests.
5. Update documentation for architecture/configuration changes.
6. Include benchmark evidence for performance/cost-sensitive changes.
7. Do not silently change accepted AI regression baselines.

## Branch names

Examples:

```text
feature/s3-bulk-upload
feature/bedrock-kb
feature/source-router
feature/semantic-cache
fix/cache-isolation
docs/security-model
```

## Pull requests

A PR should explain:

- what changed
- why
- how it was tested
- AWS resources/services affected
- security/privacy implications
- cost implications
- performance implications
- screenshots/traces/benchmark output where useful

## Engineering rules

- OpsPilot's supported product path is AWS-native.
- Localhost development is allowed; a parallel local AI product stack is not a requirement.
- Infrastructure required by the supported deployment belongs in Terraform.
- Do not log private document/prompt content by default.
- Retrieval and caching must be workspace-scoped.
- External web retrieval must respect source policy/privacy routing.
- New AI behavior needs an evaluation story, not only manual examples.
- Resume/README performance or cost claims must come from reproducible project benchmarks.
- Avoid adding AWS services only to increase the service count; every service must have a justified responsibility.

## AWS development safety

- Use a dedicated dev environment/account where possible.
- Never commit AWS credentials.
- Prefer least-privilege roles.
- Tag resources consistently.
- Know which resources continue billing while idle.
- Run teardown when an experiment no longer needs persistent resources.

## Development setup

Concrete commands will land with the foundation/AWS-core issues.

Until then, see:

- [Architecture](docs/ARCHITECTURE.md)
- [Security](docs/SECURITY.md)
- [Evaluation](docs/EVALUATION.md)
- [Roadmap](docs/ROADMAP.md)
