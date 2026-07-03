# Ecosystem Adoption Map

This document tracks the adoption and independent verification of the Correctover Agent Conformance Specification across major agent frameworks and by independent contributors.

## Framework Adoption Status

### Microsoft AutoGen
- **Status**: Partial Implementation
- **Key Issues**: 
  - [#7525](https://github.com/microsoft/autogen/issues/7525): MoltBridge trust verification — Required(τ)/Supported(τ) framework adopted
  - [#7353](https://github.com/microsoft/autogen/issues/7353): AAR encryption governance — Four-axis verification implemented
- **Conformance Level**: Level 1 (Partial)
- **Notes**: Core concepts (Required/Supported, Recovery Classification) have been adopted by community contributors

### CrewAI
- **Status**: In Progress
- **Key References**:
  - [PR #6432](https://github.com/crewAIInc/crewAI/pull/6432): Reliability improvements
  - [#4877](https://github.com/crewAIInc/crewAI/issues/4877): Agent recovery mechanisms
- **Conformance Level**: Level 0 (Discussion)
- **Notes**: Framework architecture aligns with conformance requirements; formal verification pending

### LangGraph (LangChain)
- **Status**: Under Discussion
- **Key References**:
  - [#7303](https://github.com/langchain-ai/langgraph/issues/7303): Trust-gated checkpoints and governance nodes
- **Conformance Level**: Level 0 (Discussion)
- **Notes**: Partial trust decomposition maps to capability-based authorization (Axis 1)

### Semantic Kernel (Microsoft)
- **Status**: Under Discussion
- **Key References**:
  - [#13957](https://github.com/microsoft/semantic-kernel/issues/13957): Compliance-as-Code plugin proposal
- **Conformance Level**: Level 0 (Discussion)
- **Notes**: Compliance mapping to EU AI Act Articles 9, 13, 14, 15 under review

### PHI-OMEGA Runtime
- **Status**: Aligned
- **Key References**:
  - [#1](https://github.com/Tuttotorna/PHI-OMEGA-RUNTIME/issues/1): Conformance verification request
- **Conformance Level**: Level 1 (Partial)
- **Notes**: Runtime architecture implements Required(τ)/Supported(τ) split; formal conformance assessment in progress

## Independent Verification

The following independent implementations have verified the portability and reproducibility of the conformance specification:

### giskard09/argentum-core
- **Repository**: [giskard09/argentum-core](https://github.com/giskard09/argentum-core)
- **File**: `examples/conformance/composed-decision-chain-recompute/verify.py`
- **Contribution**: Complete four-axis conformance verification (Axis 4 fork-matrix implementation)
- **Verification**: Byte-identical output with babyblueviper1/preaction-governance-conformance
- **Significance**: Demonstrates cross-implementation portability of the standard

### babyblueviper1/preaction-governance-conformance
- **Repository**: [babyblueviper1/preaction-governance-conformance](https://github.com/babyblueviper1/preaction-governance-conformance)
- **Contribution**: Three-axis verification framework (Admission / Recompute / Chain-Fork)
- **Verification**: Cold-cloned giskard09 repository, executed verify.py, four-axis PASS with byte-level consistency
- **Significance**: Two independent implementations from same specification yield identical output — proves reproducibility

### pshkv (AutoGen community)
- **Contribution**: Required(τ)/Supported(τ) framework formulation
- **Adoption**: Framework adopted into SPEC-v1.md Section 4 (Conformance Model)
- **Significance**: Community-driven refinement of conformance model

### Tuttotorna (PHI-OMEGA)
- **Contribution**: Runtime architecture aligned with conformance requirements
- **Role**: Requested @Correctover for conformance verification authority
- **Significance**: Recognition of Correctover as conformance verification authority

### kevinkaylie (LangGraph community)
- **Contribution**: Mapping of partial trust decomposition to capability-based authorization
- **Reference**: [langgraph #7303](https://github.com/langchain-ai/langgraph/issues/7303)
- **Significance**: Extension of Axis 1 (Admission Control) to authorization frameworks

## Standard Contributors

The following individuals have contributed to the development, verification, or extension of the Correctover Agent Conformance Specification through independent implementation and community discussion:

| Contributor | Contribution | Verification |
|-------------|--------------|--------------|
| **pshkv** | Required(τ)/Supported(τ) framework | Adopted into SPEC-v1.md |
| **giskard09** | Axis 4 fork-matrix implementation | argentum-core/verify.py |
| **babyblueviper1** | Three-axis framework + cross-validation | preaction-governance-conformance |
| **Tuttotorna** | Runtime alignment + verification request | PHI-OMEGA-RUNTIME #1 |
| **kevinkaylie** | Trust decomposition → authorization mapping | langgraph #7303 |

## Adoption Criteria

For a framework to be listed as "Partial" or higher:
1. **Level 1 (Partial)**: Core concepts (Required/Supported, Recovery Classification) implemented in at least one module
2. **Level 2 (Substantial)**: Four-axis verification framework implemented and tested
3. **Level 3 (Full)**: Complete conformance assessment completed and documented

## How to Contribute

If you have implemented or verified conformance with the Correctover Agent Conformance Specification:
1. Open an issue in this repository with your implementation details
2. Provide reproducible test cases or verification code
3. Your contribution will be reviewed and added to this map

---

*Last updated: 2026-07-03*
