# Security, Privacy and Control Matrix

## Control priorities

| Area | P0 MVP | P1 enterprise | Evidence required |
|---|---|---|---|
| Publisher | Verified account, MFA, RBAC, abuse/security contact | Org verification, SSO, delegated namespaces, payout/tax/KYC policy | identity record, role audit, acceptance records |
| Artifact | Immutable digest, signature verification, package/schema/size/path checks | HSM keys, reproducible builds, signed mirrors | digest, signature result, provenance/SBOM refs |
| Supply chain | Dependency/license/secret/malware checks | Automated re-review on CVE/dependency changes, policy attestation | CI result per digest, SBOM, provenance |
| Runtime | Capability allowlist, sandbox/host bridge policy, compatibility rejection | Tenant policy overlays, offline cache policy, stronger isolation | manifest, host decision, denial reason |
| Privacy | Data-use declaration, consent UX, minimization, retention owner | Regional policy, DPA/controller mapping, deletion/export workflows | notice version, consent event, retention/deletion evidence |
| Review | Automated gates plus human policy/security review | Risk-tiered review, appeals, independent review for high risk | submission, reviewer, policy version, findings |
| Release | Preview, canary/gray, pause, rollback | Multi-region, automated health gates, emergency kill switch | cohort, metrics, approval, rollback event |
| API | OAuth/OIDC role scopes, idempotency, ETags, audit events | mTLS/service identity, tenant isolation, rate/cost quotas | API spec, auth decision, correlation ID |
| Incident | Quarantine, stop installs, last-known-good, notify | Trust revocation, cache purge, forensics/postmortem, regulatory mapping | incident timeline and signed evidence |
| Accessibility | WCAG-based keyboard/focus/contrast/name/role/value checks for store UI | Conformance statement and recurring regression testing | test report, assistive-tech findings |

## Security design rules

1. **Least privilege:** expose only the declared native capabilities and backend scopes required by the app; deny by default.
2. **Consent is not authorization:** user consent UX must be separate from host policy and server-side authorization.
3. **Verify before execute:** the host verifies digest/signature/provenance and compatibility before launch; cache must not bypass policy.
4. **No implicit trust from catalog placement:** featured/ranked does not mean security-approved beyond the published review tier.
5. **Audit state changes:** publisher, package, review, release, permission, quarantine and deprecation transitions need append-only evidence.
6. **Minimize telemetry:** use app/release/device-class/cohort/region dimensions; exclude user identifiers by default and apply retention/redaction.

## Privacy and consent record

For each capability/data category, store: purpose, data elements, source, recipient/backend, legal/policy basis, retention, deletion/export behavior, user-facing notice version, consent timestamp and revocation behavior. The store should expose a short human-readable summary and link to the full notice.

## Review tiers

- **Low risk:** no sensitive data, no payments, no privileged native capability; automated checks plus standard review.
- **Medium risk:** personal data, external APIs, notifications, location/device access or partner-owned backend; expanded privacy/security review and canary requirement.
- **High risk:** financial transactions, identity, health, regulated data, privileged device operations or broad user reach; independent security review, threat model, stronger release approval and emergency rollback test.

## Accessibility minimum

The store itself must support keyboard/focus navigation where applicable, visible focus, semantic names/roles/values, sufficient contrast, non-color status communication, meaningful error messages, accessible search/filter/sort and screen-reader-readable permission/review states. Apply the same quality gate to the miniapp onboarding/review surfaces, not only the host shell.
