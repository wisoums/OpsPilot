# Roadmap

OpsPilot is intentionally scoped as a **production-grade, measurable AWS RAG/LLMOps project**.

The goal is not to build every possible enterprise assistant feature. The goal is to demonstrate that a RAG system can be deployed, observed, evaluated, costed, cached, and regression-gated like production software.

## 1 — Foundation + Terraform

Definition of done:

> The repository has a runnable application skeleton, CI baseline, AWS configuration model, and Terraform deployment layout.

Includes:

- backend/frontend skeleton required for the demo
- configuration and secrets model
- Terraform state/environment conventions
- least-privilege IAM needed by the implemented path
- deploy/destroy workflow
- lint/type/unit-test CI baseline
- AWS budget/cost guardrails

## 2 — AWS RAG over private documents

Definition of done:

> A user can upload the evaluation document corpus, wait for indexing, ask a private-knowledge question, and receive a grounded answer with citations.

Includes:

- direct-to-S3 document upload
- Bedrock Knowledge Bases
- S3 Vectors
- Bedrock generation
- chunking/metadata configuration
- corpus-version tracking
- grounded answers and refusals
- end-to-end RAG acceptance test

## 3 — Private + web source routing

Definition of done:

> OpsPilot routes internal, public, and mixed questions to the allowed evidence sources without leaking private terminology externally.

Includes:

- INTERNAL / WEB / MIXED routing
- Bedrock Web Search
- privacy gate before web retrieval
- normalized provenance
- mixed-source synthesis
- routing/privacy tests

## 4 — Semantic cache

Definition of done:

> Semantically equivalent eligible queries can reuse a grounded cached answer without violating corpus/config/privacy boundaries.

Includes:

- ElastiCache for Valkey
- semantic lookup
- cache eligibility contract
- corpus/config invalidation
- isolation/safety checks
- exact reproducible cache OFF vs ON benchmark

## 5 — OpenTelemetry observability

Definition of done:

> A request can be traced through routing, cache lookup, retrieval, generation, and grounding.

Includes:

- OpenTelemetry instrumentation
- GenAI/RAG span attributes
- CloudWatch/X-Ray export where appropriate
- privacy-safe telemetry
- latency/token/cost inputs needed by benchmark reporting

## 6 — Evaluation harness

Definition of done:

> A fixed, versioned test set produces reproducible retrieval, grounding, routing, citation, latency, and cost results.

Includes:

- synthetic enterprise corpus
- fixed query set
- deterministic retrieval metrics
- Ragas or equivalent semantic grading for faithfulness/relevancy
- routing/privacy evaluation
- semantic-cache correctness/performance evaluation
- machine-readable results

Release targets include:

- faithfulness >= 0.85
- answer relevancy >= 0.80
- private-query external leakage = 0
- stale/unsafe semantic-cache hits = 0

## 7 — AI regression gate in CI

Definition of done:

> Relevant prompt/model/retrieval/routing/cache changes cannot merge when they violate accepted quality, latency, privacy, or cost budgets.

Includes:

- versioned accepted baseline
- GitHub Actions evaluation job
- blocking quality/privacy thresholds
- blocking latency/cost regression budgets
- explicit baseline-update workflow
- proof that an intentionally bad change fails CI

## 8 — Benchmark + technical communication

Definition of done:

> A reviewer can understand the architecture, reproduce the deployment, and see measured evidence of the project's quality/performance/cost behavior.

Includes:

- final cache OFF vs ON benchmark
- p95 latency
- Bedrock cost per 1,000 queries
- quality/retrieval scores
- README results table
- raw machine-readable benchmark artifacts
- one-page technical write-up or short demo video

## Out of scope for the flagship release

These ideas may be revisited after the flagship release, but they are **not dependencies for completion**:

- agent actions and mutating business tools
- LangGraph
- polished admin/product-management surfaces
- large-scale ingestion stress testing
- multipart-upload optimization unless required by the evaluation corpus
- broad release/package automation
- nonessential dashboards

## Project-level acceptance criterion

A stranger with an AWS account and the required permissions can:

1. clone the repository,
2. provision OpsPilot with Terraform,
3. upload the fixed/private evaluation corpus,
4. ask internal and permitted web questions,
5. receive source-backed answers,
6. inspect OpenTelemetry traces,
7. run the evaluation suite,
8. reproduce cache OFF vs ON latency/cost results,
9. observe CI block an intentionally regressive change,
10. destroy deployed resources cleanly.
