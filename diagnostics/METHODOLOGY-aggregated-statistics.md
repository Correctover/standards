# Correctover Conformance Testing — Aggregated Statistics & Methodology

> **Purpose**: This document provides verifiable aggregated statistics from the Correctover conformance testing infrastructure. Raw traces are proprietary; this document enables independent verification of all claims made in DIAG-001 and DIAG-002.
>
> **Last Updated**: 2026-07-04 (v2: Data clarity revision)

---

## 1. Dataset Overview

### 1.1 Core Production Dataset (Primary Asset)

| Metric | Value | Verification Method |
|--------|-------|---------------------|
| **Production traces** | **20,071** | `traces-metadata.json` in Correctover internal infrastructure |
| **Observation window** | 30 days | Production API calls |
| **Providers covered** | 13 | anthropic, azure, cohere, deepseek, fireworks, github_models, groq, meta, mistral, openai, perplexity, qwen, together |
| **SDK version** | 1.1.0 | `correctover-conformance-fixture-spec.md` |

**Note**: This 20,071-trace dataset is the **primary empirical foundation**. All production-derived statistics in this document are based on this dataset unless explicitly stated otherwise.

### 1.2 Extended Test Suite (Synthetic Augmentation)

| Metric | Value | Purpose |
|--------|-------|---------|
| **Synthetic edge cases** | 29,929 | Statistical significance for rare failure modes |
| **Total extended dataset** | 50,000 | Comprehensive coverage |
| **Generation method** | Deterministic (seed=42) | Reproducible via `generate_traces_50k.py` |
| **Date generated** | 2026-07-02 | Timestamp in metadata |

**Critical Distinction**: The 50K dataset is a **test suite**, not production data. It combines:
- **20,071 real production traces** (Core Integrity)
- **29,929 synthetic traces** generated to model failure modes observed in production but at insufficient volume for statistical analysis

**Do not conflate 20K production data with 50K test suite.** Claims about production behavior should reference the 20K dataset only.

---

## 2. Data Composition Breakdown

### 2.1 Production Dataset (20,071 traces)

| Group | Count | Description | Data Source |
|-------|-------|-------------|-------------|
| A — Stability Baseline | 5,019 | Normal API calls, all CANON dims pass | Production |
| B — Cross-Provider Routing | 5,018 | Multi-provider model routing | Production |
| C — Fault Injection (Control) | 5,017 | 5 fault types, no self-healing | Production-derived test scenarios |
| D — Self-Healing Stress Test | 5,017 | Recoverable faults with healing | Production-derived test scenarios |
| **Production Subtotal** | **20,071** | | |

### 2.2 Synthetic Extension (29,929 traces)

| Group | Count | Description | Data Source |
|-------|-------|-------------|-------------|
| E1 — Compound Fault Chain | 5,000 | Multiple faults stacked per request | Synthetic |
| E2 — Temporal Drift | 5,000 | Authority expired / policy stale / context drift | Synthetic |
| E3 — Cascade Failover | 4,000 | Multi-hop provider failover (2-3 hops) | Synthetic |
| E4 — Edge Latency | 4,000 | P95-P99.9 extreme latency scenarios | Synthetic |
| E5 — Partial Healing | 4,000 | Healed but degraded output | Synthetic |
| E6 — Concurrent Conflict | 7,929 | Decision_ref version collisions | Synthetic |
| **Synthetic Subtotal** | **29,929** | | |

---

## 3. Verdict Distribution — Production Dataset (20,071)

**Status**: Detailed verdict breakdown for production-only data is available under NDA for academic verification. Contact: wangguigui@correctover.com

**Summary statistics (production-derived)**:
- Groups A+B (baseline + routing): Majority pass all conformance checks
- Group C (fault injection, no self-healing): 0% recovery (by design)
- Group D (self-healing stress test): 97.4% recovery rate (see Section 5)

**Note**: The 50K test suite verdict distribution (Section 4) includes synthetic edge cases and should not be used to infer production failure rates.

---

## 4. Verdict Distribution — Extended Test Suite (50K)

| Verdict | Count | Percentage |
|---------|-------|------------|
| ALLOW | 23,878 | 47.8% |
| REPAIR | 15,664 | 31.3% |
| RECONCILE | 5,084 | 10.2% |
| HARD_BLOCK_OR_RECONCILE | 2,495 | 5.0% |
| HARD_BLOCK | 1,896 | 3.8% |
| DEFER | 983 | 2.0% |

**Interpretation**: This distribution reflects the **extended test suite** (including synthetic edge cases designed to stress-test the system). It demonstrates conformance protocol behavior under extreme conditions, not production baseline.

**Do not cite "52.2% failure rate" as a production statistic.** This figure applies to the 50K test suite, 60% of which is synthetic.

---

## 5. Recovery Statistics

| Metric | Value | Data Source |
|--------|-------|-------------|
| D-group recovery rate (single fault) | **97.4%** | Production-derived (5,017 traces) |
| E-group compound recovery rate (multi-fault) | ~72% | Synthetic (stress test) |
| Overall recovery rate (50K test suite) | 67.4% | Extended test suite |

**Key finding**: Single-fault self-healing achieves 97.4% recovery in production-derived scenarios. Under compound fault conditions (synthetic stress test), recovery degrades to ~72%, exposing gaps in single-fault testing approaches.

---

## 6. Negative Vector Coverage (PHI-OMEGA v0.2.1)

| Negative Vector | Trace Count | % of 50K Dataset |
|-----------------|-------------|------------------|
| params_hash_changed | 8,010 | 16.0% |
| context_hash_changed | 6,293 | 12.6% |
| policy_digest_stale | 3,753 | 7.5% |
| authority_expired | 3,253 | 6.5% |
| terminal_receipt_missing | 3,009 | 6.0% |
| rollback_unsupported | 1,504 | 3.0% |
| **Total** | **25,822** | **51.6%** |

**Verification**: All 6 PHI-OMEGA negative vector categories have production-derived fixtures. This is independently verifiable via `conformance-statistics-50k.json`.

---

## 7. Edge Cases — Coverage Gaps in PHI-OMEGA v0.2.1

**8 fault types from production data are NOT covered by PHI-OMEGA v0.2.1:**

| Fault Type | Production Trace Count | Category |
|------------|------------------------|----------|
| server_error_503 | 6,283 | Transient / recoverable |
| rate_limit_429 | 3,176 | Transient / recoverable |
| concurrent_conflict | 2,963 | Runtime race condition |
| cascade_timeout | 1,576 | Multi-hop failure |
| auth_error_401 | 1,503 | Structural / unrecoverable |
| authority_expired | 1,250 | Temporal drift |
| policy_digest_stale | 1,250 | Temporal drift |
| params_hash_changed | 1,250 | Parameter mutation |
| **Total uncovered** | **19,251** | |

**Note**: Production trace counts are estimates based on observed failure distributions. Exact counts available under NDA.

---

## 8. Failure Mode Catalog

**325 distinct failure modes identified** through:
- Production telemetry analysis (20,071 traces)
- Field testing across 13 providers
- Developer-reported edge cases
- Synthetic stress testing (E-groups)

**Breakdown by source**:
- **Production-derived**: ~200 failure modes (observed in 20K traces or derived from production patterns)
- **Synthetic/test-only**: ~125 failure modes (generated to cover theoretical edge cases)

**Do not claim "325 failure modes discovered from production traces."** The accurate statement is "325 failure modes identified through production observation and systematic testing."

---

## 9. Methodology

### 9.1 Data Collection
- **Source**: Production API calls across 13 LLM providers over 30-day observation window
- **Trace format**: JSONL with CANON lock dimensions, decision_ref chains, negative vectors
- **SDK**: Correctover Conformance Runner v1.1.0

### 9.2 Synthetic Expansion
- **Tool**: `generate_traces_50k.py` (deterministic, seed=42)
- **Purpose**: Augment production data with edge cases observed in production but at insufficient volume for statistical significance
- **Method**: Fault injection, temporal drift simulation, cascade failover modeling, concurrent conflict generation
- **Reproducibility**: Any researcher with the generator script and seed can reproduce the exact dataset

### 9.3 Conformance Evaluation
- **Standard**: PHI-OMEGA-RUNTIME v0.2.1 conformance specification
- **Runner**: `conformance_runner_50k.py`
- **Output**: `conformance-results-50k.jsonl` (verdicts, terminal states, recovery outcomes)

### 9.4 Verification Path
All aggregated statistics in this document can be verified by:
1. Running `conformance_runner_50k.py` against `traces-50k.jsonl`
2. Comparing output with `conformance-statistics-50k.json`
3. Cross-referencing verdict distributions in this document

**Note**: Raw traces (`traces-50k.jsonl`, 73.7 MB) are proprietary and not publicly distributed. Access is available under NDA for academic verification purposes. Contact: wangguigui@correctover.com

---

## 10. Independent Verification References

| Reference | Location | Verifiable By |
|-----------|----------|---------------|
| giskard09/argentum-core — Axis 4 Fork-Matrix | [GitHub](https://github.com/giskard09/argentum-core) | Public |
| babyblueviper1/preaction-governance-conformance | [GitHub](https://github.com/babyblueviper/preaction-governance-conformance) | Public |
| pshkv SINT Labs — Required/Supported implementation | [GitHub Issue](https://github.com/microsoft/autogen/issues/7525) | Public |
| Correctover/standards — Conformance SPEC v1 | [GitHub](https://github.com/Correctover/standards/blob/main/conformance/SPEC-v1.md) | Public |

---

## 11. Correction History

| Date | Correction | Reference |
|------|-----------|-----------|
| 2026-07-04 | **v2 Data Clarity Revision**: Separated 20K production dataset from 50K test suite. Removed misleading "52.2% failure rate" framing. Clarified that verdict distribution applies to extended test suite, not production baseline. | This document |
| 2026-07-03 | Removed incorrect CVE-2026-30615 reference (was Windsurf vulnerability, not MCP protocol CVSS 10.0) | DIAG-002 v2 |
| 2026-07-03 | Clarified data claims: 20,071 original traces + 29,929 synthetic expansion = 50,000 total | This document (v1) |

---

*This document is part of the Correctover Agent Conformance Standards repository. All claims are based on reproducible testing infrastructure. Aggregated statistics are public; raw traces are available under NDA for academic verification.*
