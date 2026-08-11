---
name: minimax-speech
description: Generate speech, voiceovers, narration and audiobooks using the MiniMax CLOUD Text-to-Speech API (T2A v2), plus voice cloning and text-described voice design. Supports 332 preset voices across 40+ languages, precise inline pause control, word- and sentence-level timestamp output, emotion and prosody control, streaming, and async long-form synthesis up to 1M characters. Use when the user asks to create a voiceover, narrate a script, read text aloud, generate speech or TTS, clone a voice from a sample, design a new voice from a description, produce an audiobook or podcast narration, add narration to a video, or any task involving speech synthesis with MiniMax. Keywords: speech, TTS, text to speech, voiceover, VO, narration, narrator, voice, audio, read aloud, speak, dub, dubbing, audiobook, podcast, voice clone, voice cloning, voice design, timestamps, subtitles, MiniMax, t2a, speech-2.8-hd.
license: MIT
compatibility: Requires a MiniMax platform API key (https://platform.minimax.io), curl, jq, and ffmpeg for post-processing
metadata:
  author: ylafrimi
  version: "1.0"
  model: speech-2.8-hd
  api_base: https://api.minimax.io/v1/t2a_v2
  verified: 2026-08-04 against the live API
allowed-tools: Bash(curl:*) Bash(ffmpeg:*) Bash(ffprobe:*) Bash(jq:*) Read Write
---

# MiniMax Speech — Cloud Text-to-Speech, Voice Cloning & Voice Design

**This skill covers the CLOUD platform API only** (`api.minimax.io`, Bearer auth with a
platform API key). It does not cover the local Electron gateway — see `minimax-local-audio`
for that, and do not mix their parameter shapes: this API uses real JSON types
(`"speed": 0.9`), the local gateway uses strings (`"speed": "1"`).

---

## Quick start

```bash
curl -s -X POST "https://api.minimax.io/v1/t2a_v2" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "speech-2.8-hd",
    "text": "The text to speak.",
    "output_format": "url",
    "language_boost": "English",
    "voice_setting": {"voice_id": "English_CaptivatingStoryteller", "speed": 0.9, "vol": 1, "pitch": 0, "emotion": "calm"},
    "audio_setting": {"sample_rate": 44100, "bitrate": 256000, "format": "wav", "channel": 1}
  }' | jq -r '.data.audio' | xargs curl -sL -o narration.wav
```

No `GroupId` parameter is needed. The call is **synchronous** — audio comes back in one response.

---

## Phase 0: Choose the approach

| Need | Use |
|------|-----|
| Any script under 10 000 characters | **Sync `/v1/t2a_v2`** — simplest, returns audio directly |
| Long-form (audiobook, > 10 000 chars, up to 1M) | **Async `/v1/t2a_async_v2`** — see [async reference](references/api-reference.md) |
| Real-time / low latency playback | `"stream": true`, or the WebSocket endpoint |
| A voice that matches a real person's sample | **Voice cloning** — Phase 5 |
| A specific timbre with no sample to work from | **Voice design** — Phase 6, describe it in words |

**For voiceover against picture, always use the sync endpoint and render each line as its own
call.** See Phase 4 — this is the single most important technique in this skill.

---

## Phase 1: Pick the model

| Model | Character |
|-------|-----------|
| **`speech-2.8-hd`** | **Best quality. Supports interjection tags. Default choice for narration.** |
| `speech-2.8-turbo` | Faster, natural flow |
| `speech-2.6-hd` | Ultra-low latency, enhanced naturalness |
| `speech-2.6-turbo` | Faster, cheaper, good for agents |
| `speech-02-hd` / `speech-02-turbo` | Superior rhythm and stability |
| `speech-01-hd` / `speech-01-turbo` | Legacy |

`emotion: "whisper"` is **not available on `speech-2.8` models**. Interjection tags are
**`speech-2.8` only**.

---

## Phase 2: Pick a voice

**332 preset voices** across 40+ languages. Full list in
[references/voice-catalog.md](references/voice-catalog.md).

English narration shortlist — the names are the only descriptor the platform publishes, so
**audition 2–3 before committing to a long script**:

| Intent | Candidates |
|--------|-----------|
| Warm, low, documentary confidant | `English_Deep-VoicedGentleman`, `English_ManWithDeepVoice`, `English_magnetic_voiced_man` |
| Storytelling / trailer | `English_CaptivatingStoryteller`, `English_expressive_narrator`, `English_Steadymentor` |
| Calm female | `English_CalmWoman`, `English_SereneWoman`, `English_Graceful_Lady`, `English_Wiselady` |
| Trustworthy / corporate | `English_Trustworth_Man` *(spelled exactly so)*, `English_PatientMan`, `English_ConfidentWoman` |
| Bright / upbeat | `English_Upbeat_Woman`, `English_radiant_girl`, `English_Jovialman` |

To list every voice available on the account (system + cloned + designed):

```bash
curl -s -X POST "https://api.minimax.io/v1/get_voice" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" -H "Content-Type: application/json" \
  -d '{"voice_type": "all"}' | jq
```

---

## Phase 3: Shape the delivery

### `voice_setting`

| Field | Range | Default | Notes |
|-------|-------|---------|-------|
| `voice_id` | string | — | **Required** |
| `speed` | `[0.5, 2]` | `1.0` | `0.85–0.95` reads unhurried without sounding stretched |
| `vol` | `(0, 10]` | `1.0` | |
| `pitch` | `[-12, 12]` integer | `0` | `-1`/`-2` warms a voice; below `-4` sounds processed |
| `emotion` | see below | `auto` | `happy, sad, angry, fearful, disgusted, surprised, calm, fluent, whisper` |
| `text_normalization` | boolean | `false` | Improves digit reading; slightly higher latency |
| `latex_read` | boolean | `false` | Read LaTeX notation |

### `audio_setting`

| Field | Options | Default |
|-------|---------|---------|
| `sample_rate` | `8000, 16000, 22050, 24000, 32000, 44100` | `32000` |
| `bitrate` | `32000, 64000, 128000, 256000` | `128000` |
| `format` | `mp3, pcm, flac, wav, pcmu_raw, pcmu_wav, opus` | `mp3` |
| `channel` | `1` mono, `2` stereo | `1` |

**For anything that will be mixed or edited, request `wav` at `44100`.** Start lossless; the
edit will re-encode anyway.

### `language_boost`
Set it explicitly (`"English"`, `"French"`, …, or `"auto"`). 40+ values — see the reference.
It measurably improves pronunciation of proper nouns.

### Other controls
- **`pronunciation_dict`** — force pronunciations: `{"tone": ["Omg/Oh my god", "moments-hq/moments H Q"]}`.
  Essential for brand names, URLs and acronyms.
- **`voice_modify`** — `pitch`, `intensity`, `timbre` each `[-100, 100]`, plus `sound_effects`
  ∈ `spacious_echo`, `auditorium_echo`, `lofi_telephone`, `robotic`.
- **`timbre_weights`** — blend up to 4 voices, weights `[1, 100]`. Legacy.

---

## Phase 4: Timing — pauses and timestamps

This is what makes MiniMax genuinely good for narration against picture.

### Inline pauses — `<#x#>`

Insert silence of `x` seconds directly in the text:

```
"One code.<#0.45#>No app.<#0.45#>No sign-up."
```

- Range `[0.01, 99.99]`, up to 2 decimal places.
- **Must sit between speakable segments.** Not at the very start or end, and never two in a row
  (`<#1#><#1#>` is invalid).
- ✅ Verified working on the sync endpoint.
- ⚠️ On the **async** endpoint, `<#x#>` is documented only for `text_file_id` (txt input), *not*
  for the inline `text` parameter. If a pause comes back spoken literally, that asymmetry is why.

**There is no SSML.** `<#x#>` plus interjection tags are the entire inline control surface.

### Interjection tags — `speech-2.8` models only

`(laughs) (breath) (sighs) (inhale) (exhale) (clear-throat) (coughs) (chuckle) (groans) (pant)
(gasps) (sniffs) (snorts) (burps) (lip-smacking) (humming) (hissing) (emm) (whistles) (sneezes)
(crying) (applause)`

A `(breath)` before an emotional line is the cheapest way to make a read sound human.

### Timestamps — `subtitle_enable`

```json
"subtitle_enable": true,
"subtitle_type": "word"
```

`subtitle_type` ∈ `sentence` (default) · `word` · `word_streaming`.

Returns `data.subtitle_file` — a URL to a JSON array. ✅ Verified live schema:

```json
[
  {
    "text": "One code.",
    "pronounce_text": "One code.",
    "time_begin": 0.0,
    "time_end": 1037.78,
    "text_begin": 0, "text_end": 9,
    "is_final_segment": true,
    "timestamped_words": [
      { "word": "One", "time_begin": 47.41, "time_end": 189.63,
        "word_begin": 0, "word_end": 3 }
    ]
  }
]
```

**All times are milliseconds.** This gives word-accurate positions for cutting picture to
narration, animating captions, or syncing subtitles.

### The technique for voiceover against picture

**Render each line as its own request**, with identical `voice_id`, `speed`, `pitch` and
`emotion` across all of them, then place the clips on a timeline yourself:

1. Every line's exact length arrives as `extra_info.audio_length` (ms).
2. Silence between lines is placed by you, frame-accurately — not guessed by the model.
3. One bad line can be re-rendered without disturbing the others.

Do **not** slow an entire read with `speed: 0.7` to fill a duration — that stretches delivery and
sounds artificial. Keep speed natural and buy time with placed silence.

```bash
# Place lines at exact timestamps
ffmpeg -i l1.wav -i l2.wav -i l3.wav -filter_complex \
 "[0]adelay=0|0[a];[1]adelay=9000|9000[b];[2]adelay=15000|15000[c];[a][b][c]amix=inputs=3:normalize=0" vo.wav
```

---

## Phase 5: Voice cloning

Three steps. **Requires account verification** — status `2038` means the account is not cleared
for cloning.

```bash
# 1. Upload the reference audio: mp3/m4a/wav, 10s–5min, ≤ 20 MB
curl -s -X POST "https://api.minimax.io/v1/files/upload" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -F "purpose=voice_clone" -F "file=@sample.wav"
# -> .file.file_id

# 2. Clone
curl -s -X POST "https://api.minimax.io/v1/voice_clone" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" -H "Content-Type: application/json" \
  -d '{"file_id": 123456, "voice_id": "MomentsNarrator01",
       "need_noise_reduction": true, "need_volume_normalization": true}'

# 3. Use it — pass voice_id straight into t2a_v2
```

**`voice_id` naming rules:** 8–256 chars · must start with a letter · letters, digits, `-`, `_`
only · must not end with `-` or `_` · must be unique on the account.

| Param | Notes |
|-------|-------|
| `clone_prompt` | `{prompt_audio: <file_id>, prompt_text: "..."}` — a <8s sample uploaded with `purpose=prompt_audio`. Improves similarity and stability. |
| `text` + `model` | Optional preview, ≤1000 chars. **Billed by character** and returns `demo_audio` URL. |
| `text_validation` | Expected transcript (≤200 chars); triggers an ASR similarity check |
| `accuracy` | `[0,1]`, default `0.7` — threshold for that check. Failing returns `1043` |
| `need_noise_reduction` / `need_volume_normalization` | default `false`. Turn both on for phone-recorded samples |
| `aigc_watermark` | default `false` |

⚠️ **A cloned voice is deleted if unused for 7 days.** Generate at least one line with it inside
that window, or re-clone.

---

## Phase 6: Voice design — describe a voice in words

The fastest way to a specific timbre with no sample to source. Costs **$30 per 1M characters**
of preview text.

```bash
curl -s -X POST "https://api.minimax.io/v1/voice_design" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" -H "Content-Type: application/json" \
  -d '{
    "prompt": "A warm, low, unhurried male narrator in his forties. Intimate and confiding, like a close friend telling a story late at night. Slight gravel, no announcer polish.",
    "preview_text": "Two hundred people came to your wedding. Two hundred phones."
  }'
```

Returns `voice_id` (shape `ttv-voice-<timestamp>-<hex>`) and `trial_audio` (**hex-encoded**).
`preview_text` max 500 chars. Pass an explicit `voice_id` to name it yourself. Then use that
`voice_id` in `t2a_v2` exactly like a preset.

---

## Phase 7: Response handling

✅ Verified live response shape:

```json
{
  "data": {
    "audio": "<hex string, OR a download URL when output_format=url>",
    "status": 2,
    "subtitle_file": "<URL, present when subtitle_enable=true>"
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

**Key facts, all verified:**
- **`output_format: "url"` puts the URL in `data.audio` itself** — there is no separate
  `audio_url` field. The default is `hex`, which puts hex-encoded bytes in the same field.
  Always set `output_format` explicitly so you know which you are parsing.
- URLs are valid **24 hours** — download immediately.
- `data.status`: `1` = synthesizing, `2` = complete.
- `extra_info.audio_length` is **milliseconds** — use it for timeline math.
- `extra_info.audio_size` can read `0` even for a valid file. Do not use it as a sanity check.
- `extra_info.usage_characters` is what you are billed for.
- An undocumented empty `data.ced` field is also returned. Ignore it.
- Always check `base_resp.status_code == 0`. HTTP 200 does **not** imply success.

```bash
# hex (default)
jq -r '.data.audio' resp.json | xxd -r -p > out.mp3
# url
jq -r '.data.audio' resp.json | xargs curl -sL -o out.wav
```

---

## Error codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1000` / `1001` | Unknown error / timeout |
| `1002` | RPM rate limit — back off |
| `1004` | Authentication failed |
| `1008` | Insufficient balance |
| `1039` | TPM rate limit |
| `1042` | Invalid characters exceed 10% |
| `1043` | Clone ASR similarity below `accuracy` |
| `2013` | Invalid parameters |
| `2038` | **No cloning permission — account not verified** |
| `2049` | Invalid API key |

**Rate limits:** T2A **60 RPM** · Voice Cloning **60 RPM** · Voice Design **20 RPM** ·
async query 10 QPS.

---

## Common failures and fixes

| Symptom | Cause | Fix |
|---------|-------|-----|
| Pause markers spoken aloud | Used `<#x#>` in async inline `text` | Use the sync endpoint, or async with `text_file_id` |
| Garbage audio file | Decoded `url` output as hex | Set `output_format` explicitly and branch on it |
| Brand name mispronounced | No pronunciation hint | `pronunciation_dict.tone` |
| Read sounds artificially slow | Filled duration with low `speed` | Keep `speed ≥ 0.85`; add silence between lines instead |
| Emotion ignored | `whisper` on a 2.8 model | Not supported there — pick another emotion or model |
| Interjection tag spoken literally | Not a 2.8 model | Switch to `speech-2.8-hd` |
| Cloned voice vanished | Unused for 7 days | Re-clone; use it at least once per week |
| HTTP 200 but no audio | Only checked HTTP status | Check `base_resp.status_code` |

---

## Supporting files

- [Complete API reference](references/api-reference.md) — every parameter, async endpoints, file upload, full schemas
- [Voice catalogue](references/voice-catalog.md) — all 332 preset voice IDs by language
