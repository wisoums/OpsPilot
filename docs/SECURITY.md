# Security and Trust Model

OpsPilot is an AWS-native assistant for private enterprise knowledge. Security is therefore part of the architecture, not a later hardening exercise.

## 1. Authenticated workspace identity

Amazon Cognito is the initial authentication layer.

The backend must derive workspace/user identity from trusted authentication context. A client-supplied `workspace_id` is not sufficient authorization.

## 2. Private routing before web access

Source policy and privacy routing occur before Amazon Bedrock Web Search is allowed to receive the query.

Internal terminology should not be sent externally merely to determine that it was internal.

For uncertain/private-looking prompts, prefer the safer route unless the administrator has explicitly chosen a policy that permits broader web use.

## 3. Evidence-backed generation

In evidence-required modes, answers should be produced from allowed retrieved evidence.

If evidence is insufficient, the preferred behavior is explicit refusal/uncertainty rather than unsupported model-memory output.

## 4. S3 isolation and upload safety

Document uploads must use scoped, short-lived presigned requests.

Controls should cover:

- workspace-specific object prefixes
- allowed content types/extensions
- maximum file size
- upload expiration
- server-side validation after upload
- encryption
- public-access blocking

No private document bucket should be public.

## 5. Least-privilege IAM

Use separate/scoped roles where practical.

Examples:

- upload signer: permission to sign/authorize only intended upload locations
- ingestion orchestration: start/inspect allowed knowledge-base ingestion operations
- query runtime: retrieve from the configured KB, invoke allowed Bedrock models/tools, use allowed cache
- telemetry exporter: write only required logs/metrics/traces

Wildcard actions/resources require documented justification.

## 6. Workspace isolation

These are always workspace-scoped:

- S3 documents
- retrieval metadata/filters
- cache entries
- conversation state
- citations
- evaluation fixtures when tenant-specific
- telemetry correlation metadata

Cross-workspace retrieval/cache behavior is a security defect.

## 7. Permission-aware retrieval

If document-level permissions are introduced, effective permissions must participate in both retrieval authorization and cache eligibility.

A cached response is not safe merely because the question is semantically similar.

## 8. Semantic-cache safety

A semantic cache entry must be ineligible when relevant context changes, including:

- corpus version
- permissions
- workspace
- source policy
- model/prompt configuration
- cache schema/eligibility rules

TTL supplements versioned identity; it does not replace it.

## 9. Safe observability

OpenTelemetry/CloudWatch/X-Ray should capture by default:

- trace/span IDs
- operation names
- duration
- cache hit/miss
- route
- model identifier
- token counts where available
- retrieval counts/scores
- opaque document IDs
- error/status metadata

Do not capture by default:

- full private prompts
- raw retrieved passages
- uploaded document contents
- credentials/tokens
- presigned URLs
- personal/confidential source content

## 10. Web evidence integrity

Web-derived claims must retain URL/title/retrieval timestamp/citation information.

A web source must never be presented as internal company policy.

Mixed answers should clearly distinguish company evidence from public evidence.

## 11. Prompt-injection resistance

Treat uploaded documents and web pages as **untrusted data**, not instructions.

Evaluation must include content that attempts to:

- override system/source policy
- request secrets
- alter routing
- bypass authorization
- suppress citations
- trigger future tools/actions

## 12. AWS audit and encryption

Use CloudTrail/audit facilities where they materially improve traceability.

Use service-managed or KMS encryption according to the data/resource threat model. The project should document why customer-managed KMS keys are used where enabled rather than adding them purely for complexity.

## 13. Cost as a security/reliability concern

Abuse can become spend.

Controls should eventually include:

- request/upload limits
- Bedrock invocation limits where practical
- AWS Budgets/alerts
- bounded batch concurrency
- queue/backpressure controls
- cache size/TTL policy

## Threats to test

- cross-workspace S3 access
- cross-workspace retrieval
- cross-workspace semantic cache hit
- stale cache after document update
- unauthorized cached citation
- private query sent to web search
- prompt injection in internal documents
- prompt injection in web content
- unsupported answer on empty retrieval
- oversized/malformed upload
- presigned upload scope abuse
- telemetry leaking confidential content
- excessive/replayed requests causing unbounded spend

These should become automated regression tests where feasible.
