# Correctover Data Use Agreement

> **Version**: 1.0
> **Date**: 2026-07-04
> **Parties**: Correctover (Data Provider) and [Recipient Name] (Data Recipient)

---

## 1. Purpose

This Agreement governs the provision and use of anonymized conformance trace data ("Data") from Correctover for the sole purpose of academic research, peer review, and independent verification of the Correctover Agent Conformance Specification.

## 2. Parties

**Data Provider**:
- Name: Guigui Wang
- Affiliation: Correctover
- Contact: wangguigui@correctover.com

**Data Recipient**:
- Name: [Full Name]
- Affiliation: [Institution or "Independent Researcher"]
- Contact: [Email]

## 3. Data Description

The Data consists of:
1. **Anonymized sample traces**: Up to 500 JSON-formatted production trace records from the Correctover conformance governance infrastructure
2. **Failure mode classification**: A table of 325 distinct failure modes identified through production observation and systematic testing
3. **Aggregate statistics**: Summarized metrics by test group and provider category
4. **Methodology document**: Formal description of data collection, preprocessing, and analysis methods

### 3.1 What the Data Contains
- Request/response metadata (timestamps, token counts, status codes, latency)
- CANON verification results (6-dimensional pass/fail)
- Decision reference chains (authorization/execution hash comparison)
- Negative vector flags
- Recovery action and outcome (where applicable)

### 3.2 What the Data Does NOT Contain
- Prompt content or response content (text payloads are excluded)
- Provider-specific identifiers (provider names are replaced with category labels)
- API keys, authentication tokens, or credentials
- User-identifiable information
- Complete production dataset (only a representative subset is provided)

## 4. Permitted Use

The Data Recipient may use the Data solely for:
1. **Academic verification**: Validating statistical claims in the Correctover conformance specification
2. **Independent implementation**: Developing compatible conformance verification systems
3. **Academic publication**: Citing aggregate statistics in peer-reviewed publications, with proper attribution

## 5. Restrictions

The Data Recipient SHALL NOT:
1. **Redistribute**: Share, publish, or transfer the Data to any third party
2. **Reverse-engineer**: Attempt to reconstruct the full production dataset from the sample
3. **Commercial use**: Use the Data for any commercial purpose
4. **Competitive analysis**: Use the Data to build a competing product or service
5. **Provider identification**: Attempt to identify specific LLM providers from anonymized traces
6. **Modify attribution**: Remove or alter any attribution notices

## 6. Data Protection Measures

### 6.1 Storage
- Data must be stored on encrypted storage (AES-256 or equivalent)
- Data must not be stored on shared/public cloud storage without access controls
- Data must not be committed to public repositories (GitHub, GitLab, etc.)

### 6.2 Access
- Access limited to the Data Recipient and named collaborators listed in Appendix A
- All collaborators must sign this Agreement or an equivalent acknowledgment

### 6.3 Disposal
- Upon completion of the research purpose, or within 2 years of receipt (whichever comes first), the Data Recipient shall permanently delete all copies of the Data
- Deletion confirmation shall be provided to the Data Provider upon request

## 7. Publication and Attribution

### 7.1 Citation
Any publication using the Data shall cite:
- The Correctover Agent Conformance Specification (DOI: 10.5281/zenodo.21166867)
- This Data Use Agreement as the access mechanism
- The methodology document (DOI: [TBD upon Zenodo upload])

### 7.2 Review
The Data Recipient shall provide the Data Provider with a copy of any publication at least 30 days before submission, to verify that:
- No proprietary information is disclosed
- Data is cited accurately
- Proper attribution is given

The Data Provider's review is limited to compliance with this Agreement and does not constitute editorial control.

## 8. Data Integrity

### 8.1 Accuracy
The Data Provider makes reasonable efforts to ensure the accuracy of the Data but provides it "as is" without warranty of any kind.

### 8.2 Known Limitations
- Sample traces are anonymized and may differ from the full production dataset in distribution
- Provider categories are broad (tier-1 commercial, open-source endpoint, regional provider) and do not identify specific providers
- Aggregate statistics are computed from the full 20,071-trace dataset; sample traces are provided for verification, not as the complete dataset

## 9. Term and Termination

This Agreement is effective from the date of signature by both parties and remains in effect until:
- The Data Recipient completes the permitted use and deletes all copies, OR
- 2 years from the date of Data receipt, OR
- Either party terminates with 30 days written notice

Upon termination, all restrictions in Section 5 and Section 6 survive.

## 10. Governing Law

This Agreement is governed by the laws of [Jurisdiction — to be confirmed by Data Recipient].

---

## Signatures

**Data Provider**:
- Name: Guigui Wang
- Title: Founder, Correctover
- Date: _______________
- Signature: _______________

**Data Recipient**:
- Name: _______________
- Title: _______________
- Date: _______________
- Signature: _______________

---

## Appendix A: Named Collaborators

| Name | Affiliation | Role | Signed DUA? |
|------|-------------|------|-------------|
| [To be filled] | | | ☐ |

---

*This Data Use Agreement is part of the Correctover Agent Conformance Standards data access program.*
*Contact: wangguigui@correctover.com*
