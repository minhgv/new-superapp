# Iteration 4 — Reliability, SLO, Change and Recovery Operations

**Scope:** A new structural direction focused on service reliability and operational governance for the catalog, control plane, artifact delivery, and mini-app runtime. This is distinct from earlier discovery/launch contracts, offline/cache performance, payments/trust, and privacy/accessibility/moderation work.

**Validated additions:** 10 findings, 10 unique URLs, all parent-rechecked with HTTP 200. Evidence labels distinguish official standards/guidance from implementation proposals. The NIST SP 800-34 source is explicitly marked withdrawn historical guidance and is used only for vocabulary, not current targets.

## 1. Define SLOs and error-budget behavior as release inputs

- **Event:** `slo_error_budget_governance`
- **Source / level:** Google SRE, Service Level Objectives — `primary-official-guidance`
- **Source fact:** Google defines an SLI as a quantitative service-level measure and an SLO as a target or range measured by an SLI. It advises against requiring 100% compliance, recommends tracking an error budget for allowed SLO misses, and describes using the budget gap to inform release rollout decisions.
- **Scope / limitation:** This is SRE guidance, not a universal uptime commitment or prescribed numeric target. Targets must reflect product and business impact.
- **Mini-app proposal:** Define separate user-observable SLIs/SLOs for catalog reads/search, publish and metadata operations, and runtime session start/command success. Show rolling budget and burn-rate views by service, app, release and cohort. Normal rollout, guarded rollout, reliability work and pause/approval states must be explicit.
- **URL:** <https://sre.google/sre-book/service-level-objectives/>

## 2. Make releases reproducible, canaryable and reversible

- **Event:** `release_reproducibility_canary_rollback`
- **Source / level:** Google SRE, Release Engineering — `primary-official-guidance`
- **Source fact:** Google SRE describes reproducible and automated builds/configuration, intentional release-process changes, canary strategies, uninterrupted rollout, rollback, gated operations and archived change reports.
- **Scope / limitation:** These are engineering practices, not a mandatory CI/CD or package-signing standard. Cohorts, approval roles and rollback timing require local risk calibration.
- **Mini-app proposal:** Treat each release as an immutable artifact with source revision, inputs, manifest/config schema, test results, owner and complete change list. Promote through automated checks and an SLO-linked canary cohort; stop or roll back on guardrail breaches and retain the last approved artifact for deterministic recovery.
- **URL:** <https://sre.google/sre-book/release-engineering/>

## 3. Turn material failures into blameless, closed-loop learning

- **Event:** `blameless_postmortem_action_tracking`
- **Source / level:** Google SRE, Postmortem Culture — `primary-official-guidance`
- **Source fact:** Google SRE describes written incident records containing impact, mitigation/resolution, causes and follow-up actions. It expects postmortems after significant undesirable events, treats them as blameless learning, and recommends review before archiving the document and action items.
- **Scope / limitation:** This is organizational practice guidance, not a universal root-cause method or notification requirement. The store must define significance triggers and exclude secrets and unnecessary personal data.
- **Mini-app proposal:** Automatically open a post-incident record for material SLO breaches, emergency rollbacks, customer-visible catalog/control/runtime incidents or monitoring failures. Require impact, timeline, causes, evidence, owner, due date, review status and verified closure for each preventive action.
- **URL:** <https://sre.google/sre-book/postmortem-culture/>

## 4. Propagate safe correlation context across store and runtime services

- **Event:** `cross_service_telemetry_context`
- **Source / level:** OpenTelemetry Context Specification — `primary-standard`
- **Source fact:** The stable OpenTelemetry Context specification defines immutable execution-scoped context and propagation across API boundaries and logically associated execution units. It supports cross-cutting concerns such as trace and baggage entries.
- **Scope / limitation:** OpenTelemetry does not define retention, access control, sampling, incident severity or the store's telemetry schema. Context fields still require allowlists and data-minimization review.
- **Mini-app proposal:** Propagate a host-generated request/trace context across catalog edge, catalog service, publish/rollout workers, runtime broker and mini-app callbacks. Correlate traces, metrics and logs using service, app/version, release and incident identifiers while prohibiting credentials and unnecessary user data.
- **URL:** <https://opentelemetry.io/docs/specs/otel/context/>

## 5. Publish a versioned SLO contract instead of relying on dashboard conventions

- **Event:** `slo_schema_and_budgeting`
- **Source / level:** OpenSLO Specification v1 — `open-specification`
- **Source fact:** OpenSLO is a vendor-agnostic specification for defining SLOs. Its schema supports rolling or calendar-aligned windows, multiple budgeting methods, explicit objectives, alert policies and composite SLOs.
- **Scope / limitation:** OpenSLO excludes platform-specific implementation details and is not an ISO/IETF mandate or a mini-app runtime contract. It does not select targets, queries, exclusions or retention.
- **Mini-app proposal:** Publish a versioned SLO manifest for each store/runtime service and critical app journey. Include the SLI query/reference, good/bad/total definition, window, target, exclusions, measurement point, owner, alert policy and labels for app, digest, host version, region, tenant class and cohort. Keep component budgets visible within composite journeys.
- **URL:** <https://raw.githubusercontent.com/OpenSLO/OpenSLO/main/website/docs/specification.md>

## 6. Use a documented error-budget policy to govern promotion

- **Event:** `error_budget_release_gate`
- **Source / level:** Google SRE Workbook, Error Budget Policy — `engineering-reference`
- **Source fact:** Google's example policy permits releases while the service meets its SLO, freezes ordinary changes when the preceding budget is exceeded, allows only exceptional high-priority work, and requires corrective action after material budget consumption.
- **Scope / limitation:** The example's four-week window, 20% trigger and P0 terminology are not universal thresholds or legal obligations. Local governance must define attribution and emergency exceptions.
- **Mini-app proposal:** Make budget state a release-controller input per service, app, host version and cohort. Pause feature rollout when the relevant budget is exhausted; permit narrowly recorded security, data-integrity and recovery fixes with an owner and rollback target. Persist the calculation and override reason in the release audit record.
- **URL:** <https://sre.google/workbook/error-budget-policy/>

## 7. Separate tenants and protect the runtime from noisy neighbors

- **Event:** `tenant_isolation_and_noisy_neighbor_control`
- **Source / level:** Kubernetes Documentation, Multi-tenancy — `primary-platform`
- **Source fact:** Kubernetes documents multi-tenancy as a tradeoff among security, fairness, noisy-neighbor control, complexity and cost. It identifies RBAC, namespaces, quotas and network policies as key controls, and warns that containers are weaker than VMs for untrusted code.
- **Scope / limitation:** Kubernetes has no first-class tenant concept; namespaces are not a complete security or reliability boundary, quotas do not cover every shared resource, and NetworkPolicy requires an enforcing plugin.
- **Mini-app proposal:** Define publisher, organization, app-execution and customer-data boundaries separately. Use least-privilege authorization, per-publisher/app namespaces or equivalent isolation, default-deny network/capability egress, CPU/memory/object/concurrency/download quotas and separate queues/pools for critical tenants. Use stronger sandboxing or dedicated infrastructure when shared-runtime isolation is insufficient.
- **URL:** <https://kubernetes.io/docs/concepts/security/multi-tenancy/>

## 8. Assign recovery objectives by ecosystem component

- **Event:** `recovery_objectives_rto_rpo`
- **Source / level:** NIST SP 800-34 Rev. 1 — `primary-government-guidance`
- **Source fact:** NIST defines maximum tolerable downtime, recovery time objective and recovery point objective, and ties objective selection to business-impact analysis and recovery strategy/cost tradeoffs.
- **Scope / limitation:** NIST's landing page marks SP 800-34 Rev. 1 withdrawn. The vocabulary is useful, but it is not a current universal target, mini-app SLA or substitute for a current business-impact analysis.
- **Mini-app proposal:** Assign MTD/RTO/RPO by criticality tier to catalog/control plane, registry/CDN, install/update state, publisher review, entitlement adapters and app durable state. Record last-known-good digest and backup age; define dependency order, degraded operation, failover owner and data-loss semantics; test restore/failover and report actual results separately from steady-state SLOs.
- **URL:** <https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-34r1.pdf>

## 9. Version telemetry schemas and gate incompatible changes

- **Event:** `telemetry_schema_compatibility_gate`
- **Source / level:** OpenTelemetry Telemetry Schemas — `primary-project`
- **Source fact:** OpenTelemetry describes versioned telemetry schemas identified by unique Schema URLs. Producers include the Schema URL, consumers may transform data to a target schema, and published schema files are immutable.
- **Scope / limitation:** This mechanism addresses evolution of semantic conventions and transformations; it does not define the complete telemetry shape or valid values/types.
- **Mini-app proposal:** Require catalog/runtime telemetry producers and collectors to emit and persist `telemetry_schema_url` and schema version. Maintain tested compatibility transforms, quarantine unknown or broken schema versions, and bind accepted telemetry schema to release/cohort evidence before promotion.
- **URL:** <https://opentelemetry.io/docs/specs/otel/schemas/>

## 10. Require objective cohort evidence before expanding rollout

- **Event:** `cohort_differential_release_gate`
- **Source / level:** Google SRE, Embracing Risk — `engineering-reference`
- **Source fact:** Google SRE describes canarying as testing a release on a small subset of typical workload, with size and duration guided by objective data-based metrics. It also describes error budgets as a control for continuing, slowing or halting releases.
- **Scope / limitation:** The guidance does not prescribe mini-app cohort keys, sample sizes, confidence rules or rollout percentages.
- **Mini-app proposal:** Assign each rollout an immutable cohort definition containing digest, host runtime, region, device class, eligibility rule and start/end. Retain a contemporaneous control cohort, compute SLI deltas and budget burn per cohort, require minimum observation/sample criteria, and pause or roll back when approved guardrails are breached.
- **URL:** <https://sre.google/sre-book/embracing-risk/>

## Implementation implications

For the P0 profile, add an operational contract to the release record: SLO manifest, current budget state, cohort definition, release evidence bundle, rollback target, telemetry schema version and incident owner/runbook. For P1, add tenant-specific quotas/pools, RTO/RPO tests, schema compatibility transforms and automated postmortem/action closure. Numeric SLOs, budget windows, cohort sizes, RTO/RPO values and severity thresholds remain pilot-calibration gaps; they should be set from workload, business impact and threat modeling rather than copied from examples.
