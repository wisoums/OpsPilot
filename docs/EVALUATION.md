# Evaluation and Benchmarking

OpsPilot is evaluated as an AWS AI system, not by whether a few demo answers look convincing.

## Synthetic enterprise corpus

The repository will generate fictional private business documents containing invented processes, names, codes, approval levels, and cross-document relationships that a pretrained model cannot be expected to know.

Example:

```text
The NOVA-7 procedure requires Level Amber approval before a Helios
supplier can enter Stage Kappa.
```

Ground truth is emitted alongside the corpus.

## Query categories

1. internal-only
2. public/web-only
3. mixed internal + web
4. exact repeated
5. paraphrased repeated
6. uncommon/long-tail
7. deliberately unanswerable
8. conflicting/outdated document cases
9. privacy-sensitive internal terminology
10. adversarial prompt-injection cases

## Retrieval metrics

- Recall@k
- expected-source rank / MRR
- zero-result rate
- source-filter correctness
- retrieval latency
- Bedrock Knowledge Base retrieval failures

## Routing/privacy metrics

- INTERNAL/WEB/MIXED classification accuracy
- unnecessary web-search rate
- unnecessary internal-retrieval rate
- private-query external leakage rate
- uncertain-route behavior

## Generation/grounding metrics

- factual correctness on synthetic ground truth
- unsupported-claim rate
- appropriate-refusal rate
- contradiction with retrieved evidence
- answer relevance where useful

## Citation metrics

- citation presence
- citation correctness
- citation coverage
- broken/inaccessible source references
- company-vs-web source labeling correctness

## Semantic-cache metrics

- exact hit rate
- semantic hit rate
- false-hit rate
- stale-hit rate (target: 0)
- cross-workspace/permission unsafe hit rate (target: 0)
- LLM calls avoided
- token reduction
- estimated Bedrock-cost reduction
- average/p95 latency reduction

## Ingestion metrics

Benchmark direct S3 upload separately from knowledge-base indexing.

### Upload

- files/second
- MB/second
- retry/failure rate
- large-file multipart behavior

### Ingestion/indexing

- documents/minute
- time from upload-complete to queryable
- failed-document rate
- batch completion time
- incremental update behavior

## Runtime/cost metrics

- p50 / p95 / p99 end-to-end latency
- Bedrock invocation latency
- error rate
- token usage
- estimated cost/request
- cost/1,000 queries
- cache-enabled vs cache-disabled cost

## Benchmark scenarios

### Small

- 100 documents
- 1,000 questions

### Medium

- 1,000 documents
- 10,000 questions

### Stress

Increase corpus/query/batch size until an AWS service quota, configured capacity, latency target, or cost budget becomes the limiting factor.

Report the exact AWS region, model IDs, Terraform configuration, cache configuration, and relevant quotas/capacity.

## Semantic-cache experiment

At minimum compare:

```text
A. cache disabled
B. exact-match caching
C. semantic caching
```

Across workload redundancy profiles such as:

```text
10%
30%
50%
70%
```

Resume/README percentage claims must come from these reproducible OpsPilot results.

## AI regression baselines

Version:

- prompts
- model IDs
- embedding model
- retrieval top-k/thresholds
- chunking configuration
- routing configuration
- cache similarity threshold
- grounding/refusal rules
- evaluation-dataset version

CI compares relevant changes against an accepted machine-readable baseline.

Examples of blocking regressions:

- private-query leakage increases
- stale cache hit occurs
- cross-workspace cache/retrieval succeeds
- unsupported-claim rate crosses threshold
- retrieval quality drops beyond budget

Performance/cost regressions may warn or fail according to explicit version-controlled budgets.

## Benchmark rules

- Never copy vendor benchmark percentages into OpsPilot claims.
- Record AWS region and service/model configuration.
- Separate cold and warm runs.
- Separate cache-disabled and cache-enabled runs.
- Record dataset/workload seed/version.
- Keep raw machine-readable results as artifacts.
- Make baseline updates explicit and reviewed.
- Do not print private production data into public CI logs.
