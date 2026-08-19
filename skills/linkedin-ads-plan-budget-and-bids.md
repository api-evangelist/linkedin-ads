---
name: Plan LinkedIn ad budget and bids
description: Pull LinkedIn's suggested bid and budget bounds for a targeting set so a campaign is created with values the platform will accept.
api: openapi/linkedin-ads-adbudgetpricing-api-openapi.yml
operations:
  - getAdBudgetPricing
  - getAudienceCounts
  - createAdCampaign
generated: '2026-08-13'
method: generated
source: openapi/linkedin-ads-adbudgetpricing-api-openapi.yml + learn.microsoft.com ad-budget-pricing docs
---

# Plan LinkedIn ad budget and bids

LinkedIn ads are auction-priced; there is no rate card. `getAdBudgetPricing` is
how you find out, for a given account, campaign type and targeting set, what the
platform considers a viable bid and the daily/total budget floor and ceiling.
Call it before `createAdCampaign` and you avoid a whole class of validation
rejections.

## Before you call anything

- `https://api.linkedin.com/rest`, `Linkedin-Version: 202607`,
  `X-Restli-Protocol-Version: 2.0.0`, bearer token with `r_ads` or `rw_ads`.
- Every money value is a `MoneyAmount` (`currencyCode` + `amount`), and the
  currency must match the ad account's currency exactly.

## Steps

1. **Resolve and size the audience first** (`getAudienceCounts`) — pricing is a
   function of who you are targeting.
2. **Call `getAdBudgetPricing`** with the account, campaign type / objective and
   the same `targetingCriteria`. Read back the suggested bid and the daily and
   total budget bounds.
3. **Create the campaign** with values inside those bounds. The validations that
   fire otherwise:
   - `FIELD_VALUE_TOO_LOW` / `FIELD_VALUE_TOO_HIGH` — outside the returned bounds
   - `CONDITIONAL_VALUE_TOO_LOW` — `totalBudget` below `dailyBudget`
   - `INSUFFICIENT_CAMPAIGN_GROUP_BUDGET` — the parent group does not have the
     headroom; the error carries the minimum you must raise it to
   - `CURRENCY_MISMATCH` — budget currency is not the account currency
   - `CURRENCY_NOT_SUPPORTED` — the currency itself is not supported
4. **Respect immutability.** Campaign floor price cannot be changed once set for
   Sponsored Updates, InMail and Dynamic campaigns
   (`IMMUTABLE_CAMPAIGN_FLOOR_PRICE`), and budget-shaped fields are frozen once a
   campaign leaves draft.

## Reporting the outcome

Spend and delivery come from the Ad Analytics reporting surface, not from these
endpoints. Reporting is a separate approved product with its own scopes
(`r_ads_reporting`).
