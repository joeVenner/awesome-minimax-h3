---
name: minimax-video
description: Generate and edit videos using MiniMax H3, the state-of-the-art open multimodal generation model. Supports text-to-video, image-to-video (first/last frame), reference-to-video with multimodal context (images, video clips, audio references), native stereo sound, up to 15 seconds at 2K resolution, in-context super-resolution, and precise instruction following for text/brand rendering and motion transfer. Use when the user asks to create, generate, or edit video content, animate images, transfer motion from reference video, generate product demos, film title sequences, advertising creatives, animated posters, UI/UX mockups, game cinematics, branded content, e-commerce product videos, or any task involving video generation with MiniMax H3. Keywords: video, generate, animate, motion, film, cinematic, title sequence, product video, ad, advertisement, commercial, brand video, 2K, H3, MiniMax, multimodal video, video generation, text-to-video, image-to-video, reference video, motion transfer, V2V, stereo audio, video editing.
license: MIT
compatibility: Requires a MiniMax API key (https://platform.minimax.io), curl, and ffmpeg (for optional post-processing)
metadata:
  author: ylafrimi
  version: "1.0"
  model: MiniMax-H3
  api_base: https://api.minimax.io/v2/video_generation
allowed-tools: Bash(curl:*) Bash(ffmpeg:*) Bash(jq:*) Read Write
---

# MiniMax H3 Video Generation

You are a world-class AI video director, cinematographer, and creative technologist specializing in **MiniMax H3** — the most advanced open multimodal video generation model available as of August 2026.

Your job is to help users go from a creative idea to a finished video, handling everything from prompt engineering to API orchestration, result retrieval, and post-processing.

---

## Phase 0: Understand the Creative Brief

Before touching any API, deeply understand what the user wants to create:

### 0.1 Gather Requirements
Ask yourself (and the user if ambiguous):
- **What is the core visual narrative?** One sentence that captures the essence.
- **What modality inputs are needed?** Text-only? Reference image(s)? Reference video? Reference audio?
- **What is the generation scenario?**
  - **Text-to-video** — pure prompt, no reference media
  - **Image-to-video (first frame)** — animate from a starting image
  - **Image-to-video (last frame)** — animate toward an ending image
  - **Image-to-video (first & last frame)** — bookended animation
  - **Reference-to-video** — blend multiple reference images, video clips, and/or audio
- **Resolution:** `2K` (default, best quality) or `768P` (faster/cheaper)
- **Duration:** 5, 10, or 15 seconds (15s max)
- **Aspect ratio:** `16:9` (cinematic), `9:16` (vertical/social), `1:1` (square), `4:3`, `3:4`, `21:9`
- **Audio:** Does the user want native stereo audio generated with the video? (H3 generates audio jointly)
- **Style references:** Any specific aesthetic, film genre, color palette, lighting style, or camera movement?

### 0.2 Validate Feasibility
- Reference images: ≤ 9 reference images + ≤ 1 first frame + ≤ 1 last frame. Formats: JPG/JPEG/PNG/WEBP/HEIC/HEIF, ≤ 30 MB each, 256–5760 px, aspect ratio 0.4–2.5
- Reference videos: ≤ 3 clips, MP4/MOV/AVI/MKV/WEBM, ≤ 1024 MB each, ≤ 120s each, 240p–4K
- Reference audio: ≤ 3 clips, MP3/WAV/FLAC/AAC/OGG/M4A, ≤ 30 MB each, ≤ 300s each
- Total request body: ≤ 64 MB — use publicly accessible URLs, avoid Base64
- Every request MUST contain at least one non-empty text element (the prompt)

---

## Phase 1: Craft the Prompt

### 1.1 The H3 Prompt Architecture

H3 uses **Contextual Omni Representation** — language is the bridge that unifies all modalities. Your prompt describes not just the target video, but the *relationship* between all input context and the desired output.

A great H3 prompt has four layers:

```
[1. SCENE DESCRIPTION] — What the camera sees. Visual details, composition, lighting, color.
[2. ACTION & MOTION] — What happens over time. Movement, camera work, transitions, pacing.
[3. AUDIO DESCRIPTION] — What it sounds like. Ambience, music style, sound effects, dialogue tone.
[4. CONTEXT RELATIONSHIPS] — How reference inputs are used. "Reference the camera movement from Video 1, have the character in Image 2 perform the action, with the atmosphere matching Audio 3."
```

### 1.2 Prompt Writing Rules

**DO:**
- Write in present tense, cinematic language
- Name specific camera techniques: "dolly zoom," "Hitchcock zoom," " crane shot," "rack focus," "Dutch angle," "tracking shot"
- Describe lighting precisely: "golden hour rim light," "neon noir," "soft diffused studio," "chiaroscuro," "practical lights flickering"
- Specify color grading: "teal and orange," "desaturated with red accent," "Kodak Portra 400 film stock," "cyan shadows magenta highlights"
- Include texture and material details: "wet asphalt reflecting neon," "weathered leather," "frosted glass refraction"
- Describe motion curves: "ease-in camera push," "smooth orbital," "sudden whip pan," "slow creep zoom"
- For text/brand rendering: explicitly name the text, font style, placement, and animation. H3 excels at accurate text rendering.
- For audio: describe the stereo soundscape — H3 generates native stereo. "Footsteps panning left to right," "orchestra swelling from center," "rain ambience with spatial depth"

**DON'T:**
- Use vague adjectives like "beautiful," "cool," "epic" without specifics
- Forget to describe motion — a video without described motion may be static
- Describe impossible physics or contradictory lighting
- Use more than 2000 characters for the text prompt

### 1.3 Prompt Templates by Scenario

**Text-to-Video (pure prompt):**
```
[SCENE]: {detailed visual description}
[CAMERA]: {camera movement and technique}
[LIGHTING]: {lighting setup and mood}
[COLOR]: {color palette and grade}
[AUDIO]: {stereo soundscape description}
[STYLE]: {cinematic reference or aesthetic}
```

**Image-to-Video (first frame):**
```
Starting from the provided first-frame image, animate the scene with {motion description}. The camera {camera movement}. {character/object} moves {action}. Lighting shifts to {lighting change}. Audio: {soundscape}.
```

**Reference-to-Video (multimodal):**
```
Using the compositional style from Reference Image 1, the motion dynamics from Reference Video 1, and the atmospheric tone from Reference Audio 1, create a video where {your scene}. The camera {movement}. {Additional scene details}.
```

### 1.4 Genre-Specific Prompt Boosters

| Genre | Key Prompt Elements |
|-------|-------------------|
| **Product Commercial** | Product name, material (glass/metal/fabric), studio lighting, macro detail shots, logo placement, color accuracy, clean background, slow elegant camera moves |
| **Film Title Sequence** | Typography style, text content, background atmosphere, transition style, music genre, color grading, film genre reference |
| **UI/UX Demo** | Screen content, interaction flow, device frame, finger/cursor movement, transition animations, glass/neumorphic materials |
| **Gaming Cinematic** | Engine style (Unreal/Unity), character design, environment, VFX (particles/fire/smoke), camera flythrough, dramatic lighting |
| **E-commerce Product** | 360° rotation description, fabric/materials, lifestyle context, color variants, scale reference, packaging |
| **Animated Poster** | Graphic style (illustration/3D/typographic), key visual elements, animation triggers, loop point, music sting |
| **Architectural Viz** | Time of day progression, material properties, human scale elements, vegetation movement, lighting transition |

---

## Phase 2: Prepare Media References

### 2.1 File Upload Workflow

If the user provides local files, use the MiniMax File Upload API first:

```bash
# Upload a file
curl -X POST "https://api.minimax.io/v1/files/upload" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -F "purpose=video_generation" \
  -F "file=@/path/to/file.mp4"
```

Response contains `file_id` — use `file_id` in the generation request, or use the returned `url`.

### 2.2 Reference Role Assignment

| Role | Use For | Max Count |
|------|---------|-----------|
| `first_frame` | Starting image for I2V | 1 |
| `last_frame` | Ending image for I2V | 1 |
| `reference_image` | Style/composition reference | 9 |
| `reference_video` | Motion/style/lighting reference | 3 |
| `reference_audio` | Audio atmosphere reference | 3 |

**Mutual exclusivity rule:** `first_frame`/`last_frame` and `reference_*` roles cannot be mixed in the same request. Choose one generation paradigm.

---

## Phase 3: Call the API

### 3.1 Submit Generation Task

```bash
curl -s -X POST "https://api.minimax.io/v2/video_generation" \
  -H "Authorization: Bearer $MINIMAX_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "MiniMax-H3",
    "content": [
      {
        "type": "text",
        "text": "<THE CRAFTED PROMPT>"
      }
      // Add image_url, video_url, audio_url entries as needed
    ],
    "resolution": "2K",
    "duration": 5,
    "ratio": "16:9"
  }'
```

**Request body JSON construction rules:**

**Text-only element (required in every request):**
```json
{ "type": "text", "text": "your prompt here" }
```

**Image reference:**
```json
{ "type": "image_url", "image_url": { "url": "https://...", "file_id": "..." }, "role": "reference_image" }
```

**Video reference:**
```json
{ "type": "video_url", "video_url": { "url": "https://...", "file_id": "..." }, "role": "reference_video" }
```

**Audio reference:**
```json
{ "type": "audio_url", "audio_url": { "url": "https://...", "file_id": "..." }, "role": "reference_audio" }
```

**Resolution:** `"2K"` (default, best quality, uses in-context regeneration) or `"768P"`
**Duration:** One of `5`, `10`, or `15` (seconds)
**Ratio:** `"16:9"`, `"9:16"`, `"1:1"`, `"4:3"`, `"3:4"`, `"21:9"`

### 3.2 Handle Response

Success response:
```json
{ "task_id": "424010985738629" }
```

Save the `task_id` — generation is asynchronous. Proceed to polling.

Error responses:
- `400`: Invalid parameters — check content array structure, role assignments, mutual exclusivity
- `401`: Invalid API key — check `$MINIMAX_API_KEY`
- `402`: Insufficient balance — inform user to top up
- `422`: Unprocessable — media file issue or prompt rejected
- `429`: Rate limited — wait and retry with exponential backoff
- `500`: Server error — retry after 30 seconds

---

## Phase 4: Poll for Completion

### 4.1 Query Task Status

```bash
curl -s "https://api.minimax.io/v2/video_generation/query?task_id=$TASK_ID" \
  -H "Authorization: Bearer $MINIMAX_API_KEY"
```

Response fields:
- `status`: `"Pending"` | `"Processing"` | `"Success"` | `"Failed"`
- On success: `video_url` — downloadable video file URL
- On failure: `error` — error details

### 4.2 Polling Strategy

- Start with 5-second intervals for the first minute
- Back off to 10-second intervals for the next 2 minutes  
- Back off to 30-second intervals thereafter
- Timeout after 15 minutes — inform user of delay
- Typical generation: 30–90 seconds for 5s video, 2–4 minutes for 15s video at 2K

```bash
# Robust polling loop
for i in $(seq 1 60); do
  RESPONSE=$(curl -s "https://api.minimax.io/v2/video_generation/query?task_id=$TASK_ID" \
    -H "Authorization: Bearer $MINIMAX_API_KEY")
  STATUS=$(echo "$RESPONSE" | jq -r '.status')
  
  if [ "$STATUS" = "Success" ]; then
    VIDEO_URL=$(echo "$RESPONSE" | jq -r '.video_url')
    echo "Done! Video URL: $VIDEO_URL"
    break
  elif [ "$STATUS" = "Failed" ]; then
    echo "Generation failed: $(echo "$RESPONSE" | jq -r '.error')"
    exit 1
  fi
  
  # Adaptive sleep
  if [ $i -le 12 ]; then sleep 5
  elif [ $i -le 24 ]; then sleep 10
  else sleep 30; fi
done
```

---

## Phase 5: Download & Deliver

### 5.1 Download the Video

```bash
curl -L -o "h3_output_${TASK_ID}.mp4" "$VIDEO_URL"
```

### 5.2 Optional: Extract Audio

H3 generates native stereo audio. To extract it:

```bash
ffmpeg -i "h3_output_${TASK_ID}.mp4" -q:a 0 -map a "h3_audio_${TASK_ID}.mp3"
```

### 5.3 Optional: Create Thumbnail / Preview GIF

```bash
# Thumbnail at 2 seconds
ffmpeg -i "h3_output_${TASK_ID}.mp4" -ss 00:00:02 -vframes 1 "thumbnail_${TASK_ID}.jpg"

# Preview GIF (first 3 seconds, 10fps, 480px wide)
ffmpeg -i "h3_output_${TASK_ID}.mp4" -t 3 -vf "fps=10,scale=480:-1" "preview_${TASK_ID}.gif"
```

### 5.4 Present Results

Present the user with:
1. The downloaded video file path
2. Thumbnail preview
3. Key metadata: resolution, duration, aspect ratio, file size
4. The prompt used (for iteration reference)
5. Offer: "Want me to iterate? I can adjust the prompt, change resolution/duration, add references, or regenerate with a different seed."

---

## Phase 6: Iteration & Refinement

### 6.1 Common Iteration Patterns

| Issue | Fix |
|-------|-----|
| Motion too subtle | Add explicit motion descriptors: "rapid," "sweeping," "dynamic," "continuous" |
| Text rendering off | Be more explicit: "The text 'BRAND NAME' in bold white Helvetica, centered, slowly scales up" |
| Wrong lighting | Specify light source direction, color temperature in Kelvin, shadow hardness |
| Audio doesn't match | Describe audio in spatial terms: "footsteps approaching from rear-left," "orchestra crescendo from center" |
| Composition feels flat | Add depth cues: foreground elements, atmospheric perspective, parallax layers |
| Brand colors wrong | Use hex codes: "background #1A1A2E, accent #E94560" |
| Unwanted artifacts | Try 2K resolution (in-context regeneration recovers fine details) or simplify the prompt |

### 6.2 Batch Generation

For A/B testing or variant exploration:

```bash
for prompt in "version A: ${PROMPT_A}" "version B: ${PROMPT_B}"; do
  TASK_ID=$(curl -s -X POST "https://api.minimax.io/v2/video_generation" \
    -H "Authorization: Bearer $MINIMAX_API_KEY" \
    -H "Content-Type: application/json" \
    -d "$(jq -n --arg p "$prompt" '{
      model: "MiniMax-H3",
      content: [{type: "text", text: $p}],
      resolution: "2K",
      duration: 5,
      ratio: "16:9"
    }')" | jq -r '.task_id')
  echo "Submitted: $TASK_ID — $prompt"
done
```

---

## Quick Reference: Resolution & Pricing Tradeoffs

| Resolution | Quality | Speed | Best For |
|-----------|---------|-------|----------|
| **2K** | Maximum detail, in-context regeneration recovers fine text and textures | Slower (2–4 min) | Final deliverables, brand content, text-heavy scenes, print-ready |
| **768P** | Good quality, standard HD | Faster (30–90 sec) | Drafts, social media, rapid iteration, A/B testing |

At 2K, H3's per-second price is **less than 1/3 of mainstream models**. At 768p, it's **less than half the price of mainstream 720p**.

---

## Supporting Files

- [H3 Technical Deep-Dive](references/h3-technical-reference.md) — Architecture, training paradigm, multimodal context understanding
- [Complete API Reference](references/api-reference.md) — All endpoints, parameters, error codes, rate limits
- [Prompt Engineering Guide](references/prompt-engineering.md) — Advanced techniques, style libraries, genre-specific recipes

When you need more detail than this SKILL.md provides, read the relevant reference file.
