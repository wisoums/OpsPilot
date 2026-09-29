# Evaluation and Benchmarking

OpsPilot should be measured as an AI system, not judged only by whether a demo answer looks good.

## Synthetic enterprise corpus

The repository will include a generator for fictional company documents containing invented names, processes, codes, and policies that a pretrained model could not reliably know.

Example:

```text
The NOVA-7 procedure requires Level Amber approval before a Helios
supplier can enter Stage Kappa.
```

This creates a controlled benchmark for retrieval and grounding.

## Query categories

Evaluation workloads should contain:

1. internal-only questions
2. general/web-only questions
3. mixed internal + external questions
4. paraphrased repeated questions
5. exact repeated questions
6. uncommon long-tail questions
7. deliberately unanswerable questions
8. conflicting/outdated document cases

## Metrics

### Retrieval

- Recall@k where ground truth is available
- MRR / rank of expected source
- retrieval latency
- zero-result rate
- source filtering correctness

### Routing

- internal/web/mixed classification accuracy
- private-query external leakage rate
- unnecessary web-search rate
- unnecessary internal-retrieval rate

### Generation and grounding

- answer correctness on synthetic ground truth
- citation coverage
- citation correctness
- unsupported-claim rate
- appropriate refusal rate

### Semantic cache

- exact hit rate
- semantic hit rate
- false-hit rate
- stale-hit rate (target: 0)
- LLM calls avoided
- latency reduction
- estimated cost reduction

### Ingestion

- documents/minute
- pages or chunks/minute
- failure rate
- incremental re-index efficiency
- indexing completion latency

### Runtime

- end-to-end average latency
- p50 / p95 / p99 latency
- token usage
- estimated cost/request
- error rate

## Benchmark scenarios

### Small

- 100 documents
- 1,000 questions

### Medium

- 1,000 documents
- 10,000 questions

### Large/local stress

Scale until the test machine becomes the bottleneck and report hardware.

## Benchmark rules

- Report hardware/environment.
- Separate cold and warm runs.
- Separate cache-disabled and cache-enabled results.
- Do not reuse AWS marketing benchmark numbers as OpsPilot results.
- Preserve benchmark configuration in version control.
