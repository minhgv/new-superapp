# Iteration 6 — Runtime resource, tenant isolation and cost governance
## Scope and validation
This iteration adds a structurally distinct direction: runtime/API resource governance, tenant isolation, abuse resistance, and transparent cost attribution for mini-app ecosystems. The parent rechecked 16 unique source URLs with HTTP 200 and rejected one duplicate candidate record whose primary URL duplicated another candidate in the same batch. Thirteen findings were appended to the canonical append-only state, increasing the evidence count from 186 to 199. No credential or spreadsheet content is included.

The evidence separates normative IETF material, official platform/project guidance, and proposals. Kubernetes, OWASP, FinOps, OpenTelemetry, and OpenCost are precedents or semantic foundations; they do not by themselves define the WindVane package model, legal pricing rules, or a universal mini-app quota schedule.

## Decision implications

- Enforce two quota layers: per-app/workload ceilings and tenant/platform aggregate budgets. Include durable objects and paid downstream side effects, not only request counts.
- Treat resource exhaustion as a product and safety concern: stable machine-readable errors, bounded retries, partition-aware rate limits, payload/concurrency/time caps, and alerting for spend-bearing operations.
- Publish isolation strength by trust tier. Logical namespaces, RBAC, network/egress policy, storage isolation, sandboxing, and dedicated VM/node boundaries have different guarantees; namespace separation alone is not a sandbox.
- Attribute direct, shared, idle, and overhead cost separately. Require stable owner/app/environment dimensions before spend starts, retain an unallocated bucket, and show the allocation basis before chargeback.
- Keep rate-limit header drafts labeled as draft-dependent. Use authoritative gateway state for enforcement and pair quota headers with problem details rather than treating positive remaining quota as a service guarantee.

## 1. rate_limit_response_contract
- **Source:** IETF RFC 6585 — Additional HTTP Status Codes, Section 4
- **Evidence level:** `official-normative`
- **Topic:** `runtime-api/rate-limits/429`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc6585.html
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:35:21Z", "method": "curl -sL -A Mozilla/5.0 --max-time 25", "retrieval": "official RFC HTML page fetched directly and Section 4 inspected"}`

[SOURCE FACT] RFC 6585 defines 429 Too Many Requests for a client that has sent too many requests in a given amount of time (rate limiting). It says the response representation SHOULD explain the condition, MAY include Retry-After, does not prescribe how the origin identifies the user or counts requests (for example per resource, across a server, or across a server set), and 429 responses MUST NOT be stored by a cache. [MINI-APP-STORE CONTROL PROPOSAL] Return 429 at the host/API gateway when a mini-app, publisher, user, tenant, device cohort, or endpoint budget is exhausted; identify the exhausted partition and safe next action in a machine-readable problem body, emit Retry-After when a retry time is meaningful, and keep the response non-cacheable. Store the partitioning rule and numeric thresholds in an explicit, versioned policy rather than implying that RFC 6585 chooses them.

**Source URLs:**
- https://www.rfc-editor.org/rfc/rfc6585.html

## 2. retry_after_backoff_semantics
- **Source:** IETF RFC 9110 — HTTP Semantics, Section 10.2.3
- **Evidence level:** `official-normative`
- **Topic:** `runtime-api/retry-semantics/overload`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc9110.html
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:35:23Z", "method": "curl -sL -A Mozilla/5.0 --max-time 25", "retrieval": "official RFC HTML page fetched directly and Section 10.2.3 inspected"}`

[SOURCE FACT] RFC 9110 says servers send Retry-After to indicate how long a user agent ought to wait before a follow-up request; with 503 it indicates how long the service is expected to be unavailable, with 3xx it indicates the minimum wait before the redirected request, and the value is either an HTTP-date or a non-negative decimal delay in seconds. [MINI-APP-STORE CONTROL PROPOSAL] Make the host emit Retry-After for overload responses and rate-limit responses only when it has a meaningful estimate; make the SDK honor it with bounded jitter/backoff and a retry cap, and automatically retry only operations classified safe/idempotent or explicitly deduplicated. RFC 9110 does not define mini-app retry ownership, idempotency keys, or quota values, so those controls must be specified by the host profile.

**Source URLs:**
- https://www.rfc-editor.org/rfc/rfc9110.html

## 3. machine_readable_api_error_contract
- **Source:** IETF RFC 9457 — Problem Details for HTTP APIs
- **Evidence level:** `official-normative`
- **Topic:** `runtime-api/errors/problem-details`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc9457.html
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:35:23Z", "method": "curl -sL -A Mozilla/5.0 --max-time 25", "retrieval": "official RFC HTML page fetched directly and Sections 1 and 3 inspected"}`

[SOURCE FACT] RFC 9457 defines problem details as machine-readable error details in HTTP response content; JSON uses the application/problem+json media type. Its type member is a URI reference used as the problem type identifier, status conveys the origin HTTP status but is advisory and must match the actual response status, title/detail/instance carry a summary, occurrence-specific explanation, and occurrence identifier, and problem types MAY add extensions that unknown clients MUST ignore. [MINI-APP-STORE CONTROL PROPOSAL] Standardize one host problem profile with stable absolute type URIs for quota_exhausted, concurrency_limit, payload_too_large, and retry_suppressed; include status, title, detail, and instance plus explicit extensions such as policy_id, partition_scope, quota_unit, remaining, reset_after, retryable, request_id, and support_url. Keep credentials, secrets, and sensitive user data out of detail; use instance/request identifiers for support and audit correlation.

**Source URLs:**
- https://www.rfc-editor.org/rfc/rfc9457.html

## 4. quota_advertisement_and_partitioned_budget
- **Source:** IETF HTTPAPI — draft-ietf-httpapi-ratelimit-headers-11, RateLimit header fields for HTTP (May 2026 Internet-Draft)
- **Evidence level:** `official-ietf-draft`
- **Topic:** `runtime-api/quotas/ratelimit-headers`
- **Primary URL:** https://www.ietf.org/archive/id/draft-ietf-httpapi-ratelimit-headers-11.html
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:35:23Z", "method": "curl -sL -A Mozilla/5.0 --max-time 25", "retrieval": "official IETF archive HTML fetched directly; draft status and Sections 3–4 inspected"}`

[SOURCE FACT] This active Internet-Draft, not a final RFC, defines structured RateLimit-Policy and RateLimit response fields: RateLimit-Policy advertises quota policy items with q for quota, optional qu for quota units, w for a time window, and pk for a partition key; RateLimit reports available quota r, optional effective window t, and pk. It allows multiple policies, defines quota units including requests, content-bytes, and concurrent-requests, says quotas are allocated per partition key, and warns that positive available quota is not a guarantee that a later request will be served because the server may alter quota and window between responses. [MINI-APP-STORE CONTROL PROPOSAL] Pilot this profile for host-exposed budgets by naming partitions such as mini_app, publisher, user, tenant, and endpoint and quota units such as requests, content-bytes, and concurrent-requests, while keeping authoritative enforcement in gateway state. Pair headers with the RFC 9457 problem profile on rejection, document which policies stack, do not expose sensitive partition identifiers, and label the host profile as draft-dependent until the IETF document is finalized.

**Source URLs:**
- https://www.ietf.org/archive/id/draft-ietf-httpapi-ratelimit-headers-11.html

## 5. tenant_aggregate_quota_and_object_exhaustion
- **Source:** Kubernetes Resource Quotas — official documentation
- **Evidence level:** `platform-engineering-precedent+mini-app-store-proposal`
- **Topic:** `tenant_aggregate_quotas_object_exhaustion`
- **Primary URL:** https://kubernetes.io/docs/concepts/policy/resource-quotas/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:33:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 30", "retrieval": "official Kubernetes HTML page fetched directly; aggregate namespace, object-count, and enforcement text inspected"}`

[PLATFORM ENGINEERING PRECEDENT] Kubernetes defines ResourceQuota as an aggregate constraint on resource consumption per namespace; it can also cap object creation by API kind and total infrastructure resources, and it becomes enforced when a ResourceQuota exists in that namespace. The documentation ties team separation to namespaces plus RBAC or another authorization mechanism. [MINI-APP-STORE PROPOSAL] Model each tenant or trust domain as an explicit quota subject, enforce aggregate CPU/memory/storage and object-count budgets at the control plane/API gateway, and charge durable runtime-created objects—sessions, jobs, subscriptions, artifacts, webhooks, and cache entries—to the owning tenant. Return stable quota-exceeded errors and meter usage; do not rely only on per-request checks that omit durable objects.

**Source URLs:**
- https://kubernetes.io/docs/concepts/policy/resource-quotas/

## 6. per_workload_limits_and_admission_defaults
- **Source:** Kubernetes Limit Ranges — official documentation
- **Evidence level:** `platform-engineering-precedent+mini-app-store-proposal`
- **Topic:** `per_workload_limits_runtime_admission`
- **Primary URL:** https://kubernetes.io/docs/concepts/policy/limit-range/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:33:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 30", "retrieval": "official Kubernetes HTML page fetched directly; LimitRange constraints, defaults, admission, and rejection text inspected"}`

[PLATFORM ENGINEERING PRECEDENT] Kubernetes LimitRange constrains resource allocations for applicable object kinds; it can enforce minimum and maximum compute usage per Pod or Container, minimum and maximum storage requests per PersistentVolumeClaim, request-to-limit ratios, and default requests/limits injected at runtime. Its admission controller applies defaults, tracks usage, and rejects a violating Pod or PersistentVolumeClaim with HTTP 403. [MINI-APP-STORE PROPOSAL] Define per-mini-app workload ceilings for CPU, memory, execution duration, concurrent workers, payload/upload size, and storage; inject safe defaults at launch; and reject configurations outside the cap before execution. Keep per-workload limits separate from tenant aggregate quotas so one app cannot monopolize the tenant budget.

**Source URLs:**
- https://kubernetes.io/docs/concepts/policy/limit-range/

## 7. admission_policy_for_tenant_runtime_requests
- **Source:** Kubernetes Validating Admission Policy — official documentation
- **Evidence level:** `platform-engineering-precedent+mini-app-store-proposal`
- **Topic:** `admission_enforcement_tenant_policy`
- **Primary URL:** https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:33:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 30", "retrieval": "official Kubernetes HTML page fetched directly; CEL, binding, failurePolicy, and validation-action text inspected"}`

[PLATFORM ENGINEERING PRECEDENT] Kubernetes Validating Admission Policy is a declarative, in-process alternative to validating admission webhooks, uses CEL expressions, and can be parameterized and scoped to resources. A policy needs a binding; failed expressions are handled according to failurePolicy, and validation actions such as Deny can reject the request. [MINI-APP-STORE PROPOSAL] Put tenant/app identity, capability, namespace, egress, resource, and quota checks in a fail-closed admission path for creation and update of runtime workloads and API objects. Require binding coverage for every tenant boundary and route, emit auditable denial reasons, and reserve webhooks for logic the declarative policy cannot express.

**Source URLs:**
- https://kubernetes.io/docs/reference/access-authn-authz/validating-admission-policy/

## 8. layered_tenant_isolation_boundary
- **Source:** Kubernetes Multi-tenancy — official documentation
- **Evidence level:** `platform-engineering-precedent+mini-app-store-proposal`
- **Topic:** `layered_tenant_isolation_boundaries`
- **Primary URL:** https://kubernetes.io/docs/concepts/security/multi-tenancy/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:33:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 30", "retrieval": "official Kubernetes HTML page fetched directly; isolation layers, namespace limits, and virtual-control-plane caveat inspected"}`

[PLATFORM ENGINEERING PRECEDENT] Kubernetes frames multi-tenancy around security, fairness, and noisy neighbors, and describes layered isolation: control-plane isolation; namespaces, access controls, and quotas; data-plane network and storage isolation; sandboxed containers; and node isolation. Namespace-per-tenant is supported but does not cover cluster-scoped resources, while a virtual control plane improves control-plane isolation without solving data-plane isolation. [MINI-APP-STORE PROPOSAL] Publish an isolation matrix by threat and trust tier: namespace/account as a logical boundary, RBAC/capability gateway, default-deny network and egress, tenant-scoped storage, sandboxed execution, and an optional dedicated node/VM boundary for untrusted apps. Document which controls are soft versus hard and never present namespace separation alone as a sandbox.

**Source URLs:**
- https://kubernetes.io/docs/concepts/security/multi-tenancy/

## 9. api_resource_exhaustion_and_abuse_controls
- **Source:** OWASP API Security Top 10 2023 — API4 Unrestricted Resource Consumption
- **Evidence level:** `platform-engineering-precedent+mini-app-store-proposal`
- **Topic:** `api_resource_exhaustion_abuse_resistance`
- **Primary URL:** https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `{"http_status": 200, "checked_at": "2026-10-02T11:33:27Z", "method": "curl -sL -A Mozilla/5.0 --max-time 30", "retrieval": "official OWASP API4 HTML page fetched directly; vulnerable-limit list and prevention guidance inspected"}`

[PLATFORM/API SECURITY PRECEDENT] OWASP API4 identifies missing or inappropriate limits on execution time, memory, file descriptors, processes, upload size, operations per request or batching, records per page, and third-party spending. Its prevention guidance recommends payload/array/upload caps, rate limiting, per-client or per-operation throttling, server-side validation of pagination controls, and provider spending limits or billing alerts. [MINI-APP-STORE PROPOSAL] Apply these controls at the API gateway and runtime bridge per tenant, app, user, device/IP, and operation: bound request cost and page size, concurrency and timeouts, body/upload size, batch or GraphQL complexity, and downstream spend. Treat paid calls such as SMS, notifications, payments, AI, and storage as quota-bearing side effects with authorization, hard ceilings, and alerting.

**Source URLs:**
- https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/

## 10. cost_allocation_owner_dimensions
- **Source:** FinOps Foundation Cloud Cost Allocation Guide
- **Evidence level:** `official-guidance`
- **Topic:** `cost_attribution_showback_chargeback`
- **Primary URL:** https://www.finops.org/wg/cloud-cost-allocation/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `"http_200_curl_browser_ua_body_inspected_2026-10-02"`

[SOURCE FACT] The guide defines cost allocation as identifying, categorizing, and assigning cloud-resource costs to users, departments, projects, or other groups through structural hierarchies, tags, and labels; it says allocation supports chargeback and showback. Its summary calls out required metadata such as Cost Center and Environment, warns that tags cannot be applied retroactively, and names tag-compliance percentage plus time from cost incurred to cost displayed as maturity metrics. [PROPOSAL] Require every mini-app cost-bearing release, workload, and platform resource to carry stable owner_id, app_id, environment, and cost_center dimensions before spend starts; retain an explicit unknown/unallocated bucket and measure mapping coverage and attribution latency. [LIMIT] This is cloud-cost guidance, not a prescribed mini-app schema, rate card, or legal chargeback policy.

**Source URLs:**
- https://www.finops.org/wg/cloud-cost-allocation/

## 11. shared_cost_attribution_governance
- **Source:** FinOps Foundation Managing Shared Cloud Costs
- **Evidence level:** `official-guidance`
- **Topic:** `shared_costs_fair_allocation`
- **Primary URL:** https://www.finops.org/wg/identifying-shared-costs/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `"http_200_curl_browser_ua_body_inspected_2026-10-02"`

[SOURCE FACT] The guide says allocating shared costs back to the business areas that created the spend creates transparency and better insight into unit cost. Its summary frames shared infrastructure such as container clusters, data warehouses, and AI foundation models as a common substrate distinct from the workloads consuming it, and says system telemetry plus runtime metadata can turn shared spend into granular attribution. [PROPOSAL] The mini-app standard should classify each cost as direct, shared-substrate, idle, or overhead and publish the allocation basis, denominator, attribution window, version, and residual bucket; show the resulting allocation before applying any chargeback. [LIMIT] The guide does not define a universal mini-app economic model, prices, or a mandatory allocation formula.

**Source URLs:**
- https://www.finops.org/wg/identifying-shared-costs/

## 12. cost_telemetry_owner_resource_dimensions
- **Source:** OpenTelemetry Semantic Conventions
- **Evidence level:** `official-semantic-conventions`
- **Topic:** `cost_telemetry_owner_resource_dimensions`
- **Primary URL:** https://opentelemetry.io/docs/specs/semconv/resource/service/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `"http_200_curl_browser_ua_body_inspected_2026-10-02"`

[SOURCE FACT] OpenTelemetry defines service.namespace as a required namespace that groups related services and notes that it may distinguish a group such as the team owning the services; service.name is expected to remain the same for horizontally scaled instances. Its cloud conventions define cloud.account.id, cloud.provider, cloud.region, and cloud.resource_id, with cloud.resource_id representing the provider-native identity of the monitored resource. [PROPOSAL] Emit these dimensions on every mini-app workload and cost-relevant telemetry record so app identity and owner grouping can join runtime usage to provider billing; use service.name for the stable mini-app identity, service.namespace for the platform owner or domain, and cloud account/region/resource identity for reconciliation. [LIMIT] OpenTelemetry standardizes attribute names and meanings, not pricing, billing joins, allocation algorithms, or chargeback authority.

**Source URLs:**
- https://opentelemetry.io/docs/specs/semconv/resource/service/
- https://opentelemetry.io/docs/specs/semconv/registry/attributes/cloud/
- https://opentelemetry.io/docs/specs/semconv/

## 13. workload_shared_idle_cost_attribution
- **Source:** OpenCost Specification and API Examples
- **Evidence level:** `official-project-documentation`
- **Topic:** `usage_attribution_shared_idle_costs`
- **Primary URL:** https://opencost.io/docs/specification/
- **Verification:** Parent HTTP status 200 recheck; source record verification: `"http_200_curl_browser_ua_body_inspected_2026-10-02"`

[SOURCE FACT] The OpenCost specification supports workload cost aggregation by container, pod, deployment, statefulset, job, controller name/kind, label, annotation, namespace, and cluster. It identifies shared workload, cluster-idle, and overhead costs and describes uniform, consumption-proportional, or custom-metric distribution; the API examples show aggregate=namespace and shareIdle=true, including idleByNode=true. [PROPOSAL] Expose a mini-app cost ledger with direct workload, shared-substrate, idle, and overhead components plus owner/app/environment/namespace dimensions, allocation method, metric, window, and version; never hide idle or shared allocation inside a single unexplained total. [LIMIT] OpenCost models Kubernetes/cloud allocation and is not a complete mini-app pricing, entitlement, or chargeback standard.

**Source URLs:**
- https://opencost.io/docs/specification/
- https://opencost.io/docs/integrations/api-examples/

## Implementation acceptance tests

- Exhaust per-app, per-tenant, per-user/device, and per-endpoint budgets; verify 429/problem responses, Retry-After behavior, non-caching, audit correlation, and bounded retry for safe operations only.
- Attempt to create durable sessions, jobs, webhooks, artifacts, cache entries, and subscriptions beyond quota; verify admission rejection and no orphaned objects.
- Run noisy-neighbor tests across trust tiers; demonstrate the documented difference between logical isolation, sandbox, and hard VM/node isolation.
- Send oversized payloads, deep pagination, expensive batch requests, and paid downstream calls; verify request-cost caps, concurrency/timeouts, spend ceilings, alerts, and operator kill switches.
- Reconcile platform billing to owner/app/environment dimensions; report direct/shared/idle/overhead and unallocated buckets with allocation method, window, version, and residuals.
- Create a rate-limit policy change and verify the host profile remains compatible when the IETF draft changes; do not make rollout dependent on draft-only headers.
