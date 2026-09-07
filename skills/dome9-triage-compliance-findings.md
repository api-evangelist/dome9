---
name: dome9-triage-compliance-findings
description: >-
  Triage CloudGuard (Dome9) compliance findings — search by cloud account and
  severity, read one finding in full, then acknowledge, comment, reassign,
  re-severity or archive it. Use when asked to work a posture backlog, review
  critical findings for an account, or hand findings to an owner.
api: Dome9 API
base_url: https://api.dome9.com/v2
generated: '2026-09-07'
method: generated
source: openapi/dome9-api-openapi.json (https://api.dome9.com/swagger/docs/v2)
operations:
  - Finding_Search
  - Finding_GetFinding
  - Finding_Acknowledge
  - Finding_AddComment
  - Finding_Assign
  - Finding_ChangeSeverity
  - Finding_Archive
  - CloudAccounts_Get
---

# Triage CloudGuard compliance findings

## Before you start

- Auth is **HTTP Basic**: username = V2 API key id, password = key secret.
  `curl -u $D9_KEY_ID:$D9_KEY_SECRET https://api.dome9.com/v2/CloudAccounts`
- The API is **region-pinned**. `api.dome9.com` is US. Use `api.eu1.dome9.com`,
  `api.ap1/ap2/ap3.dome9.com` or `api.cace1.dome9.com` for other tenants —
  see `conventions/dome9-conventions.yml`.
- There is **no idempotency key** on this API. Never blind-retry a POST.

## 1. Resolve the cloud account

`GET /v2/CloudAccounts` (`CloudAccounts_Get`) returns every onboarded AWS
account. Azure and Google live at `GET /v2/AzureCloudAccount`
(`AzureCloudAccount_Get`) and `GET /v2/GoogleCloudAccount`
(`GoogleCloudAccount_Get`). Keep the `id` — nearly every downstream call is
scoped by `cloudAccountId`.

## 2. Search findings

`POST /v2/Compliance/Finding/search` (`Finding_Search`) takes a filter body —
cloud account, region, VPC, IP, instance name, severity, entity type, entity
tags. Prefer this over listing: there is no pagination convention on the
collection GETs.

For counts rather than rows use
`POST /v2/Compliance/Finding/searchAggregate` (`Finding_SearchAggregate`) or
`GET /v2/Compliance/Finding/stats/aggregatedbyproperty`
(`Finding_GetStatsByProperty`).

Severities are `Informational | Low | Medium | High | Critical`.

## 3. Read one finding

`GET /v2/Compliance/Finding/{id}` (`Finding_GetFinding`). If you only hold the
finding key rather than the id, use
`POST /v2/Compliance/Finding/getByKey` (`Finding_GetFindingByKey`).

## 4. Act on it

| Intent | Call |
|---|---|
| Mark as reviewed | `PUT /v2/Compliance/Finding/{id}/acknowledge` (`Finding_Acknowledge`) |
| Leave a note | `POST /v2/Compliance/Finding/{id}/comment` (`Finding_AddComment`) |
| Give it an owner | `PUT /v2/Compliance/Finding/{id}/assign` (`Finding_Assign`) |
| Re-rate it | `PUT /v2/Compliance/Finding/{id}/severity` (`Finding_ChangeSeverity`) |
| Take it off the board | `POST /v2/Compliance/Finding/{id}/archive` (`Finding_Archive`) |
| Put it back | `POST /v2/Compliance/Finding/{id}/unarchive` (`Finding_Archive`) |

Archive is the **only reversible write on this API** — and no time window is
documented for the reversal, so do not assume one.

## 5. Bulk — and the blast radius

Every action above has a `bulk/*` form taking a list of ids, and a
`selectAll/*` form taking a **filter**:
`Finding_SelectAllAcknowledge`, `Finding_SelectAllAssign`,
`Finding_SelectAllSeverity`, `Finding_SelectAllComment`,
`Finding_SelectAllClose`.

`POST /v2/Compliance/Finding/selectAll/close` (`Finding_SelectAllClose`) closes
everything matching the filter. There is **no undo for close** and **no
idempotency key**. Confirm with a human, and dry-run the same filter through
`Finding_Search` first so you can show exactly what will be hit.

## Errors

The contract declares no 4xx/5xx responses and Check Point publishes no error
reference. Treat any non-2xx as opaque, surface the raw body, and do not retry
mutations automatically.
