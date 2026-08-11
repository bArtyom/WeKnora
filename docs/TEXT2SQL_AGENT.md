# Enterprise Text2SQL Agent Blueprint

## 1. Positioning

This document defines the maintainer-led Text2SQL direction for this public WeKnora fork. It is a product and engineering blueprint, not a claim that every capability is already shipped.

The system is intended for enterprise analytics teams that need reliable answers over fragmented data platforms. Its primary promise is grounded data discovery and governed query execution: find the right tables, resolve business meaning, produce explainable SQL, and make unsafe or ambiguous requests visible.

## 2. Problems to solve

Enterprise Text2SQL fails for reasons that go beyond SQL syntax:

1. **Schema discovery:** the correct table is hidden among similarly named tables, historical snapshots, marts, and tenant-specific schemas.
2. **Business semantics:** terms such as “active customer”, “revenue”, or “conversion” have governed definitions that are not recoverable from column names alone.
3. **Join correctness:** multiple valid-looking join paths can produce duplicated or silently wrong results.
4. **Dialect and version drift:** SQL syntax, functions, permissions, and cost behavior vary across engines and versions.
5. **Governance:** a syntactically valid query can still violate row-level, column-level, tenant, PII, or export policies.
6. **Reliability and trust:** users need evidence for table selection, metric definitions, filters, and execution results—not just generated SQL.

## 3. Target workflow

User intent
  -> intent decomposition and ambiguity detection
  -> catalog / glossary / lineage retrieval
  -> candidate table and column ranking
  -> join-path and metric-plan construction
  -> dialect-aware SQL compilation
  -> AST, policy, cost and read-only validation
  -> dry-run / execution gateway
  -> repair or clarification loop
  -> result + SQL + evidence + audit trace

The agent should stop and ask a clarification question when the requested metric, time range, tenant scope, or join path is not sufficiently grounded.

## 4. Proposed components

### 4.1 Catalog and semantic ingestion

Connectors should capture schema names, table and column comments, types, owners, tags, freshness, sample statistics, lineage edges, access policies, and approved glossary definitions. Ingestion must be incremental and versioned so that generated SQL can be tied to the metadata snapshot used for planning.

### 4.2 Hybrid discovery

Candidate retrieval should combine:

- lexical matching for exact identifiers and abbreviations;
- embeddings for business-language similarity;
- graph traversal for lineage and join relationships;
- freshness, ownership, quality, and usage signals for ranking;
- evidence snippets that can be shown to the user and stored in the trace.

The ranking contract should expose why a table or column was selected and allow a human to override or pin governed sources.

### 4.3 Semantic planning

Before SQL generation, the planner should represent the request as entities, measures, dimensions, filters, time windows, grain, ordering, limits, and required joins. Metric definitions and policy constraints must be first-class inputs rather than hidden prompt text.

### 4.4 SQL compilation and repair

The compiler should target an explicit dialect adapter and produce an intermediate plan before rendering SQL. Validation should include parsing, unknown identifiers, aggregate/grain consistency, join cardinality warnings, dangerous statements, missing predicates, cost limits, and explain or dry-run results. Repairs must be bounded and revalidated; repeated failure should return a clear diagnostic instead of silently guessing.

### 4.5 Governed execution

The execution gateway should enforce:

- read-only by default;
- tenant, row, and column policy evaluation;
- sensitive-field masking and export restrictions;
- statement timeout, scan/cost limits, and result-size limits;
- approval for write, DDL, or high-risk queries;
- query fingerprinting, audit events, and trace correlation.

### 4.6 Evidence and observability

Every answer should be able to show the metadata snapshot, selected tables and columns, metric definitions, join path, generated SQL, validation results, execution metadata, and user feedback. Langfuse and repository-native audit facilities can provide tracing, but the product contract should remain inspectable without a specific observability vendor.

## 5. Evaluation plan

The evaluation suite should include synthetic and de-identified enterprise schemas with hard negatives, aliases, stale tables, ambiguous metrics, multi-hop joins, dialect differences, and policy constraints.

Recommended metrics:

| Area | Metrics |
| --- | --- |
| Discovery | table recall@k, column recall@k, evidence precision |
| Semantics | metric-definition accuracy, ambiguity-detection rate |
| Planning | join-path accuracy, grain consistency |
| SQL | execution accuracy, parse success, repair success, cost regression |
| Safety | policy violation rate, unsafe-statement rejection, PII leakage rate |
| Operations | p50/p95 latency, trace completeness, reproducibility |

No specialist capability should be marked shipped until it has targeted tests, representative evaluation cases, and documented failure modes.

## 6. Rollout

1. **Foundation:** metadata contracts, catalog snapshot model, evidence schema, and read-only execution boundary.
2. **Discovery:** table/column retrieval, glossary search, ranking explanations, and feedback capture.
3. **Planning:** semantic intermediate representation, join graph, dialect adapters, and clarification loop.
4. **Reliability:** AST validation, dry-run, bounded repair, cost controls, and regression suite.
5. **Governance:** row/column policies, masking, approval flow, audit, and tenant isolation.
6. **Operations:** benchmark dashboards, trace correlation, deployment guides, and contributor documentation.

## 7. Upstream relationship and license

This repository is a maintainer-led derivative of [Tencent/WeKnora](https://github.com/Tencent/WeKnora). Upstream attribution, fork metadata, and the [MIT License](../LICENSE) remain part of the project. Text2SQL-specific work should be clearly separated into new commits, tests, docs, and release notes so downstream users can distinguish the existing WeKnora foundation from this specialization.
