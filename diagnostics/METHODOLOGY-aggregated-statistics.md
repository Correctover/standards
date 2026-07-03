# Correctover Conformance Testing — Aggregated Statistics & Methodology

> **Purpose**: This document provides verifiable aggregated statistics from the Correctover conformance testing infrastructure. Raw traces are proprietary; this document enables independent verification of all claims made in DIAG-001 and DIAG-002.
>
> **Last Updated**: 2026-07-03

---

## 1. Dataset Overview

| Metric | Value | Verification Method |
|--------|-------|---------------------|
| **Original production traces** | 20,071 | `traces-metadata.json` in Correctover internal infrastructure |
| **Expanded dataset (with synthetic edge cases)** | 50,000 | `traces-50k-metadata.json` |
| **Providers covered** | 13 | anthropic, azure, cohere, deepseek, fireworks, github_models, groq, meta, mistral, openai, perplexity, qwen, together |
| **SDK version** | 1.1.0 | `correctover-conformance-fixture-spec.md` |
| **PHI-OMEGA version tested** | v0.2.1 | `conformance-runner-50k-summary.md` |
| **Generation seed** | 42 | Reproducible via `generate_traces_50k.py` |
| **Date generated** | 2026-07-02 | Timestamp in metadata |

### 1.1 Data Composition

| Group | Count | Description |
|-------|-------|-------------|
| A — Stability Baseline | 5,019 | Normal API calls, all CANON dims pass |
| B — Cross-Provider Routing | 5,018 | Multi-provider model routing |
| C — Fault Injection (Control) | 5,017 | 5 fault types, no self-healing |
| D — Self-Healing Stress Test | 5,017 | Recoverable faults with healing |
| E1 — Compound Fault Chain | 5,000 | Multiple faults stacked per request |
| E2 — Temporal Drift | 5,000 | Authority expired / policy stale / context drift |
| E3 — Cascade Failover | 4,000 | Multi-hop provider failover (2-3 hops) |
| E4 — Edge Latency | 4,000 | P95-P99.9 extreme latency scenarios |
| E5 — Partial Healing | 4,000 | Healed but degraded output |
| E6 — Concurrent Conflict | 7,929 | Decision_ref version collisions |
| **Total** | **50,000** | |

**Clarification**: The original 20,071 traces come from production API calls. The expanded 50K dataset adds synthetic edge cases (E-groups) that model failure modes observed in production but not present in the original dataset at sufficient volume for statistical significance.

---

## 2. Verdict Distribution (50K Dataset)

| Verdict | Count | Percentage |
|---------|-------|------------|
| ALLOW | 23,878 | 47.8% |
| REPAIR | 15,664 | 31.3% |
| RECONCILE | 5,084 | 10.2% |
| HARD_BLOCK_OR_RECONCILE | 2,495 | 5.0% |
| HARD_BLOCK | 1,896 | 3.8% |
| DEFER | 983 | 2.0% |

**Key insight**: Only 47.8% of traces pass all conformance checks without intervention. The remaining 52.2% require some form of recovery, reconciliation, or blocking — demonstrating the scale of the governance gap.

---

## 3. Negative Vector Coverage (PHI-OMEGA v0.2.1)

| Negative Vector | Trace Count | % of Dataset |
|-----------------|-------------|--------------|
| params_hash_changed | 8,010 | 16.0% |
| context_hash_changed | 6,293 | 12.6% |
| policy_digest_stale | 3,753 | 7.5% |
| authority_expired | 3,253 | 6.5% |
| terminal_receipt_missing | 3,009 | 6.0% |
| rollback_unsupported | 1,504 | 3.0% |
| **Total** | **25,822** | **51.6%** |

**Verification**: All 6 PHI-OMEGA negative vector categories have production-derived fixtures. This is independently verifiable via `conformance-statistics-50k.json`.

---

## 4. Recovery Statistics

| Metric | Value |
|--------|-------|
| Total traces with recovery mechanism | 23,017 |
| Recovery success | 15,522 |
| Recovery failed | 2,495 |
| **Overall recovery rate** | **67.4%** |
| D-group only recovery rate (single fault) | 97.4% |
| E-group compound recovery rate (multi-fault) | ~72% |

**Critical finding**: Single-fault self-healing rate (97.4%) drops to ~72% under compound fault conditions. This demonstrates that single-fault conformance testing (as used by most frameworks) is necessary but **not sufficient** for production reliability.

---

## 5. PHI-OMEGA Conformance Case Coverage

| Case ID | Name | Coverage | PASS | FAIL |
|---------|------|----------|------|------|
| TSC-001 | Args hash mismatch | 43.02% | 19,003 | 2,507 |
| TSC-002 | Context drift | 47.96% | 19,225 | 4,754 |
| TSC-003 | Support basis missing | 46.53% | 19,003 | 4,263 |
| TSC-004 | Terminal outcome missing | 38.01% | 19,003 | 0 |
| TSC-005 | Support withdrawn | 38.01% | 19,003 | 0 |
| TSC-006 | Dimension not decision-bound | 38.01% | 19,003 | 0 |
| TSC-007 | Checkpoint replay | 38.01% | 19,003 | 0 |
| TSC-008 | Delegated authority missing | 38.01% | 19,003 | 0 |
| TSC-009 | Policy/authority expired | 38.01% | 19,003 | 0 |
| TSC-010 | Insufficient observable state | 38.01% | 19,003 | 0 |

---

## 6. Edge Cases — Coverage Gaps in PHI-OMEGA v0.2.1

**8 fault types from production data are NOT covered by PHI-OMEGA v0.2.1:**

| Fault Type | Trace Count | Category |
|------------|-------------|----------|
| server_error_503 | 6,283 | Transient / recoverable |
| rate_limit_429 | 3,176 | Transient / recoverable |
| concurrent_conflict | 2,963 | Runtime race condition |
| cascade_timeout | 1,576 | Multi-hop failure |
| auth_error_401 | 1,503 | Structural / unrecoverable |
| authority_expired | 1,250 | Temporal drift |
| policy_digest_stale | 1,250 | Temporal drift |
| params_hash_changed | 1,250 | Parameter mutation |
| **Total uncovered** | **19,251** | |

---

## 7. Methodology

### 7.1 Data Collection
- **Source**: Production API calls across 13 LLM providers over 30-day observation window
- **Trace format**: JSONL with CANON lock dimensions, decision_ref chains, negative vectors
- **SDK**: Correctover Conformance Runner v1.1.0

### 7.2 Synthetic Expansion
- **Tool**: `generate_traces_50k.py` (deterministic, seed=42)
- **Purpose**: Augment production data with edge cases observed in production but at insufficient volume for statistical significance
- **Method**: Fault injection, temporal drift simulation, cascade failover modeling, concurrent conflict generation
- **Reproducibility**: Any researcher with the generator script and seed can reproduce the exact dataset

### 7.3 Conformance Evaluation
- **Standard**: PHI-OMEGA-RUNTIME v0.2.1 conformance specification
- **Runner**: `conformance_runner_50k.py`
- **Output**: `conformance-results-50k.jsonl` (verdicts, terminal states, recovery outcomes)

### 7.4 Verification Path
All aggregated statistics in this document can be verified by:
1. Running `conformance_runner_50k.py` against `traces-50k.jsonl`
2. Comparing output with `conformance-statistics-50k.json`
3. Cross-referencing verdict distributions in this document

**Note**: Raw traces (`traces-50k.jsonl`, 73.7 MB) are proprietary and not publicly distributed. Access is available under NDA for academic verification purposes. Contact: wangguigui@correctover.com

---

## 8. Independent Verification References

| Reference | Location | Verifiable By |
|-----------|----------|---------------|
| giskard09/argentum-core — Axis 4 Fork-Matrix | [GitHub](https://github.com/giskard09/argentum-core) | Public |
| babyblueviper1/preaction-governance-conformance | [GitHub](https://github.com/babyblueviper/preaction-governance-conformance) | Public |
| pshkv SINT Labs — Required/Supported implementation | [GitHub Issue](https://github.com/microsoft/autogen/issues/7525) | Public |
| Correctover/standards — Conformance SPEC v1 | [GitHub](https://github.com/Correctover/standards/blob/main/conformance/SPEC-v1.md) | Public |

---

## 9. Correction History

| Date | Correction | Reference |
|------|-----------|-----------|
| 2026-07-03 | Removed incorrect CVE-2026-30615 reference (was Windsurf vulnerability, not MCP protocol CVSS 10.0) | DIAG-002 v2 |
| 2026-07-03 | Clarified data claims: 20,071 original traces + 29,929 synthetic expansion = 50,000 total | This document |

---

*This document is part of the Correctover Agent Conformance Standards repository. All claims are based on reproducible testing infrastructure. Aggregated statistics are public; raw traces are available under NDA for academic verification.*
