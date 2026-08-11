# MiniMax Speech API — Complete Cloud Reference

> Host: `https://api.minimax.io` (low-TTFA alternative: `https://api-uw.minimax.io`)
> Auth: `Authorization: Bearer <API_KEY>` — a JWT-format platform key from
> [Account Management → API Keys](https://platform.minimax.io)
> `Content-Type: application/json`
>
> **No `GroupId` query parameter is required.** The legacy `api.minimax.chat` T2A v2 endpoint did
> require one; the `.io` platform does not.
>
> Verified 2026-08-04 against the live API and the official OpenAPI source.

---

## Endpoint map

| Purpose | Method & path |
|---|---|
| Text-to-speech (sync) | `POST /v1/t2a_v2` |
| Text-to-speech (async, long form) | `POST /v1/t2a_async_v2` |
| Async status | `GET /v1/query/t2a_async_query_v2?task_id={id}` |
| Download async result | `GET /v1/files/retrieve_content?file_id={id}` |
| Upload audio (cloning / async input) | `POST /v1/files/upload` |
| Voice clone | `POST /v1/voice_clone` |
| Voice design | `POST /v1/voice_design` |
| List voices | `POST /v1/get_voice` |
| Delete a voice | `POST /v1/delete_voice` |

---

## POST `/v1/t2a_v2` — synchronous TTS

### Top-level parameters

| Field | Type | Required | Default | Values |
|---|---|---|---|---|
| `model` | string | ✅ | — | `speech-2.8-hd`, `speech-2.8-turbo`, `speech-2.6-hd`, `speech-2.6-turbo`, `speech-02-hd`, `speech-02-turbo`, `speech-01-hd`, `speech-01-turbo` |
| `text` | string | ✅ | — | **< 10 000 characters** |
| `voice_setting` | object | ✅ | — | `voice_id` required within it |
| `audio_setting` | object | no | — | see below |
| `stream` | boolean | no | `false` | Streaming forces `output_format: hex` |
| `stream_options` | object | no | — | `{exclude_aggregated_audio: bool}` |
| `output_format` | string | no | **`hex`** | `hex` or `url`. **Non-streaming only** |
| `subtitle_enable` | boolean | no | `false` | Emits `data.subtitle_file` |
| `subtitle_type` | string | no | `sentence` | `sentence`, `word`, `word_streaming` |
| `language_boost` | string | no | `null` | see language list below |
| `pronunciation_dict` | object | no | — | `{tone: ["original/replacement", ...]}` |
| `voice_modify` | object | no | — | see below |
| `timbre_weights` | array | no | — | legacy; blend up to 4 voices |

> Text over 3000 characters: streaming output is recommended.
> If invalid characters are ≤ 10% of the text, audio still generates normally; above that,
> error `1042`.

### `voice_setting`

| Field | Type | Range | Default |
|---|---|---|---|
| `voice_id` | string | — | **required** |
| `speed` | number | `[0.5, 2]` | `1.0` |
| `vol` | number | `(0, 10]` | `1.0` |
| `pitch` | integer | `[-12, 12]` | `0` |
| `emotion` | string | `happy, sad, angry, fearful, disgusted, surprised, calm, fluent, whisper` | `auto` |
| `text_normalization` | boolean | — | `false` |
| `latex_read` | boolean | — | `false` |

`emotion: "whisper"` is **not supported on `speech-2.8` models**.

> ⚠️ **Sync vs async naming differs — this is a real trap:**
> | Concept | Sync `/v1/t2a_v2` | Async `/v1/t2a_async_v2` |
> |---|---|---|
> | Normalization flag | `text_normalization` | `english_normalization` |
> | Sample rate field | `audio_setting.sample_rate` | `audio_setting.audio_sample_rate` |
> | Channel default | `1` | `2` |
>
> A wrong name is silently ignored rather than rejected. Do not copy one body into the other.

### `audio_setting`

| Field | Type | Options | Default |
|---|---|---|---|
| `sample_rate` | integer | `8000, 16000, 22050, 24000, 32000, 44100` | `32000` |
| `bitrate` | integer | `32000, 64000, 128000, 256000` | `128000` |
| `format` | string | `mp3, pcm, flac, wav, pcmu_raw, pcmu_wav, opus` | `mp3` |
| `channel` | integer | `1` mono, `2` stereo | `1` |
| `force_cbr` | boolean | — | `false` |

### `voice_modify`

| Field | Range |
|---|---|
| `pitch`, `intensity`, `timbre` | `[-100, 100]` each |
| `sound_effects` | `spacious_echo`, `auditorium_echo`, `lofi_telephone`, `robotic` |

Voice effects are limited to `mp3`, `wav` and `flac` output formats.

### `pronunciation_dict`

```json
"pronunciation_dict": { "tone": ["Omg/Oh my god", "moments-hq/moments H Q"] }
```
Format `"original/replacement"`. Supports Pinyin (tones 1–5), IPA, and Cantonese Jyutping
(tones 1–6).

### `language_boost` values

`Chinese`, `Chinese,Yue`, `English`, `Arabic`, `Russian`, `Spanish`, `French`, `Portuguese`,
`German`, `Turkish`, `Dutch`, `Ukrainian`, `Vietnamese`, `Indonesian`, `Japanese`, `Italian`,
`Korean`, `Thai`, `Polish`, `Romanian`, `Greek`, `Czech`, `Finnish`, `Hindi`, `Bulgarian`,
`Danish`, `Hebrew`, `Malay`, `Persian`, `Slovak`, `Swedish`, `Croatian`, `Filipino`,
`Hungarian`, `Norwegian`, `Slovenian`, `Catalan`, `Nynorsk`, `Tamil`, `Afrikaans`, `auto`

### Inline pause markers — `<#x#>`

Documented on the `text` parameter itself:

> "You can customize speech pauses by adding markers in the form `<#x#>`, where `x` is the pause
> duration in seconds."

- Valid range `[0.01, 99.99]`, up to two decimal places.
- "Pause markers must be placed between speakable text segments and cannot be used consecutively."
- ✅ Verified working on the sync endpoint.
- ⚠️ On async, `<#x#>` is documented only for `text_file_id` input, **not** for inline `text`.

**There is no SSML support.** `<#x#>` and interjection tags are the complete inline control set.

### Interjection tags — `speech-2.8-hd` / `speech-2.8-turbo` only

`(laughs) (breath) (sighs) (inhale) (exhale) (clear-throat) (coughs) (chuckle) (groans) (pant)
(gasps) (sniffs) (snorts) (burps) (lip-smacking) (humming) (hissing) (emm) (whistles) (sneezes)
(crying) (applause)`

### Response

```json
{
  "data": {
    "audio": "<hex string, OR download URL when output_format=url>",
    "status": 2,
    "subtitle_file": "<URL to subtitle JSON, when subtitle_enable=true>",
    "ced": ""
  },
  "extra_info": {
    "audio_length": 4441,
    "audio_sample_rate": 44100,
    "audio_size": 0,
    "bitrate": 256000,
    "word_count": 43,
    "invisible_character_ratio": 0,
    "usage_characters": 43,
    "audio_format": "wav",
    "audio_channel": 1
  },
  "trace_id": "06c11f53...",
  "base_resp": { "status_code": 0, "status_msg": "success" }
}
```

- **`data.audio` carries the URL when `output_format: "url"`** — there is no separate URL field.
  Default `hex` puts hex-encoded bytes in the same field. Decode with `xxd -r -p`.
- URLs are valid **24 hours**. They are signed — never truncate them.
- `data.status`: `1` = synthesizing, `2` = complete.
- `data.ced` is undocumented and returns empty. Ignore.
- `extra_info.audio_length` is **milliseconds**.
- `extra_info.audio_size` may read `0` for a valid file — not a reliable check.
- `extra_info.usage_characters` is the billed quantity.

### Subtitle JSON schema — ✅ verified live

`data.subtitle_file` is a URL to a JSON array. **All times in milliseconds.**

```json
[
  {
    "text": "One code.",
    "pronounce_text": "One code.",
    "time_begin": 0.0,
    "time_end": 1037.7777777777778,
    "text_begin": 0,
    "text_end": 9,
    "pronounce_text_begin": 0,
    "pronounce_text_end": 9,
    "is_final_segment": true,
    "timestamped_words": [
      {
        "word": "One",
        "word_begin": 0,
        "word_end": 3,
        "pronounce_word": "One",
        "pronounce_word_begin": 0,
        "pronounce_word_end": 3,
        "time_begin": 47.407407407407405,
        "time_end": 189.62962962962962
      }
    ]
  }
]
```

`timestamped_words` is present when `subtitle_type` is `word` or `word_streaming`. Whitespace
appears as its own word entry. Subtitle URL lifetime is not documented — download immediately.

### Verified example request

```json
{
  "model": "speech-2.8-hd",
  "text": "One code.<#0.45#>No app.<#0.45#>No sign-up.",
  "stream": false,
  "language_boost": "English",
  "output_format": "url",
  "subtitle_enable": true,
  "subtitle_type": "word",
  "voice_setting": {
    "voice_id": "English_CaptivatingStoryteller",
    "speed": 0.9, "vol": 1, "pitch": -1, "emotion": "calm"
  },
  "audio_setting": { "sample_rate": 44100, "bitrate": 256000, "format": "wav", "channel": 1 }
}
```

---

## POST `/v1/t2a_async_v2` — asynchronous long-form TTS

For scripts too long for the sync endpoint.

| Field | Notes |
|---|---|
| `model` | Same eight-model enum |
| `text` | **or** `text_file_id`, mutually exclusive, one required. Max **50 000 characters** |
| `text_file_id` | Max **1 000 000 characters**. Formats: txt, zip. **`<#x#>` pauses supported here.** zip must contain files of one type |
| `voice_setting` | `voice_id`, `speed`, `vol`, `pitch`, `emotion`, **`english_normalization`**, `latex_read` |
| `audio_setting` | **`audio_sample_rate`** (default `32000`), `bitrate` (`128000`, MP3 only), `format` (`mp3`), `channel` (**default `2`**) |
| `pronunciation_dict`, `language_boost`, `voice_modify` | as sync |

**`output_format`, `subtitle_enable` and `subtitle_type` are NOT accepted here.**

Create response:
```json
{ "task_id": "95157322514444", "task_token": "eyJhbGciOiJSUz",
  "file_id": 95157322514444, "usage_characters": 101,
  "base_resp": { "status_code": 0, "status_msg": "success" } }
```

Poll: `GET /v1/query/t2a_async_query_v2?task_id={task_id}` — **max 10 queries per second**.
Returns `task_id`, `status`, `file_id`, `base_resp`.
Status values: `success`, `processing`, `failed`, `expired`. The published example shows
`"Processing"` capitalized — **compare case-insensitively.**

Download: `GET /v1/files/retrieve_content?file_id={file_id}` — write the bytes to disk.
The download URL is valid **9 hours** (32 400 s).

Optional input upload: `POST /v1/files/upload` with `purpose=t2a_async_input`.

Async output includes sentence-level subtitle information, but the exact retrieval mechanism is
not documented.

---

## POST `/v1/files/upload`

`Content-Type: multipart/form-data`. Required: `purpose`, `file`.

| `purpose` | Use | Limits |
|---|---|---|
| `voice_clone` | Reference audio for cloning | mp3/m4a/wav, **10 s – 5 min**, ≤ 20 MB |
| `prompt_audio` | Sample to improve clone similarity | mp3/m4a/wav, **< 8 s**, ≤ 20 MB |
| `t2a_async_input` | txt/zip script for async TTS | — |
| `video_understanding` | Video for multimodal understanding | MP4/AVI/MOV/MKV, 7-day retention |
| `video_generation_input` | Video-generation inputs | Image 30 MB, video 50 MB, audio 15 MB |

**There is no music-related `purpose` value.**

```json
{ "file": { "file_id": 0, "bytes": 0, "created_at": 0, "filename": "string", "purpose": "voice_clone" },
  "base_resp": { "status_code": 0, "status_msg": "string" } }
```

---

## POST `/v1/voice_clone`

| Field | Type | Required | Notes |
|---|---|---|---|
| `file_id` | int64 | ✅ | From a `purpose=voice_clone` upload |
| `voice_id` | string | ✅ | 8–256 chars · starts with a letter · letters, digits, `-`, `_` · must not end with `-` or `_` · must be unique |
| `clone_prompt` | object | no | `{prompt_audio: <file_id>, prompt_text: "..."}` — improves similarity and stability |
| `text` | string | no | Preview text ≤ 1000 chars. **Billed per character** |
| `model` | string | required if `text` given | Same eight-model enum |
| `language_boost` | string | no | Default `null` |
| `text_validation` | string | no | Expected transcript ≤ 200 chars; triggers ASR comparison |
| `accuracy` | double | no | `[0, 1]`, default `0.7` — similarity threshold; failure returns `1043` |
| `need_noise_reduction` | boolean | no | Default `false` |
| `need_volume_normalization` | boolean | no | Default `false` |
| `aigc_watermark` | boolean | no | Default `false` — appends a watermark tone |

Response includes `demo_audio` (URL, present only when `text` + `model` were given),
`input_sensitive.type` (0 Normal · 1 Severe violation · 2 Pornographic · 3 Advertisement ·
4 Prohibited · 5 Abusive · 6 Terror/violence · 7 Other), `extra_info`, `base_resp`.

⚠️ **Requires account verification** — `2038` means the account lacks cloning permission.
⚠️ **A cloned voice unused for 7 days is deleted.**

---

## POST `/v1/voice_design`

| Field | Type | Required | Notes |
|---|---|---|---|
| `prompt` | string | ✅ | Natural-language voice description |
| `preview_text` | string | ✅ | Max **500 characters** |
| `voice_id` | string | no | Auto-generated if omitted |

```json
{ "trial_audio": "<hex-encoded audio>",
  "voice_id": "ttv-voice-2025060717322425-xxxxxxxx",
  "base_resp": { "status_code": 0, "status_msg": "success" } }
```

`trial_audio` is **hex-encoded**. Cost: **$30 per 1M characters** of preview text.

---

## POST `/v1/get_voice`

Body: `{"voice_type": "system" | "voice_cloning" | "voice_generation" | "all"}`

Returns arrays `system_voice` (`voice_id`, `voice_name`, `description[]`, `created_time` as
`YYYY-MM-DD`), `voice_cloning`, `voice_generation`, plus `base_resp`.

---

## Error codes

| Code | Meaning |
|---|---|
| `0` | Success |
| `1000` | Unknown error |
| `1001` | Timeout |
| `1002` | RPM rate limit exceeded |
| `1004` | Authentication failed |
| `1008` | Insufficient balance |
| `1013` | Internal service error |
| `1027` | Output content error |
| `1039` | TPM rate limit exceeded |
| `1042` | Invalid characters exceed 10% |
| `1043` | Clone ASR similarity below `accuracy` |
| `1044` | Clone prompt similarity check failed |
| `2013` | Invalid input parameters |
| `2037` | Voice sample duration too short or too long |
| `2038` | **No cloning permission — check account verification** |
| `2039` | Duplicate clone `voice_id` |
| `2042` | No access to that `voice_id` |
| `2048` | Prompt audio too long |
| `2049` | Invalid API key |
| `20132` | Invalid samples or `voice_id` |

Always check `base_resp.status_code` — HTTP 200 does not imply success.

---

## Rate limits

| API | RPM |
|---|---|
| T2A (all `speech-2.8` / `2.6` / `02` models) | **60** |
| Voice cloning | **60** |
| Voice design | **20** |
| Async status query | 10 QPS |

No TPM or concurrency figure is published for speech, though error `1039` (TPM) exists.
To raise limits: `api@minimax.io`.
