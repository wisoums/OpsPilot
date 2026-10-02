# Roadmap

OpsPilot is now AWS-native. The roadmap no longer contains a separate local AI implementation.

## Milestone 0 — Foundation

Definition of done:

> The repository has a runnable application skeleton, CI, AWS configuration model, Terraform layout, and a documented development workflow.

Includes:

- repository/application structure
- AWS-native domain boundaries
- configuration/secrets model
- frontend/backend skeleton
- CI/lint/type/test baseline
- Terraform structure
- AWS dev-environment conventions
- contribution/security documentation

## Milestone 1 — AWS Core

Definition of done:

> A developer can provision the foundational AWS stack and reach an authenticated OpsPilot API/UI skeleton.

Includes:

- Terraform state/environment conventions
- S3/CloudFront frontend delivery
- Cognito
- API Gateway
- Lambda API/runtime
- IAM
- logging
- AWS Budgets/cost guardrails
- deploy/destroy workflow

## Milestone 2 — AWS RAG and Bulk Ingestion

Definition of done:

> A user can deploy OpsPilot, upload their own documents, wait for indexing, ask a company-specific question, and receive a cited answer.

Includes:

- direct-to-S3 presigned bulk uploads
- multipart large-file path
- batch/document status
- asynchronous ingestion orchestration
- Bedrock Knowledge Bases
- S3 Vectors
- Bedrock model generation
- chunking/metadata configuration
- grounded answers/refusals
- citation/source viewer
- end-to-end AWS acceptance test

## Milestone 3 — Smart Routing + Web Search

- source policy configuration
- internal/web/mixed router
- Bedrock Web Search
- privacy gate before web access
- normalized provenance
- mixed-source synthesis

## Milestone 4 — Semantic Cache

- cache contract/eligibility
- ElastiCache for Valkey
- semantic query embeddings/search
- corpus-version invalidation
- workspace/permission isolation
- cache correctness tests

## Milestone 5 — Observability

- OpenTelemetry instrumentation
- GenAI/RAG span schema
- CloudWatch/X-Ray export
- AI operations metrics/dashboard
- privacy-safe telemetry controls

## Milestone 6 — Evaluation and AI Regression CI

- synthetic enterprise corpus
- query/workload generator
- retrieval/grounding evaluation
- citation evaluation
- routing/privacy benchmark
- cache benchmark
- bulk-ingestion benchmark
- prompt-injection tests
- security regression tests
- versioned AI baselines
- CI regression gate

## Milestone 7 — Productization

- polished document manager
- ingestion progress
- admin settings
- citation/source explorer
- AWS preflight/developer workflow
- guided deployment/teardown
- security scanning
- real demo GIF/video
- release/versioning
- performance/cost targets
- license decision

## Future — Agent Actions

Only after the knowledge system is reliable:

- read-only business tool calls
- ticket/customer lookup
- proposed actions
- human approval
- authorized write actions

## Project-level acceptance criterion

A stranger with an AWS account and the required permissions can:

1. clone the repository,
2. provision OpsPilot with Terraform,
3. upload their own private documents,
4. wait for indexing,
5. ask internal and permitted web questions,
6. receive source-backed answers,
7. inspect observability data,
8. benchmark semantic caching/evaluation behavior,
9. destroy the deployed resources cleanly.
