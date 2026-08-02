# MiniMax H3 Video API — Complete Reference

> Base URL: `https://api.minimax.io/v2/video_generation`
> Auth: `Authorization: Bearer <API_KEY>` (get key from https://platform.minimax.io → Account → API Keys)

---

## Endpoints

### POST `/v2/video_generation` — Create Generation Task

Submits a video generation task. Returns a `task_id` for polling.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | ✅ | `"MiniMax-H3"` |
| `content` | object[] | ✅ | Array of multimodal inputs (see below) |
| `resolution` | string | No | `"2K"` (default) or `"768P"` |
| `duration` | int | No | `5`, `10`, or `15` (seconds). Default: `5` |
| `ratio` | string | No | `"16:9"` (default), `"9:16"`, `"1:1"`, `"4:3"`, `"3:4"`, `"21:9"` |

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

---

### GET `/v2/video_generation/query` — Query Task Status

```
GET /v2/video_generation/query?task_id={task_id}
Authorization: Bearer <API_KEY>
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `status` | string | `"Pending"` → `"Processing"` → `"Success"` or `"Failed"` |
| `video_url` | string | Download URL (only when `Success`) |
| `error` | object | Error details (only when `Failed`) |
| `created_at` | string | Task creation timestamp |
| `updated_at` | string | Last status update timestamp |

**Example:**
```json
{
  "status": "Success",
  "video_url": "https://minimax-output.cos.ap-southeast-1.myqcloud.com/videos/424010985738629.mp4",
  "created_at": "2026-07-31T10:30:00Z",
  "updated_at": "2026-07-31T10:32:15Z"
}
```

---

### GET `/v2/video_generation/list` — List Tasks

```
GET /v2/video_generation/list?page=1&page_size=20
Authorization: Bearer <API_KEY>
```

Returns paginated list of recent tasks with statuses.

---

### DELETE `/v2/video_generation/cancel` — Cancel Task

```
DELETE /v2/video_generation/cancel?task_id={task_id}
Authorization: Bearer <API_KEY>
```

Cancels a pending or processing task.

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
