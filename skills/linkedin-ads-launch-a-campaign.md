---
name: Launch a LinkedIn ad campaign
description: Create the ad account, campaign group, campaign and creative needed to put a sponsored ad in market on LinkedIn.
api: openapi/linkedin-ads-adaccounts-api-openapi.yml
operations:
  - createAdAccount
  - getAdAccount
  - createAdCampaignGroup
  - createAdCampaign
  - createCreative
generated: '2026-08-13'
method: generated
source: openapi/linkedin-ads-adaccounts-api-openapi.yml + learn.microsoft.com marketing docs
---

# Launch a LinkedIn ad campaign

LinkedIn advertising is a four-level hierarchy and you must build it top down:
**Ad Account -> Campaign Group -> Campaign -> Creative**. Nothing serves until a
creative exists under a campaign whose group and account are active.

## Before you call anything

- Base URL is `https://api.linkedin.com/rest`.
- Send **both** headers on every request:
  - `Linkedin-Version: 202607` (dated version, `YYYYMM`; there is no default — an
    unversioned call is rejected, and a sunset version fails with `426`)
  - `X-Restli-Protocol-Version: 2.0.0`
- Authenticate with a 3-legged OAuth bearer token holding `rw_ads`. Tokens live
  60 days; refresh before expiry rather than re-prompting the member.
- Know your access tier: on **Development** you may create exactly one *test* ad
  account through the API and edit up to five accounts; real accounts are created
  in Campaign Manager. On **Standard** both are unlimited.
- **There is no idempotency key.** If a POST times out, do not blind-retry —
  `GET` the collection and check whether the entity landed. A retried create is a
  duplicate campaign spending real money.

## Steps

1. **Have an account.** If you are testing, `createAdAccount` with `test: true`
   (immutable, one per app). Otherwise `getAdAccount` to confirm the account you
   administer and read its `currency` — every budget you send must match it or
   you get `CURRENCY_MISMATCH`.
2. **Create the campaign group** with `createAdCampaignGroup`, setting `account`
   to the account URN, a `runSchedule`, and a `totalBudget` if the group is
   budgeted. On success the new id comes back in the `x-restli-id` response
   header, not the body.
3. **Create the campaign** with `createAdCampaign`. Set `account`,
   `campaignGroup`, `objectiveType`, `type`, `costType`, `dailyBudget` and/or
   `totalBudget`, and `targetingCriteria`. Two constraints bite immediately:
   `totalBudget` cannot be lower than `dailyBudget` (`CONDITIONAL_VALUE_TOO_LOW`),
   and the combination of objective, format, type, optimization target and cost
   type must be a valid campaign configuration (`INVALID_CAMPAIGN_CONFIGURATION`).
   If the account has more than one campaign group you must name one
   (`MISSING_CAMPAIGN_GROUP_ID`).
4. **Create the creative** with `createCreative` against the campaign URN. A
   campaign cannot be created while it has creatives in a failed state
   (`CREATE_NOT_ALLOWED_WITH_FAILED_CREATIVES`).
5. **Move out of draft last.** Many campaign fields are immutable once the
   campaign leaves draft (`FIELD_IMMUTABLE_FOR_NON_DRAFT_CAMPAIGNS`), and objective
   type can never be changed after it is set. Get the shape right in draft.

## Errors you will actually hit

Read the `code` field, never the `message` — LinkedIn changes messages without
notice and says so. The full registry is in
`errors/linkedin-ads-error-codes.yml`. Common ones here:
`MISSING_FIELD`, `INVALID_VALUE_FOR_FIELD`, `CONDITIONAL_VALUE_TOO_LOW`,
`INSUFFICIENT_CAMPAIGN_GROUP_BUDGET`, `NO_PERMISSION_ON_ENTITY` (403),
`AUDIENCE_SIZE_TOO_SMALL`.

Note that `code`/`errorDetails` are only returned by `adAccounts`,
`adCampaignGroups` and `adCampaigns`; other resources still return the older
`message`/`serviceErrorCode`/`status` envelope only.

## Rate and retry behaviour

There are no rate-limit headers. A `429` means "resource level throttle limit for
calls to this resource is reached" and there is no `Retry-After` to obey — back
off on your own schedule and check the Developer Portal Analytics tab for the
endpoint's daily quota.
