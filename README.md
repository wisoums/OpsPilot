# OpsPilot

> **AWS-native RAG and LLMOps system for private company knowledge, live web research, and source-backed answers.**

OpsPilot is a production-focused AI engineering project built to make RAG **measurable, observable, and regression-tested** on AWS.

> 🚧 **Status:** in development.

## What OpsPilot does

- Answers questions from **private documents**, the **live web**, or both.
- Uses **Amazon Bedrock Knowledge Bases + S3 Vectors** for private-document retrieval.
- Routes internal, public, and mixed questions to the correct evidence source.
- Uses **ElastiCache for Valkey** as a semantic cache to reduce repeated model calls.
- Evaluates retrieval quality, faithfulness, answer relevancy, routing, latency, and cost on a fixed test set.
- Traces retrieval, generation, and cache behavior with **OpenTelemetry**.
- Runs an **AI regression gate in CI** that blocks changes when quality, privacy, latency, or cost crosses accepted thresholds.
- Deploys the AWS infrastructure with **Terraform**.

## Architecture

```text
React / TypeScript
        |
   CloudFront
        |
   API Gateway
        |
      Lambda
        |
  +-----+-------------------+
  |                         |
Semantic Cache          Source Router
Valkey                  /           \
  |             Private RAG       Live Web
  |             Bedrock KB        Bedrock
  |             + S3 Vectors      Web Search
  |                    \           /
  +--------------------- Bedrock
                           |
                    Answer + Citations
```

Supporting services include S3, Cognito, CloudWatch/X-Ray, IAM, and AWS Budgets.

## Evaluation targets

Final README/resume claims will come only from reproducible OpsPilot benchmark runs.

| Metric | Target / Output |
|---|---:|
| Faithfulness | >= 0.85 |
| Answer relevancy | >= 0.80 |
| Private-query external leakage | 0 |
| Invalid semantic-cache hits | 0 |
| Retrieval quality | Recall@k / MRR reported |
| p95 latency | Cache OFF vs ON |
| Cost / 1,000 queries | Cache OFF vs ON |
| CI regression gate | Must block an intentionally bad change |

The final benchmark will report semantic-cache hit rate, LLM calls avoided, p95 latency, and estimated Bedrock cost with caching disabled vs enabled.

## Tech stack

**Python · React · TypeScript · AWS Bedrock · Lambda · S3 · S3 Vectors · ElastiCache/Valkey · OpenTelemetry · Terraform · GitHub Actions**

## Roadmap

1. Foundation + Terraform
2. AWS RAG over private documents
3. Private + web source routing
4. Semantic cache
5. OpenTelemetry observability
6. Evaluation harness
7. AI regression gate in CI
8. Final benchmark + technical write-up/demo

See [docs/ROADMAP.md](docs/ROADMAP.md) for implementation details.

## Documentation

- [Architecture](docs/ARCHITECTURE.md)
- [Evaluation & Benchmarking](docs/EVALUATION.md)
- [Security & Trust](docs/SECURITY.md)
- [Roadmap](docs/ROADMAP.md)
- [Contributing](CONTRIBUTING.md)
