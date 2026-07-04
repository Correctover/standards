# Correctover Benchmark Methodology — Formal Reproducibility Document

> **Purpose**: This document provides a complete, formal description of the benchmark methodology used to derive all statistics in the Correctover conformance specification. It addresses reproducibility concerns raised during peer review.
>
> **Status**: DRAFT v1.0 (2026-07-04)
> **Authors**: Guigui Wang (Correctover), with input from Massimiliano Brighendi (PHI-OMEGA)

---

## 1. Scope and Limitations

### 1.1 What This Document Covers
- Complete description of data collection, preprocessing, and analysis methodology
- Formal definitions of all metrics reported in the conformance specification
- Reproducibility statement for each claim
- Description of the data sharing program for independent verification

### 1.2 What Cannot Be Reproduced Without Access
- **Raw production traces** (20,071) are proprietary and contain sensitive production data from multiple LLM providers. They cannot be publicly released.
- **Synthetic trace generation engine** — a proprietary governance component that depends on internal Correctover infrastructure. Its input/output schemas are described in SPEC-v1.md; the implementation is proprietary.
- **Conformance verification runtime** — part of the Correctover Compliance Architecture (closed-source). Its verification logic is specified in SPEC-v1.md; the implementation remains independent to preserve test integrity.

### 1.3 What CAN Be Independently Verified
- **Conformance fixture specifications** (SPEC-v1.md) — fully public, CC BY 4.0
- **5-case conformance test suite** — fully public, implementable by any researcher
- **Aggregate statistics** — reported in this document with exact definitions
- **Independent verification ecosystems** — PHI-OMEGA Runtime (transition-sufficiency verification) and preaction-governance-conformance (authorization gap detection) provide public, independently verifiable conformance verification

---

## 2. Data Collection Methodology

### 2.1 Production Dataset (20,071 traces)

**Collection Period**: 30-day observation window (2026-05-30 to 2026-06-29)

**Collection Method**:
- Correctover SDK v1.1.0 deployed as middleware intercepting all LLM API calls
- Each API call generates a structured trace containing:
  - Request metadata (provider, model, timestamp, token counts)
  - Response metadata (status code, latency, content hash)
  - CANON verification results (6 dimensions: Structure, Schema, Latency, Cost, Identity, Integrity)
  - Decision reference chain (authorization → execution → verification)
  - Negative vector flags (params_hash_changed, context_hash_changed, policy_digest_stale, authority_expired, terminal_receipt_missing, rollback_unsupported)

**Anonymization Protocol**:
- Provider names are recorded but not disclosed in public reports (referred to by category: tier-1 commercial, open-source endpoint, regional provider)
- Prompt content and response content are NOT stored in traces (only metadata and verification results)
- Timestamps are recorded at second granularity but reported at hour granularity in public documents

**Trace Format** (simplified schema):
```json
{
  "trace_id": "uuid-v4",
  "timestamp": "2026-06-15T14:30:22Z",
  "provider_category": "tier-1-commercial",
  "model_id": "[redacted]",
  "group": "A|B|C|D",
  "request_tokens": 0,
  "response_tokens": 0,
  "status_code": 200,
  "latency_ms": 622,
  "canon_dimensions": {
    "structure": "pass",
    "schema": "pass",
    "latency": "pass",
    "cost": "pass",
    "identity": "pass",
    "integrity": "pass"
  },
  "decision_ref": {
    "authorization_hash": "sha256:abc123...",
    "execution_hash": "sha256:abc123...",
    "match": true
  },
  "negative_vectors": {
    "params_hash_changed": false,
    "context_hash_changed": false,
    "policy_digest_stale": false,
    "authority_expired": false,
    "terminal_receipt_missing": false,
    "rollback_unsupported": false
  },
  "recovery_action": null,
  "recovery_outcome": null,
  "terminal_state": "COMPLETED"
}
```

### 2.2 Group Assignment

Traces are assigned to groups based on test design:

| Group | Purpose | Fault Injection | Self-Healing Enabled |
|-------|---------|----------------|---------------------|
| A | Stability baseline | None | Yes (no faults to heal) |
| B | Cross-provider routing | None (routing changes only) | Yes |
| C | Fault injection control | 5 fault types | No (control group) |
| D | Self-healing stress test | Recoverable faults only | Yes |

**Fault types injected in Groups C/D**:
1. Invalid API key → 401 (structural, unrecoverable)
2. Invalid model name → 400/404 (structural, unrecoverable)
3. DNS/connection failure (structural, unrecoverable)
4. Invalid request body → 400 (structural, unrecoverable)
5. Timeout (transient, recoverable via failover)

### 2.3 Synthetic Extension (29,929 traces)

**Generation Method**: Deterministic expansion using proprietary fault injection engine
- **Seed**: 42
- **Purpose**: Augment production data with edge cases observed in production at insufficient volume
- **6 categories** (E1-E6): compound faults, temporal drift, cascade failover, extreme latency, partial healing, concurrent conflicts

**Reproducibility**: The synthetic generator exists but is proprietary. An independent researcher cannot reproduce the exact 29,929 synthetic traces without access to the generator. However, the statistical properties (distribution of fault types, recovery rates) are fully described in this document and can be independently validated using the 5-case conformance test suite.

---

## 3. Metric Definitions

### 3.1 Recovery Rate

**Definition**: Recovery Rate = (traces where self-healing succeeded AND post-healing CANON verification passed) / (traces where self-healing was attempted)

**Single-fault recovery (D-group)**: 97.4% = 4,887 / 5,017

**Compound-fault recovery (E-groups)**: ~72% = estimated from synthetic stress testing where multiple simultaneous faults are injected

### 3.2 Verification Latency

**Definition**: Time from response receipt to completion of all 6 CANON dimension checks.

**Measurement**: P50 = 22μs, measured using Python `time.perf_counter_ns()` on dedicated verification thread.

**Note**: This measures verification overhead only, not the total request-response cycle.

### 3.3 Non-Conformance Rate

**Definition**: Percentage of traces where at least one CANON dimension failed OR at least one negative vector was triggered.

**Production dataset**: 3,847 / 20,071 = 19.2% non-conformant traces

### 3.4 Failure Mode Count

**Definition**: A unique failure mode is a distinct combination of (fault_type, provider_category, recovery_outcome) observed in traces.

**Total**: 325 distinct failure modes
- ~200 from production traces (Groups A-D)
- ~125 from synthetic traces (Groups E1-E6)

---

## 4. Reproducibility Statement

### 4.1 Fully Reproducible (Public)
- [ ] 5-case conformance test specification (SPEC-v1.md, Section 4)
- [ ] Conformance fixture schema (JSON schema in SPEC-v1.md)
- [ ] Aggregate statistics tables (this document)
- [ ] Independent verification ecosystems (PHI-OMEGA Runtime, preaction-governance-conformance)

### 4.2 Reproducible Under NDA
- [ ] 200-500 anonymized sample traces (available upon request for academic verification)
- [ ] Full aggregated statistics by provider category
- [ ] Detailed fault injection specifications
- [ ] Contact: wangguigui@correctover.com

### 4.3 Not Reproducible (Proprietary)
- [ ] Raw production traces (20,071) — contain provider-specific production data
- [ ] Synthetic trace generator — proprietary code
- [ ] Conformance runner implementation — part of closed-source SDK
- [ ] Flywheel rule set (88 rules) — core intellectual property

### 4.4 Justification for Non-Disclosure
The proprietary elements constitute Correctover's core technology (LLM reliability engine). Disclosing them would:
1. Eliminate the competitive moat of a solo-founder project
2. Enable direct cloning by well-resourced competitors
3. Violate the trust of production environment contributors

The public elements (specification, fixture schema, aggregate statistics, independent implementations) provide sufficient basis for:
1. Understanding the conformance protocol
2. Implementing compatible verification systems
3. Validating the statistical claims through independent testing
4. Building upon the framework without depending on Correctover's proprietary code

---

## 5. Data Access Program

### 5.1 Academic Verification Access
Researchers may request access to anonymized sample traces for the purpose of:
- Validating statistical claims in the conformance specification
- Implementing independent conformance verification
- Academic publication (with citation)

**Requirements**:
- Signed Data Use Agreement (one-time academic use, no redistribution)
- Institutional affiliation verification
- Brief description of intended use

**What is provided**:
- 200-500 anonymized traces (JSON format, provider names redacted)
- 325 failure mode classification table
- Aggregate statistics by group and provider category
- This methodology document

**What is NOT provided**:
- Raw prompt/response content
- Provider-specific API response data
- Flywheel rule logic
- Complete production dataset

### 5.2 Contact
wangguigui@correctover.com

---

## 6. Revision History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2026-07-04 | Initial formal methodology document. Addresses reproducibility concerns from Section 5 peer review. |

---

*This document is part of the Correctover Agent Conformance Standards repository.*
*Licensed under CC BY 4.0.*
