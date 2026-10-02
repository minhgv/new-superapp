# Reference Architecture and Lifecycle

## 1. Trust boundaries

```text
Publisher / partner
      | identity, namespace, source, build evidence
      v
Control plane
  Publisher/RBAC -> Submission API -> Policy + automated checks
        -> Review queue -> OCI-compatible registry + signed catalog
        -> Release orchestrator -> Audit/event ledger
        -> Incident/deprecation service
      | approved digest + policy decision
      v
Data plane
  Store/catalog UI -> compatibility resolver -> CDN/cache
      -> host signature/provenance verifier -> miniapp container/runtime
      -> capability broker / consent UX -> miniapp backend APIs
      -> telemetry with redaction and retention policy
```

The control plane decides what may be published. The data plane decides what may be downloaded, verified, launched and authorized on a specific host. They must not share an implicit trust boundary.

## 2. Core resources

### Publisher

`publisher_id`, verified identity/org, namespace, MFA status, roles, security contact, abuse contact, legal/payout state where applicable, terms/privacy acceptance, trust status, suspension/quarantine state.

### App

`app_id`, publisher, display name, localized metadata, category, support URL, privacy URL, data-use summary, target audience, screenshots, supported locales, host/runtime families, required capabilities, monetization/entitlement model, lifecycle status.

### Version/release

`release_id`, `app_id`, SemVer, immutable package digest, size, media type, entry point, min/max host SDK, required host APIs, capability declaration, network endpoints, SBOM/provenance/signature refs, release notes, reviewer decision, rollout policy, rollback target, created/approved/activated/deprecated timestamps.

### Review decision

`submission_id`, automated checks, reviewer identity/role, policy version, findings, remediation, approval/rejection, appeal/re-review state, evidence links.

### Audit event

Append-only event with actor/service, action, resource/version, timestamp, reason, policy version, before/after state references and correlation ID.

## 3. Lifecycle state machine

```text
draft -> submitted -> automated_checks_passed -> human_review -> approved
  -> internal -> preview -> canary -> staged -> active

Side states: rejected, needs_changes, quarantined, paused, aborted, withdrawn, deprecated,
sunset_scheduled, blocked
```

Every transition must be authorized and recorded. A release cannot move to `active` unless the package digest, publisher identity, compatibility result, capability policy, automated-check results, reviewer decision, rollout target and rollback target are queryable by `release_id`.

## 4. Release algorithm

1. Validate publisher and namespace.
2. Validate manifest/schema, package size/path/entry point and declared capabilities.
3. Build in a trusted CI context; generate digest, SBOM, provenance and signature.
4. Run tests across the host SDK/device matrix plus static, malware, secret and sandbox checks.
5. Submit immutable artifact and evidence.
6. Human reviewer evaluates content, privacy, permissions, UX, security and policy.
7. Publish to internal/preview cohort.
8. Run canary/gray release with health thresholds and automatic pause.
9. Promote to staged/all users only after explicit approval.
10. Retain last-known-good digest and one-click rollback.
11. On incident: pause promotion, quarantine digest, stop new installs, revoke/constrain trust, invalidate unsafe cache and notify according to severity.

## 5. Compatibility resolver

Reject before download when platform, host SDK, bridge/API range, required capability, locale, package format or policy version is incompatible. The resolver should evaluate `min_host_version`, tested host range, required host APIs and capability policy intersection. “Installable” and “launchable” are separate outcomes; a release can be visible but unavailable on an incompatible host with an explanatory reason.

## 6. API profile

Use versioned OpenAPI for the control plane. Minimum resource groups: `/publishers`, `/apps`, `/apps/{id}/versions`, `/submissions`, `/reviews`, `/releases`, `/compatibility`, `/incidents`, `/deprecations`, `/audit-events`. Use cursor pagination and ETags for catalog reads; idempotency keys for submissions and release actions; explicit problem details and role-scoped OAuth/OIDC authorization.

## 7. Deprecation

`active -> deprecated -> sunset_scheduled -> blocked -> withdrawn/deleted` only when legal, security and retention policy permit. Publish successor/migration guide, owner, notice date, sunset date, scope and data-export/retention behavior. Use machine-readable HTTP deprecation/sunset signals where APIs are involved, but do not rely on headers alone for store UX.
