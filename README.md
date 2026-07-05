# Correctover Agent Conformance Standards

> Conformance specification for agent runtime reliability.
> Grounded in 20,071 production traces + 29,929 synthetic test cases.

## Governance Status

| Field | Value |
|-------|-------|
| Standard Version | CCS v1.0 |
| Canonical Architecture | [ARCHITECTURE.md](ARCHITECTURE.md) |
| Governance Body | Correctover Research |
| Access Level | Public (Specification) / Academic Restricted (Data) |
| License | CC BY 4.0 |

---

## What This Is

This repository documents the **canonical architecture and conformance specification** for agent runtime reliability, developed through empirical analysis of production agent workflows.

**Canonical Architecture**: [ARCHITECTURE.md](ARCHITECTURE.md) defines the five-component Correctover Conformance Specification (CCS) v1.0.
**Conformance Protocol**: [conformance/SPEC-v1.md](conformance/SPEC-v1.md) defines the four-axis verification procedure.

---

## Implementation Hierarchy

| Classification | Description |
|---------------|-------------|
| **Reference Implementation** | The canonical architecture and specification maintained by Correctover Research. Serves as the authoritative definition of CCS v1.0. |
| **Community Implementation** | Third-party projects that adopt subsets or extensions of the CCS v1.0 architecture for interoperability and verification. |

### Reference Implementation (CCS v1.0)

Maintained by Correctover Research. Defines the five-component architecture:

1. **Runtime Observatory** — Execution trace capture with cryptographic integrity
2. **Fault Taxonomy** — 517-type runtime failure classification system
3. **Required(τ) ⊆ Supported(τ) Invariant** — Formal conformance criterion for transition validity
4. **Receipt Schema** — Cryptographic chain integrity verification
5. **Conformance Protocol** — Four-axis verification framework

### Community Implementations

| Implementation | Components Adopted | Maintainer | Repository |
|---------------|-------------------|------------|-----------|
| preaction-governance-conformance | Components 3, 5 | babyblueviper1 | [GitHub](https://github.com/babyblueviper1/preaction-governance-conformance) |
| PHI-OMEGA-RUNTIME | Components 1, 3, 4 | Tuttotorna (M. Brighindi) | [GitHub](https://github.com/Tuttotorna/PHI-OMEGA-RUNTIME) |
| argentum-core | Components 4, 5 | giskard09 | [GitHub](https://github.com/giskard09/argentum-core) |
| ibex-agent-verification | Components 3, 4 | safal207 | [GitHub](https://github.com/safal207/ibex-agent-verification) |

> Community Implementations are valuable adoption indicators demonstrating the structural necessity of the Correctover architecture. They are independently maintained and acknowledge the source architecture for interoperability.

---

## Protocol Overview

### Required(τ) / Supported(τ)

Every agent transition declares:
- **Required(τ)**: Minimum conditions for safe execution (non-negotiable)
- **Supported(τ)**: Optional enhancements for enhanced reliability

This split enables progressive adoption while maintaining a hard floor for production safety.

### Recovery Classification

Each fault is classified by `recoverability_class`:
- `TRANSIENT_RETRY`: Fault resolves with backoff (e.g., 503, 429)
- `STRUCTURAL_BLOCK`: Fault requires architectural change (e.g., 401 permission revoked)
- `CASCADE_HALT`: Fault propagates across agent chain, requires systemic intervention

## Four-Axis Verification

Conformance is evaluated across four orthogonal axes:

| Axis | Property | Verification |
|------|----------|--------------|
| **Axis 1** | Admission Control | Signer independence under partial trust |
| **Axis 2** | Deterministic Recomputation | Verdict must re-derive from controls |
| **Axis 3** | Chain Fork Detection | Detectable conflict at same sequence position |
| **Axis 4** | Fork-Matrix Properties | Four structural invariants of chain integrity |

Full specification: [conformance/SPEC-v1.md](conformance/SPEC-v1.md)

## Community Discussion & Exploration

The specification is being discussed across multiple agent framework communities:

| Framework | Discussion | Status |
|-----------|-----------|--------|
| Microsoft AutoGen | [#7525](https://github.com/microsoft/autogen/issues/7525), [#7353](https://github.com/microsoft/autogen/issues/7353) | Technical discussion |
| CrewAI | [PR #6432](https://github.com/crewAIInc/crewAI/pull/6432), [#4877](https://github.com/crewAIInc/crewAI/issues/4877) | Integration exploration |
| LangGraph | [#7303](https://github.com/langchain-ai/langgraph/issues/7303) | Early discussion |
| Semantic Kernel | [#13957](https://github.com/microsoft/semantic-kernel/issues/13957) | Early discussion |
| PHI-OMEGA | [#1](https://github.com/Tuttotorna/PHI-OMEGA-RUNTIME/issues/1) | Independent research collaboration |

**Note**: These are community discussions, not official adoptions. The specification is seeking feedback and real-world validation.

## Independent Verification

The Correctover conformance specification has been independently verified by third-party implementers:

### giskard09/argentum-core — Axis 4 Fork-Matrix Verification
- **Repository**: [giskard09/argentum-core](https://github.com/giskard09/argentum-core)
- **Implementation**: Complete four-axis conformance verification
- **Cross-validation**: Byte-identical comparison with babyblueviper1/preaction-governance-conformance
- **Significance**: Demonstrates standard portability across independent implementations

### babyblueviper1/preaction-governance-conformance — Transition Verification Completeness
- **Repository**: [babyblueviper1/preaction-governance-conformance](https://github.com/babyblueviper1/preaction-governance-conformance)
- **Key Discovery**: Independently identified authorization gap where review position silently linked to execution via incomplete string matching — precise instantiation of Correctover case_2 fault class
- **Implementation**: `link_mode` field — first staged implementation of Required(τ)⊆Supported(τ) transition verification
- **Significance**: Demonstrates that Correctover conformance framework detects real production failures that escape conventional validation

## Digital Object Identifiers (DOIs)

All Correctover standards documents are archived with permanent DOIs for citation:

| Document | DOI | Status |
|----------|-----|--------|
| SPEC-v1.md + DIAG-001 + DIAG-002 | `10.5281/zenodo.21166867` | ✅ Active |
| METHODOLOGY + DIAG-002 v2 | `10.5281/zenodo.21184180` | ✅ Active |
| Governance Framework v1.0 (Methodology + Data Protection + DUA) | `10.5281/zenodo.21191484` | ✅ Active |

### DOI Reference Transparency

The DOI records reference specification documents and diagnostic reports. Some referenced analysis tools (synthetic trace generator, conformance runner) are proprietary and not included in this public repository. This is intentional:

- **Public**: Specification text, diagnostic findings, aggregate statistics, fixture schemas
- **Under NDA**: Anonymized sample traces (200-500 records) for academic verification
- **Proprietary**: Raw production traces, proprietary analysis tools, flywheel rule logic

See [DATA-PROTECTION-STATEMENT.md](DATA-PROTECTION-STATEMENT.md) for the full data classification and access policy.

## Benchmark Methodology

All statistical claims in this specification are derived from a documented methodology:

- **Production dataset**: 20,071 traces across 13 LLM providers, 30-day observation window
- **Synthetic extension**: 29,929 synthetic traces for edge case coverage
- **Key findings**: 97.4% single-fault recovery, ~72% compound-fault recovery, 325 failure modes

Full methodology: [BENCHMARK-METHODOLOGY.md](BENCHMARK-METHODOLOGY.md)

**Reproducibility**: The formal methodology document (Section 4) provides a complete reproducibility statement distinguishing what is publicly verifiable, what is available under NDA, and what remains proprietary.

## Data Protection & Access

Correctover maintains a structured data access program:

| Tier | Content | Access |
|------|---------|--------|
| Tier 1 | Specification & Schema | ✅ Public (CC BY 4.0) |
| Tier 2 | Aggregate Statistics | ✅ Public (Zenodo DOI) |
| Tier 3 | Anonymized Sample Traces | 🔒 Under NDA (Data Use Agreement) |
| Tier 4 | Raw Production Data | 🔒 Proprietary |

For academic verification access: wangguigui@correctover.com

Full policy: [DATA-PROTECTION-STATEMENT.md](DATA-PROTECTION-STATEMENT.md)
Data Use Agreement template: [DATA-USE-AGREEMENT.md](DATA-USE-AGREEMENT.md)

## Empirical Foundation

### Production Dataset (Core Asset)
- **20,071 production traces** from real agent workflows
- **13 LLM providers** covered
- **30-day observation window**

### Extended Test Suite (Synthetic Augmentation)
- **29,929 synthetic test cases** generated to model rare failure modes
- **50,000 total** when combined with production data
- **Purpose**: Statistical significance for edge cases not present in production at sufficient volume

### Key Findings
- **Single-fault recovery rate**: 97.4% (from production-derived test scenarios)
- **Compound-fault recovery rate**: ~72% (from synthetic stress testing)
- **325 distinct failure modes** identified through production observation and systematic testing

**Critical Distinction**: The 20,071 production traces are the empirical foundation. The 29,929 synthetic cases are a test suite designed to stress-test the system. Claims about production behavior reference the 20K dataset only.

## Seeking Adopters

This specification is seeking real-world adoption and validation. If you are:
- Building production agent systems
- Experiencing runtime reliability issues
- Interested in runtime verification

We welcome:
- **Feedback** on the specification
- **Integration attempts** in your frameworks
- **Bug reports** and edge cases
- **Collaboration** on validation studies

Contact: wangguigui@correctover.com

## License

All specification documents are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

---

*Correctover Standards Team | [correctover.com](https://correctover.com)*
