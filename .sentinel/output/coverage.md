# Sentinel Coverage Baseline

This file is the **single evidence-based coverage map** for Sentinel Phase 0 and the **intake decision log** for Phase 1.

## Purpose

- show what the current automation repo already covers
- make review easier for both technical and non-technical stakeholders
- show the **full suite inventory** separately from the **Phase 1 decision view** so row counts are not misread
- prevent duplicate automation when BrowserStack cases are still marked as manual
- record whether a manual intake item needs **new automation**, an **extension to existing coverage**, or only a **traceability update**

> This is **not** a percentage report. It is a reviewed coverage map backed by real repo evidence.

## Update rules

### Phase 0
- establish the initial baseline from current repo evidence
- group by endpoint / business flow / major scenario area
- include plain-language business context for stakeholder review

### Phase 1
For each BrowserStack manual intake, update this file with one of the following decisions:
- **Net-new automation**
- **Extend existing coverage**
- **Traceability-only gap**

## 1. Current automation suite inventory

This inventory shows the **full current suite baseline** in the repo. The row count here should match the current suite estate and is intentionally separate from the Phase 1 decision view.

| Suite | Endpoint / Domain | Business Context | Scenarios currently covered | Status |
|---|---|---|---|---|
| `CreatePreadviceAndLabelTests` | `POST /routeDeliveryCreatePreadviceAndLabel` | Generates delivery preadvice and printable labels, including ParcelShop flows. | happy-path response, label and barcode validation, next-day variants, ParcelShop handling, label content checks | Active baseline |
| `CreatePreadviceTests` | `POST /routeDeliveryCreatePreadvice` | Produces delivery preadvice and routing data before label generation. | happy-path response, source/client validation, next-day variants, ParcelShop handling | Active baseline |
| `DetermineDeliveryRoutingTests` | `POST /determineDeliveryRouting` | Determines the operational routing outcome for delivery requests. | successful routing response, source/client validation, ParcelShop-related routing checks | Active baseline |
| `CreatePreadviceReturnBarcodeAndLabelTests` | `POST /routeDeliveryCreatePreadviceReturnBarcodeAndLabel` | Supports reverse-logistics flows where return barcode and label outputs are required. | return happy path, country-of-origin validation, unsupported-postcode handling, return label and barcode validation | Active baseline |
| `CreatePreadviceReturnBarcodeTests` | `POST /routeDeliveryCreatePreadviceReturnBarcode` | Supports reverse-logistics flows where a return barcode is required without label generation. | return happy path, country-of-origin validation, unsupported-postcode handling, barcode validation | Active baseline |

## 2. Phase 1 intake decision view

This view is grouped by **intake-relevant scenario areas**. Its row count may differ from the suite inventory because multiple existing tests can support the same intake decision.

| Area / Endpoint | Business Context | Scenarios currently covered | How intake maps to current coverage | BrowserStack / Manual Intake | Phase 1 Decision | Decision rationale |
|---|---|---|---|---|---|---|
| `POST /routeDeliveryCreatePreadviceAndLabel` | Generates routing and printable label data for deliveries, including ParcelShop flows. Failure can block parcel processing or misroute consignments. | happy-path response, ParcelShop routing, label/barcode validation, label content checks | `QA-T6504` aligns to the existing ParcelShop positive path already covered in `deliveryPreadviceAndLabelParcelShopIdTwoValueTest()` and `checkParcelShopIdInResponseLabelTest()` | `QA-T6504` — ParcelShop delivery | **Extend existing coverage** | Extend `checkParcelShopIdInResponseLabelTest()` to include the exact ParcelShop routing expectations from `QA-T6504`, such as delivery method and sort-level assertions, instead of adding a duplicate test path. |
| `POST /routeDeliveryCreatePreadvice` | Produces delivery routing decisions before label generation. Failure can affect sort selection and downstream despatch handling. | happy-path response, source/client validation, next-day behavior, ParcelShop handling | No active BrowserStack intake is currently mapped to this area in Phase 1. | _No active Phase 1 intake yet_ | Baseline only | Existing coverage is already visible here; update this row only when a nominated manual case needs review against the current preadvice scenarios. |
| `POST /determineDeliveryRouting` | Determines routing logic used to place parcels onto the correct operational path. Errors may cause service or route misclassification. | successful routing response, source/client validation, ParcelShop-related routing checks | No active BrowserStack intake is currently mapped to this area in Phase 1. | _No active Phase 1 intake yet_ | Baseline only | Current suite coverage is in place; use this row when a manual routing-intent case needs a duplicate-check and disposition. |
| `POST /routeDeliveryCreatePreadviceReturnBarcodeAndLabel` | Supports reverse-logistics flows where return barcode and label outputs are required. | return happy path, country-of-origin validation, unsupported-postcode handling, return label and barcode validation | No active BrowserStack intake is currently mapped to this area in Phase 1. | _No active Phase 1 intake yet_ | Baseline only | This suite is part of the current estate and is shown here so reviewers do not mistake the Phase 1 scope for the full repo coverage picture. |
| `POST /routeDeliveryCreatePreadviceReturnBarcodeAndLabel` | NRC returns: label must not contain Zone/Sub Zones for tier 4 clients | return happy path, label validation | No explicit assertion for tier 4 label content | `QA-T8666` | **Net-new automation** | Add test for tier 4 client label content (absence of Zone/Sub Zones) |
| `POST /routeDeliveryCreatePreadviceReturnBarcode` | Supports reverse-logistics flows where a return barcode is required without label generation. | return happy path, country-of-origin validation, unsupported-postcode handling, barcode validation | No active BrowserStack intake is currently mapped to this area in Phase 1. | _No active Phase 1 intake yet_ | Baseline only | Keep this row as baseline evidence until a matching manual intake is selected for review. |

## 3. POC completion evidence

The following intake decisions are the current proof points used to close Sentinel POC stage and proceed to external delivery handoff.

| Evidence item | Value |
|---|---|
| POC milestone label | `Sentinel POC Complete - External Delivery Ready (V1 Git Package)` |
| Approved intake decisions | `QA-T6504` (Extend existing coverage), `QA-T8666` (Net-new automation) |
| Decision record location | This file (`.sentinel/output/coverage.md`) |
| Required reviewer roles | QA/Automation lead, Domain/service engineer, Product/business stakeholder |
| Next milestone | External consumer-team package adoption (V1 Git package) |

## Review notes

- **Non-technical reviewers**: use the suite inventory to see the full current automation estate, then use the Phase 1 decision view to understand what the current intake will change.
- **Technical reviewers**: confirm the covered scenarios remain accurate and that the Phase 1 decision does not duplicate existing automation.
- Any new manual intake should update the **Phase 1 intake decision view** before implementation proceeds.
- 2026-04-15: Implementation sign-off for `QA-T6504` is recorded as user-confirmed in Copilot Chat; proceed with the approved **Extend existing coverage** path in `CreatePreadviceAndLabelTests#checkParcelShopIdInResponseLabelTest`.
