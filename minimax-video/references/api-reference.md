# MiniMax H3 Video API — Complete Reference

> Base URL: `https://api.minimax.io`
> Auth: `Authorization: Bearer <API_KEY>` **only** — no `token:` header
> (get key from https://platform.minimax.io → Account → API Keys)
>
> `hub.minimax.io` is a marketing website, not an API host. Requests there return an HTML 404.

> ### ⚠️ Corrections verified against the live API, 2026-08-04
> The generation endpoint lives under `/v2`, but **task status and file retrieval live under `/v1`.**
> Earlier revisions of this file were wrong on the points below; each was confirmed by direct probe.
>
> | Was documented | Actual |
> |---|---|
> | `GET /v2/video_generation/query` | **404.** Real: `GET /v1/query/video_generation` |
> | Query returns `video_url` | Returns **`file_id`**; exchange it at `GET /v1/files/retrieve` |
> | `resolution`, `duration` optional | **Both required** |
> | `duration` ∈ {5,10,15} | Any integer accepted; output runs slightly long |
> | 6 ratios | 7 — `adaptive` included |
> | `Pending` → `Processing` | **`Preparing`** → `Processing` → `Success`/`Failed` |
> | Flat error object | Nested under `.error`, plus `request_id` |
> | `GET /v2/video_generation/list` | **404 — does not exist** |
> | `DELETE .../cancel?task_id=` | Endpoint exists but rejects the id as query param *and* as body field; correct invocation unknown |

---

## Endpoints

### POST `/v2/video_generation` — Create Generation Task

Submits a video generation task. Returns a `task_id` for polling.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | ✅ | `"MiniMax-H3"`. Any other value → `400 this model is not supported by /v2/video_generation` |
| `content` | object[] | ✅ | Array of multimodal inputs (see below) |
| `resolution` | string | ✅ | `"2K"` or `"768P"`. Omitting it → `400 cause=missing required parameter` |
| `duration` | int | ✅ | Seconds. `5`/`10`/`15` are the documented values but **any integer is accepted** (`7` → 7.29 s output). Omitting it → `400` |
| `ratio` | string | ✅ | `"adaptive"`, `"16:9"`, `"4:3"`, `"1:1"`, `"3:4"`, `"9:16"`, `"21:9"` |

**There is no `seed` parameter.** Generations are not reproducible.

**Measured output (2K @ 9:16):** 1440 × 2560, h264, **24 fps**, AAC stereo **32 kHz**.
Audio is always generated and cannot be disabled — strip with `ffmpeg -an` if supplying your own.

**Content Array Elements:**

| `type` | Required | Description |
|--------|----------|-------------|
| `text` | ✅ (at least one) | The prompt. `{"type":"text","text":"..."}` |
| `image_url` | No | `{"type":"image_url","image_url":{"url":"https://..."},"role":"first_frame"\|"last_frame"\|"reference_image"}` |
| `video_url` | No | `{"type":"video_url","video_url":{"url":"https://..."},"role":"reference_video"}` |
| `audio_url` | No | `{"type":"audio_url","audio_url":{"url":"https://..."},"role":"reference_audio"}` |

**Generation Scenarios:**

| Scenario | Content Combination |
|----------|-------------------|
| Text-to-Video | Single `text` element |
| I2V — First Frame | `text` + 1 `image_url` (role=`first_frame` or omitted) |
| I2V — Last Frame | `text` + 1 `image_url` (role=`last_frame`) |
| I2V — First & Last | `text` + 2 `image_url` (roles=`first_frame` + `last_frame`) |
| Reference-to-Video | `text` + any combination of `reference_image` + `reference_video` + `reference_audio` (at least one image or video required) |

**Reference-to-Video constraints:**
- Audio alone is NOT allowed — at least one reference image or video required
- `first_frame`/`last_frame` and `reference_*` roles are **mutually exclusive** (cannot mix paradigms)

**Media Limits:**

| Type | Format | Max Size | Max Count | Other Limits |
|------|--------|----------|-----------|--------------|
| Image | JPG, JPEG, PNG, WEBP, HEIC, HEIF | 30 MB | 9 ref + 1 first + 1 last | 256–5760 px, AR 0.4–2.5 |
| Video | MP4, MOV, AVI, MKV, WEBM | 1024 MB | 3 | ≤ 120s, 240p–4K |
| Audio | MP3, WAV, FLAC, AAC, OGG, M4A | 30 MB | 3 | ≤ 300s |
| **Total body** | — | **64 MB** | — | Use URLs, avoid Base64 |

**Example Requests:**

```bash
# Text-to-Video (minimal)
curl -X POST "https://api.minimax.io/v2/video_generation" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MiniMax-H3",
    "content": [{"type":"text","text":"Cinematic drone shot over a misty forest at dawn"}],
    "resolution": "2K",
    "duration": 10,
    "ratio": "21:9"
  }'

# Image-to-Video (first frame)
curl -X POST "https://api.minimax.io/v2/video_generation" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MiniMax-H3",
    "content": [
      {"type":"text","text":"The character in the image looks up, wind blowing through hair, camera slowly pushes in"},
      {"type":"image_url","image_url":{"url":"https://example.com/portrait.jpg"},"role":"first_frame"}
    ],
    "resolution": "2K",
    "duration": 5,
    "ratio": "16:9"
  }'

# Reference-to-Video (multimodal)
curl -X POST "https://api.minimax.io/v2/video_generation" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MiniMax-H3",
    "content": [
      {"type":"text","text":"Reference the Hitchcock camera movement from Video 1, have the character in Image 2 sing, with vocals matching Audio 3"},
      {"type":"video_url","video_url":{"url":"https://example.com/hitchcock.mp4"},"role":"reference_video"},
      {"type":"image_url","image_url":{"url":"https://example.com/character.jpg"},"role":"reference_image"},
      {"type":"audio_url","audio_url":{"url":"https://example.com/vocals.mp3"},"role":"reference_audio"}
    ],
    "resolution": "768P",
    "duration": 10,
    "ratio": "16:9"
  }'
```

**Success Response (200):**
```json
{ "task_id": "424010985738629" }
```

**Error Response — nested, not flat:**
```json
{
  "type": "error",
  "error": {
    "type": "bad_request_error",
    "message": "invalid params, binding: expr_path=duration, cause=missing required parameter (2013)",
    "http_code": "400"
  },
  "request_id": "06c112189a49e6a7fd6262fb8b8f2c91"
}
```
`.error.message` names the offending field. Include `request_id` in support tickets.

---

### GET `/v1/query/video_generation` — Query Task Status

> ⚠️ Note the path shape: **`/v1/query/video_generation`**, not `/v2/video_generation/query`.
> The `/v2` form returns a plain-text `404 page not found`. Since that body is not JSON,
> `jq -r '.status'` returns `null` forever and a poll loop hangs silently until timeout.
> Always guard the loop with a JSON-validity check.

```
GET /v1/query/video_generation?task_id={task_id}
Authorization: Bearer <API_KEY>
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `task_id` | string | Echo of the requested id |
| `status` | string | `"Preparing"` → `"Processing"` → `"Success"` or `"Failed"` |
| `file_id` | string | Empty until `Success`. **Not a URL** — exchange at `/v1/files/retrieve` |
| `video_width` | int | `0` until `Success` |
| `video_height` | int | `0` until `Success` |
| `base_resp.status_code` | int | `0` = success, `2013` = invalid params |
| `base_resp.status_msg` | string | Human-readable status |

**Example (Success):**
```json
{
  "task_id": "427190077210913",
  "status": "Success",
  "file_id": "427191192965277",
  "video_width": 1440,
  "video_height": 2560,
  "base_resp": { "status_code": 0, "status_msg": "success" }
}
```

An unknown `task_id` returns HTTP 200 with `base_resp.status_code: 2013` — check the code, not
just the HTTP status.

---

### GET `/v1/files/retrieve` — Resolve file_id to a Download URL

Mandatory second hop. The query endpoint never returns a URL.

```
GET /v1/files/retrieve?file_id={file_id}
Authorization: Bearer <API_KEY>
```

**Example:**
```json
{
  "file": {
    "file_id": "427191192965277",
    "bytes": 0,
    "created_at": 1785847844,
    "filename": "output.mp4",
    "purpose": "video_generation",
    "download_url": "https://video-product.cdn.minimax.io/inference_output/rollout/..."
  },
  "base_resp": { "status_code": 0, "status_msg": "success" }
}
```

The download field is **`download_url`**. `bytes` reads `0` even for valid files — do not use it
as a sanity check.

---

### ~~GET `/v2/video_generation/list`~~ — DOES NOT EXIST

Returns `404 page not found`. There is no documented way to enumerate recent tasks; track your
own `task_id`s.

---

### DELETE `/v2/video_generation/cancel` — Cancel Task (invocation unresolved)

The endpoint exists — it returns a structured `bad_request_error` rather than a 404 — but it
rejected the task id both as a query parameter and as a `{"task_id": "..."}` JSON body, in each
case with `invalid params, invalid task_id (2013)`. The correct parameter name or location is
unknown. **Assume a submitted task cannot be cancelled** and budget accordingly.

---

## Error Codes

| Code | Meaning | Action |
|------|---------|--------|
| `400` | Invalid parameters | Check content array structure, role assignments, mutual exclusivity, prompt length (≤2000 chars) |
| `401` | Unauthorized | Verify API key is valid and active |
| `402` | Payment required | Account balance insufficient — top up at platform.minimax.io |
| `422` | Unprocessable entity | Media file issue (format, size, corrupted) or content policy rejection |
| `429` | Rate limited | Back off and retry. Check rate limits at platform.minimax.io |
| `500` | Internal server error | Retry after 30s. If persistent, contact support |

---

## Rate Limits

Check current limits at https://platform.minimax.io/docs/guides/rate-limits. Typical defaults:
- Generation submissions: 5–10 requests per minute
- Concurrent processing: 3–5 tasks
- Query endpoint: 60 requests per minute

---

## File Upload Endpoint

```
POST https://api.minimax.io/v1/files/upload
Authorization: Bearer <API_KEY>
Content-Type: multipart/form-data

purpose: video_generation
file: @/path/to/file.mp4
```

**Response:**
```json
{
  "file": {
    "file_id": "file-abc123",
    "url": "https://minimax-cos.example.com/uploads/file-abc123.mp4",
    "purpose": "video_generation",
    "bytes": 5242880
  }
}
```

Use the returned `file_id` or `url` in subsequent generation requests.

---

## Pricing (as of July 2026)

| Resolution | Per-Second Price | vs Mainstream Models |
|-----------|-----------------|---------------------|
| 2K | Base price | **< 1/3** of mainstream model cost |
| 768P | Lower tier | **< 1/2** of mainstream 720p cost |

Check https://platform.minimax.io/docs/pricing/overview for current exact pricing.
