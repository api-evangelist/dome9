---
name: dome9-route-findings-to-a-destination
description: >-
  Wire CloudGuard (Dome9) posture findings out to a webhook, SNS topic, Slack or
  Teams channel, Jira/ServiceNow/PagerDuty, AWS Security Hub, Azure Defender for
  Cloud, GCP Security Command Center or Eventarc — and diagnose a destination
  that stopped receiving. Use for "send our findings to X" and "why did our
  webhook stop firing".
api: Dome9 API
base_url: https://api.dome9.com/v2
generated: '2026-09-07'
method: generated
source: openapi/dome9-api-openapi.json (https://api.dome9.com/swagger/docs/v2)
operations:
  - ContinuousComplianceNotification_Get
  - ContinuousComplianceNotification_Post
  - ContinuousComplianceNotification_Put
  - ContinuousComplianceNotification_Delete
  - ContinuousComplianceNotification_PublishOpenedFindingsByPolicyIdAsync
  - ContinuousComplianceNotification_GetAllCircuitBreakerAsync
  - ContinuousComplianceNotification_DeleteCircuitBreakerAsync
  - ContinuousComplianceNotification_WebhookJiraTokens
---

# Route CloudGuard findings to a destination

CloudGuard has no subscription endpoint. You create a **notification policy**
over the REST API and CloudGuard pushes to whatever destination that policy
names. Full channel catalog:
`asyncapi/dome9-notifications-webhooks.yml`.

## 1. See what already exists

`GET /v2/Compliance/ContinuousComplianceNotification`
(`ContinuousComplianceNotification_Get`), or by name:
`GET /v2/Compliance/ContinuousComplianceNotification/get-by-name`.

## 2. Create the policy

`POST /v2/Compliance/ContinuousComplianceNotification`
(`ContinuousComplianceNotification_Post`).

Pick trigger and destination in the same body:

- Triggers: `scheduledReport`, `changeDetection`, `sendOnEachOccurrence`,
  `alertsConsole`.
- Filter: `severities` (Informational/Low/Medium/High/Critical), `entityTypes`,
  `entityTags`, `entityNames`, `entityIds`.
- Destinations, each with its own object:
  `webhook`, `sns`, `slack`, `teams`, `eventarc`, ticketing
  (`ServiceNow` / `Jira` / `PagerDuty`), `awsSecurityHubIntegrationState`,
  `azureSecurityCenterIntegrationState`,
  `gcpSecurityCommandCenterIntegration` (`projectId` + `sourceId`), email.

### Generic webhook fields

Required: `url`, `httpMethod` (`Post`/`Put`/`Get`), `authMethod`
(`NoAuth`/`BasicAuth`), `formatType`.

`formatType` picks the payload dialect:
`JsonWithFullEntity`, `JsonWithBasicEntity`, `Json`, `JsonFirstLevelEntity`,
`PlainText`, `SplunkBasic`, `ServiceNow`, `QRadar`, `Jira`.

**Security note to pass on to the receiver:** CloudGuard signs nothing. The only
authentication offered is HTTP Basic to your endpoint, and `ignoreCertificate`
can disable TLS verification. The receiver cannot cryptographically prove a
delivery came from CloudGuard — put it behind an allowlist or a secret path.

## 3. Backfill the destination

`POST /v2/Compliance/ContinuousComplianceNotification/PublishOpenedFindings/{id}`
(`..._PublishOpenedFindingsByPolicyIdAsync`) replays **every open finding** for
that policy. That is the only replay available — there is no replay-by-time — so
on a large tenant it is a flood. Warn the human before firing it.

## 4. When a destination goes quiet

CloudGuard trips a **circuit breaker** on an integration that keeps failing, and
that is the only delivery-failure signal in the API.

1. `GET /v2/Compliance/ContinuousComplianceNotification/CircuitBreaker`
   (`..._GetAllCircuitBreakerAsync`) — list tripped integrations.
2. `GET /v2/Compliance/ContinuousComplianceNotification/CircuitBreaker/{id}`
   — check one policy.
3. Fix the endpoint, then
   `DELETE /v2/Compliance/ContinuousComplianceNotification/CircuitBreaker/{id}/{integrationType}`
   (`..._DeleteCircuitBreakerAsync`) to reset it.

For Jira, list issued tokens with
`GET /v2/Compliance/ContinuousComplianceNotification/webhookJiraTokens`.

## 5. Changing and removing

`PUT .../{id}` to update, `DELETE .../{id}` to remove. Deleting a policy stops
delivery immediately and there is no restore.
