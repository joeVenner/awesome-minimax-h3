# MiniMax Music API — Complete Reference

> Base URLs:
> - Music Generation: `https://api.minimax.io/v1/music_generation`
> - Lyrics Generation: `https://api.minimax.io/v1/lyrics_generation`
> - Music Cover Preprocess: `https://api.minimax.io/v1/music_cover_preprocess`
> - Auth: `Authorization: Bearer <API_KEY>` (get key from https://platform.minimax.io → Account → API Keys)

---

## Endpoint 1: Music Generation

### POST `/v1/music_generation`

Generate a song from lyrics and a style prompt, or generate instrumental music.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model` | string | ✅ | `music-3.0`, `music-2.6`, `music-cover`, or `*-free` variants |
| `prompt` | string | Conditional | 1–2000 chars. Required for instrumental or cover. Optional for vocal songs. |
| `lyrics` | string | Conditional | Lyrics with `\n` line breaks. Required for vocal songs. Max ~3000 chars. |
| `is_instrumental` | boolean | No | `true` to generate instrumental only. Default: `false` |
| `reference_audio_url` | string | Conditional | Required for `music-cover` model |
| `audio_setting` | object | No | Output audio configuration |

**`audio_setting` object:**

| Field | Type | Default | Options |
|-------|------|---------|---------|
| `sample_rate` | int | 44100 | 44100, 48000 |
| `bitrate` | int | 256000 | 128000, 256000, 320000 |
| `format` | string | mp3 | mp3, wav |

**Model Options:**

| Model | Description | RPM | Access |
|-------|-------------|-----|--------|
| `music-3.0` | Latest, best quality | 120 | Token Plan / Paid |
| `music-2.6` | Previous generation | 120 | Token Plan / Paid |
| `music-cover` | Cover generation from reference | 120 | Token Plan / Paid |
| `music-3.0-free` | Free music-3.0 | 3 | All users |
| `music-2.6-free` | Free music-2.6 | 3 | All users |
| `music-cover-free` | Free cover generation | 3 | All users |

**Prompt Requirements by Model:**

| Model | `is_instrumental` | Prompt Required? | Lyrics Required? |
|-------|-------------------|-----------------|-----------------|
| `music-3.0` / `*-free` | `true` | ✅ Required | ❌ Not used |
| `music-3.0` / `*-free` | `false` (default) | Optional | ✅ Required |
| `music-2.6` / `*-free` | `true` | ✅ Required | ❌ Not used |
| `music-2.6` / `*-free` | `false` (default) | Optional | ✅ Required |
| `music-cover` / `*-free` | N/A | ✅ Required (cover style) | ❌ |

**Example Requests:**

```bash
# Full song with lyrics
curl -X POST "https://api.minimax.io/v1/music_generation" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-3.0",
    "prompt": "Indie folk, melancholic, introspective, longing, rainy afternoon, acoustic guitar, gentle piano, warm vocals",
    "lyrics": "[Verse]\nWalking alone through autumn streets\nLeaves falling at my feet\n[Chorus]\nI remember when you said goodbye\nUnder the November sky",
    "audio_setting": {"sample_rate": 44100, "bitrate": 256000, "format": "mp3"}
  }'

# Instrumental only
curl -X POST "https://api.minimax.io/v1/music_generation" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-3.0",
    "prompt": "Ambient electronic, meditative, serene, floating, sparse piano, warm pads, gentle arpeggios, slow tempo",
    "is_instrumental": true,
    "audio_setting": {"sample_rate": 44100, "bitrate": 320000, "format": "wav"}
  }'

# Cover song
curl -X POST "https://api.minimax.io/v1/music_generation" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-cover",
    "prompt": "Acoustic folk version, gentle female vocals, fingerpicked guitar, intimate, warm, slow",
    "reference_audio_url": "https://example.com/original-pop-song.mp3",
    "audio_setting": {"sample_rate": 44100, "bitrate": 256000, "format": "mp3"}
  }'
```

**Success Response (200):**

```json
{
  "data": {
    "audio": "hex-encoded-audio-data",
    "status": 2
  },
  "trace_id": "04ede0ab069fb1ba8be5156a24b1e081",
  "extra_info": {
    "music_duration": 25364,
    "music_sample_rate": 44100,
    "music_channel": 2,
    "bitrate": 256000,
    "music_size": 813651
  },
  "analysis_info": null,
  "base_resp": {
    "status_code": 0,
    "status_msg": "success"
  }
}
```

**Response Field Reference:**

| Field | Type | Description |
|-------|------|-------------|
| `data.audio` | string | Hex-encoded audio data. Decode with `xxd -r -p` |
| `data.status` | int | `2` = success |
| `extra_info.music_duration` | int | Duration in **milliseconds** |
| `extra_info.music_sample_rate` | int | Sample rate in Hz |
| `extra_info.music_channel` | int | Channel count (always `2` = stereo) |
| `extra_info.bitrate` | int | Bitrate in bps |
| `extra_info.music_size` | int | File size in bytes |
| `base_resp.status_code` | int | `0` = success |
| `base_resp.status_msg` | string | `"success"` on success |

**Error Codes:**

| Code | Meaning | Action |
|------|---------|--------|
| `400` | Bad request | Check prompt length (≤2000 chars), lyrics length, parameter values |
| `401` | Unauthorized | Verify API key |
| `402` | Payment required | Top up balance |
| `429` | Rate limited | Back off; check RPM limits (3 for free, 120 for paid) |
| `500` | Server error | Retry after 30s |

---

## Endpoint 2: Lyrics Generation

### POST `/v1/lyrics_generation`

Generate or edit song lyrics with structure tags.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mode` | string | ✅ | `"write_full_song"` or `"edit"` |
| `prompt` | string | No | Theme/style description. Max 2000 chars. Random if empty. |
| `lyrics` | string | Conditional | Existing lyrics. Only for `edit` mode. Max 3500 chars. |
| `title` | string | No | Song title. If provided, output preserves it. |

**Example Requests:**

```bash
# Generate complete lyrics
curl -X POST "https://api.minimax.io/v1/lyrics_generation" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "write_full_song",
    "prompt": "A cheerful summer love song about meeting at the beach, upbeat, youthful, catchy chorus"
  }'

# Edit existing lyrics
curl -X POST "https://api.minimax.io/v1/lyrics_generation" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "edit",
    "prompt": "Add a bridge about overcoming distance, make the final chorus more powerful and anthemic",
    "lyrics": "[Verse 1]\nWe met under summer skies\nYour smile caught me by surprise...",
    "title": "Summer Promise"
  }'
```

**Success Response (200):**

```json
{
  "song_title": "Summer Breeze Promise",
  "style_tags": "Pop, Summer Vibe, Romance, Lighthearted, Beach Pop",
  "lyrics": "[Intro]\n(Ooh-ooh-ooh)\n\n[Verse 1]\nSea breeze gently through your hair\nSmiling face, like a summer dream...",
  "base_resp": { "status_code": 0, "status_msg": "success" }
}
```

**Response Fields:**

| Field | Description |
|-------|-------------|
| `song_title` | Generated title (preserved if provided in request) |
| `style_tags` | Comma-separated style descriptors — can be used directly in music prompt |
| `lyrics` | Complete lyrics with structure tags and `\n` line breaks — ready for music generation |

---

## Endpoint 3: Music Cover Preprocess

### POST `/v1/music_cover_preprocess`

Preprocess reference audio before cover generation. Required before using `music-cover`.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `reference_audio_url` | string | ✅ | Public URL of the reference audio file |

```bash
curl -X POST "https://api.minimax.io/v1/music_cover_preprocess" \
  -H "Authorization: Bearer $KEY" \
  -H "Content-Type: application/json" \
  -d '{"reference_audio_url": "https://example.com/song.mp3"}'
```

---

## Audio Decoding Cheat Sheet

```bash
# Save hex audio from JSON response
curl ... | jq -r '.data.audio' | xxd -r -p > output.mp3

# Save base64 audio (if response format differs)
curl ... | jq -r '.data.audio' | base64 -d > output.mp3

# Verify the file
file output.mp3
ffprobe output.mp3
```
