---
name: cc-reels
description: "Create and edit Connection Codes social media reels — short-form vertical video (1080×1920) for Instagram, TikTok, YouTube Shorts. Use this skill whenever asked to make a reel, edit a reel, create video content, add captions, or produce any social video about the 8 core emotions (Joy, Anger, Shame, Guilt, Fear, Lonely, Sad, Hurt). Invoke for both new reels from raw footage AND edits to existing reels. This skill knows the full pipeline: transcription → words.json → HyperFrames composition → karaoke captions → graphic cards → export via hyperframes-cli."
---

# Connection Codes Reels

Short-form vertical video (1080×1920) for social media. Every reel follows the same brand system and pipeline.

## Emotion → Accent Color

The reel's emotion drives the accent color used everywhere (caption highlights, card headlines). Identify the emotion first — everything else flows from it.

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

**Fonts** (compiler embeds these — just declare in CSS):
- `IBM Plex Sans` — primary for captions and cards (weight 400–800)
- `Cardo` — serif for alternate card styles
- `Rubik` weight 800 — high-impact display

**Caption style:** Karaoke word-by-word highlight pills. Active word gets rounded-rect background in accent color; inactive words plain white. Implementation uses direct `backgroundColor` GSAP animation — NOT CSS class toggling (see Karaoke section).

**Graphic cards:** Cream background `#f5f1e8`, 24px border-radius, left-aligned. Three tiers:
- Top: uppercase accent-colored headline (IBM Plex Sans Bold ~58px)
- Middle: bold dark statement (`#1a1a1a`, ~46px)
- Bottom: supporting detail (muted `#666`, ~30px)

**Video:** Ken Burns zoom (scale 1.0 → 1.08 over clip duration, restart at 1.0 per segment). Muted video + separate audio element on different tracks (HyperFrames requires the split).

**Cuts:** True hard cut — NO overlay, NO dip-to-black. Sequential clips on track 0 with no gap between `data-start` values.

**Sound FX:** `pop_in.wav` on card/caption entrances (vol 0.22), `swoosh_out.wav` on exits (vol 0.16). Files live at project root.

## Pipeline

### 1. Identify emotion & set accent color

Name it. Every decision downstream depends on it.

### 2. Transcribe with word-level timestamps

```bash
npx hyperframes transcribe <video.mov> --model medium.en
```

**Use `medium.en` — not `small.en`.** small.en misses words and produces inaccurate word-level timestamps. medium.en is slower but required for karaoke accuracy.

Alternative: run `/watch:watch <video>` and extract the word timestamps from the output.

### 3. Build words.json

After transcription, create `words.json` in the project root keyed by segment:

```json
{
  "seg1": [
    { "word": "So", "start": 0.0, "end": 0.32 },
    { "word": "when", "start": 0.34, "end": 0.58 }
  ],
  "seg2": [
    { "word": "unprocessed", "start": 45.1, "end": 45.6 }
  ]
}
```

This file is the source of truth for all karaoke timing. Build the GROUPS array from it at composition time.

### 4. Plan video segments & compute timing shifts

If removing silent pauses from the source video (tight editing), only cut at **natural boundaries** — sentence ends, pauses >600ms. Never mid-word or mid-phrase.

Each removed pause shifts ALL downstream timestamps. Track cumulative shift per segment:

| Segment | Source range | Removed before | Shift applied |
|---------|-------------|----------------|---------------|
| seg1a   | 0–30.84s    | nothing        | 0s            |
| seg1b   | 31.90–42.34s| 1.06s pause    | -1.06s        |
| seg1c   | 43.02–44.57s| +0.68s pause   | -1.74s        |
| seg2    | 58.7–67.2s  | +0.53s more    | -2.27s        |

**Formula:** `adjusted_t = source_word_t + shift_for_segment`

Apply the same shift to all GFX `tIn`/`tOut`/`tHide` values that fall within that segment's composition window.

Encode segments with dense keyframes for clean seeking:
```bash
ffmpeg -ss <start> -i source.mov -t <duration> \
  -vf "scale=1080:1920" -c:v libx264 -preset fast -crf 17 \
  -r 30 -g 30 -keyint_min 30 -movflags +faststart -c:a copy \
  seg1a.mp4
```

### 5. Init HyperFrames project

```bash
npx hyperframes init <emotion>-reel --video <clip.mp4> --non-interactive
```

### 6. Write DESIGN.md

```markdown
## Style Prompt
Connection Codes brand: warm, grounded, emotionally clear. Vertical 1080×1920. Emotion-colored accents on dark video canvas. Cream graphic cards with bold typographic hierarchy. Karaoke captions word-by-word.

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
- Don't add cut overlays or dip-to-black — true hard cuts only
```

### 7. Build the composition

Track layout:

| Track | Content |
|-------|---------|
| 0 | Video (muted) |
| 2 | Audio (same source as video) |
| 4 | pop_in.wav SFX |
| 5 | swoosh_out.wav SFX |

Caption groups and GFX cards are overlay divs (no `data-track-index`) positioned absolutely with `z-index`.

Portrait: `data-width="1080" data-height="1920"`.

Video segments — sequential hard cuts, no gaps:
```html
<div id="kb1a" class="kb-wrap">
  <video id="v1a" data-start="0" data-duration="30.84" data-track-index="0" src="seg1a.mp4" muted playsinline></video>
</div>
<audio id="a1a" data-start="0" data-duration="30.84" data-track-index="2" src="seg1a.mp4" data-volume="1"></audio>

<div id="kb1b" class="kb-wrap">
  <video id="v1b" data-start="30.84" data-duration="10.44" data-track-index="0" src="seg1b.mp4" muted playsinline></video>
</div>
<audio id="a1b" data-start="30.84" data-duration="10.44" data-track-index="2" src="seg1b.mp4" data-volume="1"></audio>
```

Ken Burns — restart scale per segment:
```js
tl.to("#kb1a", { scale: 1.08, duration: 30.84, ease: "none" }, 0);
tl.to("#kb1b", { scale: 1.08, duration: 10.44, ease: "none" }, 30.84);
// Each segment starts fresh from scale 1.0
```

### 8. Karaoke captions

**Critical:** Do NOT use GSAP `className` toggling — it's not seekable (scrubbing backwards leaves state wrong). Animate `backgroundColor` directly.

Build GROUPS array from `words.json`, grouping 3–5 words per group at sentence/pause boundaries:

```js
var GROUPS = [
  { id: "cg0", s: 0.0, e: 1.8, words: [
    { id: "cg0-w0", t: 0.0 },
    { id: "cg0-w1", t: 0.52 },
    { id: "cg0-w2", t: 0.84 }
  ]},
  // ...
];
```

Each group needs an HTML container div with word spans:
```html
<div id="cg0" class="cg">
  <div class="cap-line">
    <span id="cg0-w0" class="w">So,</span>
    <span id="cg0-w1" class="w">when</span>
    <span id="cg0-w2" class="w">people</span>
  </div>
</div>
```

CSS — pre-pad all words so highlight background doesn't cause layout shift:
```css
.cg {
  position: absolute;
  bottom: 480px;
  left: 0;
  width: 1080px;
  text-align: center;
  padding: 0 60px;
  opacity: 0;
  visibility: hidden;
  z-index: 60;
}

.cap-line {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  gap: 4px;
}

.w {
  font-family: 'IBM Plex Sans', sans-serif;
  font-weight: 800;
  font-size: 58px;
  color: #fff;
  display: inline-block;
  padding: 2px 14px 6px;        /* pre-padded — always this size */
  border-radius: 10px;
  background: transparent;      /* highlight added at runtime */
  line-height: 1.3;
  text-shadow: 0 2px 16px rgba(0,0,0,0.6);
}
```

GSAP karaoke loop:
```js
var ACCENT = "#6dac95"; // emotion accent color

GROUPS.forEach(function(g) {
  var gsel = "#" + g.id;
  tl.set(gsel, { visibility: "visible" }, g.s);
  tl.fromTo(gsel,
    { opacity: 0, y: 12 },
    { opacity: 1, y: 0, duration: 0.12, ease: "power3.out" },
    g.s
  );
  g.words.forEach(function(w, wi) {
    var wsel = "#" + w.id;
    var wEnd = wi < g.words.length - 1 ? g.words[wi + 1].t : g.e;
    tl.set(wsel, { backgroundColor: ACCENT }, w.t);
    tl.set(wsel, { backgroundColor: "transparent" }, wEnd);
  });
  tl.to(gsel, { opacity: 0, duration: 0.08, ease: "power2.in" }, g.e - 0.08);
  tl.set(gsel, { opacity: 0, visibility: "hidden" }, g.e);
});
```

### 9. Graphic cards

Cards punctuate 1–3 key conceptual moments. Position upper-middle zone, don't compete with captions.

HTML:
```html
<div id="gfx1" class="gfx" style="top: 320px; left: 80px; right: 80px;">
  <p class="gfx-hl">EMOTION LABEL</p>
  <p class="gfx-sub">Bold core statement</p>
  <p class="gfx-body">Clarifying detail</p>
</div>
```

CSS:
```css
.gfx {
  position: absolute;
  visibility: hidden;
  z-index: 70;
  clip-path: inset(0% 0% 100% 0%); /* initial state for curtain reveal */
  background: #f5f1e8;
  border-radius: 24px;
  padding: 44px 52px;
}
.gfx-hl {
  font-family: 'IBM Plex Sans', sans-serif;
  font-weight: 700;
  font-size: 58px;
  color: <ACCENT>;
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

GSAP — curtain reveal entrance, blur-fly exit:
```js
var GFXS = [
  { id: "gfx1", tIn: 1.5, tOut: 5.7, tHide: 6.05 },
  // ...
];

GFXS.forEach(function(g) {
  var sel = "#" + g.id;
  // Entrance
  tl.set(sel, { visibility: "visible" }, g.tIn);
  tl.fromTo(sel,
    { clipPath: "inset(0% 0% 100% 0%)" },
    { clipPath: "inset(0% 0% 0% 0%)", duration: 0.35, ease: "power3.out" },
    g.tIn
  );
  // Exit
  tl.to(sel, { y: -280, filter: "blur(18px)", duration: 0.22, ease: "power4.in" }, g.tOut);
  tl.set(sel, { visibility: "hidden", y: 0, filter: "blur(0px)", clipPath: "inset(0% 0% 100% 0%)" }, g.tHide);
});
```

GFX that bridges a hard cut: set `tIn` before the cut, `tOut` after — provides visual continuity through the video jump.

### 10. Lint and preview

```bash
npx hyperframes lint
npx hyperframes preview --port 3010
```

Check: captions don't cover face, cards readable, karaoke highlight tracks voice, cuts sound clean.

### 11. Export

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
- `hyperframes/references/captions.md` — caption positioning, overflow prevention
- `hyperframes/references/dynamic-techniques.md` — karaoke patterns
- `hyperframes/references/transitions.md` — cut and transition types
- `/watch:watch <video>` — transcription with word-level timestamps
