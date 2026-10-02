# Iteration 3 — Accessibility, Privacy and Marketplace Governance

This section preserves all 20 normalized findings appended in iteration 3. Four DSA article-level candidate records were consolidated because they cite one official regulation URL; their article-specific source facts and proposed controls remain in the consolidated detail. Parent verification rechecked 25 unique URLs and received HTTP 200 for every URL.

## Accessibility and inclusive UX

### Finding 145: catalog_discovery_reflow — official-normative

- **Source:** W3C WCAG 2.2 SC 1.4.10 Reflow

- **Detail:** [SOURCE FACT] WCAG 2.2 SC 1.4.10 (Level AA) requires content to be presented without loss of information or functionality and without two-dimensional scrolling at 320 CSS pixels wide (or 256 CSS pixels high for horizontally scrolling content), except parts that require two-dimensional layout. [MINI-APP PROPOSAL] Make the catalog home, search results, category and filter panels, app cards, and app-detail pages reflow at the equivalent of 400% zoom; keep search, sort/filter, ratings, permissions, version/update/takedown status, and install/update controls operable without horizontal page scrolling. Treat a game canvas, map, or data table as a documented two-dimensional exception only when its meaning or usage requires it. [EVIDENCE LEVEL] Official W3C normative Level AA success criterion; proposal is implementation guidance.

- **URL(s):** [https://www.w3.org/TR/WCAG22/#reflow](https://www.w3.org/TR/WCAG22/#reflow)

### Finding 146: app_detail_text_resize — official-normative

- **Source:** W3C WCAG 2.2 SC 1.4.4 Resize Text

- **Detail:** [SOURCE FACT] WCAG 2.2 SC 1.4.4 (Level AA) requires text, except captions and images of text, to be resizable up to 200 percent without loss of content or functionality. [MINI-APP PROPOSAL] Make app names, descriptions, ratings, permissions, version history, release notes, update instructions, error recovery text, and takedown notices usable at 200% text size and at platform accessibility font-scale settings; reflow cards and controls instead of clipping, truncating essential meaning, or hiding the primary action. Test both browser zoom and native text-size settings on supported hosts. [EVIDENCE LEVEL] Official W3C normative Level AA success criterion; proposal is implementation guidance.

- **URL(s):** [https://www.w3.org/TR/WCAG22/#resize-text](https://www.w3.org/TR/WCAG22/#resize-text)

### Finding 147: catalog_search_filter_screen_reader_semantics — official-primary-guidance

- **Source:** W3C WAI-ARIA Authoring Practices Guide - Combobox Pattern

- **Detail:** [SOURCE FACT] The WAI-ARIA Authoring Practices Guide describes a combobox as an input with an associated popup and specifies the combobox role, aria-expanded state, aria-controls relationship, popup roles such as listbox/grid/tree/dialog, and keyboard focus or active-descendant behavior. [MINI-APP PROPOSAL] Use native inputs, selects, and buttons where possible; for custom catalog search and single-select filters, expose stable accessible names, role/state/value, result count, current filter, and expanded/collapsed state. Support keyboard typing, Escape, arrow navigation, selection, and restoration of focus, and announce result/filter changes through status semantics rather than visual movement alone. [EVIDENCE LEVEL] Official W3C primary accessibility guidance; APG is informative implementation guidance rather than a standalone WCAG conformance claim.

- **URL(s):** [https://www.w3.org/WAI/ARIA/apg/patterns/combobox/](https://www.w3.org/WAI/ARIA/apg/patterns/combobox/)

### Finding 148: install_update_progress_and_result_announcements — official-normative

- **Source:** W3C WCAG 2.2 SC 4.1.3 Status Messages

- **Detail:** [SOURCE FACT] WCAG 2.2 SC 4.1.3 (Level AA) requires status messages to be programmatically determinable through roles or properties so assistive technologies can present them without receiving focus. [MINI-APP PROPOSAL] Expose install, update, download, verification, queued, paused, resumed, completed, failed, rollback, and takedown states as concise status messages; include the app and version plus the actionable next step, rate-limit progress announcements, do not steal focus, and provide a determinate percentage or remaining work when available with a text alternative when not. [EVIDENCE LEVEL] Official W3C normative Level AA success criterion; proposal is implementation guidance.

- **URL(s):** [https://www.w3.org/TR/WCAG22/#status-messages](https://www.w3.org/TR/WCAG22/#status-messages)

### Finding 149: permission_error_and_takedown_dialogs — official-normative+official-primary-guidance

- **Source:** W3C WCAG 2.2 SC 3.3.1 Error Identification + W3C WAI-ARIA APG Modal Dialog Pattern

- **Detail:** [SOURCE FACT - WCAG] When an input error is automatically detected, WCAG 2.2 SC 3.3.1 requires the item in error to be identified and the error described to the user in text. [SOURCE FACT - APG] A modal dialog makes content outside it inert, keeps Tab and Shift+Tab within the dialog, moves focus inside when it opens, and exposes a dialog name or description through ARIA properties; Escape closes it where appropriate and focus returns to the invoking context. [MINI-APP PROPOSAL] Use an accessible, labeled permission or consent dialog for each high-risk capability and a confirmation or notice dialog for uninstall, revoke, quarantine, or takedown. Associate field-level errors with controls, state what failed, provide recovery, appeal, or contact actions, preserve the invoking app or card context, and never make a background page appear active to visual or assistive-technology users. [EVIDENCE LEVEL] The WCAG portion is an official normative Level A success criterion; the APG portion is official W3C primary guidance, informative implementation guidance.

- **URL(s):** [https://www.w3.org/TR/WCAG22/#error-identification](https://www.w3.org/TR/WCAG22/#error-identification), [https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

### Finding 150: contrast_and_keyboard_focus — official-normative

- **Source:** W3C WCAG 2.2 SC 1.4.11 Non-text Contrast + SC 2.4.7 Focus Visible

- **Detail:** [SOURCE FACT] WCAG 2.2 SC 1.4.11 (Level AA) requires visual information needed to identify user-interface components and states to have at least 3:1 contrast against adjacent colors, except inactive or user-agent-determined components; SC 2.4.7 (Level AA) requires a keyboard focus indicator to be visible. [MINI-APP PROPOSAL] Apply these requirements to catalog cards, selected filters, install/update buttons, progress bars, permission and takedown banners, error states, and focus rings across light, dark, and high-contrast themes. Never use color alone for installed, updated, blocked, or removed state; add text, icon, shape, or status semantics, and test focused and unfocused states. [EVIDENCE LEVEL] Official W3C normative Level AA success criteria; proposal is implementation guidance.

- **URL(s):** [https://www.w3.org/TR/WCAG22/#non-text-contrast](https://www.w3.org/TR/WCAG22/#non-text-contrast), [https://www.w3.org/TR/WCAG22/#focus-visible](https://www.w3.org/TR/WCAG22/#focus-visible)

### Finding 151: motion_and_time_limit_controls — official-normative

- **Source:** W3C WCAG 2.2 SC 2.2.1 Timing Adjustable + SC 2.2.2 Pause, Stop, Hide + SC 2.3.3 Animation from Interactions

- **Detail:** [SOURCE FACT] WCAG 2.2 SC 2.2.1 requires users to be able to turn off or adjust content-set time limits, or receive a warning and repeated extension, subject to stated exceptions; SC 2.2.2 requires a pause, stop, hide, or update-frequency control for moving, blinking, scrolling, or auto-updating information unless essential; SC 2.3.3 says interaction-triggered motion animation can be disabled unless it is essential (Level AAA). [MINI-APP PROPOSAL] Do not expire install, update, permission, appeal, or takedown steps while focus is in them; warn and extend session timeouts, preserve state after reconnect, provide pause/cancel for downloads and auto-refresh, honor reduced-motion settings by replacing nonessential transitions or spinners with stable progress/status, and never convey a deadline only through animation. [EVIDENCE LEVEL] Official W3C normative criteria (Level A for 2.2.1 and 2.2.2; Level AAA for 2.3.3); proposal is implementation guidance.

- **URL(s):** [https://www.w3.org/TR/WCAG22/#timing-adjustable](https://www.w3.org/TR/WCAG22/#timing-adjustable), [https://www.w3.org/TR/WCAG22/#pause-stop-hide](https://www.w3.org/TR/WCAG22/#pause-stop-hide), [https://www.w3.org/TR/WCAG22/#animation-from-interactions](https://www.w3.org/TR/WCAG22/#animation-from-interactions)

### Finding 152: localized_catalog_and_lifecycle_messages — official-normative

- **Source:** W3C WCAG 2.2 SC 3.1.1 Language of Page + SC 3.1.2 Language of Parts

- **Detail:** [SOURCE FACT] WCAG 2.2 requires the default human language of a page to be programmatically determinable (SC 3.1.1 Level A) and the language of each passage or phrase to be programmatically determinable except for listed exceptions (SC 3.1.2 Level AA). [MINI-APP PROPOSAL] Make locale explicit for store chrome, catalog metadata, app descriptions, reviews, permission text, progress and error messages, and takedown notices; mark language changes in mixed-language strings, preserve script and bidirectional text, localize dates, numbers, and action labels, and provide a predictable fallback when a translation is missing. Do not silently substitute an untranslated safety or permission message. [EVIDENCE LEVEL] Official W3C normative Level A and Level AA success criteria; localization and fallback rules are store proposals.

- **URL(s):** [https://www.w3.org/TR/WCAG22/#language-of-page](https://www.w3.org/TR/WCAG22/#language-of-page), [https://www.w3.org/TR/WCAG22/#language-of-parts](https://www.w3.org/TR/WCAG22/#language-of-parts)

## Privacy and data governance

### Finding 153: purpose_minimization_and_storage_baseline — official-guidance

- **Source:** European Data Protection Board, Basic principles

- **Detail:** [SOURCE FACT] The EDPB identifies purpose limitation, data minimisation, storage limitation, integrity and confidentiality, and accountability as GDPR processing principles; controllers must be able to demonstrate compliance. [SCOPE/LIMITATION] This is EDPB guidance on GDPR concepts, not a universal law or a mini-app manifest specification; applicability depends on the processing and jurisdiction. [PROPOSED STORE CONTROL] Require a versioned per-mini-app data inventory listing each field/capability, purpose, legal basis, recipient, retention class, and deletion or anonymisation action; reject collection not tied to a declared purpose, default telemetry to minimal non-identifying fields, and trigger privacy review when a purpose or field changes.

- **URL(s):** [https://www.edpb.europa.eu/topics/key-gdpr-concepts/basic-principles_en](https://www.edpb.europa.eu/topics/key-gdpr-concepts/basic-principles_en)

### Finding 154: consent_withdrawal_and_granularity — official-guidance

- **Source:** EDPB Guidelines 05/2020 on consent under Regulation 2016/679

- **Detail:** [SOURCE FACT] The EDPB describes consent as freely given, specific, informed, and unambiguous; controllers must demonstrate it. Withdrawal must be possible at any time and as easily as consent was given, through the same service interface where appropriate and without detriment; after withdrawal, consent-based processing should stop and data should be deleted when no other lawful basis justifies continued processing. [SCOPE/LIMITATION] This guidance applies when consent is the chosen GDPR lawful basis; it does not mean every telemetry operation requires consent, and other jurisdictions or legal bases may differ. [PROPOSED STORE CONTROL] Record consent per purpose, data category, app release, SDK/processor, interface, and timestamp; provide equal accept/decline choices and one-step host-level revoke; propagate revocation to the mini-app, SDKs, processors, and stores; map each purpose to its fallback retention rule; and require fresh review when a release expands a purpose or data category.

- **URL(s):** [https://www.edpb.europa.eu/system/files/documents/files/file1/edpb_guidelines_202005_consent_en.pdf](https://www.edpb.europa.eu/system/files/documents/files/file1/edpb_guidelines_202005_consent_en.pdf)

### Finding 155: deletion_export_and_propagation — official-guidance

- **Source:** EDPB Data protection guide for small business, Respect individuals’ rights

- **Detail:** [SOURCE FACT] The EDPB guide lists access, erasure, and data portability rights; says controllers must facilitate requests and processors must assist; describes erasure grounds including no longer necessary data and withdrawn consent without another legal basis; and describes portability as structured, commonly used, machine-readable data, with direct transfer to another controller where technically possible. It also advises tracking requests and informing recipients where required. [SCOPE/LIMITATION] These are GDPR rights and conditions, so exceptions and applicability must be assessed per request; the guide is not a universal export format or deletion SLA. [PROPOSED STORE CONTROL] Provide a host/store rights center with per-app export in JSON or CSV plus metadata and source labels; implement deletion fan-out to the publisher, host, processors, subprocessors, replicas, caches, and telemetry stores; keep a restricted tombstone/status record, legal-hold exception, recipient notification evidence, and an applicable response deadline without retaining the deleted payload.

- **URL(s):** [https://www.edpb.europa.eu/sme/be-compliant/respect-individuals-rights_en](https://www.edpb.europa.eu/sme/be-compliant/respect-individuals-rights_en)

### Finding 156: processor_subprocessor_registry_and_change_notice — official-guidance

- **Source:** EDPB Guidelines 07/2020 on the concepts of controller and processor in the GDPR

- **Detail:** [SOURCE FACT] The EDPB says a processor acts on documented controller instructions; another processor requires prior written authorisation, and under general authorisation the processor must inform the controller of subprocessor changes and provide an opportunity to object. The guidance says an intended-subprocessor list should include locations, activities, and safeguards; processors assist with rights and, on termination, return or delete personal data and existing copies. [SCOPE/LIMITATION] This is role and contract guidance within the GDPR/EEA context; the actual controller or processor role depends on who determines purposes and means, and it is not a universal vendor-procurement rule. [PROPOSED STORE CONTROL] Require a data-flow registry for each mini-app covering controller/processor roles, fields, purposes, locations, access, retention, SDKs, and subprocessors; block release of undeclared collection; require written authorization and advance change notice with an objection/re-review window; and collect deletion, rights-assistance, and offboarding evidence from every processor tier.

- **URL(s):** [https://www.edpb.europa.eu/system/files/2023-10/EDPB_guidelines_202007_controllerprocessor_final_en.pdf](https://www.edpb.europa.eu/system/files/2023-10/EDPB_guidelines_202007_controllerprocessor_final_en.pdf)

### Finding 157: telemetry_profile_and_lifecycle_governance — official-standard

- **Source:** NIST Privacy Framework 1.1, Using Privacy Framework 1.1

- **Detail:** [SOURCE FACT] NIST describes Current and Target Profiles for privacy outcomes, using Identify-P and Govern-P to identify processing, privacy risks, values, and legal requirements; it aligns Target Profiles with plan, design, build/buy, deploy, operate, and decommission phases, and calls for continuous assessment and profile updates as risks or business objectives change. It also describes expressing privacy requirements to external service providers and verifying them through agreements and assessments. [SCOPE/LIMITATION] The NIST Privacy Framework is a voluntary risk-management framework, not a law or telemetry-specific standard; it supplies no universal event fields, retention period, or consent rule. [PROPOSED STORE CONTROL] Create a per-mini-app telemetry Profile containing event taxonomy, purpose, legal basis or consent, identifiers/linkability, recipients, region, aggregation, retention, access, export, deletion, and sampling; default to no user identifiers or free-form payloads; version it with the package and host SDK; and require diff-based re-review before new telemetry is emitted.

- **URL(s):** [https://www.nist.gov/privacy-framework/using-privacy-framework-11](https://www.nist.gov/privacy-framework/using-privacy-framework-11)

### Finding 158: retention_schedule_and_deletion_verification — official-regulator-guidance

- **Source:** UK Information Commissioner’s Office, Principle (e): Storage limitation

- **Detail:** [SOURCE FACT] The ICO says personal data must not be kept longer than needed, retention duration must be justified by the purpose, standard retention periods should be documented where possible, and data should be periodically reviewed and erased or anonymised when no longer needed. It says UK GDPR sets no fixed time limits, distinguishes taking data offline from deleting it, and recommends coordinating deletion across organisations that hold shared copies. [SCOPE/LIMITATION] This is UK ICO guidance on the UK GDPR, and the page warns that it is under review; it does not set a universal retention duration for mini-apps or other jurisdictions. [PROPOSED STORE CONTROL] Assign each data, log, cache, backup, and telemetry class an owner, purpose, retention rule, review cadence, expiry action, and legal-hold exception; automate expiry/anonymisation; verify deletion across replicas and processors; and treat offline or suppressed records as retained until they are either deleted or irreversibly anonymised.

- **URL(s):** [https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-protection-principles/a-guide-to-the-data-protection-principles/storage-limitation/](https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/data-protection-principles/a-guide-to-the-data-protection-principles/storage-limitation/)

### Finding 159: high_risk_privacy_review_trigger — official-guidance

- **Source:** European Data Protection Board, Data protection impact assessment

- **Detail:** [SOURCE FACT] The EDPB says DPIAs help identify and manage risks to people’s personal data and must be carried out before processing likely to result in high risk to individuals’ rights and freedoms; if risks cannot be mitigated by appropriate measures, the controller should consult the data protection authority before proceeding. [SCOPE/LIMITATION] This is a GDPR/DPIA control point, not a universal risk taxonomy or a finding that any particular mini-app data type automatically requires a DPIA; applicable authority lists and local law matter. [PROPOSED STORE CONTROL] Make risk review a release gate and trigger it for declared child, health, biometric, precise-location, large-scale monitoring/profiling, identity-linkage, novel SDK, or new-region processing; require a DPIA or documented no-DPIA decision before approval, repeat it when fields, purposes, recipients, telemetry, permissions, or risk signals change, and block promotion when residual high risk is unresolved.

- **URL(s):** [https://www.edpb.europa.eu/topics/accountability-and-compliance-tools/data-protection-impact-assessment_en](https://www.edpb.europa.eu/topics/accountability-and-compliance-tools/data-protection-impact-assessment_en)

### Finding 160: child_data_and_age_assurance_safeguards — official-guidance

- **Source:** European Data Protection Board, Children

- **Detail:** [SOURCE FACT] The EDPB states that children receive specific protection under the GDPR because they are particularly vulnerable; organisations should take extra care, provide clear, understandable and age-appropriate information, and ensure protection when implementing age-assurance mechanisms. [SCOPE/LIMITATION] This is EDPB guidance on children and GDPR concepts; it does not set a universal age threshold, prescribe one age-assurance technology, or mean every mini-app is child-directed. [PROPOSED STORE CONTROL] Require an audience and age-band declaration in the manifest; route child-directed or child-data apps to enhanced review; use minimal, purpose-limited age assurance; provide age-appropriate privacy notices and consent/withdrawal paths; default off targeted profiling and non-essential telemetry in child contexts unless legal review approves it; and test deletion/export handling for child records.

- **URL(s):** [https://www.edpb.europa.eu/topics/key-gdpr-concepts/children_en](https://www.edpb.europa.eu/topics/key-gdpr-concepts/children_en)

## Marketplace governance, moderation and appeals

### Finding 161: notice_and_action_intake — official-normative

- **Source:** European Union — Regulation (EU) 2022/2065 (Digital Services Act), Article 16

- **Detail:** EU legal control: Article 16 requires hosting services to provide easy-to-access, user-friendly electronic notice mechanisms for specific suspected illegal content. The mechanism must facilitate precise, substantiated notices with reasons, the exact electronic location, notifier name/email (subject to the Directive 2011/93/EU exception), and a good-faith accuracy/completeness statement. When contact is supplied, the provider must confirm receipt without undue delay and notify the decision and redress possibilities without undue delay; notices and decisions must be processed timely, diligently, non-arbitrarily and objectively, with automated use disclosed. Implementation candidate: store notice_id, app/listing/version/content locator, allegation and evidence, notifier contact/attestation, received/acknowledgement/decision timestamps, decision/reason, automation flag, and redress link in an append-only case record. Scope: the DSA applies to intermediary services offered to recipients established or located in the EU regardless of provider location; Article 16 is for hosting services. Limitation: legal classification, exemptions, and sector or national rules need counsel; the proposed fields are an implementation pattern, not a universal requirement.\n\n[ADDITIONAL ARTICLE-LEVEL CONTROL] statement_of_reasons_and_internal_appeal:\nEU legal control: Article 17 requires hosting services to give affected recipients a clear, specific statement of reasons for removal, disabling access, demotion or other visibility restriction, monetary-payment restriction, service suspension or termination, or account suspension or termination, where relevant electronic contact details are known and at the latest when the restriction is imposed. The reasons cover the measure, territorial scope and duration where relevant, facts and circumstances, notice versus own-initiative basis, automated means, legal or contractual ground, and clear redress routes. Article 20 requires online platforms to offer for at least six months an electronic, free, easy and user-friendly internal complaint system; complaints must be handled timely, non-discriminatorily, diligently and non-arbitrarily, with reversal without undue delay when grounds warrant, a reasoned outcome and out-of-court options, under appropriately qualified staff rather than solely automated decision-making. Implementation candidate: issue a structured decision receipt with reason code, evidence summary, scope/duration, automation flag, reviewer role and appeal deadline; retain the original artifact and superseded decision. Scope: EU DSA duties for hosting services and, for Article 20, online platforms. Limitation: the exact provider category and other redress or notice obligations require legal review; the six-month period is a statutory minimum, not a universal service-level target or guarantee of reversal.\n\n[ADDITIONAL ARTICLE-LEVEL CONTROL] proportionate_suspension_for_misuse:\nEU online-platform control against misuse: after a prior warning, providers must suspend for a reasonable period service to recipients who frequently provide manifestly illegal content, and may suspend for a reasonable period the processing of notices or complaints from people or entities that frequently submit manifestly unfounded notices or complaints. Suspension decisions must be case-by-case, timely, diligent and objective, considering at least absolute count, relative proportion within a time frame, gravity and consequences, and identifiable intent; the policy and examples of factors and suspension duration must be clear and detailed in the terms and conditions. Implementation candidate: use a progressive, scope-limited state machine (warning -> temporary restriction -> review -> restore or escalate), keep content takedown distinct from reporter/appeal-channel abuse, and record counts, ratios, severity, intent evidence, duration, warning and review decisions. Scope: Article 23 is an EU DSA rule for online platforms. Limitation: it does not supply universal numeric thresholds or authorize indefinite suspension; what is manifestly illegal or unfounded and what period is reasonable depend on applicable law and facts.\n\n[ADDITIONAL ARTICLE-LEVEL CONTROL] advertising_identity_and_paid_placement_separation:\nEU advertising control, distinct from ranking-parameter disclosure: online platforms that present advertisements must let each recipient identify in clear, concise, unambiguous and real time that content is an advertisement, the person on whose behalf it is presented, the payer when different, and meaningful information about the main targeting parameters and how to change them where applicable. Platforms must also let users declare commercial communications, which must then be marked for other recipients, and may not target ads using profiling based on special categories of personal data. Implementation candidate: make placement_type explicit (organic, editorial or sponsored), render a prominent ad marker, expose sponsor and payer identity and applicable targeting explanation, retain campaign/time and user-declaration events, and keep paid placement auditable and visibly distinct from organic catalog results. Scope: Article 26 is an EU DSA rule for online platforms presenting ads. Limitation: it does not prescribe a particular ranking algorithm or by itself settle every advertising, privacy or consumer-law issue; the separation and metadata contract are design proposals derived from the identification duty, not a global legal requirement.

- **URL(s):** [https://publications.europa.eu/resource/celex/32022R2065.ENG.xhtml.L_2022277EN.01000101.doc.html](https://publications.europa.eu/resource/celex/32022R2065.ENG.xhtml.L_2022277EN.01000101.doc.html)

### Finding 162: publisher_complaint_handling_and_mediation — official-normative

- **Source:** European Union — Regulation (EU) 2019/1150 (Platform-to-Business Regulation), Articles 1, 11 and 12

- **Detail:** EU publisher/business-user redress control: the P2B Regulation covers online intermediation services and search engines offered to business users or corporate website users established or resident in the EU who offer goods or services to consumers in the EU, regardless of provider location; it excludes certain online payment services and advertising tools or exchanges. Article 11 requires an accessible, free internal complaint system with reasonable-time handling, transparency and equal treatment for equivalent situations, proportionality, direct complaints about regulatory, technical or provider-behaviour issues, individualized plain-language outcomes, terms-and-conditions access information, and public effectiveness information verified at least annually, including total complaints, main types, average processing time and aggregated outcomes; small-enterprise providers are exempt. Article 12 requires providers other than small enterprises to identify at least two mediators in their terms and conditions and sets independence, affordability, language/access, speed, good-faith and cost-sharing safeguards. Implementation candidate: separate publisher appeals from consumer content reports; preserve listing/version, policy decision, contract or technical issue, evidence, outcome, handling time and mediator route. Scope and limitation: this is EU-scoped business-user/platform redress, not a general consumer-appeal or global mini-app-store requirement; the small-enterprise and service-type exclusions matter.

- **URL(s):** [https://publications.europa.eu/resource/celex/32019R1150.ENG.xhtml.L_2019186EN.01005701.doc.html](https://publications.europa.eu/resource/celex/32019R1150.ENG.xhtml.L_2019186EN.01005701.doc.html)

### Finding 163: moderation_transparency_and_auditability — official-guidance

- **Source:** European Commission — DSA Transparency Database Questions and Answers

- **Detail:** The Commission explains that DSA Article 17 statements of reasons cover hosting-service restrictions, and Article 24(5) requires online platforms, a subset of hosting services, to send all such statements to the Commission's DSA Transparency Database. The database is publicly accessible and machine-readable; people can search, read and download statements. The Commission says public submissions must remove personal data and that redress options are not included because they are relevant to the addressee. The FAQ also documents API or webform submission after platform onboarding and sandbox testing. Implementation candidate: maintain an append-only moderation-decision ledger and versioned export serializer, separate private notice and appeal contact data from public reason records, redact personal data, retain delivery receipts and schema versions, and publish aggregate counts for action, reason, automation, appeal reversal, handling time and suspension type. Scope: this is EU Commission guidance and infrastructure for platforms subject to DSA duties. Limitation: applicability and onboarding depend on legal classification; the public export is not permission to disclose personal data, and this EU reporting pattern is not a universal marketplace mandate.

- **URL(s):** [https://digital-strategy.ec.europa.eu/en/faqs/dsa-transparency-database-questions-and-answers](https://digital-strategy.ec.europa.eu/en/faqs/dsa-transparency-database-questions-and-answers)

### Finding 164: publisher_enforcement_ladder_and_appeal — primary-platform

- **Source:** Google Play Developer Policy — Enforcement Process

- **Detail:** Google Play-specific practice: Google says enforcement decisions may consider app metadata, in-app experience, account history, third-party code, reports and own-initiative reviews; it combines automated models with trained operators and analysts when context requires human review, and sends action information with appeal instructions. It distinguishes rejection (a new app or update is not made available while a prior published version remains and account standing is unaffected), removal, suspension, limited visibility, limited regions, restricted developer account and account termination; suspension can follow serious or repeated violations and counts as a strike, while actions generally remain unless an appeal is granted. Implementation candidate: use an explicit enforcement ladder and reasoned notice with action target (version, app, account or region), effective time, user continuity and entitlement effects, remediation, appeal route and deadline, evidence, actor or automation, and republishing rules; preserve each case in an audit record. Scope and limitation: this is Google Play and developer-account policy, not an EU, US or general legal standard; the vendor process can change, and its action ladder should not be treated as a universal fairness or suspension requirement.

- **URL(s):** [https://support.google.com/googleplay/android-developer/answer/9899234?hl=en](https://support.google.com/googleplay/android-developer/answer/9899234?hl=en)

## Iteration 3 synthesis

- P0 accessibility gate: catalog, search/filter, detail, install/update, permission, error, quarantine, takedown and appeal flows must remain operable with reflow, text resizing, keyboard/screen-reader semantics, visible focus, non-color status, and accessible status announcements.

- P0 privacy gate: every release carries a versioned data/telemetry profile, purpose and retention map, consent/revocation behavior where consent is used, rights/deletion propagation, processor/subprocessor registry, and high-risk review decision. These controls are scoped to applicable law; EDPB/ICO/NIST evidence does not create a universal legal rule.

- P0 governance gate for target markets: use notice-and-action, reasoned decisions, proportionate suspension, publisher complaint/mediation paths, and auditable ad/placement labels where applicable. DSA and P2B controls are EU-scoped; Google Play evidence is vendor-specific.

- Pilot validation: test the store shell and host bridge with keyboard-only, screen reader, 200% text, 400% zoom/reflow, reduced motion, mixed-language text, consent withdrawal, deletion fan-out, processor changes, child-directed declarations, moderation appeals and reversal evidence.
