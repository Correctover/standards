# Correctover Conformance Architecture (CCA)

**Document Status**: Canonical Specification  
**Version**: 1.0.0  
**Date**: 2026-07-06  
**Classification**: Normative Reference  
**License**: CC BY 4.0  
**DOI**: [10.5281/zenodo.21166867](https://doi.org/10.5281/zenodo.21166867)

---

## Abstract

This document defines the canonical architecture of the Correctover framework — a five-component system for runtime verification of agentic AI systems. Each component is independently necessary and jointly sufficient for production-grade agent governance. Third-party implementations that adopt subsets or extensions of this architecture are classified as **Community Implementations**; the architecture defined herein is the **Reference Implementation**.

---

## 1. Scope and Purpose

The Correctover framework was developed through empirical analysis of 20,071 production agent traces (spanning 13 LLM providers) and formalized through peer-reviewed academic research. This document serves as the single authoritative reference for:

1. The **five-component architecture** that constitutes the Correctover framework
2. The **inter-component invariants** that constrain valid implementations
3. The **conformance criteria** by which implementations are evaluated

All formal definitions, mathematical notation, and architectural constraints in this document are anchored by blockchain timestamps (OpenTimestamps, Bitcoin chain) and Zenodo DOI records dated no later than 2026-07-05.

---

## 2. Architecture Overview

The Correctover framework comprises five components arranged in a strict dependency hierarchy:

```
┌─────────────────────────────────────────────────────────────────┐
│                    CORRECTOVER ARCHITECTURE                      │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Component 5: Conformance Protocol                        │   │
│  │  (Four-axis verification framework)                       │   │
│  └─────────────────────────┬────────────────────────────────┘   │
│                            │ evaluates                           │
│  ┌─────────────────────────▼────────────────────────────────┐   │
│  │  Component 3: Required(τ) ⊆ Supported(τ) Invariant       │   │
│  │  (Formal conformance criterion)                           │   │
│  └──────┬──────────────────────────────────┬────────────────┘   │
│         │ constrains                        │ generates            │
│  ┌──────▼──────────────┐    ┌──────────────▼─────────────────┐  │
│  │  Component 1:        │    │  Component 4: Receipt Schema   │  │
│  │  Runtime Observatory │    │  (Cryptographic chain integrity)│  │
│  │  (Execution capture) │    │                                │  │
│  └──────┬──────────────┘    └────────────────────────────────┘  │
│         │ feeds                                                   │
│  ┌──────▼──────────────────────────────────────────────────────┐│
│  │  Component 2: Fault Taxonomy                                 ││
│  │  (517-type runtime failure classification)                   ││
│  └──────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

### Dependency Invariant

> No component may be implemented or evaluated independently of its dependencies. A system claiming Correctover conformance must satisfy all five components and their inter-component invariants (Section 6).

---

## 3. Component Definitions

### 3.1 Component 1: Runtime Observatory

**Purpose**: Capture, structure, and preserve agent execution traces with cryptographic integrity.

**Formal Definition**:
A Runtime Observatory is a monitoring layer that, for every agent transition τ, produces an immutable observation record:

```
Observatory(τ) → { 
    transition_id, 
    agent_identity, 
    tool_invocation, 
    input_context_hash, 
    output_context_hash, 
    timestamp, 
    prior_head_hash 
}
```

**Key Properties**:
- **Immutability**: Once recorded, observation records cannot be retroactively modified without detection
- **Completeness**: Every transition in the agent execution lifecycle is captured
- **Cryptographic chaining**: Each record references the prior record's hash, forming a tamper-evident chain

**First defined**: NeuralBridge Recovery Engine internal documentation, January 2026 (blockchain-anchored SHA-256: `5fe5b905070b4b92a475fbf3896aa6dfc5cab269e2b428bc6c7f36793bbd68da`)

---

### 3.2 Component 2: Fault Taxonomy

**Purpose**: Classify all agent runtime failures into a structured, machine-actionable taxonomy.

**Formal Definition**:
A Fault Taxonomy is a hierarchical classification system mapping each observed failure to a tuple:

```
Fault(f) → { 
    fault_class ∈ {8 primary classes}, 
    fault_type ∈ {517 distinct types}, 
    recoverability_class ∈ {TRANSIENT_RETRY, STRUCTURAL_BLOCK, CASCADE_HALT},
    severity ∈ {CRITICAL, HIGH, MEDIUM, LOW},
    recovery_strategy_set ⊆ {available strategies}
}
```

**Eight Primary Classes**:
1. Authentication & Authorization failures
2. Rate Limiting & Throttling
3. Model & Endpoint failures
4. Data Validation & Schema failures
5. Context & Memory failures
6. Timeout & Latency failures
7. Cost & Quota failures
8. Agent Runtime failures (cascade, orchestration, tool binding)

**Current version**: 517 distinct fault types across 8 classes and 3 recoverability classes. The taxonomy evolves through the data flywheel mechanism — each new production fault observation may introduce new types, but never modifies existing type definitions (append-only invariant).

**First defined**: Fault Taxonomy v1.0 internal documentation, May 6, 2026 (blockchain-anchored SHA-256: `8ee3e6618603f289a262c529b18570e6d36148604128f6c77bd0af86809662e2`)

---

### 3.3 Component 3: Required(τ) ⊆ Supported(τ) Invariant

**Purpose**: Define the formal conformance criterion for agent transition safety.

**Formal Definition**:
For every agent transition τ:

```
Required(τ) = { preconditions that MUST be satisfied for safe execution }
Supported(τ) = { optional enhancements that improve reliability but are not blocking }

Conformance Criterion: Valid(τ) ⇔ Required(τ) ⊆ Supported(τ)
```

**Violation Semantics**:
- If ∃ r ∈ Required(τ) such that r ∉ Supported(τ): transition τ **MUST be denied** (hard fail, fail-closed)
- If Required(τ) ⊆ Supported(τ) but ∃ s ∈ Supported(τ) that is degraded: transition τ **MAY proceed** with reduced confidence (soft fail)

**Structural Necessity**:
This invariant is not an arbitrary design choice. Any system that enforces transition authority — regardless of framework, language, or deployment context — must satisfy this invariant by logical necessity. Independent implementations across LangGraph (ApprovalNode HITL), AutoGen (conformance fixtures), and CrewAI (GuardrailProvider interface) have converged to this same structure, demonstrating its status as an objective design principle rather than a subjective preference.

**First public use**: PHI-OMEGA-RUNTIME#1, Comment ID 4864466750, July 2, 2026, 09:57 UTC

---

### 3.4 Component 4: Receipt Schema

**Purpose**: Provide cryptographic verification of decision chain integrity across agent boundaries.

**Formal Definition**:
A Receipt Schema defines the structure of verifiable decision records:

```
Receipt(r) → {
    sequence_position: ℕ,
    content_hash: SHA-256(input_payload),
    head_hash: SHA-256(content_hash ⊕ prior_head ⊕ decision_output),
    prior_head: head_hash of immediately preceding receipt,
    verifier_metadata: { axis_results, conflict_flags, timestamp }
}
```

**Four Structural Invariants**:

1. **Determinism**: Same inputs + same prior state → same output
2. **Branch Fork Detection**: Same position + different prior state → detectable divergence
3. **Anti-Replay**: Same payload + different chain context → different output
4. **Structured Exposure**: Verifiers must expose structured metadata, not bare booleans

**Relationship to Component 3**: The Receipt Schema operationalizes the Required ⊆ Supported invariant by providing the cryptographic infrastructure needed to verify that Required preconditions were actually checked against Supported capabilities at each transition.

---

### 3.5 Component 5: Conformance Protocol

**Purpose**: Define the multi-axis verification framework for evaluating implementation compliance.

**Formal Definition**:
The Conformance Protocol evaluates implementations across four orthogonal axes:

| Axis | Property | Verification Criterion |
|------|----------|----------------------|
| **Axis 1** | Admission Control | ∀ unauthorized invocations t: DENY(t) within one execution cycle |
| **Axis 2** | Deterministic Recomputation | ∀ replay r of decision d: verdict(r) = verdict(d) when inputs match |
| **Axis 3** | Chain Fork Detection | ∀ divergent traces at same position: FORK_DETECTED = true |
| **Axis 4** | Fork-Matrix Properties | All four structural invariants of Receipt Schema satisfied |

**Conformance Levels**:

| Level | Requirements | Applicability |
|-------|-------------|---------------|
| Level 0 | No conformance | Development/testing |
| Level 1 | Axis 1 + 2 | Single-agent systems |
| Level 2 | Axis 1 + 2 + 3 | Multi-agent systems |
| Level 3 | Axis 1 + 2 + 3 + 4 | Distributed agent systems with cryptographic chaining |

**Full specification**: [conformance/SPEC-v1.md](conformance/SPEC-v1.md)

---

## 4. Inter-Component Invariants

The following invariants constrain valid implementations of the Correctover architecture. Violation of any invariant renders an implementation non-conformant regardless of individual component correctness.

### 4.1 Observation-Taxonomy Binding

> The Fault Taxonomy (Component 2) must classify failures observed exclusively through the Runtime Observatory (Component 1). Taxonomy entries derived from sources other than direct observation are not part of this framework.

### 4.2 Invariant-Receipt Correspondence

> The Required ⊆ Supported invariant (Component 3) must be evaluated using receipt chains produced by the Receipt Schema (Component 4). Verification without cryptographic receipt chains constitutes heuristic assessment, not conformance verification.

### 4.3 Protocol-Component Totality

> The Conformance Protocol (Component 5) evaluates all four other components. An implementation that satisfies only a subset of components may not claim partial conformance — it must specify exactly which axes pass and which fail.

---

## 5. Implementation Classification

### 5.1 Reference Implementation

The **Reference Implementation** is the authoritative instantiation of all five components as defined by Correctover. It is the standard against which all other implementations are measured.

| Component | Reference Status | Access |
|-----------|-----------------|--------|
| Runtime Observatory | Proprietary (closed-source verifier) | Under NDA |
| Fault Taxonomy | 517 types, publicly documented class structure | Proprietary full dataset |
| Required ⊆ Supported | Fully specified in this document and SPEC-v1.md | Public (CC BY 4.0) |
| Receipt Schema | Fully specified in this document and SPEC-v1.md | Public (CC BY 4.0) |
| Conformance Protocol | Fully specified in SPEC-v1.md | Public (CC BY 4.0) |

### 5.2 Community Implementations

Third-party implementations that adopt subsets of this architecture are recognized as **Community Implementations**. These implementations:

- May implement any subset of the five components
- Are encouraged to reference this specification for verification test suites
- Must acknowledge the source architecture when claiming conformance
- Are independently maintained and not governed by Correctover

**Known Community Implementations**:

| Implementation | Components Adopted | Maintainer | Repository |
|---------------|-------------------|------------|-----------|
| preaction-governance-conformance | Components 3, 5 (partial) | babyblueviper1 | GitHub |
| PHI-OMEGA-RUNTIME | Components 1, 3, 4 | Tuttotorna (M. Brighindi) | GitHub |
| argentum-core | Components 4, 5 | giskard09 | GitHub |
| ibex-agent-verification | Components 3, 4 (partial) | safal207 | GitHub |
| GuardrailProvider PR #6411 | Components 3, 5 | Correctover | crewAI PR |

> Community Implementations are valuable adoption indicators. Their existence demonstrates the structural necessity of the Correctover architecture. However, the formal definition of the conformance standard resides exclusively in this specification and its associated DOI-archived documents.

---

## 6. Naming and Terminology

### 6.1 Canonical Terminology

The following terms are defined by this specification and must be used exactly as defined when referencing the Correctover framework:

| Term | Definition | First Use |
|------|-----------|-----------|
| Runtime Observatory | Component 1: execution capture layer | NeuralBridge internal, Jan 2026 |
| Fault Taxonomy | Component 2: failure classification system | Fault Taxonomy v1.0, May 2026 |
| Required(τ) ⊆ Supported(τ) | Component 3: formal conformance invariant | PHI-OMEGA-RUNTIME#1, Jul 2 2026 |
| Receipt Schema | Component 4: cryptographic chain integrity | SPEC-v1.md, Jul 3 2026 |
| Conformance Protocol | Component 5: four-axis verification | SPEC-v1.md, Jul 3 2026 |
| Valid(τ) | Transition satisfies conformance criterion | PHI-OMEGA-RUNTIME#1, Jul 2 2026 |
| recoverability_class | Fault recovery classification | SPEC-v1.md, Jul 3 2026 |

### 6.2 Derivative Terminology

Terms introduced by Community Implementations that derive from the canonical terminology are recognized as valid extensions:

- "PHI-OMEGA form" (Tuttotorna) → refers to Component 3 as applied in PHI-OMEGA-RUNTIME
- "conformance registry" (babyblueviper1) → refers to a grading system built on Component 5
- "transition authority" (safal207) → refers to Component 3 enforcement in ibex-agent-verification

These derivative terms are acceptable in community discourse but must trace their definitions back to the canonical specification.

---

## 7. Provenance and Archival

### 7.1 Blockchain Anchors

All core definitions are anchored via OpenTimestamps to the Bitcoin blockchain:

| Asset | SHA-256 | Anchor Date |
|-------|---------|-------------|
| IP-Declaration-Correctover-20260705.md | `cbdbc04828eaee568896d7c4339ed2e093e7c383cba043e5ead7f26e646f5ea0` | 2026-07-05 |
| fault-taxonomy-v1.md | `8ee3e6618603f289a262c529b18570e6d36148604128f6c77bd0af86809662e2` | 2026-07-05 |
| decision-theory-framework.md | `f746774758534dd3ff4dcd5034217e4e6008f13a6cfa75d31113b96a1ecc304d` | 2026-07-05 |
| tech-whitepaper-v1.pdf | `82e82ab305584776082f4356993055068060eae8926318893a1690a7e7` | 2026-07-05 |
| NeuralBridge Complete Knowledge Backup | `5fe5b905070b4b92a475fbf3896aa6dfc5cab269e2b428bc6c7f36793bbd68da` | 2026-05-26 |

### 7.2 Academic DOIs

| Document | DOI | Registration Date |
|----------|-----|-------------------|
| SPEC-v1.md + Diagnostic Reports | 10.5281/zenodo.21166867 | 2026-07-04 |
| Benchmark Methodology + DIAG-002 v2 | 10.5281/zenodo.21184180 | 2026-07-04 |
| Governance Framework v1.0 | 10.5281/zenodo.21191484 | 2026-07-04 |

### 7.3 Earliest Public Records

| Concept | First Public Record | Date |
|---------|-------------------|------|
| Required(τ) ⊆ Supported(τ) | PHI-OMEGA-RUNTIME#1, Comment ID 4864466750 | 2026-07-02 09:57 UTC |
| Four-object conformance model | PHI-OMEGA-RUNTIME#1 | 2026-07-02 |
| Conformance Protocol (four-axis) | standards/conformance/SPEC-v1.md | 2026-07-03 |
| Fault Taxonomy (8-class) | Internal documentation | 2026-01-25 |
| Fault Taxonomy (517 types) | fault-taxonomy-v1.md | 2026-05-06 |

---

## 8. Governance

This specification is maintained by Correctover Research. The evolution policy follows these principles:

1. **Breaking changes** to the five-component architecture require 6-month deprecation notice
2. **Component additions** require formal academic publication
3. **Community Implementations** are encouraged to submit compatibility reports
4. **Specification updates** are versioned using Semantic Versioning (SemVer 2.0)

Contact: wangguigui@correctover.com

---

## 9. License

This document and all associated specification files are licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Implementation code repositories referenced herein are licensed under their respective terms.

---

*Correctover Conformance Architecture v1.0.0 | Canonical Specification*  
*DOI: 10.5281/zenodo.21166867*
