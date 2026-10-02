# Iteration 5 — Portable catalog metadata and evidence federation
## Scope and validation
This iteration adds a structurally new direction: interoperability at the catalog, evidence-exchange, and publisher capability-negotiation boundaries. It does not treat a portable format as proof of publisher identity, package integrity, approval, or runtime safety. The parent rechecked 35 unique source URLs; all returned HTTP 200 with a browser user-agent. Twelve candidate records were validated and appended to the canonical append-only findings file, increasing the count from 174 to 186.
Evidence labels distinguish normative specifications, official project documentation, and bounded proposals. No credential or spreadsheet content is included.
## 1. manifest_discovery_and_federated_export
- **Source:** W3C Web Application Manifest Working Draft (13 August 2026)
- **Evidence level:** `primary-standard`
- **Topic:** `portable_catalog_manifest_federation`
- **Primary URL:** https://www.w3.org/TR/2026/WD-appmanifest-20260813/#link-relation-type-registration
- **Verification:** "HTTP 200 for every source URL verified with curl -sL and a browser user-agent at 2026-10-02T11:01:17Z UTC."

[SOURCE FACT] The W3C Web Application Manifest is a JSON document containing startup parameters and application defaults for launching a web application; it has a manifest URL, defined as the URL from which the manifest was fetched. The registered link relation `manifest` links to a manifest and is described as a centralized place for web-application metadata. The registered media type is `application/manifest+json` with `.webmanifest`, and a user agent MUST support the `manifest` link type and fetch/process linked resources. The conformance appendix also notes that crawlers and search engines could process manifests to build catalogs of potentially installable web applications. [SCOPE/LIMITATION] This is a W3C Working Draft; the defined conformance class is a user agent, and the specification does not define a federation API, publisher authentication, signatures/digests, freshness, approval state, or a cross-store merge policy. [MINI-APP-STORE PROPOSAL] Export a versioned record containing `manifest_url`, fetched raw bytes, `content_type`, `manifest_sha256`, fetch timestamp, source origin, and parsed projection; accept `rel=manifest` discovery but require HTTPS/origin ownership and independent signature/review evidence before importing. Preserve the raw manifest and treat W3C fields as portable metadata, not as proof of publisher or release authority.

**Source URLs rechecked:**
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#using-a-link-element-to-link-to-a-manifest
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#media-type-registration
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#link-relation-type-registration
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#conformance

## 2. localized_catalog_labels_and_icons
- **Source:** W3C Web Application Manifest Working Draft (13 August 2026)
- **Evidence level:** `primary-standard`
- **Topic:** `portable_localization_manifest_projection`
- **Primary URL:** https://www.w3.org/TR/2026/WD-appmanifest-20260813/#x_localized-members
- **Verification:** "HTTP 200 for every source URL verified with curl -sL and a browser user-agent at 2026-10-02T11:01:17Z UTC."

[SOURCE FACT] The manifest's `lang` is a BCP 47 language tag for localizable member values (unknown if absent), and `dir` sets default text direction (`ltr`, `rtl`, or `auto`). Each localizable member has a corresponding `*_localized` member; the language map is keyed by language tags, and user agents SHOULD select the best match for user preferences, falling back to the default representation. Localized text objects carry `value` plus optional `lang` and `dir`; the current draft treats `name`, `short_name`, and `icons` as localizable, and localized image resources are language-map entries containing image-resource lists. [SCOPE/LIMITATION] The document is a Working Draft and uses a SHOULD/fallback behavior; it does not require a store's locale coverage, translation quality, locale negotiation policy, or localized catalog fields outside the manifest. [MINI-APP-STORE PROPOSAL] Normalize `lang`/`dir` and every `*_localized` map into a locale-indexed catalog projection; validate BCP 47 tags, keep per-value `lang`/`dir`, define a deterministic requested-locale fallback order, and export locale-specific labels and icon candidates while retaining the default representation and raw manifest. Do not silently synthesize missing translations.

**Source URLs rechecked:**
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#lang-member
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#dir-member
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#x_localized-members
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#localizing-text-values
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#localizing-image-resources

## 3. portable_icons_presentation_and_capability_projection
- **Source:** W3C Web Application Manifest Working Draft (13 August 2026) and W3C Image Resource Working Draft
- **Evidence level:** `primary-standard`
- **Topic:** `portable_icons_display_capability_projection`
- **Primary URL:** https://www.w3.org/TR/2026/WD-appmanifest-20260813/#icons-member
- **Verification:** "HTTP 200 for every source URL verified with curl -sL and a browser user-agent at 2026-10-02T11:01:17Z UTC."

[SOURCE FACT] The manifest's `icons` member supplies images representing the web application in contexts such as application lists, task switchers, and system settings. A manifest image resource is an image resource with an optional `purpose`; allowed purposes include `any`, `monochrome`, and `maskable`, while the W3C Image Resource specification defines `src` as the image URL, optional `sizes`, optional image MIME `type`, and accessible `label`. `display` is the developer's preferred display mode; modes include `fullscreen`, `standalone`, `minimal-ui`, and `browser`, with fallback chains, and the applied mode may be changed by the user agent (for example for security or out-of-scope navigation). [SCOPE/LIMITATION] Icon selection/transformations and display-mode UI are user-agent/platform choices; the specification says display-mode conventions are advisory and supplies no browser/host version matrix, package/runtime compatibility, or icon integrity guarantee. [MINI-APP-STORE PROPOSAL] Export a typed icon set per locale and purpose with absolute `src`, dimensions, MIME, label, digest/fetch evidence, and safe-zone/purpose metadata; expose `display_preference` separately from verified host capabilities, and resolve a deterministic fallback to `browser` when the host cannot honor the requested mode. Reject off-origin or unverified icon URLs unless publisher policy permits them; do not advertise a display mode as guaranteed solely because it appears in the manifest.

**Source URLs rechecked:**
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#icons-member
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#manifest-image-resources
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#display-member
- https://www.w3.org/TR/2026/WD-appmanifest-20260813/#display-modes
- https://www.w3.org/TR/image-resource/

## 4. schema_software_application_compatibility_and_catalog_projection
- **Source:** Schema.org official SoftwareApplication type
- **Evidence level:** `primary-official`
- **Topic:** `compatibility_and_federated_schema_export`
- **Primary URL:** https://schema.org/SoftwareApplication
- **Verification:** "HTTP 200 verified with curl -sL and a browser user-agent at 2026-10-02T11:01:17Z UTC."

[SOURCE FACT] Schema.org's official `SoftwareApplication` type page lists reusable application/catalog properties including `applicationCategory`, `applicationSubCategory`, `applicationSuite`, `availableOnDevice`, `countriesSupported`, `countriesNotSupported`, `downloadUrl`, `installUrl`, `featureList`, `fileSize`, `operatingSystem`, `permissions`, `processorRequirements`, `runtimePlatform`, `softwareRequirements`, `softwareVersion`, `releaseNotes`, and `screenshot`; its inherited property table also defines `inLanguage` using IETF BCP 47 language codes. [SCOPE/LIMITATION] Schema.org is a descriptive vocabulary, not a launch/runtime or federation protocol; these properties do not establish publisher control, artifact integrity, availability, version-compatibility semantics, or that an install/download URL is safe. The official page explicitly identifies itself as the development version. [MINI-APP-STORE PROPOSAL] Offer a JSON-LD/Schema.org export projection for search and partner federation: map W3C manifest `name`/`id`/`start_url`/`scope` to the store's authoritative identity and launch fields, and map category, OS/runtime/processor/software requirements, supported countries, version, release notes, features, permissions, screenshots, and install/download URLs to the corresponding Schema.org properties. Keep structured data additive; do not use Schema.org `url`, `installUrl`, or `downloadUrl` as unverified launch authority, and include the canonical manifest URL, store release ID, digest, and verification status alongside it.

**Source URLs rechecked:**
- https://schema.org/SoftwareApplication

## 5. sbom_vex_profile_portability
- **Source:** SPDX Specification 3.0.1 (Linux Foundation SPDX)
- **Evidence level:** `primary-standard`
- **Topic:** `sbom_vex_portability`
- **Primary URL:** https://spdx.github.io/spdx-spec/v3.0.1/
- **Verification:** "HTTP 200 verified for every source URL on 2026-10-02; SPDX pages fetched successfully, including the 3.0.1 specification, serialization, conformance, VEX relationship, and vulnerability-data guidance pages."

[SOURCE FACT] SPDX 3.0.1 defines an RDF-based data model for representing and exchanging information about systems with software components; it permits RDF serializations including JSON-LD, Turtle, N-Triples, and RDF/XML, and defines a deterministic canonical JSON serialization. JSON-LD documents must use the versioned SPDX global context, and conformance requires both JSON Schema structural validation and OWL/SHACL semantic validation. The conformance section makes Core mandatory and lists Software and Security as separate profiles; Security-profile conformance covers import/export of vulnerability, severity, and software-impact information, including whether a fix is available. The Security model includes VexAffected and VexNotAffected assessment relationships; the latter maps to VEX not_affected, uses doesNotAffect, links a vulnerability to product elements, and requires a justificationType or impactStatement for a valid not_affected statement. SPDX's vulnerability guidance says static SBOM data and dynamic vulnerability data can be separated, with Security-profile updates issued more frequently, and says SPDX 3.0 supports translating or carrying CVE, CVSS, EPSS, SSVC, KEV, and VEX-related metadata. [SCOPE/LIMITATION] Core and Software interoperability does not imply that a peer implements the optional Security, Build, Licensing, or other profiles; the cited SPDX sections define data, serialization, and validation, not a store's publisher trust roots, signature transport, freshness policy, or vulnerability severity threshold. [MINI-APP-STORE PROPOSAL] Define a store exchange profile requiring Core+Software and, for security evidence, the Security profile in SPDX JSON-LD with the pinned 3.0.1 context, package PURL plus SHA-256/digest, explicit profileConformance, and published/modified timestamps. Store the immutable SBOM and append separate Security/VEX relationship updates keyed to the same package digest; verify the serialization schema and semantic model, then bind the original bytes and signer through an external signature/attestation envelope. Export both the original SPDX document and a normalized status projection; do not reject a vendor merely for omitting optional SPDX profiles, but do reject a security claim lacking an unambiguous product link or required not_affected rationale.

**Source URLs rechecked:**
- https://spdx.github.io/spdx-spec/v3.0.1/
- https://spdx.github.io/spdx-spec/v3.0.1/serializations/
- https://spdx.github.io/spdx-spec/v3.0.1/conformance/
- https://spdx.github.io/spdx-spec/v3.0.1/model/Security/Classes/VexNotAffectedVulnAssessmentRelationship/
- https://spdx.dev/capturing-software-vulnerability-data-in-spdx-3-0/

## 6. sbom_vex_signature_portability
- **Source:** CycloneDX v1.6 JSON Reference and VEX capability (OWASP CycloneDX)
- **Evidence level:** `primary-standard`
- **Topic:** `sbom_vex_signature_portability`
- **Primary URL:** https://cyclonedx.org/docs/1.6/json/
- **Verification:** "HTTP 200 verified for every source URL on 2026-10-02; CycloneDX v1.6 JSON reference and VEX capability page were fetched successfully."

[SOURCE FACT] The CycloneDX v1.6 JSON reference requires bomFormat=CycloneDX and specVersion=1.6; it recommends a unique RFC 4122 serialNumber and says the BOM version should increment when a BOM changes, with the newest version used when serial numbers match. Its lifecycle values cover design, pre-build, build, post-build, operations, discovery, and decommission. Components can have unique bom-ref identifiers; dependency objects reference bom-ref values, and the specification warns that components absent from a dependency graph may have unknown dependencies rather than being dependency-free. The BOM has a vulnerabilities collection with identifiers, cross-source references, ratings, recommendations, and an analysis object. Analysis states include resolved, resolved_with_pedigree, exploitable, in_triage, false_positive, and not_affected; not_affected should have a justification, and responses include update, rollback, workaround_available, can_not_fix, and will_not_fix. The v1.6 reference also defines an enveloped JSON Signature Format signature with multiple signers, key identifiers, optional public keys/certificate paths, and recognized asymmetric algorithms. CycloneDX's VEX capability page describes machine-readable, product-context exploitability data that integrates with broader inventories to prioritize remediation. [SCOPE/LIMITATION] A v1.6 BOM is a format/schema, not a cross-vendor trust or release-approval policy; its dependency graph may be incomplete or unknown, and the VEX capability page does not prescribe a store's signer roots, freshness window, or policy for conflicting vendor assessments. The signature fields do not by themselves establish which keys a receiving store trusts. [MINI-APP-STORE PROPOSAL] Accept a pinned CycloneDX 1.6 JSON exchange containing the mini-app release as a component with PURL, version, SHA-256 hash, supplier, lifecycle, and dependency/composition data. Require serialNumber and monotonically tracked BOM version; preserve the raw BOM and expose an internal completeness flag when aggregate is incomplete or unknown. Ingest vulnerability analysis as a time-stamped status record keyed by bom-ref and release digest, retain the original ratings/references, and allow later VEX/BOM updates without rewriting the package artifact. If a signature is supplied, verify its JSF bytes and certificate/key identity against the store's configured publisher or scanner trust policy; otherwise mark the evidence unsigned and keep it out of high-assurance release gates.

**Source URLs rechecked:**
- https://cyclonedx.org/docs/1.6/json/
- https://cyclonedx.org/capabilities/vex/

## 7. attestation_bundle_portability
- **Source:** in-toto Attestation Framework v1 (official in-toto specifications and repository layers)
- **Evidence level:** `primary-standard`
- **Topic:** `attestation_bundle_portability`
- **Primary URL:** https://in-toto.io/docs/specs/
- **Verification:** "HTTP 200 verified for every source URL on 2026-10-02; in-toto specifications page and official attestation README, Statement, Envelope, and Bundle layers were fetched successfully."

[SOURCE FACT] The in-toto site lists the Attestation Framework as a Stable v1.0 specification, while the versioned layer README on the official repository's main branch is marked v1.2; the accepted version therefore needs to be pinned rather than inferred from a mutable branch. The Statement layer binds an attestation to a subject and predicate type: subject is a required array whose artifacts must carry a digest, subjects are intended to be immutable, and matching is purely by digest regardless of content type. The Envelope layer handles serialization and digital-signature authentication, recommends DSSE v1.0, requires support for multiple signatures, and requires payloadType to be signed with the payload. The Bundle layer groups multiple attestations as JSON Lines; each line can have different keys, subjects, and predicate types, and the official example combines build evidence, a vulnerability scan, and an SPDX SBOM for one package. [SCOPE/LIMITATION] A Bundle is not authenticated as a whole: each attestation is individually authenticated, so the specification explicitly warns that an attacker may delete valid attestations, replay obsolete attestations, or inject irrelevant attestations. Consumers must parse and verify each line and must not infer predicate identity from a media type alone; the Statement predicateType is authoritative. The framework defines the transport layers, but a store still has to choose accepted predicate schemas, signer trust roots, freshness/replay rules, and conflict handling. [MINI-APP-STORE PROPOSAL] Publish one sidecar bundle per immutable mini-app package digest, with separate signed attestations for SPDX/CycloneDX inventory, VEX status, build/release metadata, automated test results, and store review evidence. Require every accepted Statement subject digest to equal the package digest (or an explicitly declared dependency digest), verify each Envelope signature and payloadType before parsing the predicate, and maintain an individual evidence index rather than trusting bundle membership or order. Pin the framework schema version and repository commit in the store profile, reject stale/replayed evidence by policy timestamp and release ID, and export the verified individual envelopes plus a generated inventory/status summary for another vendor.

**Source URLs rechecked:**
- https://in-toto.io/docs/specs/
- https://raw.githubusercontent.com/in-toto/attestation/main/spec/v1/README.md
- https://raw.githubusercontent.com/in-toto/attestation/main/spec/v1/statement.md
- https://raw.githubusercontent.com/in-toto/attestation/main/spec/v1/envelope.md
- https://raw.githubusercontent.com/in-toto/attestation/main/spec/v1/bundle.md

## 8. vex_status_portability
- **Source:** OpenVEX v0.2.0 specification (official OpenVEX project repository)
- **Evidence level:** `primary-project`
- **Topic:** `vex_status_portability`
- **Primary URL:** https://github.com/openvex/spec
- **Verification:** "HTTP 200 verified for every source URL on 2026-10-02; openvex.dev redirected to the official repository with HTTP 200, and the official repository/raw specification URLs were fetched successfully."

[SOURCE FACT] OpenVEX describes itself as a minimal, compliant, interoperable, embeddable VEX implementation that is SBOM-agnostic; its documents are JSON-LD and may be embedded in or incorporated into in-toto attestations, CSAF, or CycloneDX documents. A document groups one or more time-stamped statements and requires metadata including @context, @id, author, timestamp, version, and statements. A statement connects product(s), a vulnerability identifier, and a status; statuses are not_affected, affected, fixed, and under_investigation. Products can be identified with IRIs, PURLs, and cryptographic hashes; a not_affected statement requires a machine-readable justification or impact_statement, and the specification discourages free-form impact_statement text for automated systems. The project README says OpenVEX can reference products described in both SPDX and CycloneDX SBOMs, and can provide an evolving history as new timestamped statements supersede or enrich earlier ones. [SCOPE/LIMITATION] The official README labels the specification a draft and says it is community-owned/steered; it also says OpenVEX initially provides no namespace registry or hosting/redirection of IRIs. The format recommends cryptographic association of author identity with a signature or other exchange mechanism but does not define a store trust root or a complete signature/revocation protocol. The README notes that CycloneDX VEX uses different status/justification labels, so cross-format translation may be required. [MINI-APP-STORE PROPOSAL] Keep VEX separate from the immutable package SBOM but key each statement to the mini-app release SHA-256 and PURL, retaining the original JSON-LD, document version, author, timestamp, and status history. Require a store-approved signature/attestation wrapper for publisher or scanner claims, map OpenVEX statuses to the store's vulnerability state machine, require justification for not_affected, and expose both the original statement and normalized fields to downstream vendors. Use the package digest rather than an unversioned PURL as the execution decision key; treat OpenVEX draft compatibility and translated CycloneDX/SPDX labels as explicit evidence metadata, not as proof of equivalence.

**Source URLs rechecked:**
- https://openvex.dev/
- https://github.com/openvex/spec
- https://raw.githubusercontent.com/openvex/spec/main/OPENVEX-SPEC.md

## 9. publisher_registration_metadata_negotiation
- **Source:** IETF RFC 7591 — OAuth 2.0 Dynamic Client Registration Protocol
- **Evidence level:** `official-normative`
- **Topic:** `publisher_registration_metadata`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc7591.html
- **Verification:** {"http_status": 200, "checked_at": "2026-10-02T10:57:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 25", "retrieval": "official RFC HTML page and RFC text inspected"}

[NORMATIVE RFC FACT] RFC 7591 is an IETF Standards Track specification for JSON-based dynamic client registration. A client using a redirect-based flow MUST register its redirect_uris; token_endpoint_auth_method declares the requested token-endpoint authentication method; grant_types and response_types declare the protocol modes the client can use and MUST correspond to the values used at the relevant endpoints. The metadata model also covers scope, contacts, policy/tos URIs, jwks_uri, software_id, and software_version; jwks_uri is preferred over an inline JWK set for easier key rotation, and trusted software-statement claims take precedence over duplicate request fields. [STORE CONTROL PROPOSAL] Define a publisher-onboarding profile with an allowlisted JSON schema: exact HTTPS callback/endpoint bindings, explicit grant and client-auth method, least-privilege scope/capability declarations, stable software_id plus release version, publisher support/policy URLs, and signed evidence for high-risk capabilities. Reject inconsistent grant/response combinations before activation and persist requested versus effective metadata.

**Source URLs rechecked:**
- https://www.rfc-editor.org/rfc/rfc7591.html

## 10. publisher_registration_credential_lifecycle
- **Source:** IETF RFC 7592 — OAuth 2.0 Dynamic Client Registration Management Protocol
- **Evidence level:** `official-experimental`
- **Topic:** `publisher_registration_credential_lifecycle`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc7592.html
- **Verification:** {"http_status": 200, "checked_at": "2026-10-02T10:57:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 25", "retrieval": "official RFC HTML page and RFC text inspected"}

[NORMATIVE RFC FACT] RFC 7592 is an IETF Experimental specification for managing dynamic client registrations over their lifetime. The registration response supplies a per-client configuration URI and a registration access token; the client MUST use that token for configuration calls, and the configuration endpoint MUST be transport-protected. GET/PUT operations can return replacement client credentials or a replacement registration access token, after which the client MUST immediately discard the previous value; client_id MUST remain unchanged. PUT replaces rather than augments metadata, and the client MUST NOT choose or overwrite its own client secret. Successful DELETE invalidates the client_id, client_secret, and registration access token and SHOULD invalidate active grants and tokens when possible. [STORE CONTROL PROPOSAL] Model publisher onboarding as a revocable lifecycle: issue a narrowly scoped management credential separate from runtime capability tokens, rotate it and publisher credentials, version effective metadata, trigger re-review for redirect/capability/auth changes, and on offboarding revoke management access plus active publisher sessions/tokens with auditable evidence.

**Source URLs rechecked:**
- https://www.rfc-editor.org/rfc/rfc7592.html

## 11. authorization_server_metadata_discovery
- **Source:** IETF RFC 8414 — OAuth 2.0 Authorization Server Metadata
- **Evidence level:** `official-normative`
- **Topic:** `authorization_server_metadata_discovery`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc8414.html
- **Verification:** {"http_status": 200, "checked_at": "2026-10-02T10:57:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 25", "retrieval": "official RFC HTML page and RFC text inspected"}

[NORMATIVE RFC FACT] RFC 8414 is an IETF Standards Track specification for machine-readable authorization-server metadata. The issuer is REQUIRED, MUST use HTTPS, and MUST have no query or fragment; it anchors a well-known metadata document and helps prevent authorization-server mix-up attacks. The JSON metadata can advertise authorization_endpoint, token_endpoint, registration_endpoint, scopes_supported, response_types_supported, grant_types_supported, token_endpoint_auth_methods_supported, jwks_uri, revocation/introspection endpoints, and code_challenge_methods_supported. If code_challenge_methods_supported is omitted, the authorization server does not support PKCE. The optional signed_metadata value MUST be signed or MACed and carry an iss claim; when supported, its values take precedence over plain JSON values. [STORE CONTROL PROPOSAL] Require a discovery document for each store authorization issuer and compute an explicit compatibility result before publisher activation: bind issuer and endpoint origins, intersect supported registration/auth/grant/response/scope/capability profiles, verify PKCE and key/revocation endpoints, and optionally require signed metadata. Cache with bounded freshness, fail closed on issuer/endpoint mismatch, and return machine-readable rejection reasons.

**Source URLs rechecked:**
- https://www.rfc-editor.org/rfc/rfc8414.html

## 12. resource_bound_least_privilege
- **Source:** IETF RFC 8707 — Resource Indicators for OAuth 2.0
- **Evidence level:** `official-normative`
- **Topic:** `resource_bound_least_privilege`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc8707.html
- **Verification:** {"http_status": 200, "checked_at": "2026-10-02T10:57:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 25", "retrieval": "official RFC HTML page and RFC text inspected"}

[NORMATIVE RFC FACT] RFC 8707 is an IETF Standards Track specification that adds the resource parameter to authorization and token requests. Each resource value MUST be an absolute URI without a fragment; the client SHOULD provide the most specific URI for the API or resource it intends to access. The authorization server SHOULD audience-restrict issued access tokens to the indicated resource(s), may reject an unknown or malformed target with invalid_target, and can apply policy to token type, encryption, and audience. Resource identifies where a token is redeemed while scope identifies what access is requested; when possible the server should downscope per resource and MUST report the effective scope when it differs from the request. [STORE CONTROL PROPOSAL] Bind every publisher or mini-app capability grant to a registered resource URI—such as the control-plane API, artifact registry, review API, runtime bridge, or tenant API—and a declared scope. Prefer one audience-limited token per resource, reject undeclared or unknown resources, expose machine-readable invalid_target/compatibility errors, and treat resource binding as complementary to host capability policy and publisher identity verification rather than a replacement.

**Source URLs rechecked:**
- https://www.rfc-editor.org/rfc/rfc8707.html

## Standard implications
1. **Portable metadata is additive, not authoritative.** W3C Web App Manifest and Schema.org projections can improve discovery and partner export, but the store must retain its own immutable release ID, digest, publisher verification, compatibility result, review state, and evidence status.
2. **Evidence exchange must be digest-bound and version-pinned.** SPDX, CycloneDX, OpenVEX, and in-toto provide useful machine-readable inventory, vulnerability, and attestation structures; the store still needs trust roots, freshness/replay rules, required predicate types, conflict handling, and policy thresholds.
3. **Publisher control-plane integrations need explicit negotiation.** RFC 7591/7592/8414/8707 support machine-readable registration, credential lifecycle, issuer discovery, resource/audience restriction, and least-privilege scope; the mini-app store should turn these into an allowlisted onboarding profile and re-review triggers.
4. **Recommended next artifact:** define a versioned `MiniAppFederationProfile` with canonical app/release identity, manifest projection, locale/icon rules, package digest, evidence references, issuer/resource bindings, approval state, freshness, and rejection reasons; validate it against representative Alibaba/WindVane and partner-host cases in the pilot.
