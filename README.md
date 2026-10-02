# OpsPilot

> **An AWS-native AI knowledge assistant for private company knowledge, live web research, and source-backed answers.**

OpsPilot is a publicly developed AI engineering project built around one question:

> What does it take to make an enterprise AI assistant **useful, trustworthy, measurable, and cost-aware in production on AWS**?

The target product lets a team deploy OpsPilot into its own AWS account, upload hundreds or thousands of internal documents, ask natural-language questions, and receive answers grounded in either private company knowledge, the live web, or both — always with provenance.

OpsPilot is intentionally **AWS-native**. It is not maintaining a second local AI stack. Development can happen from a laptop, but the document storage, RAG, model inference, cache, identity, observability, and deployed runtime are AWS-backed.

> 🚧 **Status:** architecture and backlog are defined; implementation is beginning now. The README distinguishes planned behavior from completed behavior.

---

## Product goals

A user should eventually be able to:

1. Clone this repository.
2. Deploy OpsPilot into their AWS account with Terraform.
3. Open the generated application URL.
4. Upload their own private document collection.
5. Ask questions about internal processes and receive cited answers.
6. Ask general/current questions and receive cited web-backed answers when policy allows it.
7. Use semantic caching to avoid redundant model calls.
8. Inspect latency, routing, retrieval, cache behavior, token usage, and estimated cost.
9. Run automated evaluations that can block AI regressions in CI.

Example:

```text
Employee:
"What is our ZQ-17 procurement exception process?"

OpsPilot:
"ZQ-17 exceptions require Amber approval before the request enters Stage Kappa."

Sources:
[Company] Procurement Operations Manual — §7.4
```

The underlying model may know many things from pretraining, but OpsPilot's job is to control **which evidence it is allowed to use** and to refuse unsupported answers where the active policy requires evidence.

---

## AWS-native architecture

```text
                                +----------------------+
                                |      Web Client      |
                                |   React / TypeScript |
                                +----------+-----------+
                                           |
                                      CloudFront
                                           |
                              +------------+------------+
                              |                         |
                              v                         v
                        Amazon Cognito             API Gateway
                                                        |
                                                        v
                                                   AWS Lambda
                                                        |
                          +-----------------------------+-----------------------------+
                          |                             |                             |
                          v                             v                             v
                 Semantic Cache                 Source Router                 Upload / Admin
              ElastiCache / Valkey                    |                             |
                          |                 +----------+----------+                  |
                    HIT --+                 |                     |                  |
                          |                 v                     v                  v
                          |          Company Knowledge       Bedrock Web Search    Amazon S3
                          |                 |                     |                  |
                          |                 v                     |                  v
                          |       Bedrock Knowledge Base          |             ingestion sync
                          |                 |                     |                  |
                          |                 v                     |                  v
                          |             S3 Vectors                |          Bedrock Knowledge Base
                          |                 |                     |            parsing/chunking/
                          +-----------------+----------+----------+            embedding/indexing
                                                       |
                                                       v
                                                Amazon Bedrock
                                                       |
                                                       v
                                              Answer + Citations

       Supporting platform: IAM · KMS · CloudWatch · X-Ray · OpenTelemetry · CloudTrail · AWS Budgets
       Infrastructure: Terraform
```

### Why S3 Vectors?

The initial RAG design uses **Amazon Bedrock Knowledge Bases + Amazon S3 as the document source + S3 Vectors as the vector store**. This keeps the first version highly managed and AWS-native while still letting us study retrieval quality, chunking, scale, cost, and operational behavior.

The vector backend remains an architecture decision we can benchmark later; if the workload requires capabilities that S3 Vectors does not provide, the project should document and justify a migration rather than changing technology silently.

---

## Bulk document ingestion

OpsPilot is meant to be tested with more than five demo PDFs.

For AWS deployments, file bytes should go **directly from the browser to Amazon S3** using scoped presigned uploads rather than flowing through the API Lambda.

```text
Browser
  |
  +--> S3 object 1
  +--> S3 object 2
  +--> S3 object 3
  +--> ...
  +--> S3 object N
          |
          v
   ingestion orchestration
          |
          v
 Bedrock Knowledge Base sync
          |
          v
 parse -> chunk -> embed -> S3 Vectors
```

Planned behavior:

- bounded parallel uploads
- multipart upload for large files
- per-document/batch status
- asynchronous ingestion
- retry/failure reporting
- incremental sync behavior
- benchmarks at 100, 1,000, and stress-scale documents

---

## Smart source routing

The administrator controls where OpsPilot may obtain evidence.

Planned source policies:

- **Company documents only**
- **Smart routing**
- **Company + web**
- **Web only**

Under Smart Routing:

```text
                         USER QUESTION
                               |
                               v
                      +----------------+
                      |  Source Router |
                      +--------+-------+
                               |
               +---------------+---------------+
               |               |               |
               v               v               v
           INTERNAL          GENERAL          MIXED
               |               |               |
               v               v               v
        Bedrock Knowledge   Bedrock Web      Both
             Base             Search
               |               |               |
               +---------------+---------------+
                               |
                               v
                      Evidence + Provenance
                               |
                               v
                       Answer + Citations
```

**Privacy rule:** classification/policy checks happen before external web retrieval. Internal terminology should not be sent to web search merely to discover that it was private.

---

## Semantic cache

OpsPilot will use **Amazon ElastiCache for Valkey** as an optional semantic caching layer.

```text
Question
   |
   v
Create query embedding
   |
   v
Valkey semantic lookup ---- HIT ----> Cached grounded answer
   |
  MISS
   v
RAG / Web retrieval -> Bedrock -> Citations -> Cache
```

The objective is measurable, not theoretical. The evaluation suite will report:

- exact and semantic cache hit rate
- LLM calls avoided
- token reduction
- estimated inference-cost reduction
- average / p95 latency reduction
- false-hit rate
- stale-hit rate

A cache result is only eligible when its workspace, permissions, corpus version, source policy, model/prompt configuration, and other safety-relevant context match.

---

## Grounding and citations

Internal answers should cite internal evidence:

```text
[Company] incident-response.pdf — page 17
```

Web answers should cite web evidence:

```text
[Web] AWS Documentation — retrieved <timestamp>
```

Mixed answers must keep the two kinds of evidence distinct.

The normalized provenance model is expected to preserve:

- source type
- source/document ID or URL
- title
- page/section where available
- supporting passage
- retrieval score where applicable
- retrieval timestamp
- content/version metadata

If the configured evidence does not support an answer, OpsPilot should refuse or clearly state that sufficient information was not found.

---

## Observability with OpenTelemetry

OpenTelemetry is part of the architecture from the beginning.

A request trace should eventually look like:

```text
question.request
|
+-- auth
+-- source_router
+-- semantic_cache.lookup
|   +-- HIT / MISS
+-- embedding
+-- retrieval.internal
+-- retrieval.web
+-- llm.generate
+-- citation.build
+-- grounding.check
+-- cache.store
```

Planned metrics include:

- end-to-end latency and p95/p99
- retrieval latency
- Bedrock latency
- cache hit/miss rate
- LLM calls avoided
- retrieved-document counts/scores
- token usage
- estimated request cost
- citation coverage/correctness
- unsupported/refusal rate
- ingestion throughput/failures

AWS-mode traces and metrics will flow into CloudWatch/X-Ray where appropriate. Private prompt/document contents are not logged by default.

---

## AI regression gate

OpsPilot will treat prompt/model/RAG configuration changes like production software changes.

```text
Prompt / model / retrieval / router / cache change
                     |
                     v
                 GitHub PR
                     |
                     v
           Golden evaluation set
                     |
                     v
          Run evaluation pipeline
                     |
                     v
           Compare accepted baseline
                     |
              +------+------+
              |             |
          regression      acceptable
              |             |
              v             v
           FAIL CI       PASS CI
```

Regression dimensions include:

- retrieval quality
- grounding
- citation correctness
- routing accuracy
- private-query leakage
- cache false/stale hits
- latency
- token usage
- estimated cost

Accepted baselines are versioned and cannot be silently rewritten by a failing CI run.

---

## Synthetic enterprise evaluation corpus

The project will generate fictional internal knowledge that the base model cannot be expected to know:

```text
"The NOVA-7 procedure requires Level Amber approval before a
Helios supplier can enter Stage Kappa."
```

This gives us controlled ground truth for:

- retrieval
- grounding
- routing
- citations
- cache correctness
- repeated/paraphrased workloads
- unanswerable questions
- mixed internal/web questions
- regression testing

---

## Trust and security principles

1. **Evidence before answers** — evidence-required modes do not rely on unsupported model memory.
2. **Private routing first** — policy/classification occurs before external web retrieval.
3. **Workspace isolation** — documents, retrieval, citations, cache entries, and telemetry correlation are tenant-scoped.
4. **Least-privilege IAM** — each runtime component gets only required AWS actions/resources.
5. **Safe caching** — corpus/policy/permission/config changes make unsafe old entries ineligible.
6. **No sensitive telemetry by default** — metadata and timings, not confidential content.
7. **Human control for future actions** — mutating agent tools require explicit authorization/approval.
8. **Reproducible infrastructure** — supported AWS resources are created through Terraform.

See [docs/SECURITY.md](docs/SECURITY.md).

---

## Planned AWS services

| Capability | AWS service |
|---|---|
| Document storage | Amazon S3 |
| Static web delivery | Amazon S3 + CloudFront |
| Authentication | Amazon Cognito |
| API | Amazon API Gateway |
| Compute/orchestration | AWS Lambda |
| Async ingestion | Amazon SQS / Lambda where useful |
| Foundation models | Amazon Bedrock |
| Enterprise RAG | Amazon Bedrock Knowledge Bases |
| Vector store | Amazon S3 Vectors |
| Public web evidence | Amazon Bedrock Web Search |
| Semantic cache | Amazon ElastiCache for Valkey |
| Metrics/logs | Amazon CloudWatch |
| Distributed tracing | OpenTelemetry + AWS X-Ray |
| Audit trail | AWS CloudTrail |
| Permissions | AWS IAM |
| Encryption | AWS KMS / service-managed encryption as appropriate |
| Cost guardrails | AWS Budgets |
| Infrastructure as Code | Terraform |

Not every optional service will be enabled in the cheapest deployment profile.

---

## Roadmap

1. **Foundation**
2. **AWS Core**
3. **AWS RAG & Bulk Ingestion**
4. **Smart Routing + Bedrock Web Search**
5. **Semantic Cache**
6. **OpenTelemetry + AWS Observability**
7. **Evaluation + AI Regression CI**
8. **Productization**
9. **Future agent actions**

See [docs/ROADMAP.md](docs/ROADMAP.md) and [the master roadmap issue](https://github.com/wisoums/OpsPilot/issues/1).

---

## Planned project structure

```text
OpsPilot/
├── apps/
│   ├── web/                  # React/TypeScript frontend
│   └── api/                  # Python API/Lambda handlers
├── packages/
│   ├── core/                 # domain/orchestration logic
│   ├── ingestion/            # upload/sync/status orchestration
│   ├── retrieval/            # RAG + routing
│   ├── caching/              # semantic cache
│   ├── telemetry/            # OpenTelemetry instrumentation
│   └── evaluation/           # eval + regression harness
├── infrastructure/
│   └── terraform/            # AWS infrastructure
├── scripts/
│   ├── generate_synthetic_corpus.py
│   └── generate_query_workload.py
├── docs/
└── tests/
```

The exact structure can evolve through reviewed architecture decisions.

---

## Development model

OpsPilot is AWS-native, but developers do not need to redeploy the entire frontend on every code change.

The intended developer loop is:

```text
local frontend/API development
          |
          v
   dedicated AWS dev stack
   S3 / Bedrock / KB / vectors / etc.
          |
          v
   Terraform-managed deployment
```

There is **no separate Ollama/local-vector-store product mode** to keep in sync.

---

## Cost philosophy

AWS Mode is not promised to be free.

The project will:

- document billable resources
- provide minimal/optional feature flags where practical
- make semantic cache/web search/advanced observability configurable
- create AWS Budget guardrails
- document teardown
- benchmark cost instead of copying vendor marketing numbers

Any resume/README claim such as "reduced inference cost by X%" must come from OpsPilot's own reproducible benchmark.

---

## Documentation

| Document | Purpose |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | AWS components and request/ingestion flows |
| [Security & Trust](docs/SECURITY.md) | IAM, privacy, isolation, routing, cache and telemetry rules |
| [Evaluation](docs/EVALUATION.md) | Grounding, routing, cache, cost and regression benchmarks |
| [Roadmap](docs/ROADMAP.md) | AWS-first implementation order |
| [Contributing](CONTRIBUTING.md) | Development and PR workflow |

---

## Why this project exists

Organizations increasingly have two opposite problems:

- too much private operational knowledge to search manually;
- AI systems that are fast to demo but difficult to trust, measure, secure, and operate.

OpsPilot explores the engineering between those two problems — on AWS.
