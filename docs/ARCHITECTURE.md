# OpsPilot Architecture

## Architecture decision

OpsPilot is **AWS-native**.

There is no separate local model/vector/cache product stack. Developers may run the web/API code locally during development, but supported data, AI, cache, identity, observability, and deployed runtime paths are AWS-backed.

The reason is deliberate: OpsPilot is intended to go deeper on cloud AI engineering rather than maintain duplicate Ollama/local-vector/AWS implementations.

## Primary components

### Presentation

- React/TypeScript web client
- document manager
- chat
- citation/source inspector
- administrator source-policy settings
- observability/benchmark views

Target deployment: S3 + CloudFront.

### Identity

Amazon Cognito provides authentication.

Every request reaching private company knowledge must resolve a server-trusted user/workspace identity.

### API/runtime

Amazon API Gateway fronts AWS Lambda-based API/orchestration functions unless implementation measurements justify a different AWS runtime later.

Responsibilities include:

- authentication/authorization context
- upload signing
- workspace/source policy
- query orchestration
- semantic cache lookup/store
- source routing
- Bedrock invocation
- citation assembly
- telemetry

### Document storage

Private source documents live in Amazon S3.

Clients upload directly to S3 with short-lived, scoped presigned requests. The API should not proxy hundreds of document bodies through Lambda.

### Enterprise RAG

Initial design:

```text
Amazon S3 documents
        |
        v
Amazon Bedrock Knowledge Base
        |
        +-- parsing/chunking
        +-- embeddings
        |
        v
Amazon S3 Vectors
```

Bedrock Knowledge Bases is responsible for the managed RAG ingestion/retrieval path in the first implementation.

S3 Vectors is the initial vector-store choice. If benchmarks show that throughput, filtering, hybrid-search, or latency requirements require another AWS vector backend, the change must be documented as an architecture decision.

### Public web evidence

Amazon Bedrock Web Search is the initial web-retrieval path.

It may only be invoked when the administrator's source policy and the privacy/source router permit it.

### Generation

Amazon Bedrock is the supported foundation-model layer.

Model IDs remain configuration, not hard-coded business logic.

### Semantic cache

Amazon ElastiCache for Valkey is the planned semantic cache.

Cache correctness is more important than cache hit rate. Cache entries are scoped by security/relevance context such as:

```text
workspace_id
corpus_version
permission_scope
source_policy
model_id/config_version
prompt_version
cache_schema_version
```

### Observability

Use OpenTelemetry instrumentation throughout the query path.

AWS export targets include CloudWatch/X-Ray where appropriate.

### Infrastructure

Terraform owns supported AWS infrastructure.

Manual console setup may be used for exploration during development, but the documented/reproducible product deployment must not depend on undocumented console clicks.

---

## Query flow

```text
1. User submits question.
2. API resolves authenticated user + workspace.
3. Load administrator source policy.
4. Run privacy/source routing before external web access.
5. Create cache lookup context and query embedding.
6. Check workspace/version/permission-safe semantic cache.
7. If cache miss:
      a. INTERNAL -> Bedrock Knowledge Base retrieval
      b. WEB      -> Bedrock Web Search
      c. MIXED    -> both
8. Generate answer with Bedrock using allowed evidence.
9. Normalize citations/provenance.
10. Run grounding/refusal checks.
11. Store cache entry only when eligible.
12. Emit traces/metrics.
13. Return answer + citations + route/cache metadata.
```

## Bulk upload and ingestion flow

```text
Browser
  |
  | request upload batch
  v
API/Lambda
  |
  | returns scoped presigned upload requests
  v
Browser ==========================> Amazon S3
  |                                   |
  | concurrent uploads                |
  +-----------------------------------+
                                      |
                                batch completion
                                      |
                                      v
                             ingestion orchestration
                                      |
                                      v
                         Bedrock Knowledge Base sync
                                      |
                         parse / chunk / embed / index
                                      |
                                      v
                                  S3 Vectors
```

The implementation may use SQS/Lambda or another AWS-native orchestration mechanism where it improves reliability and batch-status handling.

Do not start one expensive/duplicative ingestion workflow per file without measuring/justifying that design.

## Document status model

At minimum:

```text
UPLOADING
UPLOADED
QUEUED
INDEXING
INDEXED
FAILED
DELETING
DELETED
```

Batch status should derive from document statuses.

## Source policies

Administrator policy is authoritative:

```text
DOCUMENTS_ONLY
SMART
DOCUMENTS_AND_WEB
WEB_ONLY
```

SMART routes:

- company-specific/process-specific -> internal
- general/current public knowledge -> web
- comparative question -> mixed
- uncertain/private-looking -> safest allowed route; do not leak text externally by default

## Citation/provenance model

Internal and web evidence normalize into a common model while preserving origin.

Suggested fields:

```text
source_type
workspace_id
source_id
title
document_id_or_url
page
section
snippet
retrieval_score
retrieved_at
content_hash_or_version
```

## Semantic cache flow

```text
question
   |
   v
embedding
   |
   v
Valkey similarity search
   |
+--+-----------------------+
|                          |
HIT                        MISS
|                          |
validate context            v
|                     source retrieval
|                          |
|                     Bedrock generation
|                          |
|                     citations/grounding
|                          |
+-------------> response <-+
                           |
                      safe cache store
```

Corpus/document changes must make entries from the previous corpus version ineligible even if their text similarity is high.

## OpenTelemetry span model

Target spans:

```text
question.request
auth.resolve
source_policy.load
source_router.classify
cache.embed
cache.lookup
retrieval.internal
retrieval.web
llm.generate
citation.build
grounding.check
cache.store
```

Useful attributes include model identifiers, route, cache result, token counts, retrieval counts/scores, durations, status codes, and opaque source IDs.

Do not record private prompts or document text by default.

## Multi-tenancy

Workspace identity must be enforced in:

- S3 key layout/access
- knowledge-base metadata/filtering strategy
- query authorization
- cache namespace
- citation access
- conversation state
- telemetry correlation

UI filtering alone is never a security boundary.

## Cost-aware architecture

Optional higher-cost components should be feature-gated where feasible.

Examples:

- semantic cache
- web search
- detailed observability retention
- higher-capacity cache configurations

Terraform outputs/documentation must make teardown easy.

## Future action/agent layer

Only after the knowledge system is reliable:

- read-only tool calls
- ticket/customer lookup
- proposed actions
- human confirmation
- authorized write actions

Mutating actions must be independently authorized at execution time.
