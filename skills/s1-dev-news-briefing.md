---
generated: '2026-10-07'
method: generated
name: news-briefing
description: Build a dated news briefing from Search1API news search with optional full-article text.
api: openapi/s1-dev-openapi.yml
operations:
- news
- crawl
source: Grounded in openapi/s1-dev-openapi.yml operationIds and https://s1.dev/docs (credits-and-limits, error-handling);
  conventions in conventions/s1-dev-conventions.yml.
---
# News briefing

1. `news` — `POST /news` with `{ "query": "...", "time_range": "day", "max_results": 10 }`; `search_service: "hackernews"` links to the discussion thread and adds `story_url`, `points`, `num_comments`. Results carry `published_date` when the source exposes one.
2. For articles that need full text, set `crawl_results` on the news call or call `crawl` (`POST /crawl`) per URL.
3. Order by `published_date`, cite the source URL on every item.

## Rules that apply to every call
- Send `Authorization: Bearer <OAuth access token or Search1API API key>`; without one a paid endpoint answers `402` with a payment challenge (MPP/x402), and with an exhausted balance it answers `402` insufficient credits — check `GET /usage` (`usage`) first.
- Errors are JSON `{ ok: false, error, message, errors[] }`; `422` carries an `errors[]` array naming the field. Fix the request; do not retry `400/401/402/422` unchanged.
- Retry `429` (honour `Retry-After: 60`), `502`, `503`, `504` and transient `500` with backoff. Default limit is 200 requests per minute per account (`/ask` 30, `/screenshot` 10).
- No Idempotency-Key exists; search-shaped calls have no side effects, but `POST /deepcrawl` starts a new 20-credit task on every call.
- Credits: `/search` 1, `/news` 1, `/ask` 5, `/crawl` 1, `/screenshot` 2, `/sitemap` 1, `/trending` 1, `/extract` 10, `/deepcrawl` 20; `crawl_results` adds 1 credit per page crawled.
