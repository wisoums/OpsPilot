# Evaluation and Benchmarking

OpsPilot is evaluated as an AWS AI system, not by whether a few demo answers look convincing.

The flagship release uses a **fixed, versioned evaluation set** and treats quality, latency, privacy, and cost as regression-tested software properties.

## Release acceptance targets

These are project targets, not published claims. Final README/resume numbers must come from reproducible benchmark artifacts.

| Dimension | Release target |
|---|---:|
| Faithfulness | >= 0.85 |
| Answer relevancy | >= 0.80 |
| Private-query external leakage | 0 |
| Stale semantic-cache unsafe hits | 0 |
| Cross-context unsafe cache hits | 0 |
| CI regression gate | Proven to fail on an intentionally bad change |
| p95 latency | Measured and versioned against baseline |
| Cost / 1,000 queries | Measured with cache OFF vs ON |

Retrieval Recall@k/MRR targets are established from the initial baseline and then version-controlled rather than invented in advance.

## Fixed synthetic enterprise corpus

The repository generates fictional private business documents containing invented processes, names, codes, approval levels, and cross-document relationships that a pretrained model cannot be expected to know.

Example:

```text
The NOVA-7 procedure requires Level Amber approval before a Helios
supplier can enter Stage Kappa.
```

Ground truth is emitted alongside the corpus so retrieval and answer correctness can be scored reproducibly.

## Evaluation set

The fixed set should cover:

1. internal-only questions
2. public/web-only questions
3. mixed internal + web questions
4. exact repeated questions
5. paraphrased repeated questions
6. deliberately unanswerable questions
7. privacy-sensitive internal terminology
8. conflicting/outdated evidence where useful for grounding tests

The dataset version/seed must be recorded with each accepted baseline.

## Retrieval metrics

Use deterministic retrieval metrics wherever possible:

- Recall@k
- expected-source rank / MRR
- zero-result rate
- source-filter correctness
- retrieval latency

## Generation and grounding metrics

Use **Ragas or an equivalent LLM-as-judge only where semantic grading is required**, combined with deterministic checks for synthetic facts.

Report:

- faithfulness
- answer relevancy
- factual correctness against synthetic ground truth
- unsupported-claim rate
- appropriate-refusal rate

## Routing and privacy metrics

Report:

- INTERNAL / WEB / MIXED routing accuracy
- unnecessary web-search rate
- unnecessary internal-retrieval rate
- private-query external leakage rate

Privacy leakage above zero is a blocking regression.

## Citation metrics

Report:

- citation presence
- citation correctness
- citation coverage
- company-vs-web source labeling correctness

## Semantic-cache benchmark

At minimum compare:

```text
A. cache disabled
B. semantic cache enabled
```

Use a reproducible workload containing repeated and paraphrased queries.

Report:

- semantic cache hit rate
- false-hit rate
- stale/unsafe hit rate
- LLM calls avoided
- token reduction
- average latency
- p95 latency
- estimated Bedrock cost per request
- estimated Bedrock cost per 1,000 queries

The README must include the final before/after table:

| Metric | Cache OFF | Semantic Cache ON | Change |
|---|---:|---:|---:|
| p95 latency | TBD | TBD | TBD |
| Bedrock calls / 1k queries | TBD | TBD | TBD |
| Cost / 1k queries | TBD | TBD | TBD |
| Faithfulness | TBD | TBD | TBD |
| Answer relevancy | TBD | TBD | TBD |
| Semantic cache hit rate | — | TBD | — |

## OpenTelemetry evidence

For benchmark runs, capture traces/spans for:

```text
question.request
+-- source_router
+-- semantic_cache.lookup
+-- retrieval.internal / retrieval.web
+-- llm.generate
+-- citation.build
+-- grounding.check
```

At minimum the benchmark artifact should make it possible to explain where latency and model cost came from.

## AI regression baselines

Version:

- prompts
- model IDs
- embedding model
- retrieval top-k / thresholds
- chunking configuration
- routing configuration
- cache similarity threshold
- grounding/refusal rules
- evaluation-dataset version
- accepted latency and cost budgets

CI compares relevant changes against an accepted machine-readable baseline.

### Blocking regressions

CI must fail when any of the following occurs:

- faithfulness < 0.85
- answer relevancy < 0.80
- private-query leakage > 0
- stale/unsafe cache hit occurs
- retrieval quality drops beyond its accepted budget
- p95 latency regresses beyond the configured budget
- cost/query regresses beyond the configured budget

Baseline updates must be explicit and reviewed; a failing run cannot silently rewrite the baseline.

## Benchmark artifact rules

- Record AWS region and exact model IDs.
- Record Terraform and cache configuration relevant to the run.
- Separate cold and warm measurements where meaningful.
- Separate cache-disabled and cache-enabled runs.
- Record dataset/workload seed and version.
- Keep raw machine-readable results as CI/build artifacts.
- Never copy vendor benchmark percentages into OpsPilot claims.
- Do not print private production data into public CI logs.
- Resume/README percentage claims must come from these reproducible OpsPilot results.
