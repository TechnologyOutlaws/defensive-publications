# DP-005: Quantum Random Number Generator Seeded Moving Target Defense for AI Inference Infrastructure

**Defensive Publication**  
**Technology Outlaws LLC**  
**Publication Date:** May 17, 2026  
**Inventor:** Jason Tesso  
**Related Provisional Application:** U.S. Provisional App. No. 64/031,846 (filed April 6, 2026)  

---

## 1. Field of Disclosure

This disclosure relates to a moving target defense architecture for AI inference systems in which infrastructure variants are continuously generated using entropy derived from a quantum random number generator (QRNG), such that an attacker who successfully characterizes one variant gains no advantage in attacking subsequent variants.

## 2. Background

Moving target defense (MTD) is an established concept in which system configurations are continuously changed to increase the cost of attack. Existing MTD implementations typically rotate configurations on a schedule, randomize address space layouts, or cycle through a predefined set of system variants. These approaches share a common limitation: the entropy source used to generate variants is a pseudorandom number generator (PRNG) whose output is deterministic given knowledge of its seed state.

An attacker who obtains or infers the PRNG seed can predict future variants, reducing the MTD to a fixed-target defense with extra complexity. Existing MTD systems also typically operate at a single layer — network configuration, application deployment, or cryptographic key rotation — rather than generating variants across multiple independent layers simultaneously.

No existing architecture combines a true quantum entropy source with multi-layer variant generation applied to AI inference infrastructure, such that variant prediction is computationally infeasible regardless of attacker resources.

## 3. Technical Disclosure

### 3.1 QRNG Entropy Source

The moving target defense system derives variant seeds from a quantum random number generator. QRNG output is non-deterministic — it is derived from quantum mechanical processes (photon arrival time, vacuum fluctuation, radioactive decay) that are fundamentally unpredictable regardless of computational resources. This means that even an attacker with unbounded classical computing power cannot predict future variant seeds from observation of past variants.

The QRNG source can be a hardware device (dedicated QRNG appliance or DPU-embedded entropy source), a cloud QRNG API service, or a FIPS 140-3 approved deterministic random bit generator (DRBG) seeded by hardware entropy as a fallback. The architecture does not depend on a specific QRNG implementation — it requires only that the entropy source produces output that is not reproducible from prior observations.

### 3.2 Multi-Layer Variant Generation

The QRNG entropy seed is used to generate infrastructure variants at three independent layers:

**Network Layer Variants:** Decoy network endpoints are generated and broadcast continuously. Each decoy endpoint presents network characteristics (open ports, TLS certificate subjects, protocol responses) that are indistinguishable from the real infrastructure endpoint. The number of decoy endpoints grows with the number of enrolled nodes in the defense network. An attacker performing network reconnaissance encounters N targets where only one is real, and N increases continuously.

**Session Layer Variants:** Decoy TLS session handshakes are broadcast continuously alongside real session establishment. Each decoy handshake completes a full TLS negotiation with a valid-appearing certificate but routes to a monitoring endpoint rather than the real application. An attacker attempting to establish a session cannot distinguish real session establishment from decoy sessions.

**Cryptographic Seed Layer Variants:** Decoy cryptographic key derivation operations are performed continuously alongside real key derivation. Each decoy derivation uses QRNG entropy to produce output that is structurally identical to real derived key material. An attacker who intercepts derived key material cannot determine whether it is real or decoy.

### 3.3 Enrolled Endpoint Entropy Contribution

Devices enrolled in the defense network contribute behavioral entropy to the QRNG pool. Each enrolled device computes a periodic entropy contribution derived from hardware-stable device identifiers and time-based HMAC derivation. The contribution is transmitted as a signed heartbeat at a configurable interval.

The behavioral entropy contribution does not require specialized cryptographic hardware. Software-only enrollment uses a device seed derived from machine-stable identifiers (hardware serial numbers, network interface identifiers, platform attestation values). The device seed is combined with the current time period to produce a time-varying entropy contribution via HMAC.

Enrolled endpoints do not read file content, log keystrokes, or store any data beyond the device seed. The entropy contribution is derived from structural system properties (network egress patterns, process tree topology, TLS handshake timing) — not from user data or content.

### 3.4 Attack Cost Multiplication

Because the three variant layers operate independently, the cost of a successful attack is the product of defeating each layer — not the sum. An attacker must simultaneously identify the real network endpoint among N decoys (Layer 1), establish a real session among M decoy sessions (Layer 2), and identify real key material among K decoy derivations (Layer 3). The combined attack cost is N × M × K, where each factor grows independently as more endpoints enroll and more time passes.

### 3.5 Variant Rotation Period

Variants are regenerated on a configurable rotation period (τ). The rotation period defines the maximum time any single variant configuration persists. At each rotation, new QRNG entropy is consumed, new decoy configurations are broadcast, and the previous variant set becomes stale. The rotation period is itself varied by a configurable jitter factor derived from the QRNG source, preventing an attacker from synchronizing observations to the rotation schedule.

## 4. Scope of Disclosure

This disclosure establishes prior art for the concept of using quantum random number generator entropy to seed a multi-layer moving target defense for AI inference infrastructure, software-only device enrollment contributing behavioral entropy to the QRNG pool, multiplicative attack cost across independent variant layers, and jittered rotation periods derived from quantum entropy.

This disclosure does not cover the specific integration of this defense with any particular cryptographic attestation protocol, any specific compliance wire protocol, any specific CMMC or regulatory evidence generation mechanism, or any specific anomaly detection or behavioral deviation scoring system.

---

*© 2026 Technology Outlaws LLC. All rights reserved. This publication is made solely for the purpose of establishing prior art under 35 U.S.C. § 102.*
