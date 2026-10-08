---
generated: '2026-10-07'
method: generated
name: agentic-search
description: Ask Search1API a natural-language question and let the Jev decision model route it across engines.
api: openapi/s1-dev-openapi.yml
operations:
- ask
source: Grounded in openapi/s1-dev-openapi.yml operationIds and https://s1.dev/docs (credits-and-limits, error-handling);
  conventions in conventions/s1-dev-conventions.yml.
---
# Agentic search

1. `ask` — `POST /ask` with `{ "query": "<natural-language request>" }` only. The decision model (TypeSafe Jev) detects intent, chooses up to five engines and a time window, and reranks; a response holds at most 10 results. `sources`, `time_range` and `max_results` are accepted but ignored since 2026-10-07.
2. Costs 5 credits and may take up to 35 seconds — set the client timeout above that. Separate limit: 30 requests per minute per account.
3. Fall back to `search` when you need engine control, pagination or more than 10 results.

## Rules that apply to every call
- Send `Authorization: Bearer <OAuth access token or Search1API API key>`; without one a paid endpoint answers `402` with a payment challenge (MPP/x402), and with an exhausted balance it answers `402` insufficient credits — check `GET /usage` (`usage`) first.
- Errors are JSON `{ ok: false, error, message, errors[] }`; `422` carries an `errors[]` array naming the field. Fix the request; do not retry `400/401/402/422` unchanged.
- Retry `429` (honour `Retry-After: 60`), `502`, `503`, `504` and transient `500` with backoff. Default limit is 200 requests per minute per account (`/ask` 30, `/screenshot` 10).
- No Idempotency-Key exists; search-shaped calls have no side effects, but `POST /deepcrawl` starts a new 20-credit task on every call.
- Credits: `/search` 1, `/news` 1, `/ask` 5, `/crawl` 1, `/screenshot` 2, `/sitemap` 1, `/trending` 1, `/extract` 10, `/deepcrawl` 20; `crawl_results` adds 1 credit per page crawled.
