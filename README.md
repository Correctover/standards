# Correctover Agent Conformance Standards

> Conformance specification for agent runtime reliability.
> Grounded in 20,071 production traces + 29,929 synthetic test cases.

## What This Is

This repository documents a **conformance specification** for agent runtime reliability, developed through empirical analysis of production agent workflows. The specification provides a framework for evaluating and improving agent runtime governance.

**Purpose**: Provide a reference implementation for runtime verification that frameworks and practitioners can adopt, adapt, or critique.

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
| PHI-OMEGA | [#1](https://github.com/Tuttotorna/PHI-OMEGA-RUNTIME/issues/1) | Research collaboration |

**Note**: These are community discussions, not official adoptions. The specification is seeking feedback and real-world validation.

## Independent Verification

The Correctover conformance specification has been independently verified by third-party implementers:

### giskard09/argentum-core — Axis 4 Fork-Matrix Verification
- **Repository**: [giskard09/argentum-core](https://github.com/giskard09/argentum-core)
- **File**: `examples/conformance/composed-decision-chain-recompute/verify.py`
- **Implementation**: Complete four-axis conformance verification
- **Cross-validation**: Byte-identical comparison with babyblueviper1/preaction-governance-conformance
- **Significance**: Demonstrates standard portability across independent implementations

### babyblueviper1/preaction-governance-conformance — Transition Verification Completeness
- **Repository**: [babyblueviper1/preaction-governance-conformance](https://github.com/babyblueviper1/preaction-governance-conformance)
- **Context**: Production trading system governance — independent mapping of Correctover conformance model to real execution environment
- **Key Discovery**: Independently identified authorization gap where $12 review position silently linked to $50 execution via incomplete string matching (coin+side only, no size validation) — precise instantiation of Correctover case_2 fault class
- **Implementation**: `link_mode` field (`size_matched` / `legacy_coin_side_only`) — first staged implementation of Correctover Required(τ)⊆Supported(τ) transition verification
- **Architecture**: 3-layer verification classification (VERIFIED / LEGACY_LINKED / REFUSED_LINK) enabling backward-compatible adoption
- **Significance**: Demonstrates that Correctover conformance framework detects real production failures that escape conventional validation. Staged adoption model validates progressive compliance strategy.

## Digital Object Identifiers (DOIs)

All Correctover standards documents and diagnostic reports are archived with permanent DOIs for citation:

| Document | DOI | Status |
|----------|-----|--------|
| **SPEC-v1.md** (Conformance Specification) | `10.5281/zenodo.21166867` | Active |
| **DIAG-001** (AutoGen Trust Governance Gap Analysis) | `10.5281/zenodo.21166867` | Active |
| **DIAG-002** (MCP Protocol Security Gap Analysis) | `10.5281/zenodo.21166867` | Active |

> DOIs are active and resolve to published Zenodo records. All documents are versioned and immutable.

## Empirical Foundation

This specification is grounded in empirical analysis:

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

**Critical Distinction**: The 20,071 production traces are the empirical foundation. The 29,929 synthetic cases are a test suite designed to stress-test the system. Claims about production behavior should reference the 20K dataset only.

**Data Clarity**: See [METHODOLOGY-aggregated-statistics.md](diagnostics/METHODOLOGY-aggregated-statistics.md) for detailed breakdown of production vs. synthetic data.

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
