---
name: dome9-onboard-cloud-account
description: >-
  Onboard an AWS, Azure or Google cloud account into CloudGuard (Dome9), place
  it in an organizational unit, verify CloudGuard has the permissions it needs,
  and force a data sync. Use when asked to connect a new cloud environment to
  CloudGuard posture management.
api: Dome9 API
base_url: https://api.dome9.com/v2
generated: '2026-09-07'
method: generated
source: openapi/dome9-api-openapi.json (https://api.dome9.com/swagger/docs/v2)
operations:
  - CloudAccounts_Post
  - AzureCloudAccount_Post
  - GoogleCloudAccount_Post
  - CloudAccounts_Get
  - CloudAccounts_GetMissingPermissionsAsync
  - CloudAccounts_ResetMissingPermissionsAsync
  - CloudAccounts_SyncNow
  - EntityFetchStatus_Get
  - CloudAccounts_MoveCloudAccountsToOrganizationalUnit
---

# Onboard a cloud account into CloudGuard

## Before you start

HTTP Basic with the V2 API key id/secret. Pick the regional host for the tenant
(`conventions/dome9-conventions.yml`). **Onboarding is a POST with no
idempotency key — a retry creates a second account record.** Read back with
`CloudAccounts_Get` before retrying anything.

## 1. Create the account

- **AWS** — `POST /v2/CloudAccounts` (`CloudAccounts_Post`)
- **Azure** — `POST /v2/AzureCloudAccount` (`AzureCloudAccount_Post`)
- **Google** — `POST /v2/GoogleCloudAccount` (`GoogleCloudAccount_Post`)
- **Alibaba** — `POST /v2/AlibabaCloudAccount` (`AlibabaCloudAccount_Post`)

Each takes provider-specific credentials. These are **not** polymorphic —
branch on cloud provider, do not try to reuse one body shape.

AWS also has a guided flow: `/v2/awsunifiedonboarding/*`
(`AwsUnifiedOnboarding_*`) returns the CloudFormation stack config to apply.

## 2. Verify permissions

`GET /v2/cloudaccounts/{id}/MissingPermissions`
(`CloudAccounts_GetMissingPermissionsAsync`) lists what CloudGuard still cannot
read. Per entity type: `GET /v2/cloudaccounts/{id}/MissingPermissions/EntityType`.

After fixing the cloud-side role, re-validate with
`PUT /v2/cloudaccounts/{id}/MissingPermissions/reset`
(`CloudAccounts_ResetMissingPermissionsAsync`).

For AWS specifically, check
`GET /v2/cloudaccounts/{id}/CloudAccountHasAssumeRolePermissionIssue`.

## 3. Place it in the hierarchy

`PUT /v2/cloudaccounts/{id}/organizationalUnit` sets the OU
(`null` = root). To move several at once:
`PUT /v2/cloudaccounts/organizationalUnit/move`
(`CloudAccounts_MoveCloudAccountsToOrganizationalUnit`).

## 4. Pull data now instead of waiting

`POST /v2/cloudaccounts/{id}/SyncNow` (`CloudAccounts_SyncNow`) queues an
immediate fetch. It is **asynchronous** — poll
`GET /v2/EntityFetchStatus` (`EntityFetchStatus_Get`) until the account's
entities report fetched. Do not treat the 2xx from SyncNow as completion.

## 5. Renaming and credentials

- `PUT /v2/cloudaccounts/name` (`CloudAccounts_UpdateCloudAccountName`)
- `PUT /v2/cloudaccounts/credentials` (`CloudAccounts_UpdateCloudAccountCredentials`)

## Deleting — read this first

`DELETE /v2/CloudAccounts/{id}` and
`DELETE /v2/cloudaccounts/{id}/DeleteForce` (`CloudAccounts_DeleteForce`)
remove the account **and every linked entity**. There is no restore operation
anywhere in this contract and no grace period is documented. Always require
explicit human confirmation.
