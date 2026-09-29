# Roadmap

The roadmap is ordered so OpsPilot remains runnable as capabilities are added.

## Milestone 0 — Foundation

- repository structure
- architecture/provider interfaces
- configuration model
- CI/lint/test baseline
- Docker development environment
- contribution/security documentation

## Milestone 1 — Local MVP

Definition of done:

> A stranger can clone the repo, point OpsPilot at their own local documents, and ask a question that returns a grounded answer with a citation.

Includes:

- local filesystem ingestion
- PDF/DOCX/TXT/Markdown parsing
- chunking
- local embeddings/vector store
- Ollama generation
- grounded RAG
- citation UI
- incremental indexing

## Milestone 2 — AWS MVP

Definition of done:

> A user can deploy OpsPilot into their AWS account and use the same workflow with S3/Bedrock-backed infrastructure.

Includes:

- Terraform
- IAM
- S3
- presigned bulk uploads
- async ingestion
- Bedrock
- cloud vector backend
- API deployment
- CloudWatch basics
- cost guardrails

## Milestone 3 — Smart Routing

- source policy configuration
- internal/web/mixed router
- web-search provider
- privacy gate before web access
- unified citations

## Milestone 4 — Semantic Cache

- semantic cache contract
- local cache
- AWS cache
- corpus-version invalidation
- workspace/permission isolation
- cache correctness tests

## Milestone 5 — Observability

- OpenTelemetry SDK/collector
- GenAI/RAG span model
- runtime metrics
- CloudWatch/X-Ray export
- privacy-safe telemetry controls

## Milestone 6 — Evaluation and Scale

- synthetic enterprise corpus
- workload/query generator
- routing benchmark
- grounding/citation benchmark
- cache benchmark
- ingestion/load benchmark
- reproducible reports

## Milestone 7 — Productization

- polished document manager
- indexing progress
- admin controls
- citation/source explorer
- demo GIF/video
- installation docs
- release packaging
- contributor experience

## Future — Agent Actions

Only after the knowledge system is reliable:

- read-only business tool calls
- ticket/customer lookup
- proposed actions
- human approval
- authorized write actions
