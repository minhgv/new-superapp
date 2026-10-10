# Iteration 140: Solid Protocol (Pods & WAC/ACP), W3C VISS v2 (Automotive Telematics), and FIDO MDS 3.0 & IETF RFC 9535 (JSONPath)

## 1. Executive Overview & Strategic Significance

Iteration 140 represents a major architectural milestone for the Super App Mini App Store Standard, expanding canonical technical specifications across three essential frontiers:
1. **Decentralized Personal Data Architecture & Web Access Control**: Establishing user-centric Personal Online Datastores (Solid Pods), cryptographically bound identity via Solid-OIDC and DPoP (RFC 9449), granular access control matrices (WAC/ACP) with `acl:Append` tamper-evident audit logging, and Shape Tree interoperability.
2. **Connected Vehicle Architecture & Automotive In-Cabin Governance**: Institutionalizing the W3C Vehicle Information Service Specification (VISS v2 Core and Transport) for smart cockpits, in-vehicle infotainment (IVI), VSS signal hierarchy trees, WebSocket/MQTT streaming telematics, Automotive Safety Integrity Level (ASIL) access gating, and driver privacy protection.
3. **Hardware-Anchored Biometric Attestation & Standardized Query Sandboxing**: Operationalizing FIDO Alliance Metadata Service (MDS 3.0) signed BLOBs for automated passkey authenticator verification and status reporting, alongside formalizing IETF RFC 9535 JSONPath query execution to eliminate ReDoS vulnerabilities and algorithmic complexity attacks in mini-app data gateways.

With these 15 newly validated findings, the cumulative research evidence base reaches **2,051 canonical findings** across **140 structured research iterations**.

---

## 2. Deep-Dive Domain Analyses

### Domain A: Solid Protocol, Solid-OIDC & Web Access Control (WAC/ACP)
- **Solid Protocol & Personal Data Pods (LDP 1.0)**:
  - Mini-apps traditionally operate in isolated data silos, leading to pervasive data duplication, synchronization friction, and regulatory liability under GDPR/CCPA.
  - The Solid Protocol fundamentally decouples data storage from application computation by establishing user-owned Personal Online Datastores (Pods).
  - Leveraging the W3C Linked Data Platform (LDP), resources and directory containers are exposed through normalized RDF representations (Turtle, JSON-LD) and standard HTTP methods (GET, PUT, POST, PATCH via SPARQL Update).
  - Super-app recommendation: Implement the host storage layer as an internal or federated Solid Pod node. Mini-apps request scoped data access grants rather than storing duplicate copies of personal user profiles in third-party backend servers.
- **Solid-OIDC & DPoP Token Cryptographic Binding**:
  - Replaces arbitrary opaque user IDs with dereferenceable HTTP(S) WebID URIs inside the `webid` claim of OpenID Connect identity tokens.
  - Requires Demonstration of Proof-of-Possession (DPoP, RFC 9449) at the HTTP layer, binding ID Tokens and Access Tokens to ephemeral client-generated asymmetric key pairs.
  - Eliminates the vulnerability of bearer token exfiltration and replay across heterogeneous third-party resource servers.
  - Super-app recommendation: Enforce Solid-OIDC and DPoP proof validation for all mini-app API traffic interacting with the super-app backend gateway.
- **Web Access Control (WAC) & Access Control Policy (ACP)**:
  - WAC establishes declarative access control lists (.acl) defining four primary access modes: `acl:Read`, `acl:Write`, `acl:Append`, and `acl:Control`.
  - The `acl:Append` mode enforces append-only semantics, allowing mini-apps to push telemetry, audit events, or transaction logs without granting read, update, or deletion capabilities.
  - The `acl:origin` restriction blocks untrusted cross-origin web frames from accessing sensitive pod resources even when invoked in an active user session.
  - Super-app recommendation: Enforce `acl:Append` permissions on third-party mini-app security loggers to guarantee non-repudiation and prevent retroactive log tampering.
- **Solid Application Interoperability & Shape Trees**:
  - Introduces Data Grants, Access Grants, and Type Registries, allowing users to authorize mini-apps for specific semantic classes (e.g., Contacts, Medical Invoices, Loyalty Points).
  - Integrates Shape Trees with W3C SHACL/ShEx to enforce strict structural schemas on graph documents prior to persistence, preventing schema corruption by third-party mini-apps.
  - Super-app recommendation: Publish a certified Shape Tree repository in the mini-app developer portal for shared enterprise business domains.

---

### Domain B: W3C Vehicle Information Service Specification (VISS v2) Core & Transport
- **VISS v2 Core & Vehicle Signal Specification (VSS) Data Model**:
  - Connects automotive smart cockpits, heads-up displays (HUD), and IVI mini-apps to normalized vehicle telemetry.
  - Adopts the COVESA/W3C Vehicle Signal Specification (VSS) taxonomy, representing vehicle systems as a hierarchical, standardized signal tree (e.g., `Vehicle.Speed`, `Vehicle.Powertrain.TractionBattery.StateOfCharge`).
  - Shields mini-app developers from fragmented underlying CAN, LIN, and Automotive Ethernet bus protocols.
  - Super-app recommendation: Adopt VISS v2 core data model as the standard API interface for in-cabin automotive super-app mini-apps.
- **VISS v2 Transport Bindings (WebSocket, MQTT & REST)**:
  - Specifies full-duplex WebSocket framing with request correlation identifiers (`requestId`), minimizing serialization overhead for high-frequency telematics.
  - Accommodates edge-to-cloud telemetry ingestion via MQTT broker bindings and RESTful microservices for low-frequency queries.
  - Implements subscription sampling filters (minimum time intervals, delta thresholds) to prevent bus saturation and cellular bandwidth exhaustion.
  - Super-app recommendation: Implement host-mediated VISS WebSocket client adapters with client-side subscription rate-limiting to protect in-vehicle compute and network budgets.
- **Automotive Safety Gating & Access Token Verification**:
  - Differentiates read-only telematics from safety-critical physical actuation (door locks, HVAC, window roll, acceleration limiter).
  - Mandates cryptographically signed OAuth 2.0 / OpenID Connect access tokens in request authorization headers, mapping token scopes directly to VSS signal paths.
  - Enforces Automotive Safety Integrity Level (ASIL) isolation: write operations capable of compromising dynamic vehicle control while the car is in motion are strictly rejected.
  - Super-app recommendation: Maintain a kernel-level safety gateway in the automotive super-app host container that intercepts and audits all actuator calls against vehicle motion states.
- **Curve Logging, Offline Buffering & Privacy Sanitization**:
  - Supports curve-logging algorithms allowing in-cabin brokers to buffer and downsample telematics during network blackouts, flushing ordered events upon reconnection.
  - Mandates location fuzzing, coordinate truncation, and VIN redaction when streaming telematics to untrusted third-party analytics services.
  - Super-app recommendation: Provide persistent visual privacy indicators on the automotive display whenever an active mini-app reads real-time vehicle sensors.

---

### Domain C: FIDO Alliance Metadata Service (MDS 3.0) & IETF RFC 9535 JSONPath
- **FIDO MDS 3.0 BLOB Architecture & Hardware Attestation Verification**:
  - Provides a centralized, cryptographically signed distribution channel for FIDO Authenticator metadata statements via MDS3 JWT BLOBs.
  - Signed by the FIDO Alliance MDS root authority, incorporating strict `nextUpdate` timestamps to guarantee freshness and prevent replay.
  - Indexes certified authenticators by AAGUID (FIDO2) or AAID (UAF), detailing biometric modalities, cryptographic algorithms, key protection levels, and security certification ratings (FIDO Level 1 to 3+).
  - Tracks compromised authenticator batches and revoked attestation keys via real-time `statusReports`.
  - Super-app recommendation: Deploy an automated MDS 3.0 BLOB synchronization worker on the identity gateway, rejecting authenticators marked with `ATTESTATION_KEY_COMPROMISE` for financial mini-apps.
- **FIDO Metadata Statement v3.0 Schema & Security Attributes**:
  - Exposes `keyProtection` flags distinguishing software keystores from hardware-isolated Secure Elements (`KEY_PROTECTION_SECURE_ELEMENT`) and Trusted Execution Environments (`KEY_PROTECTION_TEE`).
  - Documents biometric False Acceptance Rate (FAR), False Rejection Rate (FRR), and attachment hints (`ATTACHMENT_HINT_INTERNAL` for platform passkeys vs `ATTACHMENT_HINT_EXTERNAL` for physical security keys).
  - Super-app recommendation: Require high-assurance mini-apps (banking, enterprise ERP) to verify hardware-backed key protection via FIDO metadata statements during passkey registration.
- **IETF RFC 9535 JSONPath Standard & ReDoS Defenses**:
  - Resolves decades of vendor dialect divergence by establishing an unambiguous formal grammar for JSONPath query expressions.
  - Standardizes the root identifier `$`, current node `@`, child segments, recursive descent `..`, wildcard `*`, and slice selectors `[start:end:step]`.
  - Specifies filter selectors `?<logical-expr>` and standard function extensions (`length()`, `count()`, `match()`, `search()`), mandating RFC 9485 I-Regexp compliance to eliminate catastrophic backtracking (ReDoS).
  - Documents security guardrails: enforces implementation-defined caps on JSON nesting depth, expression AST depth, and nodelist cardinality.
  - Super-app recommendation: Deploy RFC 9535 compliant JSONPath parsers with strict AST depth and memory caps across super-app backend gateways and client SDK query bridges.

---

## 3. Normative Requirements Matrix (Milestone 140)

| Requirement ID | Standard Reference | Architectural Domain | Mandate Level | Implementation Specification |
|---|---|---|---|---|
| **REQ-140-01** | Solid Protocol Section 2 & 5 | Personal Data Architecture | **SHALL** | Store user profiles and application state in LDP-compliant Personal Data Pods using JSON-LD / Turtle serialization. |
| **REQ-140-02** | Solid-OIDC 0.1.0 & RFC 9449 | Identity & Cryptographic Attestation | **SHALL** | Anchor user identities to WebID URIs and mandate DPoP cryptographic proof-of-possession on all mini-app bearer tokens. |
| **REQ-140-03** | Solid WAC & ACP Specs | Access Control & Sandboxing | **SHALL** | Enforce declarative ACLs with `acl:Append` mode on telemetry/audit streams to guarantee write-only log integrity. |
| **REQ-140-04** | Solid App Interoperability | Semantic Data Governance | **SHOULD** | Validate cross-mini-app shared data mutations against certified SHACL/ShEx Shape Trees before persistence. |
| **REQ-140-05** | W3C VISS v2 Core Section 4 | Automotive Connected Telematics | **SHALL** | Model all automotive in-cabin mini-app telemetry endpoints according to the COVESA/W3C Vehicle Signal Specification (VSS) tree. |
| **REQ-140-06** | W3C VISS v2 Transport Section 3 | Automotive Stream Ingestion | **SHALL** | Utilize WebSocket transport with client-side subscription rate-limiting and delta sampling to conserve bus bandwidth. |
| **REQ-140-07** | W3C VISS v2 Core Section 6 | Automotive Safety & Cybersecurity | **SHALL** | Verify OAuth access tokens against ASIL safety rules, strictly prohibiting actuation (`set`) calls while the vehicle is in motion. |
| **REQ-140-08** | W3C VISS v2 Privacy Considerations | Automotive Privacy Governance | **SHALL** | Apply coordinate fuzzing and VIN redaction before exposing telemetry to third-party mini-apps, backed by visual cockpit indicators. |
| **REQ-140-09** | FIDO MDS v3.0 Section 3-4 | Hardware Key Attestation | **SHALL** | Periodically fetch and cryptographically verify FIDO MDS 3.0 signed JWT BLOBs to enforce authenticator certification policies. |
| **REQ-140-10** | FIDO Metadata Statement v3.0 | Biometric & Hardware Certification | **SHALL** | Verify `KEY_PROTECTION_SECURE_ELEMENT` or `KEY_PROTECTION_TEE` for passkeys registered with regulated financial mini-apps. |
| **REQ-140-11** | IETF RFC 9535 Section 1-3 | Declarative Data Transformation | **SHALL** | Adopt RFC 9535 compliant JSONPath engines for query evaluation across super-app host bridges and gateway proxies. |
| **REQ-140-12** | IETF RFC 9535 Section 2.4 & 4 | Query Engine Sandboxing & ReDoS | **SHALL** | Enforce RFC 9485 I-Regexp compliance and impose strict AST depth (max 16) and nodelist caps (max 1,000 items) on all JSONPath queries. |

---

## 4. Canonical Finding Ledger (Iteration 140)

| ID | Topic | Evidence Level | Authoritative Source URL | Standard Reference |
|---|---|---|---|---|
| `solid_protocol_140_01` | Personal Data Pods & LDP Architecture | official_standard | https://solidproject.org/TR/protocol | Solid Protocol Section 2 & 5 / W3C LDP 1.0 |
| `solid_protocol_140_02` | Solid-OIDC & DPoP Key Binding | official_standard | https://solid.github.io/solid-oidc/ | Solid-OIDC 0.1.0 Section 3-6 / RFC 9449 |
| `solid_protocol_140_03` | Solid WAC & ACP Authorization Matrices | official_standard | https://solidproject.org/TR/wac | Solid Web Access Control & ACP |
| `solid_protocol_140_04` | Solid App Interop & Shape Trees | official_standard | https://solidproject.org/TR/protocol | Solid Application Interoperability / SHACL |
| `solid_protocol_140_05` | Solid Real-Time Notifications Protocol | official_standard | https://solidproject.org/TR/protocol | Solid Notifications Protocol 0.2.0 |
| `viss2_automotive_140_06` | VISS v2 Core & VSS Signal Tree | official_standard | https://www.w3.org/TR/viss2-core/ | W3C VISS v2 Core Section 4-5 |
| `viss2_automotive_140_07` | VISS v2 Transport (WebSocket/MQTT/REST) | official_standard | https://www.w3.org/TR/viss2-transport/ | W3C VISS v2 Transport Section 3-6 |
| `viss2_automotive_140_08` | VISS v2 Security Tokens & ASIL Gating | official_standard | https://www.w3.org/TR/viss2-core/ | W3C VISS v2 Core Section 6 (Security) |
| `viss2_automotive_140_09` | VISS v2 Curve Logging & Offline Buffering | official_standard | https://www.w3.org/TR/viss2-core/ | W3C VISS v2 Core Section 5.4 |
| `viss2_automotive_140_10` | Automotive Privacy Fuzzing & Redaction | official_standard | https://www.w3.org/TR/viss2-core/ | W3C VISS v2 Security & Privacy Considerations |
| `fido_mds_140_11` | FIDO MDS 3.0 Attestation BLOB Verification | official_standard | https://fidoalliance.org/specs/mds/fido-metadata-service-v3.0-ps-20210518.html | FIDO Metadata Service (MDS) v3.0 Section 3-4 |
| `fido_mds_140_12` | FIDO Metadata Statement v3.0 Security Schema | official_standard | https://fidoalliance.org/specs/mds/fido-metadata-statement-v3.0-ps-20210518.html | FIDO Metadata Statement v3.0 Section 3 |
| `rfc9535_jsonpath_140_13` | IETF RFC 9535 JSONPath Core Syntax & Model | official_standard | https://www.rfc-editor.org/rfc/rfc9535.html | IETF RFC 9535 Section 1-3 |
| `rfc9535_jsonpath_140_14` | RFC 9535 Filters, Functions & I-Regexp | official_standard | https://www.rfc-editor.org/rfc/rfc9535.html | IETF RFC 9535 Section 2.3 & 2.4 |
| `rfc9535_jsonpath_140_15` | JSONPath Security, Resource Caps & ReDoS | official_standard | https://www.rfc-editor.org/rfc/rfc9535.html | IETF RFC 9535 Section 4 (Security) |
