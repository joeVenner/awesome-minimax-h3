# MiniMax H3 — Prompt Engineering Guide

> Advanced techniques for getting the best results from MiniMax H3 video generation.

---

## The H3 Difference: Why Prompts Matter More

Unlike previous video models that treat prompt as a simple visual description, H3 uses **Contextual Omni Representation** where language is the computational bridge across all modalities. This means your prompt quality directly determines output quality far more than with previous models.

### H3's Prompt Understanding Capabilities:
- Understands **relationships between reference media** (not just the media itself)
- Handles **complex multi-element instructions** with accurate sequencing
- Interprets **camera technique terminology** (Hitchcock zoom, dolly, crane, rack focus)
- Processes **brand/text instructions** with high accuracy
- Understands **audio descriptions** for joint audio-visual generation

---

## Style Libraries

### Camera Movement Lexicon

| Term | Description | When To Use |
|------|-------------|-------------|
| Dolly zoom / Hitchcock zoom | Camera moves while zooming opposite direction, creating disorienting perspective shift | Suspense, revelation, psychological moments |
| Crane shot / Jib | Camera rises or descends on a crane | Establishing shots, grandeur, reveals |
| Steadicam / Gimbal | Smooth floating camera following subject | Walk-and-talks, tracking characters, immersion |
| Whip pan | Rapid horizontal camera rotation creating motion blur | Energy, transitions, chaos |
| Rack focus | Focus shifts between foreground and background | Drawing attention, reveals, emotional shifts |
| Dutch angle | Camera tilted on roll axis | Unease, tension, disorientation |
| Push in / Dolly in | Camera moves toward subject | Intensity, importance, emotional climax |
| Pull out / Dolly out | Camera moves away from subject | Isolation, scale, ending |
| Tracking / Trucking | Camera moves laterally alongside subject | Following action, parallelism |
| Orbital / Arc shot | Camera circles subject | Power dynamics, 360° product views |
| Pedestal | Camera moves vertically (up/down) | Scale reveals, power dynamics |
| Handheld | Camera shakes slightly, organic movement | Documentary style, urgency, realism |
| Locked-off / Tripod | Camera completely still | Formality, tension, precision |
| SnorriCam | Camera attached to actor, background moves | Disorientation, character focus |
| Slow creep zoom | Extremely slow zoom in | Building dread, subtle emphasis |

### Lighting Lexicon

| Term | Visual Effect |
|------|--------------|
| Golden hour | Warm, long shadows, soft directional light |
| Blue hour | Cool twilight, ethereal, pre-dawn/dusk |
| Rembrandt lighting | Triangle of light under eye, dramatic portrait |
| Butterfly lighting | Shadow under nose, beauty/glamour |
| Split lighting | Half face lit, half dark — dramatic |
| Rim light / Backlight | Subject outlined in light, separation from background |
| Chiaroscuro | Strong contrast light/dark, Caravaggio-style |
| Neon noir | Colored neon sources, rain, wet surfaces, Blade Runner |
| Practical lights | Light sources visible in frame (lamps, screens, signs) |
| Soft diffused | Overcast, softbox, no hard shadows |
| Hard sunlight | Direct sun, sharp shadows, high contrast |
| Motivated lighting | Light that appears to come from a visible source in scene |
| Silhouette | Subject dark against bright background |
| God rays / Crepuscular | Beams of light through atmosphere, dust, or fog |
| Lens flare | Light directly hitting lens, anamorphic streaks |

### Color Grading References

| Grade | Look | Reference |
|-------|------|-----------|
| Teal & Orange | Warm skin, cool shadows — blockbuster look | Michael Bay, Transformers |
| Desaturated + single accent | Muted colors with one pop (red coat in Schindler's List) | Sin City, Schindler's List |
| Bleach bypass | High contrast, desaturated, silvery | Saving Private Ryan, Fight Club |
| Cross-processed | Color shifts, saturated, surreal | Music videos, fashion |
| Kodak Portra 400 | Warm, pastel, soft contrast — filmic | Indie films, portraits |
| Kodak Ektachrome | Cool blue shadows, realistic colors | Documentaries, 1960s |
| Fuji Velvia | Hyper-saturated, vivid, high contrast | Nature, landscapes |
| Cyan shadows + magenta highlights | Trendy color contrast | Modern Instagram look |
| Monochrome | Black and white, focus on texture/form | Noir, timeless, dramatic |
| Sepia | Warm brown tone, vintage | Historical, nostalgia |
| LUT: Blockbuster | High contrast, teal/orange, sharp | Action/adventure |

---

## Genre-Specific Recipe Book

### Product Commercial (Luxury)

```
[SCENE]: {Product Name}, {material descriptor}, centered on a {background} surface. 
The product rotates slowly, catching light on its {key feature}. 
Macro detail of {specific detail} with shallow depth of field.

[CAMERA]: Slow orbital arc around the product, starting at eye level and 
rising to a 45° overhead. Macro push into {detail}. Smooth, no sudden moves.

[LIGHTING]: Studio lighting — key light from 10 o'clock with soft diffusion, 
rim light from behind at 4 o'clock creating {material} edge glow. 
Background gradient from {color1} to {color2}.

[COLOR]: Clean, premium — {palette}. Product colors must be accurate. 
Background desaturated to make product pop.

[AUDIO]: Subtle atmospheric pad. Soft whoosh on camera moves. 
A gentle chime as light catches the product.
```

### Film Title Sequence

```
[SCENE]: Abstract {texture/pattern} background evoking {film genre}. 
The text "{TITLE}" in {font style}, {size}, {color} fades in from {direction}. 
{Secondary elements} animate in layers with staggered timing.

[CAMERA]: Slow drift through abstract space. {Depth effect}.
Occasional lens flare as virtual light sources pass frame edge.

[LIGHTING]: {Mood lighting}. Subtle light leaks at frame edges.
Volumetric rays through atmospheric haze.

[COLOR]: {Genre color palette}. Title text in {color} with {effect — glow/shadows/metallic}.

[AUDIO]: {Genre music style} building in intensity. Deep bass rumble underneath. 
Thematic sting on title reveal.
```

### E-commerce Product Showcase

```
[SCENE]: {Product} on a clean white/neutral background. Product rotates 360 degrees, 
showing all angles. Then transitions to lifestyle context: 
{product in use scenario}. Then close-up of {key feature/detail}. 
End with product + logo lockup.

[CAMERA]: Turntable rotation (smooth, consistent speed). 
Transition via {wipe/dissolve/zoom} to lifestyle scene. 
Push into macro detail. Pull out to wide brand lockup.

[LIGHTING]: Commercial product lighting — even, shadowless for turntable. 
Natural daylight for lifestyle scene. Accent light on logo.

[COLOR]: Accurate product colors critical. Lifestyle scene: {mood palette}. 
Logo: brand colors exactly.

[AUDIO]: Upbeat, commercial-friendly instrumental. Soft transition sounds.
```

### Gaming Cinematic

```
[SCENE]: {Environment} viewed from {perspective}. {Character/entity} {action}. 
Particle effects: {VFX description}. Environmental elements: 
{weather/wind/destruction}. Depth through atmospheric fog layers.

[CAMERA]: {Camera style — flythrough/orbital/tracking} through the environment. 
Dynamic speed changes — slow for scale reveals, fast for action beats. 
Screen shake on impacts.

[LIGHTING]: {Time of day/artificial sources}. Volumetric god rays. 
Dynamic light sources from {VFX/abilities}. Rim light on character silhouettes.

[COLOR]: Game engine-inspired — {reference game/genre} aesthetic. 
High contrast for drama. Color-graded for mood.

[AUDIO]: Cinematic score with {genre} elements. Deep impacts. 
Spatial environmental audio — {specific sounds}.
```

---

## Advanced Techniques

### Multimodal Context Prompting

When using multiple reference inputs, the key insight is that H3 understands **relationships** between them. Your prompt should explicitly bridge them:

**Weak:** "Generate a video using these references."
**Strong:** "Reference the camera movement and pacing from Video 1. Apply the lighting style and color grade from Image 2. Have the character match the vocal tone from Audio 3. The scene: {your description}."

### Text & Brand Rendering

H3 has specific strengths in accurate text rendering. To maximize this:

1. **Always quote the exact text:** `The text "REVOLUTION" appears in bold sans-serif`
2. **Specify font characteristics:** weight (bold/light/regular), style (serif/sans-serif/script), case (uppercase/lowercase)
3. **Describe text animation:** fade in, slide from direction, typewriter, scale up, letter-by-letter reveal
4. **Position precisely:** centered, bottom-third, top-left, overlay on {element}
5. **Include color:** white text on dark background, brand color #E94560
6. **Mention timing:** text appears at 2 seconds, fades at 8 seconds

### Motion Transfer (V2V)

When using a reference video for motion transfer, describe what aspect of the motion to transfer:

- "Match the camera movement cadence from Video 1"
- "Apply the character animation style from Video 1 to the character in Image 2"  
- "Use the lighting transitions and color timing from Video 1"
- "Match the editing rhythm and cut pacing from Video 1"
- "Reference the particle behavior and physics from Video 1"

### Native Stereo Audio Prompting

H3 generates native stereo audio jointly with video. Describe audio spatially:

- "Footsteps pan from rear-left to front-right"
- "Orchestra swells from center outward"
- "Rain ambience with distinct left/right channel variation"
- "Dialogue centered, environmental sounds stereo-wide"
- "Bass frequencies centered, highs spread across stereo field"

---

## Common Pitfalls

| Pitfall | Why It Happens | Fix |
|---------|---------------|-----|
| Static output | Prompt lacks motion verbs | Add explicit camera movement and subject action |
| Ignored reference | Relationship not described | Bridge references explicitly: "the style FROM image X applied TO..." |
| Washed-out colors | No color direction | Add specific color grade reference or hex codes |
| Unnatural motion | Contradictory physics | Ensure motion description is physically coherent |
| Garbled text | Font insufficiently specified | Quote text exactly, specify font, position, animation |
| Flat composition | No depth cues | Add foreground, midground, background layers; atmospheric perspective |
