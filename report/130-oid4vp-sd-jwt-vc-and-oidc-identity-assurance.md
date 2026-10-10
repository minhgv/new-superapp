# Milestone 130: OpenID for Verifiable Presentations (OID4VP 1.0), IETF SD-JWT-based Verifiable Credentials (SD-JWT VC), and OpenID Connect for Identity Assurance (OIDC IDA 1.0)

## Executive Overview

Milestone 130 establishes the normative decentralized identity presentation, verifiable credential profiling, and legally assured electronic identity verification (eKYC) architectural standards for the enterprise super-app platform. Building upon the token issuance primitives (OID4VCI 1.0) and selective disclosure foundations (SD-JWT) codified in Milestone 129, this milestone establishes:
1. **OpenID for Verifiable Presentations (OID4VP 1.0)**: Standardizes runtime verifiable presentation exchanges between End-User Wallets and Verifiers (third-party mini-apps or native services). Introduces `response_mode=direct_post.jwt` (JARM encrypted responses), eliminating browser URL parameter leakage, robust `client_id` trust schemes (`entity_id`, `x509_san_dns`), cross-device dynamic QR code presentations via single-use `request_uri` handles, and strict 5-step verifier cryptographic validation pipelines.
2. **IETF SD-JWT-based Verifiable Credentials (SD-JWT VC - `draft-ietf-oauth-sd-jwt-vc`)**: Profiles SD-JWT specifically for verifiable digital credentials. Mandates `vct` (verifiable credential type) schema metadata, `status` claim referencing compressed Token Status List bitstrings for sub-millisecond privacy-preserving revocation checks, hardware enclave holder key binding (`cnf.jwk`/`cnf.jkt`) with mandatory biometric confirmation, and anti-correlation safeguards utilizing fresh 128-bit cryptographic salts and automated batch issuance.
3. **OpenID Connect for Identity Assurance (OIDC IDA 1.0)**: Standardizes the representation of legally validated identity data and eKYC evidence within OpenID Connect ID Tokens and UserInfo responses. Codifies the `verified_claims` top-level structure, `trust_framework` taxonomy, `id_document` evidence schemas (document type, issuing authority, optical security verification, NFC chip validation, biometric liveness scores), `electronic_record` evidence for national/telco registry integrations, and dynamic claims filtering via standard OIDC `claims` parameter syntax.

---

## 1. OpenID for Verifiable Presentations (OID4VP 1.0)

### 1.1 Protocol Architecture & Authorization Request

OpenID for Verifiable Presentations extends OAuth 2.0 and OpenID Connect to enable Verifiers (relying parties or third-party mini-apps) to request Verifiable Presentations from End-User Wallets.

```
+-----------------------------------------------------------------------------------------+
|                                    OID4VP 1.0 ARCHITECTURE                              |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|   +-----------------------+                         +-------------------------------+   |
|   |  Verifier (Mini-App)  |                         |  End-User Wallet (Super-App)  |   |
|   +-----------------------+                         +-------------------------------+   |
|               |                                                     |                   |
|               |  1. Authorization Request (response_type=vp_token)  |                   |
|               |---------------------------------------------------->|                   |
|               |     - presentation_definition / dcql_query          |                   |
|               |     - client_id & client_id_scheme                  |                   |
|               |     - response_uri & response_mode=direct_post.jwt  |                   |
|               |     - nonce & state                                 |                   |
|               |                                                     |                   |
|               |                                                     | 2. User Consent   |
|               |                                                     |    & Selective    |
|               |                                                     |    Disclosure UI  |
|               |                                                     |                   |
|               |  3. Direct POST Encrypted Response                  |                   |
|               |<----------------------------------------------------|                   |
|               |     - vp_token (SD-JWT VC / mdoc / W3C VC)          |                   |
|               |     - presentation_submission                       |                   |
|               |     - Encrypted via Verifier Public Key (JARM)      |                   |
|               |                                                     |                   |
+-----------------------------------------------------------------------------------------+
```

#### Core Protocol Requirements:
- **Authorization Request**:
  - `response_type=vp_token` (or `response_type=id_token vp_token` when an identity assertion is requested simultaneously).
  - `presentation_definition`: JSON object conforming to the DIF Presentation Exchange or Digital Credentials Query Language (`dcql_query`), defining the required credential schemas, claim constraints, and acceptable issuers.
  - `client_id` and `client_id_scheme`: Explicitly identifies the requesting Verifier. Supported schemes include `entity_id` (OpenID Federation trust chains), `x509_san_dns` (X.509 certificate with DNS SAN), and `redirect_uri`.
  - `nonce`: Cryptographically secure random challenge (minimum 128 bits entropy) generated by the Verifier to bind the presentation to the current transaction.

### 1.2 Response Modes: direct_post & direct_post.jwt

To overcome URL length restrictions in mobile WebViews and eliminate the risk of sensitive credential leakage through HTTP Referer headers or browser history, OID4VP mandates direct out-of-band delivery.

- **`direct_post`**: The Wallet delivers the presentation payload directly to the Verifier's `response_uri` via HTTP POST (`application/x-www-form-urlencoded` or `application/json`).
- **`direct_post.jwt`**:
  - The Wallet serializes the response parameters (`vp_token`, `presentation_submission`, `state`) into a signed and encrypted JWT (JARM RFC 9101).
  - Encrypted using the Verifier's public key advertised in its metadata, ensuring end-to-end confidentiality across intermediate network components.
  - The Verifier decrypts the JWT, validates the signature, and matches the embedded `state` against its active session.

### 1.3 Verifier Authenticity & Trust Validation

Before prompting the user or releasing sensitive credentials, the Wallet must cryptographically authenticate the Verifier:
- **`entity_id` Scheme**: The Verifier's identifier is an OpenID Federation Entity ID. The Wallet resolves and validates the complete Trust Chain from the Verifier back to a trusted Root Trust Anchor (e.g. the Super-App Operator or National Trust Authority).
- **`x509_san_dns` Scheme**: The Verifier presents an X.509 certificate chain. The Wallet verifies that the subject alternative name (SAN) matches the Verifier's domain and anchors to an OS-trusted or enterprise-pinned Certificate Authority.
- **Untrusted / Unknown Schemes**: If a mini-app fails cryptographic client authentication, the super-app wallet must terminate the presentation session and flag a security intervention event.

### 1.4 Cross-Device Presentation & Dynamic QR Codes

For interactions at desktop terminals, smart TVs, or physical point-of-sale (POS) kiosks:
1. The Verifier renders a dynamic QR code encoding an `openid4vp://` deep-link URI.
2. The URI contains a short-lived `request_uri` pointing to the Verifier's endpoint.
3. The mobile super-app scans the QR code, fetches the signed Request Object via HTTPS from `request_uri`, displays the native consent sheet, and posts the verifiable presentation directly to `response_uri`.
4. `request_uri` handles must enforce a maximum Time-To-Live (TTL) of 60 seconds and must be invalidated immediately upon initial retrieval.

---

## 2. IETF SD-JWT-based Verifiable Credentials (SD-JWT VC)

### 2.1 Credential Profiling & vct Metadata

IETF SD-JWT VC (`draft-ietf-oauth-sd-jwt-vc`) profiles the base SD-JWT specification specifically for digital identity and verifiable credentials ecosystems.

```
+-----------------------------------------------------------------------------------------+
|                                  SD-JWT VC DATA MODEL                                   |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  JOSE Header:                                                                           |
|  {                                                                                      |
|    "typ": "vc+sd-jwt",                                                                  |
|    "alg": "ES256"                                                                       |
|  }                                                                                      |
|                                                                                         |
|  JWT Body (Issuer-Signed Claims):                                                       |
|  {                                                                                      |
|    "iss": "https://identity.superapp.vn",                                               |
|    "iat": 1791594000,                                                                   |
|    "exp": 1823130000,                                                                   |
|    "vct": "https://credentials.superapp.vn/vct/citizen-kyc-v1",                         |
|    "status": {                                                                          |
|      "status_list": {                                                                   |
|        "idx": 41208,                                                                    |
|        "uri": "https://identity.superapp.vn/status/kyc-list-2026.json"                  |
|      }                                                                                  |
|    },                                                                                   |
|    "cnf": {                                                                             |
|      "jkt": "0Z9nJ8uW5_vX6..."                                                          |
|    },                                                                                   |
|    "_sd": [                                                                             |
|      "SHA-256(Disclosure 1: given_name)",                                              |
|      "SHA-256(Disclosure 2: family_name)",                                             |
|      "SHA-256(Disclosure 3: birthdate)",                                               |
|      "SHA-256(Disclosure 4: national_id_number)"                                       |
|    ]                                                                                    |
|  }                                                                                      |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
```

#### Key Technical Rules:
- **`typ` Header**: Must be `vc+sd-jwt` (or `dc+sd-jwt` for dynamic credentials). Verifiers must reject any token lacking this explicit type to prevent cross-JWT confusion attacks.
- **`vct` Claim**: Absolute URI uniquely identifying the credential type schema. Wallets and verifiers fetch JSON Schema definitions, display metadata, and localized claim descriptions from the Type Metadata endpoint.
- **`iss` Claim**: Authoritative HTTPS URI identifying the issuing entity.

### 2.2 Privacy-Preserving Revocation via Token Status List

SD-JWT VC embeds the `status` claim pointing directly to an OAuth Token Status List:
- **Compressed Bitstring**: The status list is published as a compressed bitstring (Deflate `c: DEF`), where each bit (or 2-bit pair) represents the active, revoked, or suspended state of a credential.
- **Sub-Millisecond Verification**: Verifiers fetch the status list from `status.status_list.uri`, inspect the bit at `status.status_list.idx`, and verify validity in memory.
- **Issuer Blindness**: Because the status list is cached globally via HTTP RFC 9111, the Issuer never observes when or where a specific user's credential is verified.

### 2.3 Hardware Enclave Key Binding & Holder-of-Key Proofs

To prevent theft and unauthorized presentation of disclosed credentials:
- The credential body includes a `cnf` (confirmation) claim holding the holder's public key thumbprint (`cnf.jkt` RFC 7638 SHA-256 digest).
- The key pair is generated and locked inside hardware-backed secure elements (Android Keystore StrongBox / iOS Secure Enclave).
- Presentations require an ephemeral Key Binding JWT (`kb+jwt`) signed by the holder key, incorporating:
  - `aud`: Verifier's authenticated origin.
  - `nonce`: Fresh cryptographic nonce issued by the Verifier.
  - `iat`: Timestamp bounding the presentation validity to <= 60 seconds.
  - `sd_hash`: Cryptographic SHA-256 digest of the exact sequence of revealed Disclosures.

---

## 3. OpenID Connect for Identity Assurance (OIDC IDA 1.0)

### 3.1 Data Architecture & verified_claims

OpenID Connect for Identity Assurance (OIDC IDA 1.0) standardizes the representation of legally validated identity data and electronic Know-Your-Customer (eKYC) evidence within OpenID Connect ID Tokens and UserInfo responses.

```json
{
  "verified_claims": {
    "verification": {
      "trust_framework": "vn_sbv_ekyc_level_3",
      "time": "2026-10-08T10:15:30Z",
      "verification_process": "proc_biometric_chip_id_v2",
      "evidence": [
        {
          "type": "id_document",
          "method": "unsupervised_remote",
          "document_details": {
            "type": "idcard",
            "document_number": "001092008899",
            "issuer": {
              "country": "VN",
              "name": "BCA C06"
            },
            "date_of_issuance": "2023-01-15",
            "date_of_expiry": "2038-01-15"
          },
          "check_details": [
            {
              "check_method": "optical_security_features",
              "result": "passed"
            },
            {
              "check_method": "nfc_icao9303_chip_cryptographic_verification",
              "result": "passed"
            },
            {
              "check_method": "biometric_facial_liveness_and_match",
              "result": "passed",
              "match_score": 0.994
            }
          ]
        },
        {
          "type": "electronic_record",
          "record": {
            "type": "telecom_operator_record",
            "source": {
              "name": "Viettel National Telecom SIM Registry",
              "country": "VN"
            },
            "record_id": "sim_reg_98471203"
          },
          "check_details": [
            {
              "check_method": "subscriber_identity_match",
              "result": "passed"
            }
          ]
        }
      ]
    },
    "claims": {
      "given_name": "Minh",
      "family_name": "Giap",
      "birthdate": "1992-05-18",
      "national_id_number": "001092008899"
    }
  }
}
```

### 3.2 Evidence Types: id_document vs electronic_record

- **`id_document` Evidence**:
  - Classifies physical identity documents: passports, national biometric identity cards, driver licenses.
  - Documents inspection methodologies: `physical_in_person`, `supervised_remote`, `unsupervised_remote` (automated AI/optical processing), `electronic_signature`.
  - Embeds `check_details` reporting outcomes of optical MRZ checks, NFC chip cryptographic signature verification (ICAO Doc 9303 passive authentication), and facial biometric liveness detection.
- **`electronic_record` Evidence**:
  - Captures verifications performed against authoritative institutional databases (banking records, credit bureau histories, government citizen registries, and telecommunications operator subscriber databases).
  - Captures authoritative `source` and verification audit transaction handles, establishing end-to-end legal traceability.

### 3.3 Dynamic Claims Request Syntax & Step-Up KYC Filtering

Relying parties specify verified claim requirements dynamically via the standard OpenID Connect `claims` parameter:

```json
{
  "userinfo": {
    "verified_claims": {
      "verification": {
        "trust_framework": {
          "value": "vn_sbv_ekyc_level_3"
        }
      },
      "claims": {
        "given_name": { "essential": true },
        "family_name": { "essential": true },
        "birthdate": { "essential": true },
        "national_id_number": { "essential": true }
      }
    }
  }
}
```

If the user has not completed the required assurance level, the super-app identity runtime automatically triggers an in-app eKYC step-up verification flow before granting token access to the mini-app.

---

## 4. Synthesis: Super-App eKYC & Decentralized Identity Architecture

The integration of OIDC IDA, SD-JWT VC, and OID4VP creates a unified, privacy-first digital identity ecosystem:
1. **Semantic Verification Layer (OIDC IDA)**: Provides the standardized legal vocabulary, evidence hierarchy, and trust framework definitions for eKYC operations.
2. **Cryptographic Packaging Layer (SD-JWT VC)**: Encapsulates verified claims into salt-digested, tamper-evident credentials bound to user hardware keys.
3. **Runtime Presentation Layer (OID4VP)**: Mediates verifiable presentations across mini-app WebViews, native mobile bridges, and external cross-device terminals with zero data leakage.

```
+-----------------------------------------------------------------------------------------+
|                    SUPER-APP UNIFIED DECENTRALIZED IDENTITY STACK                       |
+-----------------------------------------------------------------------------------------+
|                                                                                         |
|  [PRESENTATION LAYER]       OpenID for Verifiable Presentations (OID4VP 1.0)            |
|                             - direct_post.jwt & JARM Encrypted Transports               |
|                             - client_id Schemes (entity_id, x509_san_dns)               |
|                             - DCQL / Presentation Definition Dynamic Queries            |
|                                                                                         |
|  [CREDENTIAL FORMAT]        IETF SD-JWT Verifiable Credentials (SD-JWT VC)              |
|                             - vct Schema Metadata & Type Registries                     |
|                             - Salted Digests (_sd) & Selective Disclosure               |
|                             - RFC Token Status List Privacy-Preserving Revocation       |
|                             - Hardware Enclave Holder Key Binding (cnf.jkt)             |
|                                                                                         |
|  [SEMANTIC & TRUST LAYER]   OpenID Connect for Identity Assurance (OIDC IDA 1.0)        |
|                             - verified_claims & Trust Frameworks (SBV, eIDAS, NIST)     |
|                             - id_document Evidence (ICAO NFC, Biometric Liveness)       |
|                             - electronic_record Evidence (Telco, National Registry)     |
|                                                                                         |
+-----------------------------------------------------------------------------------------+
```

---

## 5. Standard Recommendations for Super-App Mini-App Store

1. **Mandate OID4VP 1.0 for Identity Verification**:
   - Deprecate raw identity claim sharing via WebView bridges.
   - Require all mini-apps requesting user identity attributes to invoke the native wallet via OID4VP `direct_post.jwt` protocols.
2. **Standardize SD-JWT VC for Stored Credentials**:
   - Issue all super-app digital credentials (digital loyalty cards, verified employee badges, banking KYC tokens) in `vc+sd-jwt` format.
   - Enforce hardware-backed key pair generation inside Android Keystore StrongBox and Apple Secure Enclave.
3. **Implement Real-Time Token Status List Verification**:
   - Verify credential status lists on every presentation to guarantee immediate revocation propagation without compromising user privacy.
4. **Deploy User-Centric Selective Disclosure UI**:
   - Present a granular consent sheet in the super-app container allowing users to uncheck individual optional claims before presentation signing.
5. **Establish eKYC Trust Framework Categorization**:
   - Map mini-app store categories (e.g. Finance, Healthcare, E-Commerce, Utilities) to explicit minimum OIDC IDA `trust_framework` assurance tiers.
