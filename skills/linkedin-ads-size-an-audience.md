---
name: Build and size a LinkedIn ad audience
description: Discover targeting facets, resolve targeting entities, and get the forecast audience size before committing a campaign's targeting criteria.
api: openapi/linkedin-ads-adtargetingfacets-api-openapi.yml
operations:
  - listAdTargetingFacets
  - getAdTargetingEntities
  - getAudienceCounts
generated: '2026-08-13'
method: generated
source: openapi/ + learn.microsoft.com ads-targeting docs
---

# Build and size a LinkedIn ad audience

Targeting on LinkedIn is expressed as `targetingCriteria` — include/exclude sets
of **facet URNs**. You cannot guess those URNs; you resolve them, then you check
the audience is large enough to serve before you attach it to a campaign.

## Before you call anything

- `https://api.linkedin.com/rest`, `Linkedin-Version: 202607`,
  `X-Restli-Protocol-Version: 2.0.0`, bearer token with `r_ads` or `rw_ads`.
- These are read operations; they are still counted against your daily
  application and per-member quotas.

## Steps

1. **List the facets** with `listAdTargetingFacets`. This tells you which
   dimensions exist (locations, industries, job functions, seniorities, skills,
   company size, audiences) and which entity types each accepts.
2. **Resolve entities inside a facet** with `getAdTargetingEntities` — this turns
   a human phrase ("Software Development") into the `urn:li:...` value the
   campaign expects. Never hand-write a facet URN.
3. **Size it** with `getAudienceCounts`, passing the assembled
   `targetingCriteria`. Do this *before* creating the campaign: a campaign whose
   audience is under LinkedIn's minimum is rejected with `AUDIENCE_SIZE_TOO_SMALL`,
   and the error tells you the minimum in `minValue`.
4. **Attach the criteria** to the campaign (see the launch skill). Rules that
   fail late if you ignore them here:
   - the same facet value cannot be in both include and exclude
     (`FIELD_CANNOT_BE_BOTH_IN_INCLUDED_AND_EXCLUDED_TARGETING`)
   - some facet combinations are simply incompatible
     (`INCOMPATIBLE_TARGETING_COMBINATION`)
   - an audience segment must belong to the facet you filed it under
     (`INVALID_SEGMENT_ID_AND_FACET_COMBINATION_PROVIDED`)
   - the legacy `/targeting` field is dead — use `/targetingCriteria`
     (`TARGETING_FIELD_DEPRECATED`)

## Paging

Facet and entity collections page with `start`/`count` (default `count` 10) and
return a `paging` object. You are at the end of the dataset when fewer elements
come back than you asked for. Advertising collections also support cursor-based
pagination; prefer the cursor form for large targeting pulls.

## Failure modes worth handling

`FAILED_TO_RETRIEVE_AUDIENCE_SIZE` and `FAILED_TO_RETRIEVE_SEGMENTS` are **500s**,
not 400s — they are LinkedIn-side, so retry with backoff rather than editing your
request.
