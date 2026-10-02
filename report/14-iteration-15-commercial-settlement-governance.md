# Iteration 15 — Commercial settlement governance
## Scope and validation
This iteration adds a structurally distinct direction: commercial eligibility and product identity for hosted mini-app commerce; subscription, refund, void, dispute and notification reconciliation; publisher payout evidence; provider-to-bank settlement controls; and accounting treatment boundaries. Three workers produced 18 candidate records. The parent accepted 17 records, rejected one primary URL because the same Google Play RTDN source was already canonical, rechecked 30 unique candidate source URLs with HTTP 200, and verified the canonical findings file grew append-only from 214 to 231 lines. Two of the 30 URLs were already canonical supporting sources and were not duplicated in the appended evidence. No spreadsheet content, credential, or secret is included.
The sources are platform/vendor patterns or accounting/standards evidence, not a universal mini-app commerce law. The reusable requirements below are explicitly proposals and must be bound to the chosen merchant-of-record, payment providers, jurisdictions, tax model, and publisher contracts.
## Decision implications
- Separate host, mini-app partner, mini-app product, provider transaction, entitlement, payout, and bank-settlement identities. Do not infer commercial ownership from the runtime package or a user-facing listing.
- Make every commercial item a versioned, immutable mapping across the host catalog and provider product identifiers. Preserve region, currency, price, offer/base-plan, effective time, and provider SKU exactly as submitted.
- Treat provider notifications as change signals, not complete financial truth. Persist immutable event envelopes, deduplicate by provider event ID, enrich from authoritative provider APIs, and replay/reconcile after retries, loss, late arrival, refunds, voids, or chargebacks.
- Maintain an append-only settlement evidence ledger and a separate entitlement ledger. Keep gross sale, refund/reversal, provider fee, platform commission, publisher payable, reserve, dispute, tax/adjustment, payout, and bank cash-settlement legs separately attributable.
- Close payout periods only after report pagination/watermarks and control totals reconcile. Distinguish pending, posted, available, forecast, failed, unsettled, refunded, reversed and cash-settled states.
- Persist the exact localized price and subscription terms shown at purchase, plus cancellation/refund route and price/version cohort; cancellation, refund-only, refund-and-revoke, and partial refund are distinct events.
- Gate publisher payout on contractual dispute/reserve/liability policy and record negative-balance recovery or compensating entries without mutating the original sale.
- Treat provider-specific features such as Apple mini-app commerce eligibility, Google Play base plans, Stripe reserves, and Adyen value dates as capability adapters, not portable manifest requirements.

## Validated findings
### 1. `apple_mini_app_commerce_eligibility_and_host_partner_boundary`
- **Source:** Apple Developer — Mini Apps Partner Program
- **Evidence level:** `primary-platform`
- **Topic:** `apple/mini-apps/commerce-eligibility/host-partner-boundary`
- **Primary URL:** https://developer.apple.com/programs/mini-apps-partner/
- **Source URLs:**
  - https://developer.apple.com/programs/mini-apps-partner/

[APPLE PLATFORM PATTERN] Apple defines a qualifying mini app as one put out by a person or entity not directly or indirectly controlled by the host and not under common control; control includes power to direct management through ownership, voting securities, registered capital, contract, or otherwise. The host must be an iOS/iPadOS App Store app, maintain an Apple-approved guideline 4.7 manifest covering hosted mini apps, and submit metadata identifying all mini-app In-App Purchases (qualifying and non-qualifying) and the digital goods/services sold. Program participation requires the Advanced Commerce API, Declared Age Range API, Apple In-App Purchase system, and the App Store Server API Send Consumption Information endpoint; Apple says program members earn 85% of qualifying In-App Purchase sales. Qualifying purchases include consumables, non-consumables, auto-renewable subscriptions, and non-renewing subscriptions, while a qualifying consumable cannot be shared or consumed across mini apps. [REUSABLE REQUIREMENT] Model host, mini-app partner, and mini-app product as separate commercial principals; require a control/independence attestation, manifest-to-version mapping, classification of every mini-app purchase, and a per-mini-app consumption scope before enabling the Apple-specific commerce benefit. [LIMIT] This is an Apple program rule, not a universal legal control test. Direct fetch verified HTTP 200 on 2026-10-02.
### 2. `apple_mini_app_product_identity_and_sku_contract`
- **Source:** Apple Developer Documentation — Creating SKUs for the Mini Apps Partner Program
- **Evidence level:** `primary-platform`
- **Topic:** `apple/advanced-commerce-api/mini-app-product-identity-sku`
- **Primary URL:** https://developer.apple.com/documentation/advancedcommerceapi/creating-skus-for-the-mini-app-partner-program.md
- **Source URLs:**
  - https://developer.apple.com/documentation/advancedcommerceapi/creating-skus-for-the-mini-app-partner-program.md

[APPLE PLATFORM PATTERN] Apple requires mini-app product display names and SKUs to fully identify the mini app product using Mini App Name, Mini App Product Name, Mini App Partner Name, Mini App Partner ID, and Mini App SKU Identifier. For one-time charges, the display name is `[Mini App Name] - [Mini App Product Name]` with a 30-character maximum, and the SKU is `[Mini App SKU Identifier]|[Mini App Partner Name]|[Mini App Partner ID]`; the SKU must be unique within the host app, no more than 128 characters, and all three pipe-separated elements must be present. For subscriptions, the mini-app display name and product display name each have a 30-character maximum, and the same three-element SKU format and uniqueness rule apply. [REUSABLE REQUIREMENT] Make partner_id, mini_app_id, product_id, and Apple SKU a versioned, immutable mapping; validate required components, length, uniqueness, and display/SKU correspondence before purchase creation; preserve the exact submitted Apple identifiers in transaction, report, refund, and payout evidence. [LIMIT] The naming and length rules are Apple-specific API constraints, not a portable identifier standard. Direct fetch verified HTTP 200 on 2026-10-02.
### 3. `apple_advanced_commerce_eligibility_and_change_governance`
- **Source:** Apple Developer — Advanced Commerce API
- **Evidence level:** `primary-platform`
- **Topic:** `apple/advanced-commerce-api/eligibility-catalog-change-governance`
- **Primary URL:** https://developer.apple.com/in-app-purchase/advanced-commerce-api/
- **Source URLs:**
  - https://developer.apple.com/in-app-purchase/advanced-commerce-api/

[APPLE PLATFORM PATTERN] Apple limits Advanced Commerce API access to apps whose core business uses Apple In-App Purchases for an exceptionally large one-time catalog (including mini apps), an exceptionally large subscription catalog, or subscriptions with optional add-on content; a Mini Apps Partner Program app may apply by explaining how the API is needed for that program. The host must keep the purchase catalog as individual product identifiers/SKUs in its own system, use StoreKit 2, support subscription management, use the App Store Server API and App Store Server Notifications V2, and provide an in-app refund path. Access is handled per app; adding product identifiers, new business models, or significant price changes requires an update to the Advanced Commerce API Access form. Apple also states that Advanced Commerce purchases cannot be promoted on the App Store and currently cannot use subscription offers, Family Sharing, or StoreKit Testing in Xcode. [REUSABLE REQUIREMENT] Store an app-scoped commerce-eligibility record and approved capability profile; gate catalog mutations and feature flags against that record, require review evidence for product/business-model/price changes, and reject unsupported Apple commerce features instead of silently treating them as portable capabilities. [LIMIT] Eligibility and feature restrictions are Apple platform behavior, not a generic marketplace policy. Direct fetch verified HTTP 200 on 2026-10-02.
### 4. `apple_transaction_subscription_event_reconciliation`
- **Source:** Apple Developer Documentation — App Store Server API, Get Transaction History, Get All Subscription Statuses, and App Store Server Notifications V2
- **Evidence level:** `primary-platform`
- **Topic:** `apple/app-store-server-api/transaction-subscription-event-reconciliation`
- **Primary URL:** https://developer.apple.com/documentation/appstoreserverapi.md
- **Source URLs:**
  - https://developer.apple.com/documentation/appstoreserverapi.md
  - https://developer.apple.com/documentation/appstoreserverapi/get-transaction-history.md
  - https://developer.apple.com/documentation/appstoreserverapi/get-all-subscription-statuses.md
  - https://developer.apple.com/documentation/appstoreservernotifications/app-store-server-notifications-v2.md

[APPLE PLATFORM PATTERN] Apple’s App Store Server API returns App Store-signed transaction and subscription-renewal information as JWS from a server-side API that is independent of whether the customer installs, removes, or reinstalls the app. Get Transaction History covers auto-renewable, non-renewing, non-consumable, and consumable purchases, including refunded or revoked transactions and transactions marked finished or unfinished; it supports filters and paginated revision tokens, and subsequent revision requests must preserve the same optional query parameters. Get All Subscription Statuses returns a customer’s auto-renewable subscriptions grouped by subscription group and can filter statuses such as active and Billing Grace Period. App Store Server Notifications V2 delivers status events to an HTTPS/TLS 1.2+ endpoint; Apple asks the server to return 200–206 on success and 40x/50x on failure so it retries. [REUSABLE REQUIREMENT] Maintain an append-only commercial event ledger keyed by Apple transaction/product identifiers plus the host’s mini_app_id and partner_id; reconcile notification events with authoritative history/status/refund reads, retain a revision cursor per stable query, model refund/revoke/grace-period transitions explicitly, and support replay after notification loss or delivery retry. Do not make app installation state or exactly-once notification delivery the source of truth. [LIMIT] The ledger keys, ownership mapping, and replay policy are proposed host controls; Apple defines the API and delivery semantics. Direct fetches verified HTTP 200 on 2026-10-02.
### 5. `apple_refund_consumption_consent_and_deadline_governance`
- **Source:** Apple Developer Documentation — Send Consumption Information
- **Evidence level:** `primary-platform`
- **Topic:** `apple/refunds/consumption-information/consent-deadline`
- **Primary URL:** https://developer.apple.com/documentation/appstoreserverapi/send-consumption-information.md
- **Source URLs:**
  - https://developer.apple.com/documentation/appstoreserverapi/send-consumption-information.md
  - https://developer.apple.com/documentation/appstoreservernotifications/app-store-server-notifications-v2.md

[APPLE PLATFORM PATTERN] When a customer requests a refund for any In-App Purchase type, Apple can send a CONSUMPTION_REQUEST notification. If the customer has provided consent, the developer may call Send Consumption Information; if not, it must not respond with the consumption data, and Apple says to respond within 12 hours of receiving the notification. Apple makes the developer solely responsible for valid consent to share the customer’s personal data, describes consent as freely given, specific, informed, and unambiguous, and says to stop sending data when consent is withdrawn or no longer valid. The consent is separate from App Tracking Transparency, and adopting the API requires answering App Privacy questions; Apple directs access/deletion requests for this consumption data to privacy.apple.com. [REUSABLE REQUIREMENT] Keep a mini-app-scoped refund-consumption queue with Apple transaction/product identity, notification receipt time, consent version and timestamp, withdrawal state, payload hash, send/skip decision, and 12-hour SLA evidence; use a no-data path when consent is absent and keep refund decisioning separate from host entitlement revocation. [LIMIT] This is Apple’s refund-information sharing contract; it does not decide the host’s partner liability or revenue-allocation terms. Direct fetch verified HTTP 200 on 2026-10-02.
### 6. `apple_publisher_payout_evidence_and_settlement_reconciliation`
- **Source:** Apple App Store Connect Help — Financial report fields, Summary Sales Report, and Payment information
- **Evidence level:** `primary-platform`
- **Topic:** `apple/app-store-connect/financial-reports/publisher-payout-settlement-evidence`
- **Primary URL:** https://developer.apple.com/help/app-store-connect/reference/financial-report-fields/
- **Source URLs:**
  - https://developer.apple.com/help/app-store-connect/reference/financial-report-fields/
  - https://developer.apple.com/in-app-purchase/advanced-commerce-api/
  - https://developer.apple.com/help/app-store-connect/reference/summary-sales-report/
  - https://developer.apple.com/help/app-store-connect/reference/payment-information/

[APPLE REPORTING PATTERN] Apple says Advanced Commerce purchases appear in App Store Connect Summary Sales Reports and Payments and Financial Reports. In Financial report fields, Advanced Commerce one-time charges expose the submitted item.displayName in Title and item.SKU in ISAN/Other Identifier; subscriptions list item.displayName values in alphanumeric order and the corresponding item.SKU values in that display-name order. Apple’s reports expose product/SKU identity, product type, units or quantity, per-unit and extended partner share/developer proceeds, customer price, customer/proceeds currencies, country, fiscal period, sale/return, and parent app identifier. The Summary Sales Report documents refunds as negative Units and Customer Price with positive Developer Proceeds. Payment information separately reports units sold, earned, taxes and adjustments, carry-forward balances, total owed, exchange rate, estimated versus actual proceeds, and payment date. [REUSABLE REQUIREMENT] Build a versioned settlement-evidence record linking host generic product ID, Apple SKU/display name, mini_app_id, partner_id, Apple transaction events, report period, sale/return or refund state, units, customer price, proceeds, currencies, country, tax/adjustment, and payout status; reconcile transaction/event data to Summary Sales and Financial reports, then to actual payment proceeds and exchange-rate evidence. Handle negative/partial refunds and multi-item subscription ordering deterministically. [LIMIT] Apple reports evidence the platform settlement and payout; they do not prescribe the host’s contractual split or partner payout schedule, which must remain explicit in the marketplace ledger. Direct fetches verified HTTP 200 on 2026-10-02.
### 7. `google_play_catalog_identity_and_reconciliation`
- **Source:** Android Developers — Manage your product catalog
- **Evidence level:** `official-platform-doc`
- **Topic:** `monetization_settlement_catalog`
- **Primary URL:** https://developer.android.com/google/play/billing/manage-catalog
- **Source URLs:**
  - https://developer.android.com/google/play/billing/manage-catalog

[SOURCE FACT] Google's Play Developer API guide separates one-time and subscription catalog models. Subscription catalog management uses monetization.subscriptions plus base-plan and offer endpoints for regional availability and pricing; the guide recommends a diff system that matches product IDs and compares subscription/base-plan/offer attributes. It also warns that latency-tolerant batch updates can take up to 24 hours to reach devices. [SCOPE/LIMITATION] This is the Google Play catalog/publishing model for an Android app, not a universal mini-app SKU schema or a guarantee that host-catalog changes are immediately visible. [SETTLEMENT REQUIREMENT] Make the Play catalog a versioned settlement input: map each mini-app commercial item to package name, product ID, product type, base-plan/offer identity, region, price/configuration version, and effective timestamps; run a scheduled diff against Play and hold new settlement rows for unresolved identity/price drift. Persist the diff report and resolution rather than collapsing every subscription into one product ID.
### 8. `google_play_subscription_price_cohort_and_payout_timing`
- **Source:** Google Play Console Help — Understanding subscriptions
- **Evidence level:** `official-platform-guidance`
- **Topic:** `subscription_settlement_reconciliation`
- **Primary URL:** https://support.google.com/googleplay/android-developer/answer/12154973
- **Source URLs:**
  - https://support.google.com/googleplay/android-developer/answer/12154973

[SOURCE FACT] Google defines a base plan by billing period, renewal type (auto-renewing or prepaid), and price; a subscription can have multiple base plans and offers. A price change applies immediately to new purchases, while existing auto-renewing users can remain in legacy price cohorts; opt-in increases give at least 30 days, and opt-out increases use a 30/60-day region-dependent notice period. Installment subscribers are paid out as users make monthly payments rather than upfront. [SCOPE/LIMITATION] These rules describe Play subscriptions and region-specific Play behavior; they do not define the host's own tax or accounting policy. [SETTLEMENT REQUIREMENT] Maintain a line-level expected-charge schedule keyed by product ID, base plan, offer/phase, region, price cohort, billing mode, and effective date. Reconcile each renewal, top-up, or installment charge and payout timing against that schedule; never forecast from today's catalog price or assume subscription sign-up equals an upfront payout.
### 9. `google_play_voided_purchase_rolling_reconciliation`
- **Source:** Google Play Developer API — purchases.voidedpurchases.list
- **Evidence level:** `official-api-reference`
- **Topic:** `voided_purchase_settlement`
- **Primary URL:** https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.voidedpurchases/list
- **Source URLs:**
  - https://developers.google.com/android-publisher/api-ref/rest/v3/purchases.voidedpurchases/list

[SOURCE FACT] purchases.voidedpurchases.list returns canceled, refunded, or charged-back purchases. It supports filters for one-time or subscription type, pagination, and quantity-based partial refunds; startTime cannot be older than 30 days and is based on when Google saw the record as voided, while voidedTime is returned separately. For subscription voids, Google warns renewal orders share a purchase token, so consumers must use orderId to uniquely identify orders. [SCOPE/LIMITATION] The 30-day constraint is an API query limit, not a retention requirement; API records alone are not the host's full order ledger. [SETTLEMENT REQUIREMENT] Run a frequent rolling, paginated reconciliation with a persisted watermark plus overlap, key adjustments by order ID and quantity where applicable, and record purchase/voided times, reason, source, and partial quantity. Link each void to its original order and feed the resulting refund, chargeback, or quantity adjustment into payout reconciliation and exception queues.
### 10. `google_play_financial_report_and_order_ledger`
- **Source:** Google Play Console Help — Download sales and payout reports; Manage your app's orders and issue refunds
- **Evidence level:** `official-help`
- **Topic:** `financial_reporting_payout_reconciliation`
- **Primary URL:** https://support.google.com/googleplay/android-developer/answer/2482017?hl=en-sg
- **Source URLs:**
  - https://support.google.com/googleplay/android-developer/answer/2482017?hl=en-sg
  - https://support.google.com/googleplay/android-developer/answer/2741495?hl=en

[SOURCE FACT] Google Play provides monthly earnings reports and daily estimated sales reports; the latter add recently charged or refunded transactions and may take several days to complete. Financial data is UTC, downloadable as CSV, and Google instructs developers to sum Merchant currency separately for Charge/Charge refund and Google fee/Google fee refund. Order management exposes order ID, item details, cost, taxes, fees, history, and statuses such as pending refund, refunded, and partially refunded; refunds after payout are deducted from a future payout. [SCOPE/LIMITATION] Google's estimate/earnings timing and currency columns are Play reporting semantics; they do not replace the host's accounting close or local tax ledger. [SETTLEMENT REQUIREMENT] Build a UTC-period close: ingest daily estimates as provisional, reconcile against the monthly earnings report, join order-management state by order ID, and keep separate gross charges, refunds, Google fees, and net payout or adjustment buckets. Flag missing or late rows, statuses, and payout offsets as exceptions; do not treat a successful in-app purchase event as settled revenue.
### 11. `google_play_price_refund_transparency`
- **Source:** Google Play Console Help — Subscriptions; Manage your app's orders and issue refunds
- **Evidence level:** `official-policy-and-help`
- **Topic:** `pricing_refund_transparency`
- **Primary URL:** https://support.google.com/googleplay/android-developer/answer/9900533
- **Source URLs:**
  - https://support.google.com/googleplay/android-developer/answer/9900533
  - https://support.google.com/googleplay/android-developer/answer/12154973
  - https://support.google.com/googleplay/android-developer/answer/2741495?hl=en

[SOURCE FACT] Google Play's subscriptions policy requires explicit disclosure of cost, billing frequency, automatic renewal, whether a subscription is required, trial duration and price, trial-to-paid conversion, localized terms, and a clear cancellation path; it says users should not have to take extra action to review the terms. Google distinguishes cancellation (normally access through the current period, with no automatic refund) from subscription refund-and-revoke of the latest order versus refund-only for an older order; partial refunds affect payout and service-fee amounts. [SCOPE/LIMITATION] These are Google Play policy and help rules for Play-billed subscriptions; country-specific law and available refund methods can change the result, and a hosted mini-app store must not present them as universal refund rights. [SETTLEMENT REQUIREMENT] At purchase, render and persist the exact localized Play quote — total price and currency, billing period, trial-to-paid date, auto-renewal, offer or price version, and cancellation/refund route — and provide a direct subscription-management link. In the back office, model cancellation, refund, refund-and-revoke, and partial refund as distinct order events and project their payout and service-fee adjustments; never infer a refund from cancellation or present a monthly breakdown as the amount charged.
### 12. `immutable_balance_transaction_payout_batch_reconciliation`
- **Source:** Stripe Documentation — Reporting and reconciliation; Payout reconciliation
- **Evidence level:** `official-vendor-pattern+design-proposal`
- **Topic:** `settlement-ledger/immutable-provider-transactions/payout-batch-linkage`
- **Primary URL:** https://docs.stripe.com/plan-integration/get-started/reporting-reconciliation
- **Source URLs:**
  - https://docs.stripe.com/plan-integration/get-started/reporting-reconciliation
  - https://docs.stripe.com/payouts/reconciliation

[SOURCE FACT — STRIPE-SPECIFIC] Stripe recommends automatic payouts because they preserve the association between each transaction and the payout batch; its reporting guide says to retrieve payouts asynchronously after payout.paid or payout.reconciliation_completed, list balance transactions with the payout parameter, and paginate through all results. The same guide describes BalanceTransaction objects as automatically created for balance-affecting credits/debits, immutable, replayable as a ledger, and reports that a refund creates a new balance transaction negating the original. The payout-reconciliation guide exposes provider transaction types such as charge, refund, fee, and payout and links each row to its source object; Stripe says manual payouts do not have transaction-level reporting for the amount selected by the caller. [DESIGN PROPOSAL] Model each provider posting as an append-only settlement row with provider_event_id or transaction_id, payout_id, source_ref, transaction_type, signed_amount, currency, provider timestamps, and tenant_id/app_id/publisher_id attribution; close a payout only after all pages are ingested and the signed total agrees with the provider batch. Prefer automatic/provider-linked publisher payouts for governed settlement, or mark manual and instant payouts as requiring an explicit platform allocation record. [LIMITATION] This is Stripe Connect behavior and an implementation pattern, not a universal ledger standard or proof that any provider record is double-entry by itself.
### 13. `payout_report_watermarks_failed_payouts_ending_balance`
- **Source:** Stripe Documentation — Payout reconciliation report
- **Evidence level:** `official-vendor-pattern+design-proposal`
- **Topic:** `reconciliation/report-runs/watermarks/failed-payouts/ending-balance`
- **Primary URL:** https://docs.stripe.com/reports/payout-reconciliation
- **Source URLs:**
  - https://docs.stripe.com/reports/payout-reconciliation

[SOURCE FACT — STRIPE-SPECIFIC] Stripe's payout reconciliation report is designed to match a bank payout with its payment and other transaction batch. It offers itemized rows for payments, refunds, disputes, fees, and other balance transactions; separate sections for failed automatic payouts and for transactions not settled by the report end date; and CSV metadata that can match accounting-system records. Stripe states that payout arrival and reconciliation-data availability are separate, reports are grouped by the payout's estimated arrival date rather than the bank-posting date, Dashboard reports cover complete days while the Reporting API can request partial days, and reporting-availability webhooks are emitted for 00:00 UTC and 12:00 UTC data. [DESIGN PROPOSAL] Treat every imported report as a versioned reconciliation run with provider, report_type, requested interval, payout/arrival-date selector, source watermark, generated_at, freshness, row count/hash, pagination state, and status. Reconcile payout batches, failed payouts, and unsettled ending-balance rows as distinct control totals; keep pending, late, failed, and corrected rows visible instead of silently closing a period on payout arrival. [LIMITATION] These timing, report sections, and column semantics are Stripe report behavior, not a universal settlement close calendar; the marketplace must define its own cutoff, late-data, and correction policy.
### 14. `dispute_liability_reserve_and_publisher_payout_gate`
- **Source:** Stripe Connect Documentation — Disputes, Handle refunds and disputes, Account balances, and Connected-account reserves
- **Evidence level:** `official-vendor-pattern+design-proposal`
- **Topic:** `disputes/chargebacks/reserves/negative-balances/publisher-payout-controls`
- **Primary URL:** https://docs.stripe.com/connect/disputes
- **Source URLs:**
  - https://docs.stripe.com/connect/disputes
  - https://docs.stripe.com/connect/marketplace/tasks/refunds-disputes
  - https://docs.stripe.com/connect/account-balances
  - https://docs.stripe.com/connect/connected-account-reserves

[SOURCE FACT — STRIPE-SPECIFIC] Stripe says charge type and negative-balance responsibility determine who responds to a dispute and which account pays the chargeback and fees. For destination/separate or other indirect marketplace charges, Stripe debits refunds, disputed amounts, and associated fees from the platform balance; it recommends a dispute-created webhook and describes transfer reversals for recovering funds from a connected account. Stripe also documents that refunds and chargebacks can create negative connected-account balances, that payouts are blocked while such a balance is negative, and that platform reserve/collection entries can appear as reserve_transaction and connect_collection_transfer records. Its Connected-account reserves documentation says reserved funds are withheld from payouts while exposed to refunds or disputes, released according to a hold/plan or risk event, capped at 180 days, and that the reserve policy should be explained in marketplace terms. [DESIGN PROPOSAL] Keep an explicit dispute case and liability_owner beside the immutable sale/refund entries: original payment reference, dispute/chargeback event, evidence deadline, reserve hold/release, payout-block decision, transfer reversal or recovery, negative-balance state, and final outcome. Gate publisher payout on the applicable reserve/liability policy and post every recovery or reversal as a compensating ledger entry; do not mutate the original sale or assume the platform is always liable. [LIMITATION] The ownership, reserve objects, 180-day limit, and API events are Stripe-specific (the reserves page is marked private preview) and jurisdiction/contract dependent; do not present them as a universal marketplace rule or copy their thresholds without legal and risk review.
### 15. `value_date_pending_posted_and_marketplace_commission_reconciliation`
- **Source:** Adyen Documentation — Balance Platform Accounting Report and Marketplace-level reporting
- **Evidence level:** `official-vendor-pattern+design-proposal`
- **Topic:** `settlement-ledger/value-date/transfer-transaction-linkage/commission-payout-reconciliation`
- **Primary URL:** https://docs.adyen.com/platforms/reports-and-fees/balance-platform-accounting-report/
- **Source URLs:**
  - https://docs.adyen.com/platforms/reports-and-fees/balance-platform-accounting-report/
  - https://docs.adyen.com/marketplaces/reconciliation-use-cases/first-year-focus/platform-level-reporting/

[SOURCE FACT — ADYEN-SPECIFIC] Adyen's Balance Platform Accounting Report is a daily report of balance changes across liable and user balance accounts. Adyen recommends API responses and webhooks for real-time payment tracking and ledger updates, followed by daily reconciliation from the accounting report. It distinguishes a pending Transfer, whose linked lifecycle events share a Transfer Id and do not yet represent real money movement, from a posted Transaction with a Transaction Id; refunds start a new flow with a new Transfer Id. The report exposes account, transfer/transaction IDs, category/status/type, booking date, value date, currency, received/reserved/balance amounts, references, PSP references, and split-purpose fields such as Commission, PaymentFee, and VAT. Its marketplace reporting guidance separately describes monthly profitability and transaction/user-level profitability: commissions and costs are reflected on the liable account, internal transfers and invoice deductions must be included, and a Payout Report can be generated as a CSV containing payouts and the transactions that make them up; it recommends joining the Payment Accounting Report PSP Reference to the Balance Platform Accounting Report Payment PSP Reference. [DESIGN PROPOSAL] Use pending-versus-posted states and provider transfer_id/transaction_id as distinct lifecycle keys; post cash only when the provider confirms a completed transaction/value-date effect, and model refunds, chargebacks, fees, taxes, publisher shares, platform commission, reserves, internal transfers, payouts, and provider-invoice adjustments as separate attributable legs. Reconcile daily by value date, then perform a monthly publisher/platform close that joins stable PSP references and account-holder mappings; keep profitability and payout views derived from the immutable ledger. [LIMITATION] These are Adyen platform/report patterns, not a universal marketplace schema; account structures, split types, value-date behavior, and invoice/report availability depend on Adyen configuration and the marketplace contract.
### 16. `bank_statement_entry_reference_booking_value_date_and_control_totals`
- **Source:** ISO 20022 — Bank-to-Customer Cash Management Message Definition Report (2020–2021 Payments SEG review draft)
- **Evidence level:** `official-standard-draft+design-proposal`
- **Topic:** `iso20022/camt053/bank-statement/reconciliation-control-totals`
- **Primary URL:** https://www.iso20022.org/sites/default/files/2020-12/ISO20022_MDRPart2_BankToCustomerCashManagement_2020_2021_v1_ForSEGReview.pdf
- **Source URLs:**
  - https://www.iso20022.org/sites/default/files/2020-12/ISO20022_MDRPart2_BankToCustomerCashManagement_2020_2021_v1_ForSEGReview.pdf

[SOURCE FACT — ISO 20022 OFFICIAL REVIEW DRAFT, NOT A FINAL UNIVERSAL MARKETPLACE SPEC] The official ISO 20022 message-definition report defines camt.053 BankToCustomerStatement as reporting booked entries and balances for an account, with an account/report identifier, creation time, reporting period, copy/duplicate indicator, balance, transaction summary, and entry records. Entry structure includes amount, credit/debit indicator, reversal indicator, status, booking date, value date, account-servicer reference, availability, bank transaction code, charges, entry details, and additional information; related transaction details can carry message, end-to-end, transaction, account-owner, and account-servicer references. The draft says at least one reference should identify an entry and its underlying transactions, and transaction summaries can provide counts, sums, net totals, and debit/credit totals. [DESIGN PROPOSAL] Use the same separation in a marketplace bank-settlement adapter: retain source statement_id/message_id and sequence, account_scope, entry_ref, signed amount/currency, credit/debit and reversal flags, booking/value/availability timestamps, bank/provider code, charges, and end-to-end/provider references; reconcile entry totals and debit/credit control totals to the bank statement before marking a payout as cash-settled. Keep booked, available, and forecast/pending states distinct and map statement entries back to publisher/app/tenant only through a governed reference table. [LIMITATION] The fetched document is explicitly a 2020–2021 evaluation draft and describes bank-to-customer messaging, not a mini-app marketplace ledger, legal payout obligation, or universal double-entry chart of accounts; confirm the exact approved message version and local market practice before implementation.
### 17. `refund_liability_contract_liability_periodic_update`
- **Source:** IFRS Foundation — IFRS 15 Revenue from Contracts with Customers (2026 issued text)
- **Evidence level:** `official-accounting-standard+design-proposal`
- **Topic:** `accounting/refund-liability/contract-liability/publisher-payable/revenue-recognition`
- **Primary URL:** https://www.ifrs.org/content/dam/ifrs/publications/html-standards/english/2026/issued/ifrs15.html
- **Source URLs:**
  - https://www.ifrs.org/content/dam/ifrs/publications/html-standards/english/2026/issued/ifrs15.html

[SOURCE FACT — ACCOUNTING STANDARD, APPLICATION DEPENDS ON THE REPORTING ENTITY AND CONTRACT] IFRS 15 paragraph 16 requires consideration received before the relevant transfer to be recognised as a liability until the specified recognition conditions/events occur; the liability represents an obligation to transfer goods or services or refund consideration. Paragraph 55 requires a refund liability when the entity expects to refund some or all consideration, measured at the amount not expected to be entitled to, with the refund liability and corresponding contract-liability change updated at the end of each reporting period for changed circumstances. Paragraphs 60–65 address significant financing components when payment timing provides a financing benefit. [DESIGN PROPOSAL] Keep customer cash received, contract/entitlement liability, publisher payable, platform commission, refund liability, dispute reserve, and recognised revenue as separate ledger accounts or dimensions; make each refund/reversal/dispute estimate revision an append-only adjustment linked to the original sale and reporting-period close, with approval/evidence and the accounting-policy version recorded. Do not treat a payment as publisher revenue merely because funds arrived or an entitlement was issued. [LIMITATION] IFRS 15 is an accounting framework, not a payment-provider API or universal marketplace settlement contract; whether and how it applies depends on the entity's role, performance obligations, principal/agent analysis, jurisdiction, and adopted reporting framework.
## Proposed settlement data model
| Record | Required identity/evidence | Control |
|---|---|---|
| `commercial_item_version` | `mini_app_id, partner_id, host_product_id, provider_product_id/SKU, product type, base plan/offer, region, currency, localized terms, effective time` | `Immutable mapping; diff before activation; retain submitted identifiers` |
| `provider_event` | `provider, provider_event_id/message_id, event time, raw/envelope hash, notification type, purchase/order/transaction token` | `Append-only; deduplicate; enrich from authoritative API; replayable` |
| `entitlement_event` | `customer/account reference, product, grant/revoke/pending/grace/refund state, source transaction, effective time` | `Host/backend authority; never grant from an unverified pending signal` |
| `settlement_entry` | `signed amount, currency, sale/refund/fee/commission/tax/reserve/payout type, provider reference, app/tenant/publisher attribution` | `Compensating entries only; original rows immutable` |
| `reconciliation_run` | `provider/report type, period selector, watermark, pagination, generated time, row count/hash, control totals, status` | `Keep late/failed/unsettled exceptions visible; close only after evidence is complete` |
| `publisher_payout` | `publisher, payable period, reserve/liability state, deductions, payout batch, payment status, bank reference` | `Apply contract and dispute policy; retain payout and bank evidence` |
| `accounting_policy_link` | `principal/agent role, refund liability, contract liability, recognition rule, policy version, approval` | `Do not treat purchase, entitlement, or cash arrival as automatic revenue` |

## Pilot acceptance tests
1. Create one consumable, one non-consumable and one auto-renewing subscription for two mini-app partners; verify immutable host/provider identity mapping and rejection of duplicate or malformed identifiers.
1. Run pending, purchased, grace-period, renewal, cancellation, refund-only, refund-and-revoke, partial-refund, void, chargeback and out-of-order notification scenarios; verify idempotent state transitions and authoritative API enrichment.
1. Drop or duplicate notifications and force pagination retry; verify watermark overlap, replay, duplicate prevention and eventual agreement with provider history/status APIs.
1. Generate daily estimate, monthly earnings, payout, fee, tax, reserve and bank statement inputs; verify separate control totals, UTC period close, late-data exception handling and cash-settlement matching.
1. Apply a dispute to a publisher with a pending payout; verify reserve/liability decision, payout block, transfer reversal or recovery, negative-balance handling and compensating ledger entry.
1. Change a price, offer, base plan, localized term, refund route or commerce capability; verify catalog diff, review evidence, effective-date cohorting and user-facing disclosure.
1. Verify the mini-app runtime receives only the intended entitlement/result reference and cannot access provider payment credentials, private settlement data, raw age/identity data, or payout controls.

## Gaps and follow-up
- The public sources do not determine the target deployment's merchant-of-record, principal/agent analysis, tax obligations, payout schedule, reserve duration, dispute SLA, currency conversion policy, or local accounting basis. These require Legal, Finance, Tax, Risk and provider contracts.
- Apple and Google rules change by region and platform program; maintain dated provider adapters and revalidate eligibility, fees, reporting fields and refund behavior before production.
- The ISO 20022 source used here is an official 2020–2021 message-definition evaluation draft, not a final universal mini-app ledger specification; it is retained as a mapping precedent only.
- Stripe reserve documentation includes provider-specific/private-preview behavior; do not make it a mandatory control without a selected provider and contractual review.
