# Correctover Conformance Specification (CCS)

**Version 1.0**

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.21234580.svg)](https://doi.org/10.5281/zenodo.21234580)
![License](https://img.shields.io/badge/license-CC%20BY--NC--SA%204.0-green.svg)

---

---

## Executive Summary

CCS is the industry standard for verifying LLM outputs in agentic systems. It provides a **provider-agnostic, framework-agnostic** conformance protocol that ensures AI agents produce reliable, compliant, and cost-controlled outputs.

**For Enterprise Adoption**: CCS enables organizations to validate any LLM provider (OpenAI, Anthropic, Google, Azure, etc.) against a unified standard, reducing vendor lock-in and ensuring production-grade reliability.

**For Framework Integration**: CCS provides drop-in adapters for CrewAI, AutoGen, and LangGraph, enabling immediate compliance verification without architectural changes.

---

## Why CCS Matters

### The Problem
LLM outputs are non-deterministic. Agents that rely on LLM decisions without validation face:
- **Compliance violations** (HIPAA, GDPR, SOC2)
- **Cost overruns** (token waste, runaway loops)
- **Security vulnerabilities** (injection attacks, data leakage)
- **Unreliable behavior** (hallucinations, format drift)

### The Solution
CCS defines a **5-component verification architecture** that validates every LLM output against:
1. **Required(τ)** - Fields that must be present
2. **Supported(τ)** - Fields that can be present
3. **Forbidden(τ)** - Fields that must not be present
4. **Integrity constraints** - HMAC-based output binding
5. **Cost controls** - Token budget enforcement

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Agent Framework                           │
│       (CrewAI / AutoGen / LangGraph / Custom)              │
└────────────────────┬────────────────────────────────────────┘
                     │ LLM Output
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              CCS Verification Layer                          │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  Schema      │  │  Integrity   │  │  Cost        │     │
│  │  Validator   │  │  Checker     │  │  Controller  │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │  Compliance  │  │  Audit       │                        │
│  │  Engine      │  │  Logger      │                        │
│  └──────────────┘  └──────────────┘                        │
└────────────────────┬────────────────────────────────────────┘
                     │ Validated Output
                     ▼
┌─────────────────────────────────────────────────────────────┐
│              LLM Provider                                    │
│  (OpenAI / Anthropic / Google / Azure / Local / Custom)    │
└─────────────────────────────────────────────────────────────┘
```

---

## The 5 Components

### 1. CCS-Core
Core validation engine. Validates LLM outputs against Required/Supported/Forbidden schemas.

**Use Case**: Ensure every LLM response contains required fields (e.g., `decision`, `confidence`, `trace_id`) and rejects malformed outputs.

### 2. CCS-Compliance
Regulatory compliance engine. Maps LLM outputs to compliance frameworks (HIPAA, GDPR, SOC2).

**Use Case**: Healthcare agents must not output PHI without authorization. CCS-Compliance validates PII redaction.

### 3. CCS-Cost-Flow-Analysis
Token budget enforcement. Prevents cost overruns from runaway agents.

**Use Case**: Cap daily token spend at $100. CCS-Cost-Flow-Analysis blocks requests that would exceed the budget.

### 4. CCS-Integrity
Output integrity verification. HMAC-based binding prevents tampering.

**Use Case**: Financial agents must produce auditable outputs. CCS-Integrity provides cryptographic proof of output authenticity.

### 5. CCS-Audit
Comprehensive audit logging. Every validation decision is logged for traceability.

**Use Case**: SOC2 compliance requires audit trails. CCS-Audit provides immutable logs of all validation decisions.

---

## Integration Guide

### Quick Start (Python)

```python
from correctover import CCSValidator

# Initialize validator
validator = CCSValidator(
    required_fields=["decision", "confidence", "trace_id"],
    supported_fields=["reasoning", "metadata"],
    forbidden_fields=["pii", "credentials"]
)

# Validate LLM output
llm_output = {
    "decision": "approve",
    "confidence": 0.95,
    "trace_id": "abc-123",
    "reasoning": "Customer meets criteria"
}

result = validator.validate(llm_output)
if result.is_valid:
    print("✓ Output is compliant")
else:
    print(f"✗ Validation failed: {result.errors}")
```

### Framework Adapters

#### CrewAI
```python
from correctover.adapters import CrewAIAdapter

adapter = CrewAIAdapter(validator)
crew = adapter.wrap_crew(your_crew)
result = crew.kickoff()
```

#### AutoGen
```python
from correctover.adapters import AutoGenAdapter

adapter = AutoGenAdapter(validator)
agent = adapter.wrap_agent(your_agent)
result = agent.run(task)
```

#### LangGraph
```python
from correctover.adapters import LangGraphAdapter

adapter = LangGraphAdapter(validator)
graph = adapter.wrap_graph(your_graph)
result = graph.invoke(input)
```

---

## Compliance Mapping

### HIPAA
CCS validates that LLM outputs do not contain Protected Health Information (PHI) unless authorized.

**Validation Rules**:
- Forbidden fields: `ssn`, `medical_record_number`, `diagnosis_code`
- Required fields: `authorization_status`, `redaction_log`

### GDPR
CCS validates that LLM outputs comply with data protection requirements.

**Validation Rules**:
- Forbidden fields: `personal_data` without consent
- Required fields: `data_subject_consent`, `retention_period`

### SOC2
CCS provides audit trails and access controls for LLM outputs.

**Validation Rules**:
- Required fields: `audit_trail_id`, `access_control_list`
- All validation decisions logged to CCS-Audit

---

## Community Implementations

The following frameworks have integrated CCS:

1. **[correctover-crewai](https://github.com/babyblueviper1/correctover-crewai)** - Independent CCS integration for CrewAI framework
2. **[LightAgent](https://github.com/wanxingai/LightAgent)** - Lightweight agent framework with CCS conformance (1,100+ stars)

**Want to be listed?** Submit a PR to this repository with your implementation.

---

## Specification Documents

- **[CCS v1.0 Specification](conformance/SPEC-v1.md)** - Draft conformance specification (v0.1.0)
- **[CCS Standard Paper (PDF)](https://github.com/Correctover/standards/releases/download/ccs-v1.0/Correctover_CCS_Standard_v1.0_Final.pdf)** - Complete formalization with threat model and empirical evaluation

---

## Adoption Metrics

- **DOI**: 10.5281/zenodo.21234580 (cited in academic literature)
- **PyPI**: `correctover` (Python package)
- **npm**: `correctover` (Node.js package)
- **Framework Adapters**: 3 (CrewAI, AutoGen, LangGraph)
- **Verified API Traces**: 20,071 (30-day production observation window, 2026-05-30 to 2026-06-29)
- **Synthetic Test Cases**: 29,929 (6 categories, E1-E6)
- **Total Conformance Test Cases**: 50,000 (20,071 production + 29,929 synthetic)
- **Total Verified Interactions**: 80,000+ (including 30,000+ subsequent accumulation)
- **Distinct Failure Modes**: 325 (~200 production + ~125 synthetic)
- **Single-Fault Recovery Rate**: 97.4% (4,887/5,017)
- **Community Implementations**: 2+

---

## For Enterprise Buyers

### Why Acquire CCS?

1. **Standard Position**: CCS is the first provider-agnostic LLM output verification standard
2. **Framework Agnostic**: Works with any agent framework (CrewAI, AutoGen, LangGraph, and extensible)
3. **Production Proven**: 20,071 verified API traces from real-world deployments
4. **Compliance Ready**: Maps to HIPAA, GDPR, SOC2 out of the box
5. **Extensible**: RFC-based specification allows community-driven evolution

### Integration Paths

- **Acquisition**: Full ownership of CCS standard and implementation
- **Licensing**: Enterprise license with support and customization
- **Partnership**: Joint development of framework-specific adapters

**Contact**: wangguigui@correctover.com

---

## License

CCS Specification: CC BY-NC-SA 4.0  
Implementation: Proprietary (commercial license available)

---

## Resources

- **Website**: https://correctover.github.io
- **DOI**: https://doi.org/10.5281/zenodo.21234580
- **PyPI**: https://pypi.org/project/correctover
- **npm**: https://www.npmjs.com/package/correctover
- **GitHub**: https://github.com/Correctover

---

## 🔒 Security Research

We publish detailed security audits of MCP server implementations:

→ [**mcp-security-audits**](https://github.com/Correctover/mcp-security-audits)

**Findings to date:**
- **506** security findings across 3 major repositories
- **5** vulnerability types confirmed cross-repo (RCE, SSRF, credential hijacking, path traversal, privilege escalation)
- **19** CVE-class vulnerabilities identified
- **215** fault types cataloged in our living fault taxonomy
- **20** vulnerabilities submitted as ZDI reports

---

## 🔧 Upstream PR Contributions

We don't just report — we fix.

| PR | Framework | Status | Contribution |
|----|-----------|--------|--------------|
| [CrewAI#6432](https://github.com/crewAIInc/crewAI/pull/6432) | CrewAI | OPEN (10 commits) | GuardrailProvider — runtime governance protocol |
| [CrewAI#6411](https://github.com/crewAIInc/crewAI/pull/6411) | CrewAI | Discussion | Runtime verification authority |
| [ferro-labs#197](https://github.com/ferro-labs/ferro-labs/pull/197) | Ferro Labs | OPEN | Runtime validation integration |
| [agent-governance-toolkit#3347](https://github.com/microsoft/agent-governance-toolkit/pull/3347) | Microsoft | Under review | Runtime threat scanner (recursive scanning, SSRF fixes) |

---

## 👥 Community Validation

| Researcher | Framework | Contribution |
|-----------|-----------|-------------|
| [@pshkv](https://github.com/pshkv) (AutoGen maintainer) | AutoGen | Adopted Required(τ)⊆Supported(τ) framework |
| [@humbl-dev](https://github.com/humbl-dev) | CrewAI | Two-layer governance testing |
| [@safal207](https://github.com/safal207) | CrewAI | GuardrailProvider implementation (10 commits) |
| [@babyblueviper1](https://github.com/babyblueviper1) | Independent | 120,426 independent re-calculations |
| [@Tuttotorna](https://github.com/Tuttotorna) | PHI-OMEGA | ICLR paper collaboration |
| [@XYG-LUNA](https://github.com/XYG-LUNA) | CrewAI | Idempotency analysis |

---

## 🔍 Audit Case Studies

506 security findings across 3 major MCP repositories, with 5 cross-repo vulnerability patterns and 8 verified PoCs:

→ [Full audit case studies](https://github.com/Correctover/mcp-server/blob/main/audit-case-studies.md)

Includes: SSRF, command injection, credential exposure, DNS rebinding, authentication bypass — all with source code locations, CVSS scores, and disclosure status.

---

## 📦 Ecosystem

| Resource | Link |
|----------|------|
| MCP Server (npm) | [correctover-mcp-server](https://www.npmjs.com/package/correctover-mcp-server) |
| CCS SDK (PyPI) | [correctover](https://pypi.org/project/correctover/) |
| MCP Server Source (20K dataset + papers) | [Correctover/mcp-server](https://github.com/Correctover/mcp-server) |
| Security Audits | [Correctover/mcp-security-audits](https://github.com/Correctover/mcp-security-audits) |
| Agent Governance (fork w/ PRs) | [Correctover/agent-governance-toolkit](https://github.com/Correctover/agent-governance-toolkit) |
| CCS Integration Kit | [Correctover/ccs-integration-kit](https://github.com/Correctover/ccs-integration-kit) |
| Website | [correctover.com](https://correctover.com) |

---

## 📫 Contact

**Security reports**: wangguigui@correctover.com  
**BD / Enterprise**: wangguigui@correctover.com  
**GitHub**: [@Correctover](https://github.com/Correctover)
