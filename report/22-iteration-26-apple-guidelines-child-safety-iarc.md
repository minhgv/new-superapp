# Iteration 26 — Cross-Platform Host Review & Child Safety Parity: Apple Rule 4.7, UK ICO Children's Code, and IARC Rating Federation

## Executive Overview

Iteration 26 resolves the explicit research gap identified in Iteration 14 and the validation plan: previously, age ratings and store review governance were anchored almost entirely on Google Play / Android policies. This iteration establishes comprehensive cross-platform host parity and statutory child-safety standards by verifying and integrating 15 new authoritative findings across three critical domains:

1. **Apple App Store Review Guidelines (§4.7, §4.7.1–4.7.5, §1.3, §5.1.4)**:
   - **Host Responsibility (§4.7)**: The host super app is strictly responsible for all non-embedded HTML5/JavaScript software, mini-games, and plug-ins offered. Non-compliance leads directly to super-app binary rejection.
   - **Mandatory Trust & Moderation Controls (§4.7.1)**: Requires objectionable material filtering, user reporting mechanisms, timely operational responses, abusive user blocking, and adherence to §3.1 In-App Purchases for all digital goods.
   - **Native API & Permission Boundaries (§4.7.2, §4.7.3)**: Prohibits extending or exposing native platform APIs to mini-app code without prior permission from Apple. Forbids sharing host data or privacy permissions with any individual mini-app without explicit user consent in each instance.
   - **Universal Index & Discovery (§4.7.4)**: Mandates maintaining an accessible index of all available software and metadata, including universal links to every mini-app.
   - **Age Identification & Restriction (§4.7.5, §1.3, §5.1.4)**: Super apps must provide a way for users to identify software exceeding the app's age rating, enforce age restrictions based on verified/declared age, implement parental gates for out-of-app links/purchases, and strictly prohibit behavioral advertising and persistent tracking in kids' software under COPPA/GDPR.

2. **UK Information Commissioner's Office (ICO) Age-Appropriate Design Code (Children's Code, DPA 2018 s123)**:
   - **15 Flexible Statutory Standards**: Governs all online services and mini apps likely to be accessed by children under 18.
   - **Best Interests & DPIAs (Standards 1–4)**: Establishes children's best interests as the primary design consideration, mandates Data Protection Impact Assessments prior to release, requires knowing user age or applying high-watermark child protections to all users, and enforces bite-sized, age-appropriate transparency.
   - **Privacy Defaults & Data Minimisation (Standards 5–9)**: Mandates high privacy by default (all non-essential telemetry and sharing off), strictly limits data collection to what is strictly necessary, and prohibits detrimental data use or unauthorized disclosure.
   - **Hardware Capabilities & Parental Supervision (Standards 10–11)**: Geolocation must be OFF by default, show an unmistakable active host indicator when enabled, and immediately revert to off upon session end. Parental controls must be transparent to the child, displaying an active indicator whenever a parent/guardian is monitoring activity.
   - **Profiling & Dark Pattern Prohibitions (Standards 12–15)**: Behavioral profiling and targeted advertising are OFF by default. Prohibits nudge techniques and dark patterns designed to encourage weaker privacy settings, excessive screen time, or impulse digital spending. Mandates simple tools for children to exercise rights and report abuse.

3. **International Age Rating Coalition (IARC) Global Rating & Content Classification Federation**:
   - **Single Global Questionnaire SaaS Architecture**: Simplifies age classification by having developers complete a single dynamic questionnaire once, which algorithmically maps into culturally and legally distinct ratings across all participating global jurisdictions.
   - **Federated Rating Authorities**: Unifies ESRB (North America), PEGI (Europe), USK (Germany), ClassInd (Brazil), ACB (Australia), GRAC (South Korea), DGSC (Taiwan), IGRS (Indonesia), and Gmedia/GAMR (Saudi Arabia), plus generic ratings for non-participating territories.
   - **Standard Content Descriptors & Interactive Elements**: Classifies digital software along standardized axes: In-App Purchases (including randomized loot boxes), User Interaction (unfiltered chat/voice), Location Sharing (sharing coordinates with other users), and Unrestricted Web Access.
   - **Certificate ID Portability & Post-Release Governance**: Enables seamless rating federation across storefronts (Google Play, Nintendo eShop, Microsoft Store, Meta Quest). Regional authorities actively monitor live software, possess unilateral re-rating authority, and enforce rating corrections without re-questionnaire overhead.

---

## Detailed Control Matrix: 15 Verified Findings

### Domain 1: Apple App Store Review Guidelines (§4.7, §1.3, §5.1.4)

| Finding ID | Topic & Standard Reference | Specification Fact | Super-App Host Control |
|---|---|---|---|
| `apple-miniapp-host-responsibility-001` | **App Store Review Guidelines §4.7 & §4.7.1** | Apple permits non-embedded HTML5/JS mini apps and mini games but makes the host app strictly liable for all software compliance. §4.7.1 mandates privacy (5.1), objectionable content filtering, user reporting, timely response, abusive user blocking, and 3.1 IAP for digital goods. | Host must maintain pre-release automated security and content scanners, user reporting workflows with an enforceable 24h moderation SLA, remote kill-switch quarantine, and route all digital transactions through host-approved native payment gateways. |
| `apple-miniapp-api-permission-isolation-002` | **App Store Review Guidelines §4.7.2 & §4.7.3** | Apps may not extend or expose native platform APIs/technologies to mini-app code without prior Apple approval. Host may not share device data or privacy permissions with any mini-app without explicit user consent in each instance. | Host WebView/container must enforce zero-trust bridge sandboxing. Device permissions (camera, mic, location) granted to the super app must NEVER silently leak to a mini app; the host must prompt the user per-mini-app per-origin. |
| `apple-miniapp-catalog-index-links-003` | **App Store Review Guidelines §4.7.4** | Host apps must provide an accessible index of software and metadata available in the app, including universal links that lead directly to all offered software. | The mini-app marketplace must maintain an immutable, public-facing or host-indexed catalog schema with canonical universal links (`https://<domain>/apps/<app-id>`) supporting direct routing, state restoration, and external indexing. |
| `apple-miniapp-age-restriction-gating-004` | **App Store Review Guidelines §4.7.5 & §1.3** | The app must provide a way for users to identify software exceeding the app's age rating and use verified/declared age restriction mechanisms. §1.3 requires parental gates for external links and purchases in kids apps. | Host container must inspect user age tier at launch. If mini-app content rating exceeds user/profile age, access is hard-blocked. All outbound web links and IAP flows must require biometric or parental PIN verification. |
| `apple-kids-privacy-advertising-controls-005` | **App Store Review Guidelines §5.1.4 & §1.3** | Strict compliance with COPPA/GDPR. Birthdate and parental contacts collected solely for statutory compliance. No behavioral advertising or tracking allowed; only contextually relevant ads from vetted COPPA-compliant ad networks. | Container must inject an isolated privacy profile for minor accounts: disable identifier tracking (IDFA/GAID/cookie persistence), block third-party analytics SDKs, suppress targeted ad auctions, and enforce session wipe on exit. |

### Domain 2: UK ICO Age-Appropriate Design Code (Children's Code)

| Finding ID | Topic & Standard Reference | Specification Fact | Super-App Host Control |
|---|---|---|---|
| `uk-ico-childrens-code-governance-001` | **ICO Standards 1–4 & DPA 2018 s123** | Best interests of the child is primary. DPIA required prior to launch. Age-appropriate application requires knowing the age of users or applying full child-safety standards to all users. Transparency requires clear, bite-sized notices. | Mini app onboarding must include a mandatory Child Safety Impact Assessment (CSIA). The super app enforces age-tier segmentation at the account level; unverified accounts default to high-protection child mode. |
| `uk-ico-privacy-defaults-data-minimisation-002` | **ICO Standards 5–9** | Detrimental data use prohibited. High privacy default settings must be applied automatically (geolocation, profiling, social sharing off). Data collection strictly minimised. No data sharing unless compelling legal reason exists. | Container runtime enforces strict permission defaults: camera, microphone, contacts, and storage access default to DENY for minor profiles. In-app data sharing and public user directory search are permanently disabled. |
| `uk-ico-geolocation-parental-controls-003` | **ICO Standards 10–11** | Geolocation must be off by default; if activated, must display a conspicuous visual indicator and revert to off after the immediate session. Parental controls must be transparent and clearly indicate when monitoring is active. | Super app status bar displays a persistent, non-dismissible location icon during active GPS usage. Background location is forbidden for kids apps. When parental supervision or spend limits are active, an explicit indicator is shown. |
| `uk-ico-profiling-nudge-restrictions-004` | **ICO Standards 12–13** | Profiling and behavioral tracking must be off by default. Nudge techniques (streaks, infinite scroll, dark patterns, predatory countdown timers encouraging continuous engagement or spending) are strictly prohibited. | Store review enforces static and dynamic UI audits: mini apps with streak-reward gamification targeting minors or deceptive checkout count-downs are rejected. Recommendation algorithms for kids use non-personalized signals. |
| `uk-ico-connected-devices-online-tools-005` | **ICO Standards 14–15** | Connected device integration must enforce robust device security without default passwords. Online tools must provide easy, child-friendly mechanisms to exercise GDPR rights, report abuse, and obtain human support. | Mini apps accessing Bluetooth/NFC/IoT devices must use host-brokered encrypted channels. Super app chrome injects a permanent, one-touch "Report / Get Help" floating control in every mini app interface. |

### Domain 3: International Age Rating Coalition (IARC)

| Finding ID | Topic & Standard Reference | Specification Fact | Super-App Host Control |
|---|---|---|---|
| `iarc-questionnaire-saas-architecture-001` | **IARC Global Rating Architecture** | IARC SaaS solution allows developers to complete a single dynamic questionnaire once, which algorithmically maps responses into regional ratings according to distinct regional criteria and produces a generic global rating. | Super app developer portal integrates IARC questionnaire API or ingests IARC JSON payloads, validating that all submitted mini apps possess an authentic, cryptographically verifiable IARC rating certificate. |
| `iarc-participating-authorities-regions-002` | **IARC Regional Authorities & Scope** | IARC unifies 9 official statutory authorities: ESRB (US/CA), PEGI (Europe), USK (Germany), ClassInd (Brazil), ACB (Australia), GRAC (South Korea), DGSC (Taiwan), IGRS (Indonesia), and Gmedia/GAMR (Saudi Arabia). | Store catalog service dynamically displays locale-appropriate rating badges based on the user's registered IP/SIM/account jurisdiction (e.g., displaying PEGI 12 in the EU, USK 12 in Germany, ESRB Everyone 10+ in North America). |
| `iarc-content-descriptors-interactive-elements-003` | **IARC Classification Taxonomy** | Beyond numeric age tiers, IARC assigns standardized content descriptors (violence, language, crude humor) and interactive elements: In-App Purchases (including random items), User Interaction, Location Sharing, Unrestricted Web. | Super app store product pages must display interactive warning chips: e.g., "Contains In-App Purchases (Includes Random Items)", "Users Interact (Unmoderated Chat)", enabling informed consumer choice and policy gating. |
| `iarc-post-release-monitoring-corrections-004` | **IARC Post-Release Governance & Audit** | IARC participating rating authorities actively audit live digital software post-release. Authorities have full authority to re-rate an app or modify content descriptors unilaterally if live content diverges from questionnaire responses. | Super app must support an automated webhook listener for IARC rating update events: when an authority changes an age tier (e.g., 3+ to 12+), the store instantly re-indexes the app and applies age-gating to existing installs. |
| `iarc-certificate-id-portability-005` | **IARC Federation & Storefront Portability** | Developers obtain a unique IARC Certificate ID that is portable across major digital storefronts (Google Play, Microsoft Store, Nintendo eShop, Meta Quest) without needing to re-take the questionnaire. | Mini app publishing portal allows developers to input an existing IARC Certificate ID; the store backend verifies certificate authenticity against IARC registry APIs, saving developer overhead while maintaining strict compliance. |

---

## Architecture Implementation Recommendations

1. **Host-Enforced Three-Tier Minor Protection Profile**:
   - *Tier 1 (Ages 0–12)*: Strict COPPA/ICO mode. Zero advertising identifiers, no behavioral analytics, no unmoderated chat, zero geolocation, mandatory parental gate for all transactions, strict adherence to Apple §1.3 and §5.1.4.
   - *Tier 2 (Ages 13–17)*: Mixed-audience mode. No behavioral profiling or retargeting, high-privacy defaults, explicit per-app permission prompts, conspicuous location indicators, transparent parental dashboard.
   - *Tier 3 (Ages 18+)*: Standard enterprise/super-app profile with standard RBAC, audit logging, and configurable privacy controls.

2. **Cross-Platform Host Bridge Sandboxing (Apple §4.7.2 / §4.7.3 Alignment)**:
   - Mini apps run in an isolated multi-process WebView container.
   - The native bridge rejects any method call attempting to access native platform APIs directly.
   - Permission delegation is strictly origin-bound: the super app's OS-level camera or location permission is NEVER automatically inherited by a mini app.

3. **Federated Age & Rating Ingestion Pipeline**:
   ```
   [Developer Console]
           │
           ▼ (Submits App / Enters IARC Certificate ID)
   [IARC Ingestion Engine] ───► [Verifies Certificate & Descriptors]
           │
           ▼
   [Store Catalog Service] ───► [Dynamic Regional Badge Mapping (PEGI/ESRB/USK/etc.)]
           │
           ▼
   [Super App Client Runtime] ──► [Host Age-Gating & Parental Authorization Checks]
   ```
