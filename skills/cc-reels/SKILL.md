---
name: cc-reels
description: "Create and edit Connection Codes social media reels — short-form vertical video (1080×1920) for Instagram, TikTok, YouTube Shorts. Use this skill whenever asked to make a reel, edit a reel, create video content, add captions, or produce any social video about the 8 core emotions (Joy, Anger, Shame, Guilt, Fear, Lonely, Sad, Hurt). Invoke for both new reels from raw footage AND edits to existing reels. This skill knows the full pipeline: transcription → HyperFrames composition → karaoke captions → graphic cards → export via hyperframes-cli."
---

# Connection Codes Reels

Short-form vertical video (1080×1920) for social media. Every reel follows the same brand system and pipeline.

## Emotion → Accent Color

The reel's emotion drives the accent color used everywhere (caption highlights, card headlines, UI accents). Identify the emotion first — everything else flows from it.

| Emotion | Accent Color |
|---------|-------------|
| Joy     | `#6dac95`   |
| Anger   | `#cf4b3f`   |
| Shame   | `#e0a700`   |
| Guilt   | `#406096`   |
| Fear    | `#86cec2`   |
| Lonely  | `#f27775`   |
| Sad     | `#e58256`   |
| Hurt    | `#264375`   |

## Brand Identity

**Fonts** (compiler embeds these — just declare the family name in CSS):
- `IBM Plex Sans` — primary font for captions and graphic cards (weight 400–700, italic available)
- `Cardo` — elegant serif for alternate card styles
- `Rubik` weight 800 — high-impact display when needed

Font files on disk:
```
~/Desktop/Desktop/CC Stuff/Marketing Materials/Design branding/
  IBM_Plex_Sans/IBMPlexSans-VariableFont_wdth,wght.ttf
  IBM_Plex_Sans/IBMPlexSans-Italic-VariableFont_wdth,wght.ttf
  Cardo/Cardo-Bold.ttf
  Cardo/Cardo-Italic.ttf
  Cardo/Cardo-Regular.ttf
```

**Caption style:** Karaoke — word-by-word highlight pills. As each word is spoken, it gets a rounded-rect background in the emotion accent color. Inactive words are plain white. This is the signature CC look.

**Graphic cards:** Cream background `#f5f1e8`, 24px border-radius, left-aligned. Three tiers:
- Top: uppercase emotion-colored headline (`IBM Plex Sans` Bold ~60px)
- Middle: bold dark statement (`#1a1a1a`, ~48px)
- Bottom: supporting detail (muted `#666`, ~30px)

**Video:** Subtle Ken Burns zoom (scale 1.0 → 1.08 over clip duration). Muted video element + separate audio element (HyperFrames requires this split).

**Cuts:** Hard cut with black flash overlay (~2 frames) between video segments.

**Sound FX:** `pop_in.wav` on caption group entrances (vol 0.22), `swoosh_out.wav` on exits (vol 0.16). FX files live alongside the reel.

## Pipeline

### 1. Identify emotion & set accent color

Name it. Every decision downstream depends on it.

### 2. Transcribe to get word-level timestamps

```bash
npx hyperframes transcribe <video.mov> --model small.en
```

Word-level timestamps are required for karaoke. Standard SRT won't work.

Alternatively, use `/watch:watch <video>` — the watch skill produces timestamped transcripts too.

### 3. Init HyperFrames project

```bash
npx hyperframes init <emotion>-reel --video <clip.mp4> --non-interactive
```

### 4. Write DESIGN.md before touching index.html

HyperFrames requires a visual identity before authoring. Create `DESIGN.md` in the project root:

```markdown
## Style Prompt
Connection Codes brand: warm, grounded, emotionally clear. Vertical 1080×1920. Emotion-colored accents on a dark video canvas. Cream graphic cards with bold typographic hierarchy. Karaoke captions, word-by-word.

## Colors
- Accent: <EMOTION_COLOR> (caption highlights, card headlines)
- Card background: #f5f1e8
- Card text primary: #1a1a1a
- Card text secondary: #666666
- Caption inactive: #ffffff

## Typography
- IBM Plex Sans (primary — captions, cards)
- Cardo (alternate serif for cards)

## What NOT to Do
- Don't use generic blue/gray palettes — the accent color IS the emotion
- Don't center-align graphic card text — always left-align
- Don't cover the speaker's face with captions or cards
- Don't use exit animations before cuts — the cut IS the exit
- Don't use more than one graphic card visible at a time
```

### 5. Build the composition

Use the `/hyperframes` skill. Follow its rules strictly. CC reel track layout:

| Track | Content |
|-------|---------|
| 0 | Video (muted) |
| 2 | Audio (from same source as video) |
| 3 | Karaoke caption groups |
| 4 | Graphic cards |
| 5 | pop_in.wav sound FX |
| 6 | swoosh_out.wav sound FX |

Portrait format: `data-width="1080" data-height="1920"`.

### 6. Karaoke captions

Read `hyperframes/references/dynamic-techniques.md` for the full karaoke implementation pattern. Apply the CC style on top:

```css
.caption-row {
  position: absolute;
  bottom: 520px;               /* above lower third — don't cover face */
  left: 0;
  width: 1080px;
  text-align: center;
  padding: 0 80px;
  box-sizing: border-box;
}

.word {
  font-family: 'IBM Plex Sans', sans-serif;
  font-weight: 600;
  font-size: 58px;
  color: #ffffff;
  display: inline-block;
  padding: 6px 16px;
  border-radius: 10px;
  margin: 0 4px;
  line-height: 1.4;
}

.word.active {
  background-color: <EMOTION_COLOR>;
  color: #ffffff;
}
```

Group words: 3–5 per group (conversational pacing). Break on sentence boundaries and 150ms+ pauses.

Every caption group needs a hard kill:
```js
tl.to(cgEl, { opacity: 0, duration: 0.1, ease: "power2.in" }, group.end - 0.1);
tl.set(cgEl, { opacity: 0, visibility: "hidden" }, group.end);
```

### 7. Graphic cards

Cards punctuate the reel at 1–3 key conceptual moments — when the core idea of a segment lands. Position in the upper-middle zone so they don't compete with captions.

```html
<div id="card-1" class="gfx-card" 
  data-start="X" data-duration="Y" data-track-index="4"
  style="position: absolute; top: 35%; left: 60px; right: 60px; visibility: hidden; z-index: 70;">
  <p class="gfx-hl">CONCEPT TITLE</p>
  <p class="gfx-sub">Bold supporting statement</p>
  <p class="gfx-body">Smaller clarifying detail</p>
</div>
```

```css
.gfx-card {
  background: #f5f1e8;
  border-radius: 24px;
  padding: 44px 52px;
}
.gfx-hl {
  font-family: 'IBM Plex Sans', sans-serif;
  font-weight: 700;
  font-size: 58px;
  color: <EMOTION_COLOR>;
  text-transform: uppercase;
  letter-spacing: -0.02em;
  margin-bottom: 10px;
}
.gfx-sub {
  font-family: 'IBM Plex Sans', sans-serif;
  font-weight: 700;
  font-size: 46px;
  color: #1a1a1a;
  margin-bottom: 8px;
}
.gfx-body {
  font-family: 'IBM Plex Sans', sans-serif;
  font-weight: 400;
  font-size: 30px;
  color: #666;
}
```

Card entrance animation (slide from left + fade):
```js
tl.set("#card-1", { visibility: "visible" }, cardStart);
tl.from("#card-1", { x: -40, opacity: 0, duration: 0.35, ease: "power3.out" }, cardStart);
```

No exit animation — the cut handles it. (Exception: last card in reel can fade out.)

### 8. Lint and preview

```bash
npx hyperframes lint
npx hyperframes preview
```

Check: captions don't cover the face, cards are readable, timing feels right, cuts land on beats.

### 9. Export

```bash
# Draft (fast iteration):
npx hyperframes render --quality draft --output <emotion>-reel-draft.mp4

# Final delivery:
npx hyperframes render --quality high --fps 30 --output <emotion>-reel-v1.mp4
```

## Editing an Existing Reel

1. Read full `index.html` first — match existing fonts, colors, timing patterns
2. Identify exactly what needs changing
3. Touch only what was requested — preserve all other timing
4. Re-lint → re-preview → re-render when approved

## References

- `/hyperframes` skill — full HyperFrames composition rules
- `/hyperframes-cli` skill — init, lint, preview, render commands
- `hyperframes/references/captions.md` — caption positioning, overflow prevention, exit guarantees
- `hyperframes/references/dynamic-techniques.md` — karaoke implementation patterns
- `hyperframes/references/transitions.md` — cut and transition types
- `/watch:watch <video>` — transcription with word-level timestamps
