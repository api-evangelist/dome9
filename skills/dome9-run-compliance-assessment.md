---
name: dome9-run-compliance-assessment
description: >-
  Run a CloudGuard (Dome9) compliance assessment — list rulesets, evaluate a
  bundle against cloud accounts, read the latest results and trend, and export
  an executive report. Use when asked to check an environment against CIS/PCI/
  NIST-style control sets or to report posture over time.
api: Dome9 API
base_url: https://api.dome9.com/v2
generated: '2026-09-07'
method: generated
source: openapi/dome9-api-openapi.json (https://api.dome9.com/swagger/docs/v2)
operations:
  - ComplianceRuleset_GetAccountBundles
  - ComplianceRuleset_GetBundleById
  - Assessment_RunBundleV2Async
  - AssessmentHistoryV2_GetLastAssessmentResults
  - AssessmentHistoryV2_GetBundleResults
  - AssessmentHistoryV2_GetAssessmentTrendV2
  - AssessmentHistoryV2_GetLastAssessmentResultsMiniToExecutiveSummary
  - Finding_GetBundleStats
---

# Run a CloudGuard compliance assessment

## 1. Find the ruleset

`GET /v2/Compliance/Ruleset` (`ComplianceRuleset_GetAccountBundles`) returns
every ruleset on the account — the shipped control-framework bundles and any
custom ones. One by id: `GET /v2/Compliance/Ruleset/{id}`
(`ComplianceRuleset_GetBundleById`).

Rulesets are versioned:
`GET /v2/Compliance/Ruleset/{id}/version/{version}`
(`ComplianceRuleset_GetVersionedBundleById`). Pin a version when you need a
comparable run.

## 2. Run it

`POST /v2/assessment/bundleV2` (`Assessment_RunBundleV2Async`) evaluates a
bundle against a cloud environment. Note the `V2` — the older
`/v2/AssessmentHistory` generation still exists and is not marked deprecated.
Always prefer the `V2` paths.

## 3. Read the results

| Want | Call |
|---|---|
| Latest results for accounts + bundles | `POST /v2/AssessmentHistoryV2/LastAssessmentResults` (`AssessmentHistoryV2_GetLastAssessmentResults`) |
| Same, without detail rows | `POST /v2/AssessmentHistoryV2/LastAssessmentResults/view` |
| Minimized payload | `POST /v2/AssessmentHistoryV2/LastAssessmentResults/minimized` |
| Most recent for one bundle | `GET /v2/AssessmentHistoryV2/bundleResults` (`AssessmentHistoryV2_GetBundleResults`) |
| One assessment by id | `GET /v2/AssessmentHistoryV2/{id}` (`AssessmentHistoryV2_GetHistoryAsync`) |
| Per-rule failure counts | `GET /v2/Compliance/Finding/bundle/{bundleId}/stats` (`Finding_GetBundleStats`) |

Prefer the `view` / `minimized` variants when you only need scores — the full
results payload carries every finding row.

## 4. Trend and reporting

- Trend: `GET /v2/AssessmentHistoryV2/assessmentTrendV2`
  (`AssessmentHistoryV2_GetAssessmentTrendV2`) — **maximum 93 days** per the
  contract. Per OU: `assessmentTrendOrganizationalUnit`.
- Executive report:
  `GET /v2/AssessmentHistoryV2/ExecutiveReport/{vendor}/{rulesetId}`
  (`AssessmentHistoryV2_GetLastAssessmentResultsMiniToExecutiveSummary`).
- CSV of one run: `GET /v2/AssessmentHistoryV2/csv/{assessmentResultId}`.
- Email a report link:
  `POST /v2/AssessmentHistoryV2/ExecutiveReport/email/{vendor}/{rulesetId}`.
  This **sends mail** — confirm the recipient with a human first.

## 5. Continuous rather than one-shot

To keep evaluating instead of running by hand, create a policy:
`POST /v2/ContinuousCompliancePolicyV2` (`ContinuousCompliancePolicyV2_PostAsync`).
It binds a ruleset to accounts or OUs. The POST and PUT return a **multi-status
body** listing successes and per-item errors — parse it, do not assume a 2xx
means everything applied.

## Caution

`DELETE /v2/AssessmentHistoryV2` (`AssessmentHistoryV2_DeleteHistoryAsync`)
deletes assessment history. No restore exists.
