# Iteration 7 — Localization, regionalization and locale delivery contracts

## Scope and validation
This iteration adds a structurally distinct direction: deterministic locale identity, localized catalog delivery, regional formatting, and privacy-aware language negotiation. The parent validated 14 candidate records, rejected four URL duplicates, rechecked 11 unique source URLs with HTTP 200, and appended 10 canonical findings to the append-only state, increasing the evidence count from 199 to 209. No credential or spreadsheet content is included.

The sources are IETF RFCs, Unicode CLDR, W3C Internationalization guidance, and ECMA-402. They provide reusable protocol and data precedents; they do not define a universal mini-app store schema, translation QA process, legal market eligibility model, or host-specific locale support matrix.

## Decision implications
- Validate submitted locale tags as BCP 47 syntax plus registry-valid tags, while preserving the original tag and recording the validator registry date.
- Use an explicit, versioned matching profile: filtering for multi-result facets and lookup for one selected representation; expose the selected locale, available locales, fallback step, and no-match result.
- Keep language, script, region, currency, market, and entitlement as separate fields. Accept-Language may seed language preference but must not decide legal market, tax, currency, entitlement, or delivery region by itself.
- Pin CLDR data and translation bundles to revisions. Distinguish absent translation, inherited value, and intentional empty override; preserve hashes and rollback history.
- For locale-varying APIs, emit Content-Language and Vary for the selectors that affected representation selection, and prevent shared caches from mixing authenticated or entitlement state into public locale variants.
- Use typed machine values for currency and time-zone identifiers for recurring local-time behavior; localized display strings are presentation evidence, not authoritative values.
- Separate locale-sensitive search collation from catalog ordering. Persist comparator options and tie-break equivalent labels with immutable app identity/version keys.

## Findings
### 1. language_tag_validation_and_variants
- **Source:** IETF RFC 5646 (BCP 47), Tags for Identifying Languages
- **Evidence level:** `official-bcp`
- **Topic:** `localization/bcp47/language-tag-validation`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc5646.html
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] RFC 5646 defines BCP 47 language tags as hyphen-separated subtags with a language followed optionally by script, region, variants, extensions, and private use; script subtags are four letters and region subtags are two letters or three digits. Tags are case-insensitive. It distinguishes a syntactically well-formed tag from a valid tag whose registered subtags are valid for the registry date, whose variants and extension singletons are not duplicated, with grandfathered tags handled separately. [SCOPE/LIMITATION] RFC 5646 does not define product locale negotiation, translation fallback, or publisher/catalog semantics; extension subtags can require extension-specific validation, and private-use meanings come only from private agreement. Validity is also tied to the registry snapshot/date used by the validator. [MINI-APP-STORE PROPOSAL] Validate every catalog and mini-app UX locale against the BCP 47 grammar and a pinned IANA registry snapshot; report well-formed and valid as separate results; preserve the submitted tag while storing a normalized lookup form and parsed language/script/region/variant/extension fields; reject malformed or unregistered tags except explicitly governed private-use values. Treat script and region variants as distinct locale keys unless an explicit, versioned alias rule says otherwise, and record the validator registry date in catalog metadata.

**Source URLs:**
- https://www.rfc-editor.org/rfc/rfc5646.html

### 2. language_range_matching_and_fallback
- **Source:** IETF RFC 4647 (BCP 47), Matching of Language Tags
- **Evidence level:** `official-bcp`
- **Topic:** `localization/bcp47/locale-matching-fallback`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc4647.html
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] RFC 4647 defines Basic Filtering and Extended Filtering, which can return a possibly empty set of matching tags, and Lookup, which returns one best matching tag. A language priority list is considered in order. Basic filtering uses case-insensitive exact or prefix matching: `de-CH` can match `de-CH-1996`, but not `de-Latn-DE`; Lookup progressively truncates a range from the end, removes extension/private-use singleton material with its trailing subtag, and returns an explicitly defined default when no tag matches. [SCOPE/LIMITATION] RFC 4647 calls matching a tool rather than a complete language-tag procedure: each protocol or application must choose the matching scheme, result cardinality, and no-match default. Prefix matching does not guarantee mutual intelligibility, and canonicalizing ranges can lose the ability to select older tagged content unless the original and canonical forms are both handled. [MINI-APP-STORE PROPOSAL] Store an ordered user preference list and the set of locale tags actually supplied by each catalog or mini-app. Use filtering for multi-result catalog search/facets and Lookup for selecting one listing or UX resource; publish a deterministic fallback chain, explicit default locale, and tie-break rule. Do not infer script or region equivalence from a prefix alone. Record the requested ranges, matching mode, selected tag, fallback step, and no-match outcome for reproducibility.

**Source URLs:**
- https://www.rfc-editor.org/rfc/rfc4647.html

### 3. cldr_locale_inheritance_script_region_fallback
- **Source:** Unicode CLDR UTS #35 Part 1 Core, Locale Inheritance and Matching
- **Evidence level:** `official-standard`
- **Topic:** `localization/cldr/inheritance-script-region-fallback`
- **Primary URL:** https://www.unicode.org/reports/tr35/#Locale_Inheritance_and_Matching
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] UTS #35 organizes locale resources into bundles in an inheritance tree. Its example resource chain is `en_US_someVariant` -> `en_US` -> `en` -> `root`; the basic inheritance model for a language/script/region/variant locale truncates from the end after removing and then restoring `-u-` and `-t-` extensions, for example `sr_Cyrl_ME` -> `sr_Cyrl` -> `sr`. CLDR also defines `parentLocales` overrides because ordinary truncation can mix scripts, and for the main component the parent must be root or have the same script as the child. The specification distinguishes resource-bundle lookup from inherited item lookup. [SCOPE/LIMITATION] CLDR inheritance is a data-resource fallback mechanism, not language negotiation or a guarantee that inherited text is an acceptable translation. Parent-locale and component rules can override truncation, and bundle fallback is distinct from fallback for an individual catalog field. [MINI-APP-STORE PROPOSAL] Model each listing's localized metadata and mini-app UX resources as locale-keyed bundles, with a separate item-level fallback result. Resolve parent chains from a pinned CLDR release plus its `parentLocales` data; preserve script and region boundaries, prohibit cross-script inheritance unless an explicit parent rule permits it, and expose the resolved source locale and field for audit. Keep an absent translation, an inherited value, and an intentional empty override as distinct states.

**Source URLs:**
- https://www.unicode.org/reports/tr35/#Locale_Inheritance_and_Matching

### 4. cldr_release_and_translation_metadata_versioning
- **Source:** Unicode CLDR Releases/Downloads and unicode-org/cldr-json README
- **Evidence level:** `official-project`
- **Topic:** `localization/catalog-metadata/translation-versioning`
- **Primary URL:** https://cldr.unicode.org/index/downloads
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `2` source URL(s).

[SOURCE FACT] The Unicode CLDR Releases/Downloads page says each CLDR release is stable, may be cited as a normative reference, and each published version is absolutely stable and never changes; implementations may apply CLDR Corrigenda. The official `cldr-json` README says XML, not JSON, is the official format for CLDR data; the JSON distribution is programmatically generated from the corresponding XML with CLDR tooling and includes only data at `draft="contributed"` or `draft="approved"` status. [SCOPE/LIMITATION] CLDR release versioning governs shared locale data and its distribution, not a publisher's app strings, translation QA, catalog coverage, review approval, or the semantic correctness of a translation. A development snapshot or later corrigendum is not the same thing as the pinned published release. [MINI-APP-STORE PROPOSAL] Give every localized catalog record and mini-app UX bundle explicit `locale_tag`, `translation_revision`, `cldr_release`, optional `cldr_corrigendum`, source format, generated-data revision, coverage/draft status, update timestamp, and content hash. Pin production formatting and fallback data to a named CLDR release; retain prior localized bundle revisions for rollback and audit; and review publisher translation changes separately from CLDR-data upgrades instead of silently replacing displayed metadata.

**Source URLs:**
- https://cldr.unicode.org/index/downloads
- https://raw.githubusercontent.com/unicode-org/cldr-json/main/README.md

### 5. localized_representation_negotiation_and_cache_key
- **Source:** IETF RFC 9110 — HTTP Semantics, §§8.5 and 12.5.4–12.5.5
- **Evidence level:** `official-normative`
- **Topic:** `http_language_negotiation_vary_cache_correctness`
- **Primary URL:** https://datatracker.ietf.org/doc/html/rfc9110#name-accept-language
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] RFC 9110 defines Accept-Language as a request field for preferred natural languages, with language ranges and q weights; Content-Language describes the natural language(s) of the representation's intended audience; Vary identifies request fields that influenced representation selection, expands the cache key, and SHOULD be sent on cacheable responses when a representation was selected from Accept-Language. RFC 9110 also cautions that sending a user's complete linguistic preferences can conflict with privacy expectations. [MINI-APP-STORE REQUIREMENT — PROPOSAL] Any cacheable catalog/API representation that varies by locale MUST return Content-Language and Vary: Accept-Language (plus every other selector that changes the representation); the response body SHOULD expose the selected locale and available locales. Digest-addressed package/artifact URLs MUST be locale-invariant, while mutable localized indexes MUST use an explicit locale-aware cache key or validator. Do not put authenticated user, entitlement, or revocation state into a shared locale variant; use private/no-store policy for those responses and do not forward the full preference vector to endpoints that do not need it.

**Source URLs:**
- https://datatracker.ietf.org/doc/html/rfc9110#name-accept-language

### 6. locale_sticky_discovery_urls_and_user_override
- **Source:** W3C Internationalization — When to use language negotiation
- **Evidence level:** `official-i18n-guidance`
- **Topic:** `locale_aware_discovery_urls_user_override_sticky_selection`
- **Primary URL:** https://www.w3.org/International/questions/qa-when-lang-neg
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] W3C explains that HTTP language negotiation selects among language versions using the URL and browser preference information such as Accept-Language, but says negotiation should not be used alone. It recommends visible controls to other languages and describes two ways to make an explicit selection sticky: session/profile state or language-specific internal links; it notes that language-specific URLs preserve the chosen language but add translation and link-management cost. [MINI-APP-STORE REQUIREMENT — PROPOSAL] Every catalog listing, detail page, and launch/deep-link representation MUST have a stable locale-addressable URL or explicit locale parameter. Preserve app identity and route while changing only the locale variant, expose equivalent-locale links on every representation, and let an explicit URL selection override the host's Accept-Language hint. Persist the user's override in host-scoped preference state, but never silently redirect a shared locale-specific URL back to an inferred locale; include the locale variant in cache selection and audit context.

**Source URLs:**
- https://www.w3.org/International/questions/qa-when-lang-neg

### 7. accept_language_not_locale_or_region_authority
- **Source:** W3C Internationalization — Accept-Language used for locale setting
- **Evidence level:** `official-i18n-guidance`
- **Topic:** `locale_region_selection_privacy_and_shared_devices`
- **Primary URL:** https://www.w3.org/International/questions/qa-accept-lang-locales.en.html
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] W3C says Accept-Language was originally intended to specify language, not a complete user locale; using it alone can constrain users, omit region information, and be wrong on borrowed or shared machines. It recommends using it as a first-contact starting point, then allowing language and cultural settings to be changed and storing the user's chosen result for later visits. [MINI-APP-STORE REQUIREMENT — PROPOSAL] Treat Accept-Language only as an initial language-ranking hint. Never infer currency, tax, legal-market eligibility, entitlement region, or delivery region solely from it; keep language, script, region, currency, and market as separate catalog/API fields. Require explicit user, tenant, or host-market selection for region-sensitive variants, persist that selection in a scoped preference, and do not expose the raw header or derived full preference vector in public discovery URLs or routine logs.

**Source URLs:**
- https://www.w3.org/International/questions/qa-accept-lang-locales.en.html

### 8. content_language_metadata_and_text_language_separation
- **Source:** W3C Internationalization — HTTP headers, meta elements and language information
- **Evidence level:** `official-i18n-guidance`
- **Topic:** `content_language_api_metadata_html_lang_consistency`
- **Primary URL:** https://www.w3.org/International/questions/qa-http-and-lang/qa-html-language-declarations
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] W3C distinguishes HTTP Content-Language as metadata about the intended audience of the resource from HTML lang, which declares the language of text for processing; it recommends lang on the html element and on fragments in another language, and says meta http-equiv=Content-Language is deprecated. Content-Language may list multiple audience languages and can identify the language version chosen by server-side negotiation. [MINI-APP-STORE REQUIREMENT — PROPOSAL] For HTML catalog/detail responses, emit Content-Language for the negotiated representation and set lang on the document root plus mixed-language fragments. For JSON APIs, carry explicit selected_locale, available_locales, and per-field/per-string language metadata where needed because Content-Language alone is not a field-level text declaration. Reject or flag inconsistent header/body locale metadata, and do not make deprecated meta http-equiv content-language part of the store API contract.

**Source URLs:**
- https://www.w3.org/International/questions/qa-http-and-lang/qa-html-language-declarations

### 9. regional_number_currency_formatting_contract
- **Source:** ECMA International, ECMA-402 13th edition (June 2026), NumberFormat Objects
- **Evidence level:** `primary-standard`
- **Topic:** `regional_number_currency_formatting`
- **Primary URL:** https://402.ecma-international.org/#numberformat-objects
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] ECMA-402 defines Intl.NumberFormat with locale input and options for decimal, percent, currency, and unit styles; currency formatting requires a well-formed three-letter ISO 4217 code, and resolvedOptions exposes the resolved locale, numberingSystem, style, currency, currencyDisplay, currencySign, and digit/rounding options. [SCOPE/LIMITATION] Locale data and available locales are implementation-defined within the specification's constraints, so hosts can resolve different supported locales or display patterns; the API formats values but does not define a store wire schema, financial rounding policy, or audit semantics. [MINI-APP-STORE PROPOSAL] Store amounts as typed machine values with an explicit three-letter currency code and declared rounding/precision policy; keep localized strings out of authoritative comparisons, request locale/numbering-system at render time, and record resolvedOptions (or an equivalent result) in visual-regression/audit evidence.

**Source URLs:**
- https://402.ecma-international.org/#numberformat-objects

### 10. time_zone_identifier_contract
- **Source:** IETF RFC 9557, Date and Time on the Internet: Timestamps with Additional Information
- **Evidence level:** `primary-standard`
- **Topic:** `time_zone_identifier_contract`
- **Primary URL:** https://www.rfc-editor.org/rfc/rfc9557.html#section-1.2
- **Verification:** Parent recheck returned HTTP 200 for the primary URL and all `1` source URL(s).

[SOURCE FACT] RFC 9557 defines an IANA Time Zone as a named zone from the IANA Time Zone Database; its rules can change, and using a named IANA zone implies applying the rules current at interpretation. It explains that a fixed UTC offset is unsuitable for local-time operations and strongly discourages offset time zones, while the IXDTF extension carries additional time-zone information with a timestamp. [SCOPE/LIMITATION] The RFC defines timestamp syntax and semantics, not a mini-app store schema, locale display formatting, or a time-zone database version pin; named-zone rule changes can affect future local-time derivations. [MINI-APP-STORE PROPOSAL] Represent schedule metadata with an instant/offset plus an IANA timeZoneId, retain the source timestamp and tzdb version used for calculations, reject offset-only identifiers for recurring local events, and flag offset/name inconsistencies instead of silently normalizing them.

**Source URLs:**
- https://www.rfc-editor.org/rfc/rfc9557.html#section-1.2

## Implementation acceptance tests
- Feed malformed, syntactically valid but registry-invalid, script/region-variant, grandfathered, and private-use tags through catalog ingestion; verify separate validation results and pinned registry evidence.
- Run the same supported-locale set through filtering and lookup; verify deterministic selected locale, fallback trace, no-match response, and stable behavior after CLDR data updates.
- Serve localized catalog variants through a shared cache; verify Content-Language, Vary/cache keys, selected-locale metadata, explicit URL override, and no leakage of user/entitlement state.
- Change Accept-Language on a shared device and verify it only seeds the first-contact language choice; explicit user/tenant/market settings remain authoritative for region-sensitive behavior.
- Compare catalog amounts, recurring schedules, and search results across locales; verify typed currency codes, IANA time-zone IDs/tzdb evidence, separate search/sort collations, and immutable tie-breaks.
