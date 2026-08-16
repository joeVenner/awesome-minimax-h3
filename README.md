# 🎬🎙️🎵 Awesome MiniMax H3

**Production-ready [Agent Skills](https://www.anthropic.com/news/agent-skills) that turn any AI agent into a professional video director, voice director, and music producer — powered by [MiniMax H3](https://platform.minimax.io), Speech 2.8 and Music 3.0.**

Drop these into your agent's skills directory and it instantly knows how to gather creative requirements, engineer world-class prompts, orchestrate the MiniMax API end-to-end (submit → poll → download → post-process), and iterate like a working creative professional — not just fire off a raw API call.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Format: Agent Skills](https://img.shields.io/badge/format-Agent%20Skills-6366f1)](#-what-are-agent-skills)
[![Model: MiniMax H3](https://img.shields.io/badge/model-H3%20%C2%B7%20Speech%202.8%20%C2%B7%20Music%203.0-0ea5e9)](https://platform.minimax.io)

---

## 📦 What's inside

| Skill | Directory | Does what |
|---|---|---|
| 🎬 **minimax-video** | [`minimax-video/`](minimax-video) | Text-to-video, image-to-video (first/last frame), and multimodal reference-to-video generation with MiniMax H3 — up to 15s at 2K with native stereo audio |
| 🎙️ **minimax-speech** | [`minimax-speech/`](minimax-speech) | Voiceover, narration and audiobooks with Speech 2.8 — 332 preset voices in 40+ languages, voice cloning, voice design, word-level timestamps |
| 🎵 **minimax-music** | [`minimax-music/`](minimax-music) | Full song generation (lyrics + music), instrumental composition, AI lyric writing, and cover-song generation with MiniMax Music 3.0 |

Each skill is a self-contained package: a `SKILL.md` playbook (creative brief → prompt engineering → API calls → polling → delivery → iteration) plus deep-dive `references/` docs the agent pulls in on demand.

```
awesome-minimax-h3/
├── minimax-video/
│   ├── SKILL.md                          # Full video generation playbook
│   └── references/
│       ├── h3-technical-reference.md     # Architecture & multimodal context
│       ├── api-reference.md              # Endpoints, params, error codes
│       └── prompt-engineering.md         # Advanced prompting techniques
├── minimax-speech/
│   ├── SKILL.md                          # Full TTS / voiceover playbook
│   └── references/
│       ├── api-reference.md              # Sync, async, streaming, cloning endpoints
│       └── voice-catalog.md              # All 332 preset voices by language
└── minimax-music/
    ├── SKILL.md                          # Full music generation playbook
    └── references/
        ├── api-reference.md              # Endpoints, params, response schemas
        ├── lyrics-guide.md               # Meter, rhyme, structure, storytelling
        └── genre-style-reference.md      # Genre encyclopedia, BPM, instrumentation
```

---

## ✨ Why these skills

MiniMax exposes a raw HTTP API. These skills wrap it with the *craft* of using it well:

- **Verified against the live API** — endpoints, required parameters, status values and response schemas were confirmed by probing `api.minimax.io` directly, not copied from docs. Where the published docs are wrong (the video polling URL, for one), the skills say so and give the working call.
- **Structured creative intake** — the agent asks the right clarifying questions (scenario, resolution, duration, mood, voice, genre, structure) before touching the API, instead of guessing.
- **Expert prompt engineering** — dedicated frameworks baked in as playbooks: scene/action/audio/context layers for video, voice/prosody/pause/timestamp control for speech, genre/mood/lyrics/arrangement pillars for music.
- **Full lifecycle handling, per API** — video is async (submit → poll with backoff → resolve `file_id` → download); speech and music are synchronous, and music blocks for ~3 minutes, so the skill tells the agent to raise its timeout past the default instead of killing the call mid-flight. Each skill carries its own error table — HTTP codes for video, MiniMax internal codes (`1002`, `1026`, `2056`, …) for music and speech — plus post-processing (ffmpeg trimming, fades, loudness normalization, thumbnails, waveform art).
- **Iteration built in** — each skill ends by offering concrete refinement paths (motion too subtle, text rendering off, read too rushed, chorus not catchy, etc.) instead of a dead end.
- **Zero lock-in** — plain Markdown + YAML frontmatter. No SDK, no plugin runtime required.

---

## 🎯 The three together

The skills are designed to compose. A typical branded 15-second spot, end to end:

```
minimax-speech  →  narration.wav   (per-line renders + word-level timestamps)
minimax-music   →  bed.wav         (instrumental underscore, loudness-normalized)
minimax-video   →  clip.mp4        (2K, H3, cut to the narration timing)
                        ↓
              ffmpeg mux → spot.mp4
```

Because `minimax-speech` returns word-accurate timestamps (in milliseconds), the agent can cut
picture to the read rather than guessing at durations — that's the join that makes the suite
more than three separate generators. H3 generates its own stereo track, so at the mux step you
either duck it under the narration or drop it with `-an` and rebuild the bed from `minimax-music`.

---

## 🧩 What are Agent Skills?

An [Agent Skill](https://www.anthropic.com/news/agent-skills) is a folder containing a `SKILL.md` file: YAML frontmatter (name, description, tool permissions) followed by instructions the agent reads and follows like an expert playbook, loading the `references/` files only when it needs more depth. It's a portable, model-agnostic format — **any agent harness that can read a markdown file and shell out to `curl` can use these skills**, including:

- **[Claude Code](https://claude.com/claude-code)** — native support, see setup below
- **Claude in Slack, the Claude Agent SDK, or any custom agent built on the Anthropic API** — load `SKILL.md` as part of the system prompt / tool instructions
- **Cursor, Windsurf, and other IDE agents** — reference the `SKILL.md` path in your rules/instructions file
- **OpenAI-based agents, LangChain, AutoGPT-style runners, or any custom harness** — paste or inject the `SKILL.md` contents into the agent's system prompt; the bash/curl workflow works with any tool-calling agent that has shell access

There's no proprietary runtime to install — it's just well-structured instructions.

---

## 🚀 Setup

### 1. Get a MiniMax API key

Sign up at [platform.minimax.io](https://platform.minimax.io) and export your key:

```bash
export MINIMAX_API_KEY="your-api-key-here"
```

All three skills authenticate the same way: `Authorization: Bearer $MINIMAX_API_KEY` against
`https://api.minimax.io`. No `GroupId` and no `token:` header.

### 2. Install the requirements

All three skills shell out to standard CLI tools:

```bash
# macOS
brew install curl jq ffmpeg

# Debian/Ubuntu
sudo apt-get install curl jq ffmpeg
```

### 3. Install the skills into your agent

**Claude Code — project-level** (available only in this repo/project):

```bash
mkdir -p /path/to/your-project/.claude/skills
cp -r minimax-video minimax-speech minimax-music /path/to/your-project/.claude/skills/
```

**Claude Code — user-level** (available in every project, everywhere):

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/joeVenner/awesome-minimax-h3.git /tmp/awesome-minimax-h3
cp -r /tmp/awesome-minimax-h3/minimax-{video,speech,music} ~/.claude/skills/
```

Claude Code auto-discovers any folder under `.claude/skills/` (or `~/.claude/skills/`) containing a `SKILL.md` — no restart or registration step needed. It invokes a skill automatically when your request matches its description:

| You say | It runs |
|---|---|
| *"make me a 10 second product video"* | `minimax-video` |
| *"narrate this script in a warm documentary voice"* | `minimax-speech` |
| *"write me a synthwave song"* | `minimax-music` |

**Any other agent harness:**

1. Clone this repo (or just download the skill folder you need).
2. Point your agent at the `SKILL.md` file — either by adding its path to your system prompt / rules file, or by pasting its contents directly into your agent's instructions.
3. Make sure the agent has shell access to `curl`, `jq`, and `ffmpeg`, and that `$MINIMAX_API_KEY` is set in its environment.
4. The agent will follow the phased playbook (brief → prompt → API call → poll → deliver → iterate) exactly as written, pulling in `references/*.md` files as needed for deeper detail.

---

## 🎬 minimax-video highlights

- **Scenarios:** text-to-video, image-to-video (first/last/both frames), multimodal reference-to-video (blend reference images, video clips, and audio)
- **Up to 15s at 2K resolution** with in-context super-resolution for crisp text and fine detail
- **Native stereo audio** generated jointly with the video
- **Precise text/brand rendering** and motion transfer from reference clips
- **Aspect ratios:** `adaptive`, 16:9, 9:16, 1:1, 4:3, 3:4, 21:9
- **Live-verified orchestration:** poll `GET /v1/query/video_generation` (the documented `/v2/…/query` form 404s), then resolve the returned `file_id` via `/v1/files/retrieve` — `duration`, `resolution` and `ratio` are all required, and there is no seed parameter, so never discard a good take
- Genre-specific prompt boosters for product commercials, title sequences, UI/UX demos, gaming cinematics, e-commerce, and more

→ [`minimax-video/SKILL.md`](minimax-video/SKILL.md)

## 🎙️ minimax-speech highlights

- **332 preset voices across 40+ languages**, catalogued by language and intent — plus **voice cloning** from a sample and **voice design** from a written description
- **Synchronous generation** — audio comes back in one call; async endpoint available for long-form up to 1M characters
- **Word- and sentence-level timestamps** (`subtitle_enable`) in milliseconds, for cutting picture to narration or animating captions
- **Inline pause control** (`<#0.45#>`) and `speech-2.8` interjection tags — `(breath)`, `(laughs)`, `(sighs)` — the entire inline control surface, since there is no SSML
- **Prosody control:** speed, volume, pitch, emotion, `pronunciation_dict` for brand names and acronyms, `voice_modify` for timbre and effects
- Per-line rendering technique for voiceover against picture, so every line can be re-taken independently

→ [`minimax-speech/SKILL.md`](minimax-speech/SKILL.md)

## 🎵 minimax-music highlights

- **Full song generation** — AI-written or user-supplied lyrics, complete with 14 structure tags (`[Intro]`, `[Verse]`, `[Pre Chorus]`, `[Chorus]`, `[Bridge]`, `[Outro]`, …)
- **Instrumental composition** across a genre library spanning Pop, Rock, Electronic, Hip-Hop, Jazz, Classical, World, Metal, Cinematic, and more
- **Cover-song generation** from a reference audio track, with a free preprocessing step for rewriting lyrics
- **Six models**, not two — `music-3.0` flagship plus `-free` variants (3 RPM) for auditioning prompts before you spend on the winner
- Post-processing playbook: trim/fade, loudness normalization, format conversion, waveform art

→ [`minimax-music/SKILL.md`](minimax-music/SKILL.md)

---

## 🤝 Contributing

Issues and PRs welcome — new genre libraries, additional voice notes, extra reference docs, and improvements to the prompt-engineering playbooks are all fair game. Please keep changes scoped to a single skill/topic per PR.

## 📄 License

[MIT](LICENSE) — use these skills freely in personal or commercial agent projects.
