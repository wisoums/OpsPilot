# OpsPilot Architecture

## Design goal

OpsPilot should present one product experience while allowing infrastructure to change underneath it.

The core application must depend on provider interfaces rather than AWS-specific or local-specific implementations. This keeps Local Mode useful on its own and prevents AWS Mode from becoming a separate codebase.

## Logical layers

1. **Presentation**
   - document library
   - upload/folder ingestion
   - chat
   - citation viewer
   - admin source policy
   - observability dashboard

2. **Application/API**
   - authentication/session
   - workspace context
   - request orchestration
   - ingestion job orchestration
   - policy enforcement

3. **AI orchestration**
   - source router
   - semantic cache
   - retrieval
   - optional reranking
   - generation
   - citation/provenance assembly
   - refusal/grounding checks

4. **Provider layer**
   - document storage
   - vector index
   - embeddings
   - model inference
   - web search
   - cache
   - telemetry exporters

5. **Infrastructure**
   - local Docker services
   - AWS resources
   - Terraform
   - CI/CD

## Provider contracts

The exact APIs will be designed during implementation, but the core should depend on abstractions conceptually similar to:

```python
DocumentStore
VectorStore
EmbeddingProvider
GenerationProvider
WebSearchProvider
SemanticCache
TelemetrySink
```

A Local Mode implementation and an AWS Mode implementation should satisfy the same contracts.

## Query flow

```text
1. User question
2. Resolve workspace + permissions
3. Apply administrator source policy
4. Classify internal / web / mixed when Smart Routing is enabled
5. Check semantic cache within the same workspace/corpus/permission scope
6. Retrieve evidence from allowed sources
7. Generate answer constrained by evidence
8. Build normalized citations
9. Run grounding/refusal checks
10. Store safe cache entry when eligible
11. Emit trace + metrics
12. Return answer + citations + source-route metadata
```

## Ingestion flow

### Local

```text
filesystem scan
  -> fingerprint file
  -> skip unchanged
  -> parse
  -> chunk
  -> embed
  -> upsert vector index
  -> update corpus version
```

### AWS

```text
browser
  -> presigned S3 uploads
  -> ingestion event
  -> queue
  -> workers
  -> parse/chunk/embed/index
  -> status store
  -> update corpus version
```

Ingestion must be asynchronous for large batches and expose per-document status.

## Source routing

Administrator policy is authoritative.

```text
DOCUMENTS_ONLY
SMART
DOCUMENTS_AND_WEB
WEB_ONLY
```

For SMART:

- internal/business-specific query -> internal retrieval only
- general/public query -> web
- comparative/mixed query -> internal + web
- uncertain/private-looking query -> prefer internal path and avoid external leakage

Routing should be testable independently from answer quality.

## Citation model

Internal and web evidence are normalized to a common provenance model.

Suggested fields:

```text
source_type
workspace_id
source_id
title
uri_or_document_id
page
section
snippet
retrieval_score
retrieved_at
content_hash
```

## Semantic cache

Cache lookup should use semantic similarity, but cache eligibility is stricter than "looks similar."

A cache namespace must include:

```text
workspace_id
corpus_version
permission_scope
source_policy
model_version
prompt/config_version
```

Any corpus update changes the active corpus version so old cached answers cannot be served against new knowledge.

## Observability

Instrument major spans:

```text
question.request
auth
source_router
cache.lookup
embedding
retrieval.internal
retrieval.web
rerank
llm.generate
citation.build
grounding.check
cache.store
```

Capture metadata and timing by default. Do not capture full prompts, retrieved passages, or confidential documents unless an administrator explicitly enables a safe debugging mode.

## Multi-tenancy

Every document, vector, cache entry, query, trace correlation record, and citation must belong to a workspace.

Workspace identity is enforced at the storage/query layer, not only in UI filtering.

## Future agent/action layer

OpsPilot may later support actions such as ticket lookup or escalation creation.

Action tools are intentionally outside the initial MVP. Any mutating action must have authorization checks and, for sensitive actions, human confirmation.
