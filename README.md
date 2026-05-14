# CC Skills

Claude Code skills for Connection Codes content production.

## Install

```bash
/plugins install github:JonnyH1801/cc-skills
```

## Dependencies (install these first)

```bash
/plugins install github:heygen-com/hyperframes
/plugins install git:https://github.com/bradautomates/claude-video.git
```

## Skills

### `cc-reels`

Creates and edits Connection Codes social media reels (1080×1920) for Instagram, TikTok, YouTube Shorts.

**Triggers when you say things like:**
- "Make a reel about Anger"
- "Edit this reel"
- "Create a Joy reel from these clips"

**What it knows:**
- All 8 core emotion accent colors
- Karaoke caption style (word-by-word highlight pills)
- Graphic card layout (cream cards, 3-tier hierarchy)
- IBM Plex Sans / Cardo / Rubik brand fonts
- Full HyperFrames pipeline: transcribe → compose → preview → export

**Pipeline overview:**
1. Transcribe footage → word-level timestamps
2. `npx hyperframes init`
3. Build composition with karaoke captions + graphic cards
4. `npx hyperframes preview` to review
5. `npx hyperframes render` to export MP4
