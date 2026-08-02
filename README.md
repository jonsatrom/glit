# GLIT — touch the data layer

A browser-based tool for **literally corrupting media files**. No filters, no
shaders, no glitch *aesthetics* — every gesture rewrites the actual bytes of
the file, then the browser is asked to decode whatever is left. It's hex-editor
hand-glitching (cut, paste, smear, scramble) turned into a touch surface with
near-immediate feedback.

Everything runs client-side in one `index.html`. Nothing is uploaded anywhere.

## How it works

- **Open** a file (tap or drag-drop): JPG, PNG, GIF, WebP, BMP, MP3, WAV, OGG,
  MP4, WebM — anything, really. Unknown types can still be scrubbed and
  exported.
- The file's raw bytes fill the **data field** (right/bottom panel), one pixel
  per slice of file. Drag a finger or cursor across it to corrupt the bytes
  under your touch. The preview re-decodes live as you scrub.
- **Brushes** are real byte operations:
  - `SMEAR` — repeat one byte across the range
  - `SCRAMBLE` — shuffle the bytes in place
  - `STUTTER` — copy a small chunk and repeat it (the classic copy-paste move)
  - `FLIP` — flip random bits
  - `CLONE` — paste bytes grabbed from elsewhere in the file
  - `REVERSE`, `SORT`, `ZERO` — what they say
- **LOCK HEAD** protects the start of the file so it stays *openable* while
  the body gets wrecked. The default is format-aware: JPEG locks through the
  Start-Of-Scan marker, PNG through IHDR, GIF through the palette, MP3 past
  the ID3 tag, MP4 through the box headers to `mdat`. Slide it to 0 to risk
  the magic bytes too.
- **PNG CRC FIX** (PNGs only): browsers refuse PNGs with bad chunk checksums,
  so this recomputes the CRCs — the corruption stays, the decoder just stops
  vetoing it. Applies to preview and export.
- `UNDO` (per stroke), `CHAOS` (spray random corruption everywhere), `RESET`,
  and `EXPORT` (downloads `name.glit.ext`).
- The hex readout at the bottom of the preview shows the bytes under your
  finger as you scrub.

If the preview goes black or the badge says **DECODER CHOKED**, the browser
gave up on that state of the file — the bytes are still yours; undo a stroke
or export anyway and open it in something more forgiving.

## Hosting

It's a single static file with zero dependencies.

**GitHub Pages:** repo settings → Pages → deploy from branch → pick the branch
and `/ (root)`. Done — `https://<user>.github.io/glit/`.

**Dreamhost (or any web host):** copy `index.html` into any web-accessible
directory. That's the whole deployment.

**Locally:** just open `index.html` in a browser — it works from `file://`.

## Notes

- Works on phones (pointer events, no hover dependence). Big files (long
  videos) preview more slowly since the whole blob is re-fed to the decoder;
  images update at scrub speed.
- Corrupted PNGs without CRC FIX, and heavily corrupted MP4s, may only open in
  tolerant players (VLC, ffplay) — that's the medium.
