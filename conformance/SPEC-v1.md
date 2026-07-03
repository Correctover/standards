# Conformance Protocol v0.1

**Document Status**: Draft Specification  
**Version**: 0.1.0  
**Date**: 2026-07-03  
**Classification**: Normative Reference  

---

## 1. Scope

This specification defines the conformance requirements for agentic runtime systems operating under fault conditions. It establishes verifiable criteria for:

- Admission control under partial trust
- Deterministic recomputability of decisions
- Chain integrity across agent boundaries
- Fork detection in distributed execution contexts

This protocol is runtime-agnostic. Conformance is defined by observable behavior, not implementation details.

---

## 2. Normative References

- **RFC 2119**: Key words for use in RFCs to Indicate Requirement Levels
- **EU AI Act Article 15**: Robustness and resilience requirements for high-risk AI systems
- **NIST AI RMF 1.0**: Measure and Govern functions

---

## 3. Terminology

| Term | Definition |
|------|------------|
| **Transition τ** | An atomic state change in agent execution (tool invocation, message pass, decision commit) |
| **Required(τ)** | Non-negotiable preconditions for safe execution of transition τ |
| **Supported(τ)** | Optional enhancements that improve reliability but are not blocking |
| **recoverability_class** | Taxonomic classification of fault types by recovery semantics |
| **content_hash** | Cryptographic digest of decision inputs (prompt, context, tool schema) |
| **head_hash** | Cryptographic digest of decision output + prior state |
| **prior_head** | The head_hash of the immediately preceding transition in the chain |
| **sequence_position** | Monotonically increasing index of transition within execution trace |

---

## 4. Conformance Model

### 4.1 Transition Declaration

Every transition τ MUST declare:

```
Required(τ) = { preconditions that MUST be satisfied }
Supported(τ) = { optional enhancements that SHOULD be satisfied }
```

**Violation semantics**:
- Required(τ) violation → Transition MUST be denied (hard fail)
- Supported(τ) violation → Transition MAY proceed with degraded confidence (soft fail)

### 4.2 Recovery Classification

All faults MUST be classified by `recoverability_class`:

| Class | Semantics | Recovery Strategy |
|-------|-----------|-------------------|
| `TRANSIENT_RETRY` | Fault resolves with backoff/retry | Exponential backoff with jitter, max 3 attempts |
| `STRUCTURAL_BLOCK` | Fault requires architectural change | Halt and escalate; no automatic recovery |
| `CASCADE_HALT` | Fault propagates across agent boundaries | Systemic intervention required; all dependent transitions suspended |

---

## 5. Verification Axes

Conformance is evaluated across four orthogonal axes. A system MUST satisfy all four to claim conformance.

### 5.1 Axis 1: Admission Control

**Property**: Signer independence under partial trust.

**Formal requirement**:
```
∀ tool_invocation t:
  signer(t) ∉ trusted_set(context) → DENY(t)
```

**Verification method**:
- Construct a tool invocation where the signer identity is not in the current trusted set
- Observe that the invocation is denied (not queued, not retried, not silently dropped)
- Confirm that the denial is logged with structured metadata (signer, context, timestamp)

**Pass criterion**: 100% of unauthorized invocations are denied within one execution cycle.

---

### 5.2 Axis 2: Deterministic Recomputation

**Property**: Verdict must re-derive from controls.

**Formal requirement**:
```
∀ decision d at position n:
  verdict(d) = f(inputs(d), context(d), policy(d))
  
  ∀ replay r of d:
    f(inputs(r), context(r), policy(r)) = verdict(d)
    IF inputs(r) = inputs(d) ∧ context(r) = context(d) ∧ policy(r) = policy(d)
```

**Verification method**:
- Record a decision d with its inputs, context, and policy
- Replay the exact same inputs, context, and policy
- Observe that the verdict is byte-identical

**Pass criterion**: 100% of replays produce identical verdicts.

**Failure mode**: Non-deterministic verdicts (e.g., due to floating-point variance, clock skew, or uninitialized state) indicate conformance violation.

---

### 5.3 Axis 3: Chain Fork Detection

**Property**: Detectable conflict at same sequence position.

**Formal requirement**:
```
∀ transitions t₁, t₂:
  IF sequence_position(t₁) = sequence_position(t₂)
  ∧ prior_head(t₁) ≠ prior_head(t₂)
  ⇒ FORK_DETECTED(t₁, t₂)
```

**Verification method**:
- Construct two execution traces that share the same sequence position but diverge in prior_head
- Observe that the system detects and logs the fork
- Confirm that the fork is not silently merged or ignored

**Pass criterion**: 100% of forks at the same sequence position are detected and logged.

**Failure mode**: Silent fork merging (e.g., "last write wins" without detection) indicates conformance violation.

---

### 5.4 Axis 4: Fork-Matrix Properties

**Property**: Four structural invariants of chain integrity.

#### 5.4.1 Determinism

```
∀ transitions t₁, t₂:
  content_hash(t₁) = content_hash(t₂)
  ∧ prior_head(t₁) = prior_head(t₂)
  ⇒ head_hash(t₁) = head_hash(t₂)
```

**Interpretation**: Same inputs + same prior state → same output. This is the foundational invariant of chain integrity.

#### 5.4.2 Branch Fork

```
∀ transitions t₁, t₂:
  sequence_position(t₁) = sequence_position(t₂)
  ∧ prior_head(t₁) ≠ prior_head(t₂)
  ⇒ head_hash(t₁) ≠ head_hash(t₂)
  ∧ FORK_DETECTED(t₁, t₂) = true
```

**Interpretation**: Same position but different prior state → detectable divergence. This ensures that chain forks are not silently merged.

#### 5.4.3 Anti-Replay

```
∀ transitions t₁, t₂:
  payload(t₁) = payload(t₂)
  ∧ chain_context(t₁) ≠ chain_context(t₂)
  ⇒ head_hash(t₁) ≠ head_hash(t₂)
```

**Interpretation**: Same payload but different chain context → different output. This prevents replay attacks where an attacker reuses a valid decision in a different chain context.

#### 5.4.4 Structured Exposure

```
∀ verifier v:
  v(t) → { sequence_position, head_hash, prior_head, conflict_detected }
```

**Interpretation**: Verifier MUST expose structured metadata, not a bare boolean. This enables downstream systems to make informed decisions about chain integrity.
---

## 6. Conformance Levels

| Level | Requirements | Use Case |
|-------|--------------|----------|
| **Level 0** | No conformance | Testing/development |
| **Level 1** | Axis 1 + Axis 2 | Single-agent systems |
| **Level 2** | Axis 1 + Axis 2 + Axis 3 | Multi-agent systems without distribution |
| **Level 3** | Axis 1 + Axis 2 + Axis 3 + Axis 4 | Distributed agent systems with cryptographic chaining |

---

## 7. Verification Procedure

### 7.1 Test Fixture Requirements

A conformance test fixture MUST provide:

1. **Transition trace**: Sequence of ≕10 transitions with full metadata
2. **Fault injection**: At least one fault of each `recoverability_class`
3. **Replay capability**: Ability to re-execute transitions with identical inputs
4. **Fork construction**: Ability to create divergent traces at specified positions

### 7.2 Pass/Fail Criteria
- **Pass**: All four axes satisfy their respective pass criteria
- **Fail**: Any axis fails to satisfy its pass criteria
- **Partial**: Some axes pass, others fail (must specify which)

### 7.3 Reporting

Conformance reports MUST include:

- Test fixture identifier
- Date and time of verification
- Axis-by-axis pass/fail status
- Failure mode description (if any axis fails)
- Verifier identity (human or automated)

---

## 8. Data-Driven Validation Foundation

The conformance criteria defined in this specification are grounded in a rigorous empirical baseline, ensuring industrial-grade reliability of the standard.

### 8.1 Production Real-world Experience

- **Scope**: 20,206 verified API calls extracted directly from production agent systems
- **Definition**: Ground truth observations from real-world execution environments
- **Value**: Ensures this specification accounts for transient latency, boundary-value jitter, and unexpected runtime states that only manifest under production load

This dataset identifies 420 distinct fault types, providing the empirical foundation for the `recoverability_class` taxonomy (Section 4.2).

### 8.2 Conformance Test Suite

- **Scope**: 50,000 synthetic test cases generated via statistical modeling of production sequences and state transitions
- **Definition**: Full-space coverage test suite derived from production distribution characteristics
- **Value**: Systematically maps compatibility blind spots across the current ecosystem

Through conformance scanning of these 50,000 test cases, we identified 19,251 potential failure paths that remain uncovered by current mainstream implementations (38.5% coverage gap).

### 8.3 Key Empirical Findings

| Metric | Value | Interpretation |
|-------|-------|----------------|
| Single-fault recovery rate | 97.4% | Baseline reliability under isolated faults |
| Composite-fault recovery rate | ~72% | Degradation under multi-fault scenarios |
| Coverage gap (alternative frameworks) | 38.5% | 19,251 uncovered failure paths |

The 25.4% gap between single-fault and composite-fault recovery rates (Section 5.2 Axis 2) demonstrates the necessity of systematic conformance testing beyond ad-hoc fault injection.

Full statistical breakdown: [conformance-runner/summary.md](conformance-runner/summary.md)

---

## 9. Governance

This specification is maintained by Correctover as a community resource. Contributions are welcome via:

- Edge case submissions (via GitHub Issues)
- Implementation feedback (via GitHub Discussions)
- Conformance test fixture contributions (via Pull Requests)

The specification follows a conservative evolution policy: breaking changes require 6-month deprecation notice.

---

## 10. License

This specification is licensed under CC BY 4.0. Implementation code is licensed under Apache 2.0.

---

*Correctover: Because failover switches. Correctover verifies.*
