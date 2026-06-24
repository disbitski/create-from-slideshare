# Recovery Notes

Slideshare often exposes low-resolution viewer URLs such as:

```text
https://image.slidesharecdn.com/<deck>/<tier>/<title>-<slide>-320.jpg
```

High-resolution variants may use `/75/` plus `-2048.jpg`. Some responses use
WebP bytes even when the URL ends in `.jpg`, so inspect the `Content-Type` and
convert to JPEG before creating PPTX/PDF outputs.

The recovery should prefer visible slide assets already present in the page.
Avoid scraping unrelated recommendations or thumbnail URLs from other decks.
