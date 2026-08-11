---
name: minimax-music
description: Generate professional-quality music using the MiniMax CLOUD Music API (music-3.0). Supports full songs with vocals, purely instrumental tracks, AI lyrics writing, and cover generation from reference audio. Typical output is 2-3 minutes of finished stereo audio. Use when the user asks to create music, generate a song, write lyrics, compose an instrumental, make a cover of an existing song, produce background or underscore music for a video, film or advert, create a jingle or soundtrack, or any task involving AI music creation with MiniMax. Keywords: music, song, compose, generate music, lyrics, instrumental, underscore, background music, music bed, cover song, jingle, soundtrack, score, beat, melody, audio generation, music production, AI music, MiniMax, music-3.0, songwriting, genre, BPM, vocal style.
license: MIT
compatibility: Requires a MiniMax platform API key (https://platform.minimax.io), curl, jq, and ffmpeg for post-processing
metadata:
  author: ylafrimi
  version: "2.0"
  model: music-3.0
  api_base: https://api.minimax.io/v1/music_generation
  verified: 2026-08-04 against the live API
allowed-tools: Bash(curl:*) Bash(ffmpeg:*) Bash(ffprobe:*) Bash(jq:*) Read Write
---

# MiniMax Music — Cloud Music Generation

**This skill covers the CLOUD platform API only** (`api.minimax.io`, Bearer auth with a platform
API key). It does not cover the local Electron gateway — see `minimax-local-audio` for that, and
do not mix their parameter shapes: this API takes real JSON types (`"is_instrumental": true`),
the local gateway takes strings (`"is_instrumental": "instrumental"`).

---

## Quick start — instrumental bed

```bash
curl -s -X POST "https://api.minimax.io/v1/music_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-3.0",
    "is_instrumental": true,
    "prompt": "Cinematic indie folk, warm, nostalgic, hopeful, felt piano, soft string swell, slow tempo, 76 BPM",
    "audio_setting": {"sample_rate": 44100, "bitrate": 256000, "format": "mp3"},
    "output_format": "url"
  }' -o resp.json

jq -r '.data.audio' resp.json | xargs curl -sL -o track.mp3
```

> ⏱ **Generation takes ~3 minutes and the call blocks the whole time.** Set a client timeout of
> at least 600s and run it in the background. A 120s shell timeout will kill it mid-flight.

---

## ✅ Verified facts that contradict older documentation

| Older claim | Reality (verified 2026-08-04) |
|---|---|
| Output is ~25 seconds | **~161 seconds measured.** The "25364" in the docs is an *example* value, not a limit |
| Audio returns as hex only | `output_format: "url"` returns a **signed download URL in `data.audio`** |
| `data.audio` is a separate field from any URL | **Same field** carries either hex or the URL, depending on `output_format` |
| Only `music-3.0` and `music-cover` exist | **Six models exist**, including free-tier variants |

**There is still no duration parameter and no way to place a climax at a chosen timestamp.**
Output length is emergent — but it is long, so for anything under ~2 minutes the practical
approach is: generate, measure, then trim and fade to fit in ffmpeg.

---

## Phase 0: The brief

```
[1. GENRE & STYLE] — musical DNA: genre, subgenre, era, reference sound, BPM, key
[2. MOOD & EMOTION] — emotional arc, energy, atmosphere, time of day
[3. LYRICS & THEME] — the story (skip entirely for instrumentals)
[4. ARRANGEMENT] — instrumentation, dynamics, section structure, production style
```

Decide first: **song with vocals**, **instrumental**, or **cover**? That choice determines
which parameters are required — see the table in Phase 2.

---

## Phase 1: Models

| Model | Use | RPM | Cost |
|-------|-----|-----|------|
| **`music-3.0`** | **Best quality. Default choice.** | 120 | $0.15 per track (up to 5 min) |
| `music-2.6` | Previous generation | 120 | $0.15 per track |
| `music-cover` | Cover from reference audio | 120 | Not stated |
| **`music-3.0-free`** | **Free tier of music-3.0 — ideal for auditioning prompts** | 3 | Free |
| `music-2.6-free` | Free tier of music-2.6 | 3 | Free |
| `music-cover-free` | Free tier of cover | 3 | Free |

Paid models require a Token Plan or paid account; `-free` variants work on any API key.

> 💡 **Billing is flat per generation, bucketed at "up to 5 minutes".** A 30-second track costs
> the same $0.15 as a 5-minute one — there is no incentive to aim short. And since the peak
> position cannot be controlled, plan on generating several takes and selecting.
> **Audition prompts on `music-3.0-free` first** (3 RPM = one every 20s), then spend on the winner.

---

## Phase 2: Parameters

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `model` | string | ✅ | One of the six above |
| `prompt` | string | conditional | Comma-separated descriptors. See the table below |
| `lyrics` | string | conditional | `\n` line breaks + `[section tags]`. Max 3500 chars |
| `is_instrumental` | boolean | no | `true` = no vocals. Default `false` |
| `lyrics_optimizer` | boolean | no | `true` + empty `lyrics` → auto-writes lyrics from `prompt` |
| `output_format` | string | no | `url` or `hex`. **Default `hex`** |
| `stream` | boolean | no | Default `false`. Only `hex` is supported when streaming |
| `audio_setting` | object | no | See below |
| `audio_url` / `audio_base64` | string | conditional | Cover only. Exactly one. Mutually exclusive |
| `cover_feature_id` | string | conditional | Cover only, two-step flow. Mutually exclusive with the above |

### What is required, by mode

| Mode | `prompt` | `lyrics` |
|------|----------|----------|
| **Instrumental** (`is_instrumental: true`) | ✅ **Required**, 1–2000 chars | Not required — **omit the key entirely** |
| Song with vocals | Optional, 0–2000 chars | ✅ Required, 1–3500 chars |
| Vocals with auto-lyrics | ✅ Required | Omit, and set `lyrics_optimizer: true` |
| Cover | ✅ Required, 10–300 chars — describes the *target style* | Optional (ASR-extracted if omitted) |

⚠️ Instrumental is the one mode where `prompt` is mandatory. And although the docs only say
`lyrics` is "not required" there, the schema sets `minLength: 1` — so sending `"lyrics": ""`
may be rejected. **Omit the key rather than passing an empty string.**

`is_instrumental` and `lyrics_optimizer` are supported on the four text-to-music models only,
not on `music-cover`.

### `audio_setting`

| Field | Options | Notes |
|-------|---------|-------|
| `sample_rate` | `16000, 24000, 32000, 44100` | Use `44100` |
| `bitrate` | `32000, 64000, 128000, 256000` | Use `256000` |
| `format` | `mp3, wav, pcm` | `wav` if the track will be heavily edited |

No defaults are documented — set all three explicitly.

### Lyrics section tags

`[Intro]` `[Verse]` `[Pre Chorus]` `[Chorus]` `[Post Chorus]` `[Bridge]` `[Interlude]`
`[Transition]` `[Break]` `[Hook]` `[Build Up]` `[Inst]` `[Solo]` `[Outro]`

⚠️ The **lyrics-generation** endpoint emits a slightly different set — it can return `[Drop]`,
`[Instrumental]`, `[Breakdown]`, `[Pre-Chorus]` (hyphenated) and `[Build-up]` (hyphenated), none
of which appear in the list above. Whether the music API errors on, ignores, or interprets an
unlisted tag is undocumented. **Normalise tags before passing lyrics from one endpoint to the
other.** Official code examples use lowercase (`[verse]`), so case appears not to matter.

There is **no BPM, key, arrangement or climax-placement parameter.** Naming a BPM inside the
`prompt` string is an accepted convention and does influence the result, but it is a hint, not a
control. Section tags are not documented to affect instrumental arrangement at all.

---

## Phase 3: Writing the prompt

The `prompt` is a **comma-separated list of descriptors** — think of it as tagging a track.

```
{genre}, {subgenre}, {mood ×2-3}, {scenario}, {instrumentation}, {production}, {tempo}, {BPM}
```

**Worked examples:**

```
Cinematic indie folk, neoclassical, warm, nostalgic, hopeful, intimate, golden hour wedding,
felt piano, fingerpicked acoustic guitar, soft string swell, cello, brushed drums,
sparse opening building to a full emotional crescendo then gentle resolve, analog warmth,
slow tempo, 76 BPM
```

```
Synthwave, retro electro, euphoric, driving, night city drive, pulsating analog synthesizers,
gated reverb drums, neon atmosphere, 80s production, medium tempo, 110 BPM
```

```
Lo-fi hip-hop, chill, laid-back, rainy coffee shop, warm vinyl crackle, jazzy piano samples,
relaxed boom bap drums, slow tempo, 78 BPM
```

⚠️ **When `is_instrumental: true`, strip every vocal descriptor from the prompt.** Many stock
genre strings contain `warm harmonies`, `female vocals`, `choir` or `layered vocals` — these
contradict the instrumental flag. `choir` counts as a vocal token even though it reads as
orchestral.

Deeper vocabulary — genres, BPM ranges, instrumentation, mood lexicon — lives in
[references/genre-style-reference.md](references/genre-style-reference.md).

---

## Phase 4: Lyrics generation (optional)

```bash
curl -s -X POST "https://api.minimax.io/v1/lyrics_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" -H "Content-Type: application/json" \
  -d '{"mode": "write_full_song", "prompt": "A melancholic ballad about lost love in a rainy city", "title": "Rain on Windows"}'
```

| Param | Required | Notes |
|-------|----------|-------|
| `mode` | ✅ | `write_full_song` or `edit` |
| `prompt` | no | Max 2000 chars. Empty → a random song |
| `lyrics` | no | Existing lyrics; `edit` mode only. Max 3500 |
| `title` | no | Preserved in the output if given |

Returns `song_title`, `style_tags`, `lyrics`, `base_resp`.

💡 **`style_tags` is a comma-separated descriptor string designed to be dropped straight into a
music `prompt`.** Even for an instrumental, one lyrics call gives you model-native style
vocabulary — then discard the lyrics. Costs **$0.01 per song**.

Craft guidance in [references/lyrics-guide.md](references/lyrics-guide.md).

---

## Phase 5: Cover generation

**One-step** — pass the reference directly:

```bash
curl -s -X POST "https://api.minimax.io/v1/music_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" -H "Content-Type: application/json" \
  -d '{"model": "music-cover",
       "audio_url": "https://example.com/original.mp3",
       "prompt": "Lo-fi bedroom pop, dreamy, soft vocals, ukulele, intimate, warm",
       "audio_setting": {"sample_rate": 44100, "bitrate": 256000, "format": "mp3"},
       "output_format": "url"}'
```

**Two-step** — preprocess first when you want to rewrite the lyrics. **Preprocessing is free.**

```bash
curl -s -X POST "https://api.minimax.io/v1/music_cover_preprocess" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" -H "Content-Type: application/json" \
  -d '{"model": "music-cover", "audio_url": "https://example.com/original.mp3"}'
```

Returns `cover_feature_id` (valid 24h), `formatted_lyrics` (ASR-extracted with section tags),
`structure_result` (**JSON with per-section start/end timestamps in seconds**) and
`audio_duration`. Then pass `cover_feature_id` + your rewritten `lyrics` (10–1000 chars) to
`music_generation`.

**Reference audio limits:** 6 seconds – 6 minutes, ≤ 50 MB, mp3/wav/flac.

⚠️ **The Files API cannot host it.** `/v1/files/upload` has no music-related `purpose` value,
and the music endpoints accept no `file_id`. Your only options are a publicly reachable
`audio_url` or inline `audio_base64` (which inflates the request ~33%). A signed object-storage
URL is the clean route.

`music_cover_preprocess` accepts `music-cover` only — not `music-cover-free`.

---

## Phase 6: Response handling

✅ Verified live response:

```json
{
  "data": {
    "audio": "<hex string, OR a signed download URL when output_format=url>",
    "status": 2
  },
  "trace_id": "06c11f46...",
  "extra_info": {
    "music_duration": 160992,
    "music_sample_rate": 44100,
    "music_channel": 2,
    "bitrate": 256000,
    "music_size": 5158939
  },
  "analysis_info": null,
  "base_resp": { "status_code": 0, "status_msg": "success" }
}
```

**Key facts, all verified:**
- **`output_format: "url"` puts a signed URL in `data.audio`.** There is no separate `audio_url`
  field. The default `hex` puts hex-encoded bytes in that same field. **Always set
  `output_format` explicitly** so you know what you are parsing.
- **The URL is signed and expires in 24 hours.** It must be used whole — truncating it strips the
  signature and the download returns an `AccessDenied` XML body that will masquerade as an audio
  file. Always verify with `ffprobe` after downloading.
- `extra_info.music_duration` is in **milliseconds** (160992 = 161 s). `data.status`: `1` = in
  progress, `2` = complete.
- `trace_id`, `extra_info` and `analysis_info` sit at the **top level**, siblings of `data` — not
  nested inside it.
- Always check `base_resp.status_code == 0`. HTTP 200 does not imply success.

```bash
# url
jq -r '.data.audio' resp.json | xargs curl -sL -o track.mp3
# hex (default)
jq -r '.data.audio' resp.json | xxd -r -p > track.mp3
# always verify
ffprobe -v error -show_entries format=duration -of default=nw=1 track.mp3
```

---

## Phase 7: Post-processing

Because length is emergent and the climax cannot be placed, **fitting the track to picture
happens here.**

```bash
# Inspect
ffprobe -v quiet -print_format json -show_format -show_streams track.mp3

# Find the loudest moment — locates the natural crescendo to cut against
ffmpeg -i track.mp3 -af astats=metadata=1:reset=1,ametadata=print:key=lavfi.astats.Overall.RMS_level -f null - 2>&1 | grep RMS

# Trim a window and fade (e.g. 48s ending at a fade-out)
ffmpeg -i track.mp3 -ss 12 -t 48 -af "afade=t=in:d=2,afade=t=out:st=43:d=5" bed_48s.mp3

# Normalise (do this on the FINAL mix, not the bed alone)
ffmpeg -i mix.wav -af "loudnorm=I=-14:LRA=11:TP=-1.5" master.wav

# Waveform graphic
ffmpeg -i track.mp3 -filter_complex "showwavespic=s=1920x400:colors=#d97757|#ededed" wave.png
```

**Technique for hitting a specific emotional beat:** generate 2–3 takes, run the RMS scan on
each, and pick the take whose natural crescendo sits nearest your target — then slide the trim
window so it lands exactly. Selection plus trimming replaces the control the API does not offer.

---

## Error codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1002` | Rate limit — back off (free tier is 3 RPM = one per 20s) |
| `1004` | Authentication failed |
| `1008` | Insufficient balance |
| `1026` | Content flagged as sensitive |
| `1039` | Token limit |
| `2013` | Invalid parameters |
| `2049` | Invalid API key |
| `2056` | Usage limit exceeded — wait for the next 5-hour window |

**Rate limits:** paid models 120 RPM, 20 concurrent · free variants 3 RPM.

---

## Common failures and fixes

| Symptom | Cause | Fix |
|---------|-------|-----|
| Command killed at ~2 min | Generation takes ~3 min and blocks | Timeout ≥ 600s; run in background |
| Downloaded "audio" is a few hundred bytes | Signed URL was truncated | Never truncate the URL; `ffprobe` every download |
| Corrupt audio from hex decode | Response was a URL, not hex | Set `output_format` explicitly and branch |
| Vocals on an "instrumental" | Vocal tokens left in the prompt | Strip `vocals`, `harmonies`, `choir` |
| `400` on instrumental | `prompt` missing, or `lyrics: ""` sent | `prompt` is required here; omit `lyrics` entirely |
| Track too short/long for picture | No duration control exists | Generate, measure, trim + fade in ffmpeg |
| Crescendo in the wrong place | No timestamp control exists | Generate several takes, RMS-scan, pick and slide the trim window |
| `1002` on every other call | Free tier is 3 RPM | Sleep 20s between free calls |

---

## Supporting files

- [Complete API reference](references/api-reference.md) — every parameter, full schemas, error codes
- [Genre & style reference](references/genre-style-reference.md) — genre encyclopedia, BPM ranges, instrumentation
- [Lyrics writing guide](references/lyrics-guide.md) — meter, rhyme, storytelling, structure
