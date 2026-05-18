# DP-001: Compliance Wire Protocol Frame Schema for AI Inference Pipeline Audit Records

**Defensive Publication**  
**Technology Outlaws LLC**  
**Publication Date:** May 17, 2026  
**Inventor:** Jason Tesso  
**Related Provisional Application:** U.S. Provisional App. No. 64/013,702 (filed March 23, 2026)  

---

## 1. Field of Disclosure

This disclosure relates to a structured wire protocol frame schema for wrapping AI inference pipeline outputs in tamper-evident compliance records suitable for regulated industries including legal, financial, healthcare, defense, and environmental compliance.

## 2. Background

AI inference systems operating in regulated environments produce outputs that may be subject to audit, litigation hold, regulatory examination, or chain-of-custody requirements. Existing approaches to compliance logging treat the compliance record as a separate artifact from the inference output — typically an after-the-fact log entry appended to a database. This separation creates a temporal gap between output generation and compliance record creation, introduces the possibility of record omission, and allows the compliance record to be separated from the output it describes.

No existing wire protocol defines a mandatory compliance frame that wraps every AI inference output at the moment of generation, such that the output cannot exist without its compliance record and the compliance record cannot be separated from the output after creation.

## 3. Technical Disclosure

### 3.1 Frame Structure

The compliance wire protocol frame is a structured data envelope that wraps every output produced by an AI inference pipeline. The frame is generated at write time — not after the fact — and becomes an inseparable part of the record. A frame that has been sealed cannot be retroactively modified, separated from its content, or deleted without detection.

The frame schema contains the following mandatory fields:

- **frame_id**: A globally unique identifier generated at frame creation time.
- **timestamp**: The UTC timestamp at which the frame was sealed, with nanosecond precision.
- **tenant_id**: The organizational tenant that owns the inference session.
- **workload_type**: An enumerated classification of the record type (e.g., LEGAL_RESEARCH, COMPLIANCE_CHECK, SECURITY_RESEED, CARBON_ATTESTATION, DOCUMENT_REVIEW).
- **content_hash**: A SHA-256 hash of the wrapped content, computed before sealing.
- **source_confidence**: A structured object containing the confidence tier, cross-verification status, and any pending reverification flags for every source cited in the output.
- **chain_of_custody_seq**: A monotonically increasing sequence number per tenant, ensuring ordering and completeness detection.
- **derivative_attestation**: A boolean flag confirming that the wrapped content is a derivative representation and that no raw source material is present.
- **anonymization_attestation**: A boolean flag confirming that the content passed through the anonymization pipeline before inference.

### 3.2 SimHash Content Fingerprinting at Deploy Time

Every deployable artifact in the inference pipeline — including Azure Function code, Bicep infrastructure templates, configuration files, and dependency manifests — is fingerprinted using SimHash at the moment of deployment. The SimHash value is computed over the normalized text content of the artifact and stored in the compliance frame metadata. This creates a verifiable binding between the code that produced an inference output and the compliance record for that output.

SimHash is chosen over cryptographic hashing for content fingerprinting because SimHash produces similar hash values for similar inputs, enabling detection of minor modifications (such as single-line code changes) that would produce entirely different cryptographic hashes. The SimHash distance between the deployed artifact and the expected artifact provides a continuous similarity metric rather than a binary match/no-match result.

### 3.3 Derivative-Only Persistence Pattern

The wire protocol enforces a derivative-only persistence constraint. No raw source document content is stored in any compliance frame or in any data store accessible to the AI inference pipeline. All content within a sealed frame is a derivative representation — meaning it captures the substance of the source material (headings, citations, legal propositions, dates, party identifiers) without preserving the original document structure, formatting, or raw text.

This constraint is enforced at the storage layer through a forbidden-fields mechanism: a defined set of field names (including any field name ending in `_raw`, `_original`, or `_pii_`) is rejected at write time. Any attempt to write a document containing a forbidden field name to the compliance data store results in a write rejection, not a sanitization — the write fails entirely rather than silently stripping the offending field.

### 3.4 Chain-of-Custody Sequence Integrity

The chain_of_custody_seq field provides tamper-evidence and completeness detection. Each tenant maintains an independent monotonically increasing sequence counter. Every frame sealed for that tenant increments the counter by exactly one. A gap in the sequence indicates a missing or deleted record. A duplicate sequence number indicates a forged record. An auditor can verify completeness by confirming that the sequence is contiguous from the first record to the last.

The sequence counter is stored in a separate atomic counter service, not in the frame store itself, to prevent an attacker who compromises the frame store from also controlling the sequence counter.

## 4. Scope of Disclosure

This disclosure establishes prior art for the concept of a mandatory compliance wire protocol frame that wraps AI inference outputs at generation time with tamper-evident chain-of-custody metadata, SimHash content fingerprinting of deployed code artifacts, derivative-only content persistence enforced at the storage layer through field-name rejection, and monotonic sequence-based completeness verification.

This disclosure does not cover the specific implementation of the frame within any particular cloud infrastructure, the integration of the frame with any specific cryptographic attestation protocol, or the application of the frame to any specific regulatory compliance framework.

---

*© 2026 Technology Outlaws LLC. All rights reserved. This publication is made solely for the purpose of establishing prior art under 35 U.S.C. § 102.*
