# Mini App Store Standard for a Super App

**Research status:** Deli Deep iteration 5
**Evidence records:** 186
**Iteration 5 additions:** 12 portable catalog/evidence-federation findings / 35 unique source URLs, all rechecked HTTP 200
**Input context:** Alibaba Cloud SuperApp/WindVane POC resource sheet, tab `Software Info` (`gid=58086397`)

## Reading order

| # | File | Purpose |
|---:|---|---|
| 1 | [Executive summary](01-executive-summary.md) | Decision-level conclusions and standard shape |
| 2 | [Reference architecture and lifecycle](02-reference-architecture-lifecycle.md) | Control/data planes, catalog model, release state machine |
| 3 | [Security, privacy and control matrix](03-security-privacy-control-matrix.md) | P0/P1/P2 controls and evidence required |
| 4 | [Store UX, benchmark and operating model](04-store-ux-benchmark-operating-model.md) | Discovery, trust, analytics, monetization and support |
| 5 | [Evidence appendix](05-evidence-appendix.md) | All validated findings and URLs, without omission |
| 6 | [Research gaps and next validation](06-gaps-validation-plan.md) | Unknowns, pilot tests and decisions still required |
| 7 | [Iteration 2: launch, offline and trust](07-iteration-2-launch-offline-trust.md) | 36 detailed findings on deep links, resilience, payments, notifications and marketplace trust |
| 8 | [Iteration 3: accessibility, privacy and governance](08-iteration-3-accessibility-privacy-governance.md) | 20 normalized findings on inclusive UX, data governance, moderation, appeals and procedural controls |
| 9 | [Iteration 4: reliability, SLO and operations](09-iteration-4-reliability-slo-operations.md) | 10 findings on SLOs, error budgets, canary evidence, incident learning, tenant isolation, recovery objectives and telemetry schema governance |
| 10 | [Iteration 5: portable catalog and evidence federation](10-iteration-5-portable-catalog-and-evidence-federation.md) | 12 findings on W3C/Schema.org catalog projections, SPDX/CycloneDX/OpenVEX/in-toto evidence exchange, and OAuth publisher capability negotiation |

## Evidence rules

- `primary-standard`, `primary-government`, and `primary-official` are normative or official primary evidence.
- `official-vendor`, `official-platform-documentation`, and related labels describe vendor/platform practice, not a universal standard.
- `proposal` is a design recommendation derived from the evidence; it is not presented as a source requirement.
- `gap` means the public evidence was insufficient; it is not filled with a guess.
- Source credentials from the input spreadsheet were deliberately excluded.

## State

The append-only research state is in `../code/deli/super-app-mini-app-store-standard/state/` on the host. The repository publication should include a sanitized copy only.
