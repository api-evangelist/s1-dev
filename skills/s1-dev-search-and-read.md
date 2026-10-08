---
generated: '2026-10-07'
method: generated
name: search-and-read
description: Search the live web with Search1API, then read the full text of the pages that matter.
api: openapi/s1-dev-openapi.yml
operations:
- search
- crawl
source: Grounded in openapi/s1-dev-openapi.yml operationIds and https://s1.dev/docs (credits-and-limits, error-handling);
  conventions in conventions/s1-dev-conventions.yml.
---
# Search and read

1. `search` — `POST /search` with `{ "query": "...", "max_results": 10, "search_service": "google" }`. Optional `time_range`, `include_sites`, `exclude_sites`, `page` (bing/bingcn/baidu only). Zero results come back as `200` with an empty `results[]` and are still charged.
2. Choose the results to read. Either set `crawl_results: N` (N <= `max_results`) on the search itself (1 extra credit per page crawled) or call `crawl` — `POST /crawl` with `{ "url": "..." }` — for each chosen URL.
3. A `crawl` `404`/`410` is the target site confirming a dead link: it is a completed, charged answer — do not retry; drop the result.
4. Keep every result's source URL for citation.

## Rules that apply to every call
- Send `Authorization: Bearer <OAuth access token or Search1API API key>`; without one a paid endpoint answers `402` with a payment challenge (MPP/x402), and with an exhausted balance it answers `402` insufficient credits — check `GET /usage` (`usage`) first.
- Errors are JSON `{ ok: false, error, message, errors[] }`; `422` carries an `errors[]` array naming the field. Fix the request; do not retry `400/401/402/422` unchanged.
- Retry `429` (honour `Retry-After: 60`), `502`, `503`, `504` and transient `500` with backoff. Default limit is 200 requests per minute per account (`/ask` 30, `/screenshot` 10).
- No Idempotency-Key exists; search-shaped calls have no side effects, but `POST /deepcrawl` starts a new 20-credit task on every call.
- Credits: `/search` 1, `/news` 1, `/ask` 5, `/crawl` 1, `/screenshot` 2, `/sitemap` 1, `/trending` 1, `/extract` 10, `/deepcrawl` 20; `crawl_results` adds 1 credit per page crawled.
