<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Rednote (Xiaohongshu) API

[![SocQ Rednote (Xiaohongshu) API](https://img.shields.io/badge/API-SocQ%20Rednote%20%28Xiaohongshu%29%20API-F64C31)](https://socq.ai/apis/rednote?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/rednote/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/rednote-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/rednote-api/actions/workflows/check.yml)

8 endpoints for public Rednote (Xiaohongshu) data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Rednote (Xiaohongshu) APIs](https://socq.ai/apis/rednote?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api) · [Documentation](https://docs.socq.ai/api-manual/rednote/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Rednote Comment Replies API](#rednote-comment-replies-api) | `rednote/comment-replies` | Collect one page of public replies beneath a known Rednote comment, returning normalized reply text, author context, thread relationships, timestamps, and visible metrics when available. | 1 credit ($0.005) per call | 2026-08-18 |
| [Rednote Home Recommendations API](#rednote-home-recommendations-api) | `rednote/home-recommendations` | Collect one public Rednote home recommendation page without a seed query, then read normalized post, author, media, engagement, and collection-time fields. | 1 credit ($0.005) per call | 2026-08-17 |
| [Rednote Note Comments API](#rednote-note-comments-api) | `rednote/note-comments` | Collect one page of public comments from a Rednote note URL with normalized text, author context, timestamps, note references, and visible engagement fields. | 1 credit ($0.005) per call | 2026-08-17 |
| [Rednote Note Detail API](#rednote-note-detail-api) | `rednote/note-detail` | Retrieve one public Rednote note from a token-bearing xiaohongshu.com or xhslink.com URL with normalized text, author, media, timestamps, and visible engagement. | 1 credit ($0.005) per call | 2026-08-17 |
| [Rednote Search Notes API](#rednote-search-notes-api) | `rednote/search-notes` | Search public Rednote notes by text query with optional sort, media, and time filters, returning normalized content, authors, media, timestamps, and visible engagement. | 1 credit ($0.005) per call | 2026-08-17 |
| [Rednote Search Users API](#rednote-search-users-api) | `rednote/search-users` | Search public Rednote accounts with one text query and return normalized identity, biography, account flags, and visible audience or content metrics from one provider page. | 1 credit ($0.005) per call | 2026-08-17 |
| [Rednote User Notes API](#rednote-user-notes-api) | `rednote/user-notes` | Collect one page of public Rednote notes from a supported user URL or user ID, returning normalized text, authors, media, timestamps, and visible engagement. | 1 credit ($0.005) per call | 2026-08-17 |
| [Rednote User Profile API](#rednote-user-profile-api) | `rednote/user-profile` | Retrieve one public Rednote profile by supported account URL or user_id, returning normalized identity, profile imagery, account flags, visible counts, and collection time. | 1 credit ($0.005) per call | 2026-08-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/rednote-api.git
cd rednote-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs rednote/comment-replies

# Python 3.8+ (standard library only)
python3 examples/python/socq.py rednote/comment-replies payloads/comment-replies.json

# cURL + jq
bash examples/curl/socq.sh rednote/comment-replies '{"url":"https://www.xiaohongshu.com/explore/6a6174d6000000001d02013b?xsec_token=AB4sNisHvFy2q4C5ujG23AV858osk5rYC63eCgf8YTVOI%3D&xsec_source=pc_feed","comment_id":"6a6327f6000000002b026c1d"}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Rednote Comment Replies API

Collect one page of public replies beneath a known Rednote comment, returning normalized reply text, author context, thread relationships, timestamps, and visible metrics when available.

- **Request:** `POST https://api.socq.ai/v1/rednote/comment-replies`
- **publicId:** `rednote/comment-replies`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-08-18
- **Example body:** [`payloads/comment-replies.json`](payloads/comment-replies.json)

Fetch normalized comment replies data from Rednote.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `comment_id` | string | Yes | A public comment identifier. |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/rednote/comment-replies" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.xiaohongshu.com/explore/6a6174d6000000001d02013b?xsec_token=AB4sNisHvFy2q4C5ujG23AV858osk5rYC63eCgf8YTVOI%3D&xsec_source=pc_feed",
  "comment_id": "6a6327f6000000002b026c1d"
}'
```

Full docs: [https://docs.socq.ai/api-manual/rednote/comment-replies](https://docs.socq.ai/api-manual/rednote/comment-replies) · Landing page: [socq.ai/apis/rednote/comment-replies](https://socq.ai/apis/rednote/comment-replies?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)

### Rednote Home Recommendations API

Collect one public Rednote home recommendation page without a seed query, then read normalized post, author, media, engagement, and collection-time fields.

- **Request:** `POST https://api.socq.ai/v1/rednote/home-recommendations`
- **publicId:** `rednote/home-recommendations`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/home-recommendations.json`](payloads/home-recommendations.json)

Fetch normalized home recommendations data from Rednote.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `category` | string | No | - |
| `media_type` | string | No | (one of `all`, `image`, `video`) |
| `results_limit` | integer | No | (value 1 to 40; default `20`) |

```bash
curl -X POST "https://api.socq.ai/v1/rednote/home-recommendations" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "category": "homefeed_recommend",
  "media_type": "all",
  "results_limit": 3
}'
```

Full docs: [https://docs.socq.ai/api-manual/rednote/home-recommendations](https://docs.socq.ai/api-manual/rednote/home-recommendations) · Landing page: [socq.ai/apis/rednote/home-recommendations](https://socq.ai/apis/rednote/home-recommendations?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)

### Rednote Note Comments API

Collect one page of public comments from a Rednote note URL with normalized text, author context, timestamps, note references, and visible engagement fields.

- **Request:** `POST https://api.socq.ai/v1/rednote/note-comments`
- **publicId:** `rednote/note-comments`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/note-comments.json`](payloads/note-comments.json)

Fetch normalized comments data from Rednote.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/rednote/note-comments" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.xiaohongshu.com/explore/6a6174d6000000001d02013b?xsec_token=AB4sNisHvFy2q4C5ujG23AV858osk5rYC63eCgf8YTVOI%3D&xsec_source=pc_feed"
}'
```

Full docs: [https://docs.socq.ai/api-manual/rednote/note-comments](https://docs.socq.ai/api-manual/rednote/note-comments) · Landing page: [socq.ai/apis/rednote/note-comments](https://socq.ai/apis/rednote/note-comments?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)

### Rednote Note Detail API

Retrieve one public Rednote note from a token-bearing xiaohongshu.com or xhslink.com URL with normalized text, author, media, timestamps, and visible engagement.

- **Request:** `POST https://api.socq.ai/v1/rednote/note-detail`
- **publicId:** `rednote/note-detail`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/note-detail.json`](payloads/note-detail.json)

Fetch normalized notes data from Rednote.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | Yes | A public content or product URL. |

```bash
curl -X POST "https://api.socq.ai/v1/rednote/note-detail" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "url": "https://www.xiaohongshu.com/explore/6a2f423d000000001003cb36?xsec_token=ABtexB6bX5HgLD42SFrK1scGUmSO8YV5_isaLrDy9sLxQ%3D"
}'
```

Full docs: [https://docs.socq.ai/api-manual/rednote/note-detail](https://docs.socq.ai/api-manual/rednote/note-detail) · Landing page: [socq.ai/apis/rednote/note-detail](https://socq.ai/apis/rednote/note-detail?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)

### Rednote Search Notes API

Search public Rednote notes by text query with optional sort, media, and time filters, returning normalized content, authors, media, timestamps, and visible engagement.

- **Request:** `POST https://api.socq.ai/v1/rednote/search-notes`
- **publicId:** `rednote/search-notes`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/search-notes.json`](payloads/search-notes.json)

Fetch normalized search data from Rednote.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `media_type` | string | No | (one of `all`, `image`, `video`) |
| `published_within` | string | No | (one of `all`, `day`, `six_months`, `week`) |
| `query` | string | Yes | Search query. |
| `sort_by` | string | No | (one of `general`, `latest`, `most_liked`) |

```bash
curl -X POST "https://api.socq.ai/v1/rednote/search-notes" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "旅行攻略",
  "sort_by": "general",
  "media_type": "all"
}'
```

Full docs: [https://docs.socq.ai/api-manual/rednote/search-notes](https://docs.socq.ai/api-manual/rednote/search-notes) · Landing page: [socq.ai/apis/rednote/search-notes](https://socq.ai/apis/rednote/search-notes?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)

### Rednote Search Users API

Search public Rednote accounts with one text query and return normalized identity, biography, account flags, and visible audience or content metrics from one provider page.

- **Request:** `POST https://api.socq.ai/v1/rednote/search-users`
- **publicId:** `rednote/search-users`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/search-users.json`](payloads/search-users.json)

Fetch normalized search users data from Rednote.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `query` | string | Yes | Search query. |

```bash
curl -X POST "https://api.socq.ai/v1/rednote/search-users" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "摄影"
}'
```

Full docs: [https://docs.socq.ai/api-manual/rednote/search-users](https://docs.socq.ai/api-manual/rednote/search-users) · Landing page: [socq.ai/apis/rednote/search-users](https://socq.ai/apis/rednote/search-users?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)

### Rednote User Notes API

Collect one page of public Rednote notes from a supported user URL or user ID, returning normalized text, authors, media, timestamps, and visible engagement.

- **Request:** `POST https://api.socq.ai/v1/rednote/user-notes`
- **publicId:** `rednote/user-notes`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/user-notes.json`](payloads/user-notes.json)

Fetch normalized user notes data from Rednote.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | One of* | A public Rednote profile URL or xhslink.com/xhslink.cn share link. |
| `user_id` | string | One of* | A 24-character internal Rednote user ID, not a nickname or Rednote number. (pattern `^[0-9a-fA-F]{24}$`) |

*Required rule: send at least one of `url`, `user_id`; do not send `url` and `user_id` together.

```bash
curl -X POST "https://api.socq.ai/v1/rednote/user-notes" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "user_id": "65d8c7d40000000005033a6b"
}'
```

Full docs: [https://docs.socq.ai/api-manual/rednote/user-notes](https://docs.socq.ai/api-manual/rednote/user-notes) · Landing page: [socq.ai/apis/rednote/user-notes](https://socq.ai/apis/rednote/user-notes?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)

### Rednote User Profile API

Retrieve one public Rednote profile by supported account URL or user_id, returning normalized identity, profile imagery, account flags, visible counts, and collection time.

- **Request:** `POST https://api.socq.ai/v1/rednote/user-profile`
- **publicId:** `rednote/user-profile`
- **Price:** 1 credit ($0.005) per call
- **Launched:** 2026-08-17
- **Example body:** [`payloads/user-profile.json`](payloads/user-profile.json)

Fetch normalized profiles data from Rednote.

Successful tasks are billed per task.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `url` | string | One of* | A public Rednote profile URL or xhslink.com/xhslink.cn share link. |
| `user_id` | string | One of* | A 24-character internal Rednote user ID, not a nickname or Rednote number. (pattern `^[0-9a-fA-F]{24}$`) |

*Required rule: send at least one of `url`, `user_id`; do not send `url` and `user_id` together.

```bash
curl -X POST "https://api.socq.ai/v1/rednote/user-profile" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "user_id": "65d8c7d40000000005033a6b"
}'
```

Full docs: [https://docs.socq.ai/api-manual/rednote/user-profile](https://docs.socq.ai/api-manual/rednote/user-profile) · Landing page: [socq.ai/apis/rednote/user-profile](https://socq.ai/apis/rednote/user-profile?utm_source=github&utm_medium=readme&utm_campaign=socq-rednote-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Rednote (Xiaohongshu) data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
