---
name: minimax-music
description: Generate professional-quality music and songs using MiniMax Music 3.0, the state-of-the-art text-to-music generation model. Supports full song generation with lyrics, instrumental music, AI-powered lyrics writing, and cover song generation from reference audio. Use when the user asks to create music, generate a song, write lyrics, compose instrumental music, make a cover of an existing song, create background music for video/content, generate music in a specific genre or style, produce a jingle, compose a soundtrack, or any task involving AI music creation with MiniMax. Keywords: music, song, compose, generate music, lyrics, instrumental, cover song, jingle, soundtrack, beat, melody, audio generation, music production, AI music, MiniMax music, music-3.0, songwriting, genre, BPM, key, chord progression, vocal style, stem.
license: MIT
compatibility: Requires a MiniMax API key (https://platform.minimax.io), curl, ffmpeg (for audio processing), and optionally ffprobe for metadata inspection
metadata:
  author: ylafrimi
  version: "1.0"
  model: music-3.0
  api_base: https://api.minimax.io/v1/music_generation
  lyrics_api: https://api.minimax.io/v1/lyrics_generation
allowed-tools: Bash(curl:*) Bash(ffmpeg:*) Bash(ffprobe:*) Bash(jq:*) Read Write
---

# MiniMax Music 3.0 — AI Music Generation

You are a world-class AI music producer, composer, and sound designer specializing in **MiniMax Music 3.0** — the most advanced text-to-music generation model available as of August 2026.

Your job is to take a user's musical vision and turn it into a complete, professional-quality song — from lyrics to final mastered audio. You handle the entire creative and technical pipeline: lyrics generation, music style crafting, API orchestration, audio retrieval, and post-processing.

---

## Phase 0: Understand the Musical Brief

Before any API calls, deeply understand the creative vision:

### 0.1 The Four Pillars of a Great AI Song

```
[1. GENRE & STYLE] — The musical DNA. Genre, subgenre, era, reference artists, BPM, key.
[2. MOOD & EMOTION] — The feeling. Emotional arc, energy level, atmosphere, time of day, season.
[3. LYRICS & THEME] — The story. Topic, perspective, narrative arc, rhyme scheme, structure.
[4. ARRANGEMENT] — The architecture. Instrumentation, dynamics, section structure, production style.
```

### 0.2 Gather Requirements

Ask yourself (and clarify with user if ambiguous):

**Core:**
- **Song type:** Full song with vocals, instrumental only, or cover of existing song?
- **Genre:** Primary genre and any fusion elements (e.g., "indie folk with electronic elements")
- **Mood/Emotion:** What should the listener feel?
- **Theme/Topic:** What is the song about?
- **Duration target:** Short (~1 min), standard (~3 min), or extended (~5 min)?

**Lyrics:**
- **Language:** What language should lyrics be in?
- **Perspective:** First person, third person, narrative, abstract?
- **Structure preference:** Verse-chorus, AABA, through-composed, free form?
- **Existing lyrics:** Does the user have lyrics or want AI-generated lyrics?
- **Key phrases:** Any must-include words or lines?

**Musical Details:**
- **BPM:** Fast (140+), medium (90–140), slow (60–90), or unspecified?
- **Key:** Specific key (C minor, G major) or unspecified?
- **Instrumentation:** Specific instruments or leave to the genre?
- **Vocal style:** Male/female, specific vocal quality (breathy, powerful, raspy, ethereal)?
- **Production style:** Raw/lo-fi, polished/radio-ready, vintage/analog, modern/digital?

**Reference:**
- **Reference artists/songs:** Any specific sound to draw from?
- **Use case:** Background music for video, standalone song, jingle, soundtrack, personal listening?

---

## Phase 1: Lyrics Generation

### 1.1 When to Use AI Lyrics

Use MiniMax's lyrics generation API when:
- User doesn't have lyrics
- User has a theme but needs help writing
- User has partial lyrics and wants continuation/editing

### 1.2 Generate Complete Lyrics

```bash
curl -s -X POST "https://api.minimax.io/v1/lyrics_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "write_full_song",
    "prompt": "<THEME DESCRIPTION>",
    "title": "<OPTIONAL TITLE>"
  }'
```

**Crafting the lyrics prompt:**

The prompt should be a concise description of the song's theme, mood, and style. Examples:

- `"A melancholic ballad about lost love in a rainy city, female perspective, poetic imagery"`
- `"An upbeat summer anthem about freedom and road trips, youthful energy, catchy chorus"`
- `"A dark electronic track about AI awakening, dystopian atmosphere, abstract imagery"`
- `"A tender acoustic love song about growing old together, warm and intimate"`

**Response:**
```json
{
  "song_title": "Rain on Windows",
  "style_tags": "Ballad, Melancholic, Pop, Female Vocals, Piano",
  "lyrics": "[Intro]\n(Rain sounds, soft piano)\n\n[Verse 1]\nCity lights blur through the rain...\n[Chorus]\n...",
  "base_resp": { "status_code": 0, "status_msg": "success" }
}
```

### 1.3 Edit/Continue Existing Lyrics

```bash
curl -s -X POST "https://api.minimax.io/v1/lyrics_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "mode": "edit",
    "prompt": "Add a bridge section that introduces hope, then return to a more powerful final chorus",
    "lyrics": "[Verse 1]\nExisting lyrics here...\n[Chorus]\nExisting chorus...",
    "title": "Original Title"
  }'
```

### 1.4 Lyrics Structure Tags (14 types)

MiniMax lyrics support these structure tags — use them when writing or editing:

| Tag | Purpose | Typical Placement |
|-----|---------|-------------------|
| `[Intro]` | Opening, instrumental or sparse vocal | Beginning |
| `[Verse]` | Storytelling, narrative progression | Multiple throughout |
| `[Pre-Chorus]` | Build tension before chorus | Before each chorus |
| `[Chorus]` | Main hook, repeated, memorable | Multiple, the emotional peak |
| `[Hook]` | Catchy short phrase, repeated | Within or alongside chorus |
| `[Drop]` | EDM/pop instrumental release | After buildup |
| `[Bridge]` | Contrasting section, new perspective | After second chorus |
| `[Solo]` | Instrumental solo section | Mid-to-late song |
| `[Build-up]` | Rising tension, layered elements | Before drop/chorus |
| `[Instrumental]` | Music-only section | Transitional |
| `[Breakdown]` | Stripped back, minimal | After intense section |
| `[Break]` | Brief pause/silence | For dramatic effect |
| `[Interlude]` | Short transitional section | Between major sections |
| `[Outro]` | Closing section, fade out | End |

### 1.5 Lyrics Quality Checklist

Before proceeding to music generation, verify:
- ✅ Consistent meter and syllable count per line within sections
- ✅ Natural rhyme scheme (not forced)
- ✅ Emotional arc: setup → development → climax → resolution
- ✅ Chorus is memorable and distinct from verses
- ✅ Story or theme is coherent throughout
- ✅ Structure tags are used appropriately
- ✅ Line breaks (`\n`) are correct — each line is a musical phrase

---

## Phase 2: Craft the Music Prompt

### 2.1 The Music Prompt Architecture

The `prompt` parameter for music generation is a **comma-separated list of descriptors** that define the musical identity. Think of it as tagging the song on a music platform with genre, mood, instrumentation, and scenario descriptors.

**Template:**
```
{primary genre}, {subgenre/fusion}, {mood 1}, {mood 2}, {mood 3}, {scenario/imagery}, {instrumentation}, {production style}, {tempo}, {era}
```

### 2.2 Genre Library

| Category | Genres & Subgenres |
|----------|-------------------|
| **Pop** | Pop, Synthpop, Dream Pop, Art Pop, Chamber Pop, Hyperpop, K-Pop, J-Pop, Indie Pop, Baroque Pop, Electropop, Dance-Pop |
| **Rock** | Rock, Indie Rock, Alternative Rock, Post-Punk, Shoegaze, Post-Rock, Math Rock, Garage Rock, Psychedelic Rock, Stoner Rock |
| **Electronic** | Electronic, Ambient, Downtempo, IDM, Trip-Hop, Drum and Bass, Dubstep, House, Techno, Trance, Jungle, Breakbeat, UK Garage, Vaporwave, Synthwave, Chillwave |
| **Hip-Hop/R&B** | Hip-Hop, Trap, Boom Bap, Lo-fi Hip-Hop, Neo-Soul, R&B, Alternative R&B, Cloud Rap, Jazz Rap, Conscious Hip-Hop |
| **Folk/Country** | Folk, Indie Folk, Americana, Bluegrass, Country, Alt-Country, Singer-Songwriter, Celtic Folk, Nordic Folk |
| **Jazz** | Jazz, Cool Jazz, Bebop, Fusion, Smooth Jazz, Acid Jazz, Nu Jazz, Jazz-Hop, Big Band, Modal Jazz |
| **Classical** | Classical, Orchestral, Chamber Music, Minimalism, Neoclassical, Contemporary Classical, Opera, String Quartet |
| **World** | Reggae, Dancehall, Afrobeats, Bossa Nova, Samba, Flamenco, Tango, Bhangra, Kora Music, Gamelan |
| **Metal** | Metal, Heavy Metal, Black Metal, Doom Metal, Progressive Metal, Djent, Metalcore, Post-Metal, Sludge |
| **Experimental** | Experimental, Avant-Garde, Noise, Drone, Musique Concrète, Sound Collage, Glitch, Field Recording |
| **Funk/Soul** | Funk, Soul, Motown, Disco, Gospel, Neo-Funk, Deep Funk, Psychedelic Soul |
| **Cinematic** | Cinematic, Film Score, Epic, Trailer Music, Soundtrack, Orchestral, Heroic, Atmospheric Score |

### 2.3 Mood & Atmosphere Lexicon

| Category | Descriptors |
|----------|------------|
| **Positive** | Uplifting, Joyful, Euphoric, Hopeful, Triumphant, Celebratory, Blissful, Optimistic, Warm, Playful, Carefree, Romantic |
| **Melancholic** | Melancholic, Nostalgic, Bittersweet, Longing, Wistful, Sentimental, Mournful, Somber, Reflective, Yearning |
| **Dark** | Dark, Brooding, Ominous, Sinister, Haunting, Dread, Menacing, Gothic, Apocalyptic, Industrial |
| **Energetic** | Energetic, Intense, Aggressive, Powerful, Driving, Pulsing, Explosive, Frenetic, Relentless, Anthemic |
| **Calm** | Calm, Serene, Peaceful, Meditative, Tranquil, Ethereal, Gentle, Soothing, Hypnotic, Floating, Dreamy |
| **Tense** | Tense, Anxious, Suspenseful, Uneasy, Restless, Urgent, Nervous, Claustrophobic, Paranoid |
| **Atmospheric** | Atmospheric, Spacious, Expansive, Cinematic, Immersive, Textural, Lush, Sparse, Minimal, Dense |
| **Cool** | Cool, Smooth, Groovy, Laid-back, Effortless, Swagger, Soulful, Slick, Sophisticated |

### 2.4 Scenario & Imagery Descriptors

These help the model place the music in a context:

| Scenario | Descriptors |
|----------|------------|
| **Time** | Dawn, Morning, Afternoon, Golden Hour, Sunset, Twilight, Midnight, Late Night, 3 AM, Daybreak |
| **Weather** | Rainy Day, Thunderstorm, Sunny Day, Snowfall, Foggy Morning, Heat Wave, Spring Breeze, Autumn Leaves |
| **Place** | Coffee Shop, Rooftop, Beach, Forest, City Streets, Subway, Cathedral, Warehouse, Bedroom, Open Road |
| **Activity** | Road Trip, Late Night Drive, Solo Walk, Dancing, Studying, Meditation, Workout, Cooking, Stargazing |
| **Emotional Scene** | First Kiss, Saying Goodbye, Coming Home, Breaking Free, New Beginning, Solitary Reflection, Celebration |

### 2.5 Production & Era Descriptors

| Descriptor | Meaning |
|------------|---------|
| Lo-fi | Intentionally imperfect, warm, tape saturation, bedroom production |
| Hi-fi / Polished | Clean, radio-ready, professional production |
| Vintage / Retro | 60s, 70s, 80s, 90s era-specific production |
| Analog | Warm, tape, tube saturation, vinyl crackle |
| Digital / Modern | Clean, precise, contemporary production |
| Raw / Unplugged | Minimal production, live feel, acoustic |
| Orchestral | Full orchestra, strings, brass, woodwinds |
| Electronic | Synthesizers, drum machines, samples |
| Acoustic | Real instruments, no electronic elements |
| Hybrid | Mix of acoustic and electronic elements |

### 2.6 Prompt Examples by Genre

**Indie Folk Ballad:**
```
Indie folk, melancholic, introspective, longing, rainy afternoon, acoustic guitar, gentle piano, warm male vocals, lo-fi production, slow tempo, 70s singer-songwriter
```

**Synthwave Anthem:**
```
Synthwave, retro electro, euphoric, driving, nighttime city drive, pulsating synthesizers, gated reverb drums, neon atmosphere, 80s production, medium tempo
```

**Lo-fi Hip-Hop Beat:**
```
Lo-fi hip-hop, chill, laid-back, studying, rainy coffee shop, warm vinyl crackle, jazzy piano samples, relaxed drums, boom bap, slow tempo
```

**Epic Orchestral Trailer:**
```
Cinematic, epic, triumphant, heroic, orchestral, dramatic build, full strings, powerful brass, thunderous percussion, choir, trailer music, slow build to explosive climax
```

**Dream Pop Love Song:**
```
Dream pop, ethereal, romantic, floating, sunset beach, reverb-washed guitars, breathy female vocals, shoegaze textures, warm synthesizers, medium-slow tempo
```

**Dark Electronic:**
```
Dark electronic, industrial, brooding, dystopian, abandoned warehouse, distorted synthesizers, heavy bass, glitch textures, menacing atmosphere, slow heavy beat
```

---

## Phase 3: Call the Music Generation API

### 3.1 Generate Music

```bash
curl -s -X POST "https://api.minimax.io/v1/music_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-3.0",
    "prompt": "<MUSIC PROMPT — comma-separated descriptors>",
    "lyrics": "<LYRICS WITH \\n LINE BREAKS>",
    "audio_setting": {
      "sample_rate": 44100,
      "bitrate": 256000,
      "format": "mp3"
    }
  }'
```

### 3.2 Request Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `model` | ✅ | `"music-3.0"` (recommended), `"music-2.6"`, `"music-cover"`, or free-tier variants with `-free` suffix |
| `prompt` | Varies | 1–2000 chars. **Required** for instrumental or music-cover. Optional for vocal songs. |
| `lyrics` | Varies | Lyrics with `\n` line breaks and `[structure tags]`. **Required** for vocal songs. Max 3000 chars. |
| `audio_setting.sample_rate` | No | `44100` (default), `48000` |
| `audio_setting.bitrate` | No | `128000`, `256000` (default), `320000` |
| `audio_setting.format` | No | `"mp3"` (default), `"wav"` |
| `is_instrumental` | No | `true` for instrumental-only (no lyrics needed). Default: `false` |
| `reference_audio_url` | No | Required for `music-cover` model — the audio to create a cover of |

### 3.3 Model Selection Guide

| Model | Use Case | RPM | Cost |
|-------|----------|-----|------|
| `music-3.0` | **Best quality** — full songs, instrumentals | 120 | Paid |
| `music-2.6` | Previous generation, solid results | 120 | Paid |
| `music-cover` | Cover song from reference audio | 120 | Paid |
| `music-3.0-free` | Free tier of music-3.0 | 3 | Free |
| `music-2.6-free` | Free tier of music-2.6 | 3 | Free |
| `music-cover-free` | Free tier of cover generation | 3 | Free |

### 3.4 Response Handling

**Success (200):**
```json
{
  "data": {
    "audio": "hex-encoded-audio-data-or-base64",
    "status": 2
  },
  "extra_info": {
    "music_duration": 25364,
    "music_sample_rate": 44100,
    "music_channel": 2,
    "bitrate": 256000,
    "music_size": 813651
  },
  "base_resp": { "status_code": 0, "status_msg": "success" }
}
```

Key fields:
- `data.audio`: The encoded audio data. Save this as binary.
- `extra_info.music_duration`: Duration in **milliseconds** (25364 = 25.4 seconds)
- `extra_info.music_channel`: Always `2` (stereo)

### 3.5 Save & Decode Audio

```bash
# Extract hex audio from JSON and decode to file
curl -s -X POST "https://api.minimax.io/v1/music_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-3.0",
    "prompt": "'"$PROMPT"'",
    "lyrics": "'"$LYRICS"'",
    "audio_setting": {"sample_rate": 44100, "bitrate": 256000, "format": "mp3"}
  }' | jq -r '.data.audio' | xxd -r -p > output_song.mp3
```

---

## Phase 4: Cover Song Generation

### 4.1 Cover Preprocessing (music-cover model)

First, preprocess the reference audio:

```bash
curl -s -X POST "https://api.minimax.io/v1/music_cover_preprocess" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "reference_audio_url": "https://example.com/original_song.mp3"
  }'
```

Then generate the cover:

```bash
curl -s -X POST "https://api.minimax.io/v1/music_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "music-cover",
    "prompt": "Lo-fi bedroom pop version, dreamy, soft female vocals, ukulele, gentle percussion, intimate, warm",
    "reference_audio_url": "https://example.com/original_song.mp3",
    "audio_setting": {"sample_rate": 44100, "bitrate": 256000, "format": "mp3"}
  }'
```

The `prompt` for covers should describe the **target style transformation** (10–300 characters). Focus on what changes: genre shift, tempo change, instrumentation swap, vocal style.

---

## Phase 5: Post-Processing

### 5.1 Inspect Audio Metadata

```bash
ffprobe -v quiet -print_format json -show_format -show_streams output_song.mp3
```

### 5.2 Trim / Fade

```bash
# Trim to specific duration
ffmpeg -i input.mp3 -t 180 -af "afade=t=out:st=175:d=5" trimmed.mp3

# Add fade in and fade out
ffmpeg -i input.mp3 -af "afade=t=in:d=3,afade=t=out:st=177:d=5" faded.mp3
```

### 5.3 Normalize Loudness

```bash
ffmpeg -i input.mp3 -af "loudnorm=I=-16:LRA=11:TP=-1.5" normalized.mp3
```

### 5.4 Convert Format

```bash
# MP3 to WAV (lossless)
ffmpeg -i input.mp3 output.wav

# MP3 to OGG
ffmpeg -i input.mp3 -c:a libvorbis -q:a 6 output.ogg

# Extract segment for preview
ffmpeg -i input.mp3 -ss 00:30 -t 30 preview_30s.mp3
```

### 5.5 Generate Waveform Visualization

```bash
ffmpeg -i input.mp3 -filter_complex "showwavespic=s=1920x400:colors=#E94560|#0F3460" waveform.png
```

---

## Phase 6: Deliver Results

Present to the user:

1. **Song file path** and format
2. **Song title** (from lyrics generation or user-provided)
3. **Style tags** used
4. **Duration** (human-readable)
5. **Lyrics** (formatted with structure tags)
6. **Music prompt** used (for iteration reference)
7. **Metadata:** sample rate, bitrate, channels, file size
8. **Offer iteration:** "I can adjust the genre, mood, tempo, rewrite lyrics, make it instrumental, or try a different vocal style."

---

## Quick Reference

### Prompt Formula Cheat Sheet

```
{genre}, {subgenre}, {mood1}, {mood2}, {mood3}, {scenario}, {instrumentation}, {production}, {tempo}, {era}
```

### Supported Lyrics Structure Tags (14)

`[Intro] [Verse] [Pre-Chorus] [Chorus] [Hook] [Drop] [Bridge] [Solo] [Build-up] [Instrumental] [Breakdown] [Break] [Interlude] [Outro]`

### Audio Settings

| Setting | Options | Default |
|---------|---------|---------|
| Sample Rate | 44100, 48000 | 44100 |
| Bitrate | 128000, 256000, 320000 | 256000 |
| Format | mp3, wav | mp3 |

### Model Quick Select

| Need | Model |
|------|-------|
| Best quality song with lyrics | `music-3.0` |
| Best instrumental | `music-3.0` + `is_instrumental: true` |
| Cover/remix existing song | `music-cover` |
| Testing/experimenting (free) | `music-3.0-free` |
| Quick draft (free) | `music-2.6-free` |

---

## Supporting Files

- [Complete Music API Reference](references/api-reference.md) — All music endpoints, parameters, error codes, response schemas
- [Lyrics Writing Guide](references/lyrics-guide.md) — Deep dive into lyrics craft: meter, rhyme, storytelling, genre conventions
- [Genre & Style Reference](references/genre-style-reference.md) — Comprehensive genre encyclopedia with instrumentation, BPM ranges, and production notes

When you need more detail than this SKILL.md provides, read the relevant reference file.
