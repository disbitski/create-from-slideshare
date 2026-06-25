# Create From Slideshare

Recover full local copies of Slideshare decks when the official download, Save
to Drive, or export buttons fail, but the slide viewer still renders the deck.

This repository contains a Codex skill named `$create-from-slideshare` plus a
deterministic recovery script that rebuilds a visible Slideshare presentation
into archival local files:

- Image-based PowerPoint deck (`.pptx`)
- Multi-page PDF (`.pdf`)
- Individual high-resolution slide images
- Raw CDN downloads
- JSON manifest with source URLs and recovered file paths
- Contact sheet for quick visual QA

The rebuilt PowerPoint preserves the visual deck. It is not the original
editable PowerPoint source unless Slideshare exposes a working native download
endpoint.

## Why This Exists

Old technical talks, conference decks, and personal archives can get trapped
behind broken Slideshare UI flows. In many cases, the viewer still exposes all
slide images even when the platform's own download buttons no longer work.

This skill turns that fragile manual recovery process into a repeatable Codex
workflow: inspect the page, extract slide image URLs, prefer the highest
available image variant, download every slide in order, convert formats safely,
rebuild PPTX/PDF outputs, and verify the result before handing it back.

## Skill Install Location

For local Codex discovery, install this folder at:

```text
~/.agents/skills/create-from-slideshare
```

Once installed, invoke it in Codex with:

```text
$create-from-slideshare
```

Example prompt:

```text
Use $create-from-slideshare to recover this Slideshare deck into local PPTX,
PDF, and slide images: https://www.slideshare.net/slideshow/example/123456
```

## What The Skill Does

The skill guides Codex through this recovery flow:

1. Fetch the Slideshare page HTML with a browser-like user agent.
2. Extract `image.slidesharecdn.com` slide image URLs from metadata and viewer
   markup.
3. Prefer high-resolution variants such as `/75/...-2048.jpg`.
4. Download every slide image in slide-number order.
5. Convert raw CDN images to JPEG for broad PowerPoint/PDF compatibility.
6. Rebuild an image-based PPTX and a multi-page PDF.
7. Verify slide counts and visually inspect the contact sheet.
8. Return absolute local links to the recovered deck files.

## Script Usage

The bundled script can also be run directly.

```bash
python3 scripts/recover_slideshare.py \
  "https://www.slideshare.net/slideshow/example-title/123456" \
  --output-dir "/absolute/path/to/output-folder"
```

If you know the expected slide count, make the run fail fast on mismatch:

```bash
python3 scripts/recover_slideshare.py \
  "$SLIDESHARE_URL" \
  --output-dir "$OUTPUT_DIR" \
  --expected-slides 51
```

The output folder will contain:

```text
<output-dir>/
+-- deck-recovered-from-slideshare.pptx
+-- deck-recovered-from-slideshare.pdf
+-- contact-sheet.jpg
+-- manifest.json
+-- images/
|   +-- slide-001.jpg
|   +-- ...
+-- raw/
    +-- slide-001.webp
    +-- ...
```

## Dependencies

The script uses:

- Python 3.12+
- `Pillow`
- `python-pptx`

For Codex Desktop, prefer the bundled workspace Python when available. The
skill's validation helper also expects `PyYAML` because the system
`quick_validate.py` script imports `yaml`.

## Validation Checklist

Before treating a recovery as complete:

- Confirm the discovered slide count matches the deck.
- Confirm raw downloads, JPEG images, PDF pages, and PPTX slides all match.
- Inspect `contact-sheet.jpg` for ordering, blanks, duplicates, or missing
  slides.
- Spot-check at least the first and last recovered slide at full size.
- Tell the user the PPTX is image-based unless a native source file was truly
  recovered.

## Known Behavior

Slideshare may serve WebP bytes from URLs that end in `.jpg`. The script checks
the response content type and keeps raw downloads as delivered, then converts to
JPEG for compatibility with PowerPoint and PDF workflows.

If a page exposes only low-resolution viewer images, the script falls back to
those rather than inventing unavailable URLs. The manifest records exactly which
source URL was used for each slide.

## Responsible Use

Use this skill for decks you own, decks you created, or decks you are explicitly
authorized to archive. Always try Slideshare's official download flow first,
including the normal download button and any account-based export options the
site provides.

Do not use this skill to bypass access controls, copy private or restricted
content, redistribute someone else's work, or ignore Slideshare's terms of
service. The recovery flow is intended for personal archival and legitimate
preservation when a public viewer still displays the slides but official
download/export controls are unavailable or broken.

You are responsible for making sure your use is lawful and permitted. The
authors and contributors of this repository are not responsible for misuse,
copyright infringement, policy violations, or other claims arising from how this
tool is used.

## Repository Layout

```text
.
+-- SKILL.md
+-- agents/
|   +-- openai.yaml
+-- references/
|   +-- recovery-notes.md
+-- scripts/
    +-- recover_slideshare.py
```

## Status

Validated against two of Dave Isbitski's own real Slideshare decks from his
time at Microsoft:

- `living-the-dream-make-the-video-game-youve-always-wanted-and-get-paid-for-it/13618324`
  recovered 51 slides.
- `windows-phone-app-development/13618764` recovered 40 slides.

Both runs produced matching image, PDF, and PPTX slide counts.
