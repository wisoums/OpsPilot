# Security and Trust Model

OpsPilot is intended for internal knowledge, so security is part of the product architecture rather than an optional hardening phase.

## Principles

### 1. Private routing before external access

A question must be classified under the administrator's source policy before any external web search call is made.

Internal terminology should not be leaked to a public search provider merely to discover that the query is internal.

### 2. Evidence-backed generation

When evidence is required, the model should answer from retrieved sources rather than unsupported model memory.

If evidence is insufficient, the preferred behavior is an explicit refusal such as:

> I could not find enough information in the configured sources to answer this question.

### 3. Workspace isolation

The following must always be workspace-scoped:

- documents
- chunks/vectors
- retrieval filters
- cache entries
- conversation state
- citations
- telemetry correlation metadata

Cross-workspace cache hits or retrieval results are security defects.

### 4. Permission-aware retrieval

If document-level permissions are introduced, the retrieval query and semantic cache scope must include the user's effective permission set.

A cache hit is only safe when the current user is authorized to see the evidence that produced it.

### 5. Safe telemetry

OpenTelemetry should capture:

- span names
- timing
- cache hit/miss
- model/provider identifiers
- token counts
- retrieval counts/scores
- error codes
- opaque document IDs

It should not capture by default:

- full private prompts
- raw document text
- credentials
- access tokens
- personal or confidential source content

### 6. Least-privilege AWS access

AWS components should have separate IAM roles where practical.

Examples:

- ingestion worker: read uploaded objects, write index/status
- query API: invoke configured Bedrock model, query allowed index/cache
- upload signer: create scoped presigned upload requests
- observability exporter: write only required telemetry

### 7. Cache safety

Semantic cache entries must be invalidated or namespaced when:

- corpus changes
- permissions change materially
- source policy changes
- model/prompt configuration changes

TTL may be used in addition to, not instead of, versioned cache identity.

### 8. Web citation integrity

Web-derived answers must store the source URL/title and retrieval timestamp.

The generated answer should never present a web claim as company policy unless internal evidence supports that interpretation.

## Threats to test

- prompt injection embedded in uploaded documents
- malicious instructions inside web content
- cross-tenant retrieval
- cross-tenant semantic cache hit
- stale cache after policy/document update
- citation pointing to a source not used
- unsupported answer when retrieval is empty
- private term sent to external search
- oversized/malformed upload
- path traversal in local ingestion
- telemetry accidentally logging confidential content

These should eventually become automated tests where feasible.
