# OpsPilot

> **Private knowledge when it matters. Web knowledge when it helps. Always cited.**

OpsPilot is an early-stage open-source AI knowledge assistant for teams. It is designed to ingest an organization's own documents, answer questions with source-backed evidence, intelligently route questions between private knowledge and the public web, cache repeated questions, and expose the full retrieval/inference path through observability.

The goal is not to build another chatbot wrapper. OpsPilot is being built as a **trustworthy, deployable AI system** that people can clone, run with their own documents, benchmark, and inspect.

> 🚧 **Status:** active development. The architecture and implementation backlog are public, and features will land incrementally behind runnable milestones.

---

## What OpsPilot is building

A user should eventually be able to:

1. Clone OpsPilot.
2. Start it locally with Docker or deploy it into their own AWS account.
3. Add a folder or upload hundreds/thousands of internal documents.
4. Ask natural-language questions.
5. Get an answer backed by citations.
6. See whether the answer came from internal documents, the public web, or both.
7. Inspect latency, cache behavior, retrieval quality, token usage, and cost.

Example:

```text
Employee:
"What is our ZQ-17 procurement exception process?"

OpsPilot:
"ZQ-17 exceptions require Amber approval before the request enters Stage Kappa."

Sources:
[Company] Procurement Operations Manual — §7.4
```

If the answer is not supported by the configured sources, OpsPilot should say so instead of inventing one.

---

## Two deployment modes

OpsPilot is designed around the same product experience with interchangeable infrastructure backends.

| | **Local Mode** | **AWS Mode** |
|---|---|---|
| Best for | Trying OpsPilot, privacy, contributors | Cloud deployment, scale, AWS learning |
| Documents | Local filesystem | Amazon S3 |
| Vector search | Local vector database | AWS-managed/vector backend |
| Model | Local model via Ollama | Amazon Bedrock |
| App runtime | Docker Compose | Serverless/container AWS services |
| Observability | OpenTelemetry locally | OpenTelemetry + CloudWatch/X-Ray |
| Cost | Uses your own machine | AWS usage charges may apply |

The application should not require a different UI or workflow just because the infrastructure changes.

---

## Smart source routing

OpsPilot will support administrator-controlled knowledge policies.

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
           BUSINESS          KNOWLEDGE        QUESTION
               |               |               |
               v               v               v
          Company RAG       Web Search        Both
               |               |               |
               +---------------+---------------+
                               |
                               v
                    Evidence + Provenance
                               |
                               v
                         Answer + Citations
```

Planned administrator modes:

- **Company documents only** — never use the public web.
- **Smart routing** — decide between internal, web, or mixed evidence.
- **Company + web** — search both when useful.
- **Web only** — ignore the private corpus.

**Important:** internal/private queries must be classified *before* any external web request so private terminology is not leaked to a search provider.

---

## Grounded answers and citations

Every factual answer should preserve provenance.

For internal evidence:

```text
[Company] incident-response.pdf — page 17
```

For external evidence:

```text
[Web] AWS Documentation — retrieved 2026-09-29
```

For mixed answers, both should be shown separately.

Each citation record is expected to retain metadata such as:

- source type
- document ID or URL
- title
- page/section
- retrieved passage
- retrieval score
- retrieval timestamp

This enables citation inspection and later evaluation of grounding and source quality.

---

## Bulk ingestion

OpsPilot is intended to work with more than a handful of demo files.

### Local mode

Users should be able to point OpsPilot at a directory containing hundreds or thousands of files. Ingestion will be incremental:

```text
files -> parse -> chunk -> embed -> index
```

Unchanged files should not be reprocessed unnecessarily.

### AWS mode

Browser uploads should go directly to S3 with presigned URLs rather than proxying every file through the application server. Indexing should happen asynchronously so large batches do not block the UI.

```text
Browser -> S3 -> Queue -> Ingestion Workers -> Parse -> Embed -> Index
```

The UI should show indexing progress and per-document status.

---

## Semantic cache

Repeated company questions are common. OpsPilot will include a semantic cache so paraphrases can reuse validated answers when safe.

```text
Question
   |
   v
Semantic cache ---- HIT ----> Cached grounded answer
   |
  MISS
   v
Retrieval -> LLM -> Citations -> Cache
```

The cache must not return stale or cross-tenant information. Cache identity will include at minimum:

- workspace/tenant
- corpus version
- permission scope
- model/configuration version

When the document corpus changes, old entries must no longer be eligible.

---

## Observability with OpenTelemetry

OpenTelemetry is a first-class part of the architecture, not an afterthought.

A request trace should eventually make the system inspectable at this level:

```text
question.request
|
+-- auth
+-- source_router
+-- semantic_cache.lookup
|   +-- HIT / MISS
+-- embedding
+-- retrieval
+-- reranking
+-- llm.generate
+-- citation.build
+-- cache.store
```

Planned metrics include:

- end-to-end latency and p95
- retrieval latency
- LLM latency
- embedding latency
- cache hit/miss rate
- LLM calls avoided
- retrieved document count/scores
- input/output tokens
- estimated request cost
- citation coverage
- answer refusal/unsupported rate
- ingestion throughput and failures

Telemetry must avoid recording sensitive document contents by default.

---

## Synthetic enterprise test corpus

The repo will include tools to generate fictional internal documentation that the underlying model cannot know from pretraining.

Example:

```text
"The NOVA-7 procedure requires Level Amber approval before a
Helios supplier can enter Stage Kappa."
```

This lets us test whether OpsPilot actually retrieves company information instead of relying on model memory.

The generator will support large corpora and workloads with:

- repeated questions
- paraphrased repeats
- uncommon questions
- deliberately unanswerable questions
- internal-only questions
- web-only questions
- mixed internal/web questions

---

## Trust and security principles

OpsPilot is being designed around the following rules:

1. **Evidence before answers** — unsupported claims should be refused or clearly marked.
2. **Private routing first** — private text is not sent to external search accidentally.
3. **Tenant isolation** — retrieval and cache results cannot cross workspaces.
4. **Least privilege** — AWS permissions are scoped to what each component needs.
5. **Human control** — future write/action tools require explicit authorization.
6. **Observable by default** — important AI-system behavior is measurable.
7. **No sensitive prompt logging by default** — telemetry captures metadata, not confidential content.

See [docs/SECURITY.md](docs/SECURITY.md).

---

## Architecture

```text
                                  +------------------+
                                  |    Web Client    |
                                  +--------+---------+
                                           |
                                           v
                                  +------------------+
                                  |   API / Backend  |
                                  +--------+---------+
                                           |
                                  +--------+---------+
                                  |  Policy / Router |
                                  +--------+---------+
                                           |
                      +--------------------+--------------------+
                      |                    |                    |
                      v                    v                    v
              +---------------+    +---------------+    +---------------+
              | Semantic Cache|    | Company RAG   |    |  Web Search   |
              +-------+-------+    +-------+-------+    +-------+-------+
                      |                    |                    |
                      +--------------------+--------------------+
                                           |
                                           v
                                  +------------------+
                                  | Model / Bedrock  |
                                  +--------+---------+
                                           |
                                           v
                                  +------------------+
                                  | Citations + Eval |
                                  +------------------+

         Local providers                                  AWS providers
  ---------------------------                     ---------------------------
  filesystem / local vector DB                     S3 / managed vector store
  Ollama                                             Amazon Bedrock
  local cache                                        ElastiCache / Valkey
  OTel collector                                     OTel + CloudWatch/X-Ray
```

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the detailed design.

---

## Engineering goals

OpsPilot is intentionally being built to demonstrate more than prompt engineering:

- production-style RAG
- intelligent source routing
- privacy-aware web access
- vector retrieval and embeddings
- semantic caching
- cache invalidation
- multi-tenant isolation
- asynchronous document ingestion
- OpenTelemetry tracing
- AI-system evaluation
- performance and cost benchmarking
- AWS serverless/cloud architecture
- Infrastructure as Code
- CI/testing
- product-quality documentation

---

## Roadmap

Development is organized into runnable milestones:

1. **Foundation** — architecture, interfaces, repository structure, CI.
2. **Local MVP** — ingest local documents and answer with citations.
3. **AWS MVP** — deploy the same experience using AWS infrastructure.
4. **Smart Routing** — internal/web/mixed source selection.
5. **Semantic Cache** — safe semantic reuse and invalidation.
6. **Observability** — OpenTelemetry traces, metrics, dashboards.
7. **Evaluation & Scale** — synthetic corpus, routing/grounding/load benchmarks.
8. **Productization** — polished UI, docs, demo assets, releases.

See [docs/ROADMAP.md](docs/ROADMAP.md) and the repository's GitHub Issues for the implementation backlog.

---

## Planned project structure

```text
OpsPilot/
├── apps/
│   ├── web/                  # frontend
│   └── api/                  # backend API
├── packages/
│   ├── core/                 # provider-independent domain logic
│   ├── ingestion/            # parsing/chunking/indexing
│   ├── retrieval/            # RAG + source routing
│   ├── caching/              # semantic cache
│   ├── telemetry/            # OpenTelemetry instrumentation
│   └── evaluation/           # eval + benchmark harness
├── providers/
│   ├── local/                # local filesystem/vector/model/cache
│   └── aws/                  # S3/Bedrock/queue/cache integrations
├── infrastructure/
│   └── terraform/
├── scripts/
│   ├── generate_synthetic_corpus.py
│   └── generate_query_workload.py
├── docs/
└── tests/
```

The exact structure may evolve as implementation begins.

---

## Documentation

| Document | Purpose |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | Components, provider boundaries, request/ingestion flows |
| [Security & Trust](docs/SECURITY.md) | Privacy, isolation, routing, telemetry, IAM principles |
| [Evaluation](docs/EVALUATION.md) | Grounding, routing, caching, load, and citation benchmarks |
| [Roadmap](docs/ROADMAP.md) | Milestones and delivery sequence |
| [Contributing](CONTRIBUTING.md) | Development and contribution workflow |

---

## Contributing

OpsPilot is being developed in public. Issues are designed to be small enough to review independently while still building toward runnable milestones.

See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## Why this project exists

Organizations increasingly have two opposite problems:

- too much internal knowledge to search manually;
- AI assistants that answer confidently without enough evidence.

OpsPilot explores what it takes to build an assistant that is **useful, fast, inspectable, and careful about where its answers come from**.
