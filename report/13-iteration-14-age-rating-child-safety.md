# Iteration 14 — Age rating, child safety and age-aware release gates

## Scope and validation
This iteration adds a structurally distinct direction: age/audience classification, content-rating evidence, child-directed capability controls, and proportional age-signal handling for a mini-app store. The parent accepted five candidates from the Google Play and Android official documentation worker, rechecked six unique source URLs with HTTP 200, and appended five canonical findings. The evidence count increased from 209 to 214. The Apple and independent child-safety workers failed before producing candidate records; this is recorded as a research gap rather than filled with inference. No credential or spreadsheet content is included.

Google Play policies are platform-specific practice, not a universal legal taxonomy. They are useful as reusable control patterns, but the host must map age thresholds, child definitions, parental authority, and market-specific obligations with legal/product owners.

## Decision implications

- Make target audience, child-inclusive status, content rating, age-restriction rationale, and covered regions versioned release metadata—not free-text marketing claims.
- Separate target-audience declaration from content rating. The former describes intended users; the latter describes content suitability and may be produced by a rating authority.
- Bind child/mixed-audience classification to a host capability policy. Unknown-age or unavailable age-signal states should enter a conservative safe mode rather than silently becoming adult access.
- Trigger re-review when audience, content, advertisements, offers, social features, sensitive permissions, SDKs, or data practices change.
- Keep raw age signals and parental decisions inside the host control plane; publishers receive only the minimum capability outcome and must not use age signals for advertising, profiling, or marketing.
- Treat child safety as both a catalog/review gate and a runtime enforcement boundary. A listing declaration alone cannot prevent an unsafe API, SDK, social flow, or location access.

## 1. target_audience_age_band_declaration

- **Source:** Google Play Help, Manage target audience and app content settings
- **Evidence level:** `official-guidance`
- **Topic:** `target_audience_declaration`
- **URL(s):**
  - https://support.google.com/googleplay/android-developer/answer/9867159?hl=en

[SOURCE FACT] Google Play requires a new app or an update to declare its target age group in the App content section; multiple age groups should be selected only when the app is designed and appropriate for each group. Apps with any child-inclusive target age group must follow the Families Policy Requirements, and Google may review whether the declaration is accurate; the guide notes that some reviews can take up to 7 days or longer in exceptional cases. Before completing the section, the developer must declare ad presence, provide app-access instructions, and add a privacy policy. [SCOPE/LIMITATION] This is Google Play Console guidance for apps submitted or updated on Google Play, not a universal age taxonomy or a legal determination of who is a child; the page says local laws and contexts matter, and its review-time statement is not an SLA. It does not define a portable mini-app manifest. [MINI-APP-STORE PROPOSAL] Require every mini-app manifest to declare controlled target-age bands, child-inclusive status, intended regions, and an age-appropriateness rationale; require the publisher to update the declaration when content, ads, in-app purchases, social features, permissions, or SDKs change. Gate publication on a complete, reviewable declaration and keep jurisdiction-specific child rules separate from the portable manifest rather than hard-coding one global threshold.

## 2. content_rating_questionnaire_and_release_gate

- **Source:** Google Play Help, Content rating requirements for apps, games, and the ads served on both
- **Evidence level:** `official-policy`
- **Topic:** `content_rating_release_gating`
- **URL(s):**
  - https://support.google.com/googleplay/android-developer/answer/9859655?hl=en

[SOURCE FACT] Google Play says content ratings are assigned by separate rating authorities from the developer's questionnaire responses; the questionnaire is required for new apps, existing unrated apps, and updates whose content or features change the answers. Ratings help parents assess suitability and can support legally required blocking or filtering, while ads and associated offers must be appropriate for the app's rating. Misrepresentation may lead to removal or suspension. [SCOPE/LIMITATION] The rating authorities use their own methodologies, and the page's examples of minor-user filtering are limited to listed jurisdictions; a content rating is not the same thing as a target-audience declaration and the page does not prescribe a mini-app runtime policy. [MINI-APP-STORE PROPOSAL] Add a versioned content-rating record to each mini-app manifest, with questionnaire revision, authority/region results, and the maximum rating permitted for ads and promotional offers. On a release diff, require a new rating attestation when content, features, ads, or offers could change questionnaire answers; quarantine or reject a missing/stale result, and let the host enforce region and child-mode filters without treating the rating as a substitute for age-aware runtime controls.

## 3. child_safety_capabilities_and_mixed_audience_controls

- **Source:** Google Play Families Policies
- **Evidence level:** `official-policy`
- **Topic:** `child_safety_controls`
- **URL(s):**
  - https://support.google.com/googleplay/android-developer/answer/9893335?hl=en

[SOURCE FACT] Google Play requires child-inclusive apps to keep content accessible to children appropriate, keep Target Audience and Content, Data safety, and IARC questionnaire answers accurate, disclose child data collected through APIs and SDKs, and follow restrictions such as no precise location for apps solely targeting children and no non-approved child-directed APIs or SDKs. For mixed-audience apps, non-approved APIs or SDKs must be behind a neutral age screen or otherwise avoid collecting data from children. Child-inclusive social apps/features must show an online-safety reminder before free-form exchange, provide adult action before children share personal information, and provide adult controls for social features; anonymous chat with strangers must not target children. [SCOPE/LIMITATION] These are Google Play policy requirements for apps distributed through Google Play; the policy notes that the meaning of children varies by locale and context and that legal compliance remains the developer's responsibility. Requirements depend on the app's declared audience and feature/API design, so they are not a complete jurisdiction-neutral child-safety law or a claim that every mini-app is child-directed. [MINI-APP-STORE PROPOSAL] Classify each mini-app as child-only, mixed-audience, or not child-directed and bind that classification to a capability policy. Default unknown-age and child sessions to the stricter path: block precise location and disallowed identifiers/SDKs, require an approved SDK inventory, keep social exchange disabled until neutral age or adult action is satisfied, show safety reminders, expose parent-controlled social settings, and require an AR physical-safety warning where applicable. Enforce these controls in the host runtime and release review, not only through publisher self-attestation.

## 4. app_content_declarations_and_release_readiness

- **Source:** Google Play Help, Prepare your app for review
- **Evidence level:** `official-guidance`
- **Topic:** `release_gating_declarations`
- **URL(s):**
  - https://support.google.com/googleplay/android-developer/answer/9859455?hl=en

[SOURCE FACT] Google Play describes the App content page as the place to manage safety, policy, and legal information, including target audience and content, ad presence, reviewer access instructions, permissions declarations, content ratings, and privacy/security practices. Its Needs attention tab identifies declarations requiring action, while actioned declarations should be reviewed and kept current. Apps targeting children must provide a privacy-policy link on the store listing and in the app even when they do not access personal or sensitive data; high-risk permission requests may require approval; and unrated apps may be removed. [SCOPE/LIMITATION] This is a Play Console release workflow for Android apps, and some declarations are app-level or policy-specific rather than directly transferable to a hosted mini-app. It does not prescribe a host-side schema, review SLA, or runtime enforcement model. [MINI-APP-STORE PROPOSAL] Treat age audience, content rating, ads, access path, sensitive capabilities, SDK inventory, and child-safety attestations as versioned release artifacts with owners and review timestamps. Block promotion when any required artifact is missing, stale, or marked for attention; trigger re-review on a release diff that changes audience, content, ads, offers, permissions, social behavior, or SDKs; and retain an auditable decision linking the declaration set to the exact mini-app version and regions.

## 5. optional_age_signals_safe_mode_and_parental_approval_gate

- **Source:** Android Developers, Google Play Age Signals API (beta)
- **Evidence level:** `official-platform-doc`
- **Topic:** `age_assurance_host_adapter`
- **URL(s):**
  - https://developer.android.com/google/play/age-signals/overview
  - https://developer.android.com/google/play/age-signals/request-age-signals

[SOURCE FACT] Android's Play Age Signals documentation describes a beta API for age-related signals and for notifying Google Play about significant app changes that require parental approval or revoked approvals. In the request flow, requestAgeSignalsAccess can return SHARED, NOT_SHARED, or VERIFICATION_REQUIRED; a supervised user's parent can choose age sharing in Family Link settings, and the app receives age signals only when sharing is enabled. The terms limit use to age-appropriate content and legal compliance, prohibit advertising, marketing, profiling, and analytics uses, and restrict the API to apps updated by Google Play with the information usable only by the requesting app. [SCOPE/LIMITATION] This is an Android/Google Play runtime API in beta, not a portable mini-app standard or universal proof of age; sharing and verification behavior varies by jurisdiction, supervision, and user/parent choice, and developers remain responsible for applicable law and policy. A hosted mini-app may not itself be a Google-Play-updated app. [MINI-APP-STORE PROPOSAL] Define an optional host adapter, never a mandatory portable field: map SHARED, NOT_SHARED, and VERIFICATION_REQUIRED to explicit capability policies; use a conservative safe mode for NOT_SHARED or VERIFICATION_REQUIRED, such as withholding age-restricted content and child free-form social exchange; surface parental-approval and revoked-approval events as release/review signals; keep raw age signals inside the host, do not expose them to publishers or telemetry, and never use them for marketing or profiling. When unavailable, fall back to declared audience/rating plus conservative controls rather than silently treating the user as an adult.

## Proposed minimum data model

| Field | Purpose | Release/review rule |
|---|---|---|
| `audience_class` | `child_only`, `mixed_audience`, `not_child_directed`, or `unknown` | Required; `unknown` cannot unlock child-sensitive capabilities |
| `target_age_bands` | Controlled age-band declaration with jurisdiction scope | Re-attest when content, feature, ad, offer, permission, SDK, or data behavior changes |
| `content_rating` | Authority, region, rating, questionnaire revision, and decision timestamp | Must be present and current before publication where required |
| `child_capability_profile` | Host-enforced allow/deny policy for location, identifiers, social, ads, payments, and SDKs | Evaluated at release and runtime; default deny for unresolved states |
| `age_signal_state` | Minimal host-only state such as `shared`, `not_shared`, `verification_required`, `unavailable` | Never expose raw age data to publishers; retain decision/audit reference only |
| `parental_control_state` | Approval, revocation, and supervised-user policy outcomes | Revocation must be able to disable affected features without waiting for a package update |

## Acceptance tests for the pilot

1. Submit a mini-app with child-only, mixed-audience, and not-child-directed declarations; verify distinct review queues and capability profiles.
2. Change an advertisement, social feature, SDK, permission, or content element; verify that the release is blocked until the relevant declarations/rating evidence are refreshed.
3. Run the app with age signal shared, not shared, verification required, unavailable, and parent approval revoked; verify safe-mode behavior and absence of raw age data in publisher telemetry.
4. Attempt precise location, child-directed social exchange, unknown SDK use, and age-restricted content from a child or unresolved session; verify host-side denial and auditable reason codes.
5. Validate rating and audience metadata at catalog render, install/activation, and deep-link launch; confirm stale or missing metadata cannot be bypassed by direct launch.

## Gap and follow-up

The Apple-specific worker failed because the delegated session could not persist to disk, and the independent child-safety worker stopped before writing candidates. No Apple or independent-standard claims are included in this iteration. Re-run those directions after disk capacity is restored; preserve this section and the five validated Google/Android findings.
