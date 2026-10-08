# Iteration 27: DOM XSS Mitigation, WebAssembly Sandboxing & Cross-Origin Isolation

**Research status:** Deli Deep iteration 27
**Findings added:** 15 (canonical findings count: 341 -> 356)
**Evidence topics:**
1. Code injection & DOM XSS mitigation: W3C Trusted Types, CSP require-trusted-types-for, W3C Subresource Integrity (SRI), OWASP DOM-based XSS prevention safe sinks.
2. WebAssembly sandbox runtime boundaries: W3C WebAssembly Core 1/2, Wasm Web API streaming compilation, linear memory isolation, table indirect call type safety, and CSP wasm-unsafe-eval.
3. Cross-Origin Isolation & timing attack defense: WHATWG HTML COOP / COEP / CORP headers, crossOriginIsolated status, SharedArrayBuffer restrictions, and W3C High Resolution Time clock coarsening.

---

## 1. Domain Overview & Super-App Container Implications

In enterprise and regulated super-app architectures, mini-apps execute client-side code within embedded WebViews or multi-threaded JavaScript runtimes. This iteration establishes three critical layers of runtime defense-in-depth:

1. **DOM XSS & Code Injection Mitigation**:
   - Web application vulnerabilities frequently emerge from injection into dangerous DOM execution sinks (`innerHTML`, `outerHTML`, `document.write`, `eval`, `setTimeout(string)`).
   - W3C Trusted Types eliminates DOM XSS by converting string assignments to typed objects (`TrustedHTML`, `TrustedScript`, `TrustedScriptURL`) created only through approved, reviewed policy factories (`window.trustedTypes.createPolicy`).
   - Combined with W3C Subresource Integrity (SRI), every external or sub-packaged script and stylesheet must match a cryptographic digest (`sha256`, `sha384`, `sha512`) before execution, defeating CDN compromise and tampering.

2. **WebAssembly Sandbox Boundaries**:
   - High-performance mini-apps (games, interactive canvas, cryptography, media codecs) increasingly rely on WebAssembly.
   - W3C WebAssembly specifications enforce a capability-isolated runtime: a Wasm module has no ambient OS or host access and can only interact through explicitly provided import functions.
   - Wasm linear memory is completely isolated from JavaScript object heaps and host pointers, bounded by strict page allocations (`initial` and `maximum` 64 KiB pages).
   - Dynamic function calls via `call_indirect` are bound to typed tables, throwing runtime traps on out-of-bounds indices or signature mismatches.
   - Under CSP Level 3, runtimes can permit WebAssembly compilation via `'wasm-unsafe-eval'` without opening the dangerous doorway to arbitrary JavaScript string evaluation (`'unsafe-eval'`).

3. **Cross-Origin Isolation & Side-Channel Timing Resistance**:
   - Modern browser hardware side-channel attacks (Spectre/Meltdown) enable malicious threads to read process memory across origins via high-resolution timing.
   - WHATWG Cross-Origin Isolation requires explicit deployment of `Cross-Origin-Opener-Policy: same-origin` (COOP) and `Cross-Origin-Embedder-Policy: require-corp` or `credentialless` (COEP).
   - High-privilege concurrency primitives such as `SharedArrayBuffer` and `Atomics` are gated strictly behind `crossOriginIsolated === true`.
   - W3C High Resolution Time Level 3 mandates that clocks (`performance.now()`) are coarsened (e.g. to 100 microseconds or 5 microseconds with jitter) in non-isolated contexts to prevent high-precision micro-architectural timing attacks.

---

## 2. Iteration 27 Finding Records

### Finding 27.1: Enforce Trusted Types at every mini-app DOM XSS sink

- **Finding ID:** `iter27-trusted-types-csp-sink-enforcement`
- **Topic:** `trusted-types-dom-xss`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** W3C Trusted Types §4.2.1 require-trusted-types-for directive; §2.1.1 DOM XSS injection sinks; §3.5 Get Trusted Type compliant string
- **Primary URL:** [https://www.w3.org/TR/trusted-types/](https://www.w3.org/TR/trusted-types/)
- **Supporting URLs:**
  - [https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/require-trusted-types-for](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/require-trusted-types-for)
  - [https://web.dev/articles/trusted-types](https://web.dev/articles/trusted-types)

**Specification & Control Detail:**

[SPEC FACT] The specification groups the DOM XSS sinks under the 'script' sink group, including script URL/text setters, eval-like code execution, javascript: navigation, HTML parsing setters such as Element.innerHTML, ShadowRoot.innerHTML, Element.outerHTML, Document.write, and parseFromString. The require-trusted-types-for directive configures whether that group requires matching Trusted Types; when enforcement is active, compliant-input processing accepts the matching Trusted Type, otherwise invokes an allowed default policy or blocks the string with a TypeError. MDN documents the deployed header form as Content-Security-Policy: require-trusted-types-for 'script'. [SUPER-APP CONTROL] Make the mini-app document and worker realms enforcement boundaries: require the header (or an equivalent host-enforced policy) for every registered package, fail closed on raw-string sink violations, and include host-created templates and bridge-generated DOM in the same boundary. Capture sink name, package identifier, version, realm, and code location in a privacy-minimized violation event; do not treat a report-only deployment as protection.

**Super-App Applicability:**

container: provision Trusted Types CSP and isolate each mini-app realm; store: require this control in the package security profile and reject runtimes that cannot enforce it; runtime: block raw strings at DOM/script sinks and emit actionable, non-sensitive violation telemetry.

---

### Finding 27.2: Allowlist and review Trusted Type policy factories

- **Finding ID:** `iter27-trusted-types-policy-allowlist`
- **Topic:** `trusted-types-policy-governance`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** W3C Trusted Types §2.3 Policies; §2.4.1 Content Security Policy; §4.2.2 trusted-types directive; §4.2.5 Should Trusted Type policy creation be blocked by Content Security Policy; §5.4 Best practices for policy design
- **Primary URL:** [https://www.w3.org/TR/trusted-types/](https://www.w3.org/TR/trusted-types/)
- **Supporting URLs:**
  - [https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)
  - [https://web.dev/articles/trusted-types](https://web.dev/articles/trusted-types)
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/require-trusted-types-for](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/require-trusted-types-for)

**Specification & Control Detail:**

[SPEC FACT] TrustedHTML, TrustedScript, and TrustedScriptURL objects are created through application-defined policies; the specification treats policy creation as security-critical and says insecure policies can still expose sinks to attacker-controlled data. The trusted-types directive controls policy creation: an enforced allowlist blocks a policy name not listed, while the default policy routes strings passed to enforced sinks through its callbacks. The specification advises secure-for-all-input policies or tightly limited access to policies and self-contained, reviewed policy code. [SUPER-APP CONTROL] Require a finite policy-name allowlist in each mini-app manifest and deploy production CSP with only approved names; prohibit wildcard and duplicate-name allowances, and permit a default policy only as a time-bounded migration profile. Review every createHTML/createScript/createScriptURL callback as privileged code, pin its sanitizer configuration and dependency versions, and make TrustedScriptURL policies allow only registered HTTPS origins and paths (or disable dynamic script URLs). Quarantine packages that create unregistered policies or mutate policy decisions from unreviewed global state.

**Super-App Applicability:**

container: enforce the policy-name allowlist and restrict policy creation per realm; store: record policy names, callback purpose, sanitizer dependency, and approved resource origins during review; runtime: expose only approved policy capabilities and block or quarantine policy-creation violations.

---

### Finding 27.3: Pin executable and stylesheet bytes with SRI before mini-app execution

- **Finding ID:** `iter27-sri-byte-integrity-before-execution`
- **Topic:** `subresource-integrity`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** W3C Subresource Integrity §3.1 Integrity metadata; §3.2 Cryptographic hash functions; §3.3.3 Get the strongest metadata from set; §3.3.4 Do bytes match metadataList?; §3.7 Handling integrity violations
- **Primary URL:** [https://www.w3.org/TR/SRI/](https://www.w3.org/TR/SRI/)
- **Supporting URLs:**
  - [https://w3c.github.io/webappsec-subresource-integrity/](https://w3c.github.io/webappsec-subresource-integrity/)
  - [https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)

**Specification & Control Detail:**

[SPEC FACT] SRI requires integrity metadata containing a hash function and digest to validate a response; conformant user agents must support SHA-256, SHA-384, and SHA-512. When multiple algorithms are present, the user agent selects the strongest supported metadata, compares the computed digest to the expected value, and refuses to render or execute a response that fails the check, returning a network error. [SUPER-APP CONTROL] The store should bind every executable script and security-relevant stylesheet in a mini-app package manifest to an exact URL or package path, approved version, and SRI digest; generate a new approval record whenever bytes change. The container must not execute a resource with a missing, malformed, or mismatched digest, must quarantine the package and surface a safe update or rollback path, and should retain a failure event linking package/version/resource without logging sensitive payloads. Pair SRI with source review and signer or provenance controls: a matching hash proves byte identity, not that the approved bytes are benign.

**Super-App Applicability:**

container: verify integrity metadata before applying styles or executing code; store: calculate, review, and persist hashes for package and permitted remote resources; runtime: fail closed on mismatch, quarantine the mini-app, and support a trusted fallback or rollback.

---

### Finding 27.4: Require CORS and integrity metadata for cross-origin mini-app resources

- **Finding ID:** `iter27-sri-cors-integrity-policy`
- **Topic:** `subresource-integrity-cross-origin`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** W3C Subresource Integrity §3.3.4 Do bytes match metadataList? (CORS requirement); §3.4 Verification of HTML document subresources; §3.8 Integrity-Policy; §3.8.2 Should request be blocked by Integrity Policy
- **Primary URL:** [https://www.w3.org/TR/SRI/](https://www.w3.org/TR/SRI/)
- **Supporting URLs:**
  - [https://w3c.github.io/webappsec-subresource-integrity/](https://w3c.github.io/webappsec-subresource-integrity/)
  - [https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)

**Specification & Control Detail:**

[SPEC FACT] The SRI specification states that integrity-protected cross-origin requests require CORS and that using SRI without CORS is a logical error. Its Integrity-Policy and Integrity-Policy-Report-Only headers govern integrity metadata for script and style destinations: an enforcing policy blocks requests lacking integrity metadata or using no-CORS mode, while report-only mode allows them and reports violations. MDN likewise requires a CORS-enabled response and the crossorigin attribute for cross-origin SRI. [SUPER-APP CONTROL] The mini-app loader should classify every cross-origin script and stylesheet request, require an explicit CORS response and crossorigin=anonymous, and deny no-CORS external code or styles. Deploy Integrity-Policy-Report-Only in staging and on existing packages to inventory legacy loads, then enforce the script/style destinations with a reporting endpoint after the store's manifest corpus is clean. Treat a CORS failure, missing integrity metadata, or integrity-policy block as a package-load failure rather than falling back to an unverified URL.

**Super-App Applicability:**

container: own the URL loader, CORS mode, crossorigin attribute, and Integrity-Policy headers; store: reject manifests that reference cross-origin code without CORS and SRI metadata; runtime: report-only during migration, then block nonconforming script/style loads and expose a safe recovery state.

---

### Finding 27.5: Default-deny dangerous DOM sinks and use text or DOM construction for untrusted mini-app data

- **Finding ID:** `iter27-dom-xss-safe-sinks`
- **Topic:** `dom-xss-safe-sinks`
- **Evidence Level:** `security_standard`
- **Standard Reference:** OWASP DOM Based XSS Prevention Cheat Sheet RULE #6 Populate the DOM using safe JavaScript functions or properties; RULE #7 Fixing DOM Cross-site Scripting Vulnerabilities; GUIDELINE #1; GUIDELINE #3; GUIDELINE #4; GUIDELINE #5
- **Primary URL:** [https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html)
- **Supporting URLs:**
  - [https://www.w3.org/TR/trusted-types/](https://www.w3.org/TR/trusted-types/)
  - [https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)
  - [https://web.dev/articles/trusted-types](https://web.dev/articles/trusted-types)

**Specification & Control Detail:**

[SPEC FACT] OWASP recommends textContent for untrusted display data, says innerText or textContent should replace innerHTML for ordinary text updates, and identifies createElement, setAttribute (only for limited non-executable attributes), and appendChild as safer construction primitives. It says to avoid innerHTML, outerHTML, document.write/writeln, and implicit eval paths, and warns that event-handler attributes such as onclick are command-execution contexts. W3C Trusted Types independently identifies these classes as DOM XSS injection sinks. [SUPER-APP CONTROL] Make the host SDK's default rendering helpers use textContent or vetted DOM-node builders for catalog metadata, reviews, permissions, and error text; allow URL-valued attributes only after scheme, origin, and path validation. Reject or gate innerHTML/outerHTML/document.write, eval, Function, string setTimeout/setInterval, javascript: URLs, and event-handler attributes in static analysis and runtime wrappers. Permit an HTML sink only through a reviewed TrustedHTML policy with a pinned sanitizer, and test every package against a DOM-XSS payload corpus in the same sandbox used in production.

**Super-App Applicability:**

container: expose safe DOM APIs and enforce sink telemetry or blocking; store: lint and review package code for dangerous sinks and context-specific encoding mistakes; runtime: render untrusted values as text, restrict executable attributes and dynamic code, and use Trusted Types as a fail-closed backstop.

---

### Finding 27.6: Make Wasm imports the only host capability and keep linear memory app-scoped

- **Finding ID:** `wasm-runtime-capability-memory-isolation-001`
- **Topic:** `wasm/runtime-sandboxing/linear-memory/capability-imports`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** WebAssembly Core Specification §1.1.1 (Design Goals), §1.1.3 (Security Considerations), and §4.2.6 (Module Instances); WebAssembly Web API 2 §6 (Security and Privacy Considerations)
- **Primary URL:** [https://www.w3.org/TR/wasm-core-2/](https://www.w3.org/TR/wasm-core-2/)
- **Supporting URLs:**
  - [https://www.w3.org/TR/wasm-core-1/](https://www.w3.org/TR/wasm-core-1/)
  - [https://www.w3.org/TR/wasm-web-api-2/](https://www.w3.org/TR/wasm-web-api-2/)
  - [https://developer.mozilla.org/en-US/docs/WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)
  - [https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/Memory](https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/Memory)

**Specification & Control Detail:**

[SPEC FACT] Core says validated code executes in a memory-safe, sandboxed environment; a module has no ambient access, and I/O, resources, or operating-system calls can only occur through functions provided by the embedder and imported into the module; the embedder controls or limits those capabilities. The module-instance model collects imported, defined, and exported entities. The JavaScript Memory API permits memory to be imported/exported, while shared memory is an explicit constructor mode. [SUPER-APP CONTROL] Start each mini-app in an app- and tenant-bound Wasm instance with an empty default import set; expose only host-broker functions named in a signed manifest, validate import module/name/signature, and never hand out raw OS handles, host pointers, or a Memory object shared with another app. Use fresh unshared linear memory by default; make shared Memory an explicit capability with owner, scope, lifecycle, and revocation checks; put high-risk packages in a separate process or equivalent OS/hardware boundary because the Core security section leaves embedder isolation and side-channel mitigations to the host.

**Super-App Applicability:**

container: isolate each mini-app runtime and capability namespace; store: require manifest-declared imports and app/tenant ownership for any persisted or shared memory; runtime: deny undeclared imports, raw host handles, cross-app Memory sharing, and side-channel-sensitive execution without an additional OS boundary.

---

### Finding 27.7: Fail closed on out-of-range, empty, or type-mismatched indirect calls

- **Finding ID:** `wasm-runtime-indirect-call-table-safety-002`
- **Topic:** `wasm/tables/call-indirect/type-and-bounds-safety`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** WebAssembly Core Specification 1 §2.3.6 (Table Types), §3.3.5.11 (Validation of call_indirect), and §4.4.5.11 (Execution of call_indirect)
- **Primary URL:** [https://www.w3.org/TR/wasm-core-1/](https://www.w3.org/TR/wasm-core-1/)
- **Supporting URLs:**
  - [https://www.w3.org/TR/wasm-core-2/](https://www.w3.org/TR/wasm-core-2/)
  - [https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/Table](https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/Table)
  - [https://developer.mozilla.org/en-US/docs/WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)

**Specification & Control Detail:**

[SPEC FACT] Core 1 defines a table as an array of opaque values whose funcref entries may have heterogeneous function types and constrains its minimum and optional maximum number of entries. The call_indirect validation rule requires a table and funcref element type. At execution, an index not smaller than the table length traps, an uninitialized entry traps, and a callee whose actual function type differs from the instruction's expected type traps before invocation. MDN also documents that a WebAssembly.Table is accessible and mutable from both JavaScript and Wasm and exposes get, set, and grow operations. [SUPER-APP CONTROL] Allocate a table per mini-app, require an explicit initial/max-entry declaration, and never import a host table containing another app's or the container's callable references. Bind table imports/exports to the owning package and runtime instance, wrap JavaScript get/set/grow paths with ownership and bounds checks, reject unauthorized table mutation, and test all three trap cases plus cross-app function-reference attempts. Treat a table entry as a code capability, not as inert data.

**Super-App Applicability:**

container: prevent cross-runtime table and function-reference sharing; store: lint and sign table initial/max-entry and import declarations; runtime: preserve Wasm bounds, initialization, and signature checks and surface traps as contained app faults rather than host crashes.

---

### Finding 27.8: Gate instantiateStreaming on exact media type, CORS status, and package identity

- **Finding ID:** `wasm-runtime-streaming-instantiation-gate-003`
- **Topic:** `wasm/web-api/instantiate-streaming/mime-cors-integrity`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** WebAssembly Web API 1 §1 (Streaming Module Compilation and Instantiation); current WebAssembly Web API 2 §2 (Streaming Module Compilation and Instantiation)
- **Primary URL:** [https://www.w3.org/TR/wasm-web-api-1/](https://www.w3.org/TR/wasm-web-api-1/)
- **Supporting URLs:**
  - [https://www.w3.org/TR/wasm-web-api-2/](https://www.w3.org/TR/wasm-web-api-2/)
  - [https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/instantiateStreaming](https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/instantiateStreaming)
  - [https://www.w3.org/TR/wasm-core-1/](https://www.w3.org/TR/wasm-core-1/)

**Specification & Control Detail:**

[SPEC FACT] The WebAssembly Web API algorithm for instantiateStreaming compiles a potential WebAssembly Response and then instantiates the resulting module with the import object. It rejects with TypeError when the response is not CORS-same-origin, does not have an ok status, or does not have the exact application/wasm MIME type; extra parameters such as an empty application/wasm; are not allowed. Compilation is asynchronous and may be performed in a streaming manner, while compilation or instantiation failures reject with a CompileError or another relevant error. [SUPER-APP CONTROL] Make the package gateway serve only the registered artifact with Content-Type application/wasm, an approved origin/CORS policy, and an ok response; bind the response URL and digest/signature to the store manifest before allowing compilation, because MIME, CORS, and status checks do not authenticate package identity. Fail closed on redirects or response identities outside the manifest, validate the import object against the capability policy, and cache compiled modules only under artifact digest plus runtime-policy version. Do not treat streaming or browser MIME checks as a substitute for signature verification or host sandboxing.

**Super-App Applicability:**

container: mediate fetch/Response creation and reject untrusted origins or response metadata; store: publish exact Wasm media type and immutable digest/signature metadata; runtime: enforce import policy, contained compile/link/runtime errors, and cache isolation per package and policy version.

---

### Finding 27.9: Allow Wasm compilation narrowly without granting JavaScript eval

- **Finding ID:** `wasm-runtime-csp-execution-policy-004`
- **Topic:** `wasm/csp/script-src/wasm-unsafe-eval`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** W3C Content Security Policy Level 3 §2.3.1 (Source Lists), §4.5.1 (EnsureCSPDoesNotBlockWasmByteCompilation), and §6.1.10 (script-src)
- **Primary URL:** [https://www.w3.org/TR/CSP3/](https://www.w3.org/TR/CSP3/)
- **Supporting URLs:**
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src)
  - [https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/instantiateStreaming](https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/instantiateStreaming)
  - [https://www.w3.org/TR/wasm-web-api-2/](https://www.w3.org/TR/wasm-web-api-2/)

**Specification & Control Detail:**

[SPEC FACT] CSP3 lists new WebAssembly.Module(), compile(), compileStreaming(), instantiate(), and instantiateStreaming() as WebAssembly execution sinks gated by the wasm-unsafe-eval or unsafe-eval source expressions. It defines wasm-unsafe-eval as the more specific keyword: it permits WebAssembly compilation/instantiation without permitting JavaScript eval, whereas unsafe-eval permits both. MDN likewise states that without wasm-unsafe-eval WebAssembly is blocked and that unsafe-eval is broader. [SUPER-APP CONTROL] Default a mini-app policy to omit both keywords so Wasm compilation is blocked; where Wasm is an approved capability, grant only script-src 'wasm-unsafe-eval' to the vetted package and keep unsafe-eval prohibited. Store-lint response and manifest CSP, reject broad wildcard/unsafe-eval policies, test compile and instantiate paths under enforcement and report-only modes, and keep native API/import mediation separate because CSP is a browser execution policy rather than an OS or host-bridge boundary. Treat CSP3's working-draft status as a compatibility input and verify target WebView behavior before release.

**Super-App Applicability:**

container: inject and enforce a per-app CSP on the entry document/WebView; store: require a reviewed CSP declaration and reject unsafe-eval for untrusted packages; runtime: apply the policy to all Wasm compile/instantiate entry points and separately deny undeclared native or broker capabilities.

---

### Finding 27.10: Convert Wasm page limits into enforced per-app and aggregate host memory budgets

- **Finding ID:** `wasm-runtime-host-memory-budget-005`
- **Topic:** `wasm/memory-ceilings/host-quota/memory-grow`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** WebAssembly Core Specification §1.1.2 (Scope), §2.3.12 (Limits), §2.3.15 (Memory Types), §2.4.5 (Memory Instructions), §2.5.5 (Memories), and §4.2.9 (Memory Instances); Core 1 §2.3.4–§2.3.5 and §2.4.4
- **Primary URL:** [https://www.w3.org/TR/wasm-core-2/](https://www.w3.org/TR/wasm-core-2/)
- **Supporting URLs:**
  - [https://www.w3.org/TR/wasm-core-1/](https://www.w3.org/TR/wasm-core-1/)
  - [https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/Memory](https://developer.mozilla.org/en-US/docs/WebAssembly/JavaScript_interface/Memory)
  - [https://www.w3.org/TR/wasm-web-api-2/](https://www.w3.org/TR/wasm-web-api-2/)

**Specification & Control Detail:**

[SPEC FACT] Core 2 makes the memory minimum the initial size and an optional maximum the size ceiling to which the memory can grow; limits are in page units, a page is 65,536 bytes, and the memory-instance invariant says the byte length never exceeds the declared maximum. memory.grow returns the previous size or -1 when enough memory cannot be allocated. If no maximum is present, the specification permits growth to any valid size, and the core specification leaves environment-specific resource policy outside its scope. MDN describes Memory.grow in 64 KiB pages and shows separate initial and maximum values. [SUPER-APP CONTROL] Require every submitted memory and imported Memory object to declare a finite maximum; reject missing or excessive maxima before instantiation, charge initial pages and every growth request against an app, publisher/tenant, and global budget, and make the engine/host adapter deny growth before the aggregate budget is crossed. Account Wasm linear memory separately from JavaScript heap, tables, compiled-code cache, and process RSS; use a process/cgroup/microVM ceiling for high-risk apps, rate-limit repeated growth failures, emit non-secret quota telemetry, and terminate or quarantine an app that exceeds policy. A Wasm per-memory maximum is necessary but is not a whole-container physical-memory limit.

**Super-App Applicability:**

container: enforce runtime and process-level memory ceilings per mini-app and aggregate tenant; store: lint finite initial/max page declarations and bind them to the artifact digest; runtime: meter instantiation and memory.grow, deny over-budget growth, isolate JS/compiled-cache overhead, and preserve the -1/error path without allowing host exhaustion.

---

### Finding 27.11: COOP same-origin plus COEP require-corp or credentialless is the cross-origin-isolation header gate

- **Finding ID:** `coi_timing_027_01`
- **Topic:** `cross_origin_isolation_header_pair`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** WHATWG HTML Standard §7.1.3 Cross-origin opener policies (same-origin-plus-COEP and §7.1.3.1 The headers); §7.1.4 Cross-origin embedder policies (compatible with cross-origin isolation) and §7.1.4.1 The headers
- **Primary URL:** [https://html.spec.whatwg.org/multipage/origin.html](https://html.spec.whatwg.org/multipage/origin.html)
- **Supporting URLs:**
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy)
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)
  - [https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated](https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated)

**Specification & Control Detail:**

[SPEC FACT] WHATWG defines same-origin-plus-COEP as the cross-origin-isolation mode produced by Cross-Origin-Opener-Policy: same-origin together with a Cross-Origin-Embedder-Policy value compatible with isolation; require-corp and credentialless are the compatible COEP values. MDN further states that Permissions-Policy: cross-origin-isolated must not block the feature. [SUPER-APP CONTROL] Treat the tuple COOP: same-origin + COEP: require-corp (strict resource opt-in) or credentialless (no-CORS credentials stripped) as an explicit runtime capability profile, not as independent optional headers. The container must emit and preserve both headers on the mini-app top-level response, keep the policy through redirects and worker/iframe initialization, and report an isolation failure instead of exposing privileged APIs when the tuple or permission gate is absent.

**Super-App Applicability:**

Container: owns the top-level response headers, secure-context requirement, redirects, iframe/worker policy propagation, and Permissions Policy. Store: declare whether an app requires the isolated profile and test its dependency graph against that profile. Runtime: expose isolated-only APIs only after the effective context reports the capability.

---

### Finding 27.12: COEP require-corp makes CORP or CORS an explicit opt-in for cross-origin no-CORS resources

- **Finding ID:** `coi_timing_027_02`
- **Topic:** `coep_corp_resource_admission`
- **Evidence Level:** `industry_standard`
- **Standard Reference:** MDN Cross-Origin-Embedder-Policy, Directives (require-corp and credentialless) and Description; MDN Cross-Origin-Resource-Policy, Syntax/Directives; WHATWG HTML Standard §7.1.4 Cross-origin embedder policies and §7.1.4.2 Embedder policy checks
- **Primary URL:** [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)
- **Supporting URLs:**
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy)
  - [https://html.spec.whatwg.org/multipage/origin.html](https://html.spec.whatwg.org/multipage/origin.html)
  - [https://web.dev/articles/cross-origin-isolation-guide](https://web.dev/articles/cross-origin-isolation-guide)

**Specification & Control Detail:**

[SPEC FACT] Under COEP: require-corp, a no-CORS cross-origin resource is admitted only when it is same-origin or its response explicitly permits embedding with Cross-Origin-Resource-Policy; a CORS-mode request is governed by CORS instead. CORP: same-origin, same-site, and cross-origin respectively restrict or permit which origins/sites may load a resource. COEP: credentialless permits no-CORS cross-origin loads by omitting credentials, while other credentialed cases still need CORS or CORP. [SUPER-APP CONTROL] Require every mini-app script, stylesheet, image, worker script, iframe, and other dependency to declare its intended CORS/CORP mode in a resource manifest; fail closed or use an explicitly approved credentialless profile when a dependency lacks the required opt-in. Test nested frames and worker initialization, and surface the blocked URL and policy disposition to publishers rather than silently weakening COEP.

**Super-App Applicability:**

Container: enforces subresource admission and nested-frame/worker inheritance. Store: scans declared and discovered cross-origin dependencies and records CORP/CORS evidence per release. Runtime: blocks non-opted-in resources under require-corp and prevents an app from bypassing the platform's isolation policy by loading opaque third-party assets.

---

### Finding 27.13: crossOriginIsolated is the effective Window/Worker status bit for isolated execution

- **Finding ID:** `coi_timing_027_03`
- **Topic:** `cross_origin_isolated_runtime_status`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** HTML Standard API definition #dom-crossoriginisolated-dev; MDN Window: crossOriginIsolated property, Cross-origin isolating a document and Checking if the document is cross-origin isolated
- **Primary URL:** [https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated](https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated)
- **Supporting URLs:**
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy)
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)
  - [https://web.dev/articles/cross-origin-isolation-guide](https://web.dev/articles/cross-origin-isolation-guide)

**Specification & Control Detail:**

[SPEC FACT] Window.crossOriginIsolated returns a boolean indicating whether the document is cross-origin isolated; the corresponding WorkerGlobalScope property can be checked in workers. The effective status requires COOP: same-origin, COEP: require-corp or credentialless, and permission for the cross-origin-isolated feature; otherwise the property is false and the reduced restrictions do not apply. [SUPER-APP CONTROL] Make this status the runtime source of truth for each mini-app window and worker. Do not infer capability from a stored app declaration or from seeing headers in a manifest: redirects, embedding, a failed subresource check, a worker policy mismatch, or host Permissions Policy can make the effective status false. Gate SAB, high-resolution timing, and other isolation-dependent APIs on the live status and provide a deterministic non-isolated fallback/error state.

**Super-App Applicability:**

Container: exposes/records Window and Worker isolation status and controls Permissions Policy. Store: validates an app's declared isolation requirement but does not treat declaration as proof. Runtime: checks the boolean before privileged API use and records the status with diagnostics for support and audit.

---

### Finding 27.14: SharedArrayBuffer creation and message-based shared memory require the isolated app/worker boundary

- **Finding ID:** `coi_timing_027_04`
- **Topic:** `shared_array_buffer_memory_sharing_gate`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** MDN Window: crossOriginIsolated property, Cross-origin isolated documents operate with fewer restrictions; WHATWG HTML Standard §7.1.4.2 Embedder policy checks (worker initialization); MDN COEP, Features that depend on cross-origin isolation
- **Primary URL:** [https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated](https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated)
- **Supporting URLs:**
  - [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)
  - [https://html.spec.whatwg.org/multipage/origin.html](https://html.spec.whatwg.org/multipage/origin.html)
  - [https://web.dev/articles/cross-origin-isolation-guide](https://web.dev/articles/cross-origin-isolation-guide)

**Specification & Control Detail:**

[SPEC FACT] MDN identifies SharedArrayBuffer as an API with reduced restrictions in a cross-origin-isolated document: it can be created and sent through Window.postMessage() or MessagePort.postMessage(), while the non-isolated example uses ArrayBuffer. WHATWG's embedder-policy check applies the owner's policy to dedicated-worker initialization, so an isolated page cannot assume that an unapproved worker script belongs to the same isolated execution boundary. [SUPER-APP CONTROL] Permit SAB allocation and sharing only inside a mini-app's verified isolated scope; require all dedicated-worker scripts and nested frames participating in that scope to pass the same embedder-policy checks. Never pass a SAB to an untrusted cross-app/plugin boundary; use ArrayBuffer copying or a serialized protocol when isolation is false or cannot be proven, and surface the exact fallback rather than silently downgrading shared-memory semantics.

**Super-App Applicability:**

Container: isolates app/worker agent boundaries and controls postMessage/MessagePort exposure between apps. Store: scans worker and iframe dependencies and marks shared-memory capability as an explicit permission requiring isolation evidence. Runtime: gates SAB construction and cross-context sharing, with ArrayBuffer/serialization fallback for ordinary mini-apps.

---

### Finding 27.15: performance.now() is coarsened for timing-attack resistance and gains finer resolution only with isolation

- **Finding ID:** `coi_timing_027_05`
- **Topic:** `high_resolution_timer_coarsening`
- **Evidence Level:** `normative_standard`
- **Standard Reference:** High Resolution Time Level 3 §4 Time Origin (coarsen time algorithm), §7.1 now() method, and §9.1 Clock resolution
- **Primary URL:** [https://w3c.github.io/hr-time/](https://w3c.github.io/hr-time/)
- **Supporting URLs:**
  - [https://developer.mozilla.org/en-US/docs/Web/API/Performance/now](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now)
  - [https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated](https://developer.mozilla.org/en-US/docs/Web/API/crossOriginIsolated)
  - [https://html.spec.whatwg.org/multipage/origin.html](https://html.spec.whatwg.org/multipage/origin.html)

**Specification & Control Detail:**

[SPEC FACT] High Resolution Time Level 3 defines coarsen time with a default resolution of 100 microseconds or a higher implementation-defined value; when the cross-origin-isolated capability is true, the resolution is 5 microseconds or a higher implementation-defined value, and the user agent may coarsen and potentially jitter timestamps. The security considerations identify cache, statistical-fingerprinting, and micro-architectural timing attacks and list resolution reduction and jitter as mitigations. performance.now() remains a monotonic elapsed-time API, not a promise of an exact 5-microsecond clock. [SUPER-APP CONTROL] Treat timer precision as an effective runtime property, never as a fixed entitlement: record whether the mini-app was isolated, avoid using timer deltas as authorization/fraud/security proofs, and prevent cross-app timing correlation in telemetry. Performance tests should tolerate implementation-defined coarsening, background throttling, and jitter; publish separate isolated and non-isolated benchmark classes.

**Super-App Applicability:**

Container: selects the isolation profile and must not expose a synthetic higher-resolution timer that defeats browser coarsening. Store: labels benchmark requirements and rejects apps that require an exact timer precision as a security primitive. Runtime: returns platform-controlled performance.now() values, records isolation context for diagnostics, and keeps timing data scoped to the app/worker rather than correlating unrelated apps.

---
