# MiniMax Music API — Complete Cloud Reference

> Host: `https://api.minimax.io`
> Auth: `Authorization: Bearer <API_KEY>` (JWT-format platform key)
> `Content-Type: application/json`
>
> Verified 2026-08-04 against the live API and the official OpenAPI source.

---

## Endpoint map

| Purpose | Method & path |
|---|---|
| Generate music | `POST /v1/music_generation` |
| Generate / edit lyrics | `POST /v1/lyrics_generation` |
| Preprocess audio for cover | `POST /v1/music_cover_preprocess` |

There is **no polling endpoint**. Music generation is a single blocking call.

---

## POST `/v1/music_generation`

Only `model` is required at the schema level. Everything else is conditionally required by mode —
see the matrix below.

### Parameters

| Field | Type | Notes |
|---|---|---|
| `model` | string | ✅ Required. Enum below |
| `prompt` | string | Max 2000. Style, mood, scenario as comma-separated descriptors |
| `lyrics` | string | `minLength: 1`, `maxLength: 3500`. `\n` line breaks + `[section tags]` |
| `is_instrumental` | boolean | Default `false`. Text-to-music models only |
| `lyrics_optimizer` | boolean | Default `false`. `true` + empty `lyrics` → auto-writes lyrics from `prompt`. Text-to-music models only |
| `stream` | boolean | Default `false`. Only `hex` output is supported when streaming. SSE framing is not documented |
| `output_format` | string | `url` or `hex`. **Default `hex`** |
| `audio_setting` | object | See below. **No defaults documented — set all three** |
| `audio_url` | string | Cover only. Mutually exclusive with `audio_base64` and `cover_feature_id` |
| `audio_base64` | string | Cover only. Same exclusivity |
| `cover_feature_id` | string | Cover only, two-step flow. Same exclusivity |

### `model` enum

| Model | Availability | RPM |
|---|---|---|
| `music-3.0` | Token Plan / paid only | 120 |
| `music-2.6` | Token Plan / paid only | 120 |
| `music-cover` | Token Plan / paid only | 120 |
| `music-3.0-free` | All users via API key | 3 |
| `music-2.6-free` | All users via API key | 3 |
| `music-cover-free` | All users via API key | 3 |

### Requirement matrix

| Mode | `prompt` | `lyrics` |
|---|---|---|
| Text-to-music, `is_instrumental: true` | ✅ **Required**, 1–2000 | Not required — **omit the key** |
| Text-to-music, with vocals | Optional, 0–2000 | ✅ Required, 1–3500 |
| Text-to-music, `lyrics_optimizer: true` | ✅ Required | Leave empty/omitted |
| `music-cover` | ✅ Required, **10–300** (describes the target style) | Optional, 10–1000 (ASR-extracted if omitted) |
| `music-cover` with `cover_feature_id` | ✅ Required, 10–300 | ✅ Required, 10–1000 |

> The docs say only that `lyrics` is "not required" for instrumental — they never say it is
> forbidden, ignored, or an error. But the schema sets `minLength: 1`, so `"lyrics": ""` may be
> rejected by a strict validator. **Omit the key entirely.**

### `audio_setting`

| Field | Options |
|---|---|
| `sample_rate` | `16000`, `24000`, `32000`, `44100` |
| `bitrate` | `32000`, `64000`, `128000`, `256000` |
| `format` | `mp3`, `wav`, `pcm` |

The `44100 / 256000 / mp3` triple seen throughout the docs comes from examples, not defaults.

### Lyrics section tags

`[Intro]` `[Verse]` `[Pre Chorus]` `[Chorus]` `[Post Chorus]` `[Bridge]` `[Interlude]`
`[Transition]` `[Break]` `[Hook]` `[Build Up]` `[Inst]` `[Solo]` `[Outro]`

⚠️ **The lyrics-generation endpoint emits a different set.** Only 10 of 14 overlap:

| `music_generation` accepts | `lyrics_generation` may emit |
|---|---|
| `[Pre Chorus]` (space) | `[Pre-Chorus]` (hyphen) |
| `[Build Up]` (space) | `[Build-up]` (hyphen) |
| `[Post Chorus]`, `[Transition]`, `[Inst]` | — |
| — | `[Drop]`, `[Instrumental]`, `[Breakdown]` |

The lyrics endpoint states its output "can be directly used in the lyrics parameter", yet it can
emit tags the music endpoint does not list. Behaviour on an unlisted tag is **not documented**.
Normalise before passing between them. Official examples use lowercase tags, so case appears not
to matter.

### No structural control

There is **no** `duration`, `length`, `bpm`, `tempo`, `key`, or `structure` parameter — the
complete property list is `model`, `prompt`, `lyrics`, `stream`, `output_format`, `audio_setting`,
`lyrics_optimizer`, `is_instrumental`, `audio_url`, `audio_base64`, `cover_feature_id`.

Naming a BPM inside the `prompt` string is an accepted convention and influences the result, but
it is a hint, not a control. Section tags are not documented to affect instrumental arrangement.

### Reference audio limits (cover)

- Duration **6 seconds – 6 minutes**
- Size **≤ 50 MB**
- Formats: mp3, wav, flac and other common audio formats

⚠️ **The Files API cannot host it.** `/v1/files/upload` accepts only `voice_clone`,
`prompt_audio`, `t2a_async_input`, `video_understanding` and `video_generation_input` as
`purpose` values — none music-related — and the music endpoints accept no `file_id`. Use a
publicly reachable `audio_url` or inline `audio_base64` (≈33% request inflation).

### Response — ✅ verified live

```json
{
  "data": {
    "audio": "<hex string, OR a signed download URL when output_format=url>",
    "status": 2
  },
  "trace_id": "06c11f462840300d88d20b5d0bc905d5",
  "extra_info": {
    "music_duration": 173792,
    "music_sample_rate": 44100,
    "music_channel": 2,
    "bitrate": 256000,
    "music_size": 5158939
  },
  "analysis_info": null,
  "base_resp": { "status_code": 0, "status_msg": "success" }
}
```

- **`output_format: "url"` puts a signed URL in `data.audio`.** The declared schema documents
  `data.audio` only for `hex` and defines no field for the URL case — the observed behaviour is
  that the same field carries both. Always set `output_format` explicitly.
- **The URL is signed and expires in 24 hours.** Truncating it strips the signature; the download
  then returns an `AccessDenied` XML body that will be silently saved as a fake audio file.
  Always verify with `ffprobe`.
- `data.status`: `1` = in progress, `2` = complete.
- `extra_info.music_duration` is **milliseconds**.
- `trace_id`, `extra_info` and `analysis_info` are **top-level siblings of `data`**, not nested
  inside it. They appear in the response but are absent from the declared schema.
- `analysis_info` is always `null` in observed responses and is undocumented.
- Always check `base_resp.status_code == 0`.

### ⚠️ Measured output duration — the "~25 second" claim is false

Two independent instrumental generations on `music-3.0-free`:

| Run | `music_duration` | Verified file |
|---|---|---|
| 1 | 160 992 ms | ≈ 161 s |
| 2 | 173 792 ms | **173.74 s**, 5.3 MB mp3, 44.1 kHz stereo (ffprobe) |

The `25364` figure that appears throughout older documentation is an **example value**, not a
limit or a typical length. Expect roughly **2½–3 minutes** of audio per generation. For shorter
deliverables, generate then trim — you will have ample material.

### Timing

**~170–190 seconds measured** per generation, as a single blocking HTTP call. Set a client
timeout of at least 600 s and run it in the background — a 120 s shell timeout kills it
mid-flight.

### Error codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `1000` / `1001` | Unknown error / timeout |
| `1002` | Rate limit — retry later |
| `1004` | Authentication failed |
| `1008` | Insufficient balance |
| `1013` | Internal service error |
| `1024` | Internal error |
| `1026` | Input flagged as sensitive |
| `1027` | Output content error |
| `1039` | Token limit |
| `1041` | Connection limit |
| `2013` | Invalid parameters |
| `2049` | Invalid API key |
| `2056` | Usage limit exceeded — wait for the next 5-hour window |

---

## POST `/v1/lyrics_generation`

Required: `mode`.

| Field | Type | Required | Notes |
|---|---|---|---|
| `mode` | string | ✅ | `write_full_song` or `edit` |
| `prompt` | string | no | Max 2000. Theme, style, or editing direction. Empty → a random song |
| `lyrics` | string | no | Max 3500. `edit` mode only |
| `title` | string | no | Preserved unchanged in the output |

Response: `song_title`, `style_tags` (comma-separated descriptors, designed to drop straight into
a music `prompt`), `lyrics`, `base_resp`. No `trace_id`.

**Cost: $0.01 per song.** Not free.

---

## POST `/v1/music_cover_preprocess`

Required: `model` — enum is **`music-cover` only** (`music-cover-free` is *not* accepted here,
despite being valid on `music_generation`). Plus exactly one of `audio_url` / `audio_base64`.

Response:

| Field | Notes |
|---|---|
| `cover_feature_id` | Valid **24 hours**. Identical audio returns the same id (MD5 dedupe) |
| `formatted_lyrics` | ASR-extracted lyrics with section tags |
| `structure_result` | JSON string: segment labels with **start/end timestamps in seconds** |
| `audio_duration` | Reference length in seconds |
| `trace_id`, `base_resp` | — |

```json
{"num_segments":4,"segments":[
  {"start":0,"end":15.5,"label":"intro"},
  {"start":15.5,"end":45.2,"label":"verse"},
  {"start":45.2,"end":75.0,"label":"chorus"},
  {"start":75.0,"end":90.0,"label":"outro"}]}
```

Segment labels: `intro`, `verse`, `chorus`, `bridge`, `outro`, `inst`, `silence`.

**This step is free.** Note it *analyses* reference audio — it cannot shape generated output.

---

## Rate limits

| API | Models | RPM | Concurrent |
|---|---|---|---|
| Music generation | Music-3.0 / 2.6 / Cover / 2.0 | 120 | 20 |
| Free-tier variants | `*-free` | 3 | not stated |

Free tier at 3 RPM means one call per 20 seconds. To raise limits: `api@minimax.io`.

---

## Pricing

| Model | Price |
|---|---|
| `music-3.0-free` / `music-2.6-free` | Free |
| `music-3.0` / `music-2.6` | **$0.15 per track, up to 5 minutes** |
| Lyrics generation | $0.01 per song |
| Music cover preprocess | Free |
| `music-cover` / `music-cover-free` | Not stated |

Billing is **flat per generation**, bucketed by length — not per second and not per token. A
30-second track costs the same as a 5-minute one.

---

## Known documentation inconsistencies

1. Two conflicting section-tag lists between `music_generation` and `lyrics_generation`.
2. `music-cover` has no pricing row despite appearing in the model enum and rate-limit table.
3. The rate-limits page lists `Music-2.0`, absent from the `music_generation` enum.
4. `music_cover_preprocess` accepts `music-cover` only, not `music-cover-free`.
5. Doc examples use lowercase tags while the spec documents capitalized ones.
6. The declared response schema omits `trace_id`, `extra_info` and `analysis_info`, all of which
   are returned in practice — including `music_duration`, the most useful field.
7. `output_format: "url"` has no documented response field; the URL arrives in `data.audio`.
8. The `music_duration: 25364` example is widely mistaken for a duration limit. It is not —
   measured output is 161–174 seconds.
