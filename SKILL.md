---
name: create-from-slideshare
description: Recover archival local copies of Slideshare presentations when the built-in download, Save to Drive, or export buttons are broken but the visible slide viewer still loads every slide. Use for $create-from-slideshare, Slideshare deck recovery, rebuilding a deck from slide images, saving a Slideshare presentation as PPTX/PDF/images, or preserving old talks from a Slideshare URL.
---

# Create From Slideshare

## Overview

Recover a Slideshare deck by extracting the viewer's slide image URLs, downloading the best available image for each slide, and rebuilding local archival outputs.

The default deliverables are:

- Image-based `.pptx`
- Multi-page `.pdf`
- Individual slide images
- Raw downloads and a JSON manifest for provenance
- Contact sheet for quick visual QA

The rebuilt PPTX preserves the visual deck but is not the original editable PowerPoint source. Always say this clearly when delivering results.

## Workflow

1. Fetch the Slideshare page HTML with a browser-like user agent.
2. Extract all `image.slidesharecdn.com` slide image URLs from page metadata and viewer markup.
3. Prefer highest-resolution variants such as `/75/...-2048.jpg`; fall back to observed viewer images when needed.
4. Download every slide image in slide-number order.
5. Convert raw CDN images to JPEG for broad PPTX/PDF compatibility.
6. Build the image-based PPTX and PDF.
7. Verify counts and visually inspect a contact sheet or first/last slide previews.
8. Return absolute links to the final files and mention the source URL.

## Quick Start

Use the bundled script when a normal Slideshare download fails:

```bash
python3 /path/to/create-from-slideshare/scripts/recover_slideshare.py \
  "https://www.slideshare.net/slideshow/example-title/123456" \
  --output-dir "/absolute/path/to/output-folder"
```

In Codex Desktop, prefer the bundled workspace Python if available:

```bash
/Users/wizard/.cache/codex-runtimes/codex-primary-runtime/dependencies/python/bin/python3 \
  /path/to/create-from-slideshare/scripts/recover_slideshare.py \
  "https://www.slideshare.net/slideshow/example-title/123456" \
  --output-dir "/absolute/path/to/output-folder"
```

If the user supplied an expected slide count, pass it:

```bash
python3 scripts/recover_slideshare.py "$URL" --output-dir "$OUT" --expected-slides 51
```

## Dependencies

The bundled script uses `Pillow` for image conversion/PDF assembly and `python-pptx` for the image-based PPTX. In Codex Desktop, call `load_workspace_dependencies` first and use the bundled workspace Python when possible. If either package is missing in another environment, ask before installing dependencies or switch to an equivalent local conversion path.

## Output Layout

The script writes:

```text
<output-dir>/
├── deck-recovered-from-slideshare.pptx
├── deck-recovered-from-slideshare.pdf
├── contact-sheet.jpg
├── manifest.json
├── images/
│   ├── slide-001.jpg
│   └── ...
└── raw/
    ├── slide-001.webp
    └── ...
```

Rename final files after a successful run if the user wants a specific title.

## Validation

Before finishing:

- Confirm the script found and downloaded the expected number of slides.
- Confirm PDF page count matches the image count.
- Confirm PPTX slide count matches the image count.
- Inspect `contact-sheet.jpg` for ordering, missing slides, duplicate slides, or blank placeholders.
- Inspect at least the first and last slides at full size when the deck is personally important or archival.

For PowerPoint counts, inspect the `.pptx` zip entries:

```bash
python3 - <<'PY'
import zipfile
pptx = "deck-recovered-from-slideshare.pptx"
with zipfile.ZipFile(pptx) as z:
    print(sum(1 for n in z.namelist() if n.startswith("ppt/slides/slide") and n.endswith(".xml")))
PY
```

For PDF counts, use `pypdf` when available:

```bash
python3 - <<'PY'
from pypdf import PdfReader
print(len(PdfReader("deck-recovered-from-slideshare.pdf").pages))
PY
```

## Troubleshooting

- If the script finds only one slide, open the page HTML and search for `image.slidesharecdn.com`, `ss_thumbnails`, and `__NEXT_DATA__`.
- If high-resolution URLs fail, keep the observed viewer URLs instead of inventing more variants.
- If PowerPoint cannot open a rebuilt PPTX, rebuild from converted JPEGs, not the raw CDN files; Slideshare may serve WebP bytes from `.jpg` URLs.
- If the page requires login or network access is blocked, ask for browser access, a saved HTML file, or permission to fetch the page.
- Do not claim the original native PPTX was recovered unless an actual native download endpoint succeeds.

## Source Attribution

Mention the Slideshare URL used for recovery. Do not reproduce unrelated page text or recommendations from Slideshare; the deck images are the recovery target.
