# GLIT — touch the data layer

A browser-based tool for **literally corrupting media files**. No filters, no
shaders, no glitch *aesthetics* — every gesture rewrites the actual bytes of
the file, then the browser is asked to decode whatever is left. It's hex-editor
hand-glitching (cut, paste, smear, scramble) turned into a touch surface with
near-immediate feedback.

Everything runs client-side in one `index.html`. Nothing is uploaded anywhere.

## How it works

- **Open** a file (tap or drag-drop): JPG, PNG, GIF, WebP, BMP, MP3, WAV, OGG,
  MP4, WebM, PDF, HEIC/HEIF/AVIF — anything, really. Unknown types can still
  be scrubbed and exported.
- The file's raw bytes fill the **data field** (right/bottom panel), one pixel
  per slice of file. Drag a finger or cursor across it to corrupt the bytes
  under your touch. The preview re-decodes live as you scrub.
- **Brushes** are real byte operations (every drawing metaphor targets the
  data layer, never the pixels):
  - `SMEAR` — directional smudge: drags and stretches the bytes along your
    stroke
  - `SCRAMBLE` — shuffle the bytes in place
  - `STUTTER` — copy a small chunk and repeat it (the classic copy-paste move)
  - `FLIP` — flip random bits
  - `SPIN` — keeps rotating the bytes under the brush for as long as your
    finger is down, even without moving
  - `CLONE` — clone stamp: first tap sets a source point (cyan marker), then
    draw anywhere and the source (dashed green marker) travels with your
    stroke, copying from one to the other
  - `PIXEL` — quantize: posterizes values and blockifies positions
  - `LIGHT` / `DARK` — shift byte values up/down (brightens/darkens the data
    field, and whatever that does downstream)
  - `GRAD`, `WHITE`, `ZERO` — pencils: draw a clean 0→255 ramp, 0xFF ink
    ("blank"/eraser), or 0x00 ink straight onto the data
  - `REVERSE`, `SORT` — what they say
- **Brush cursor** — a dot with a faint circle stands in for your finger on
  the data field; the circle scales with brush size.
- **BAKE / ROT ⟳ / FLIP ↔ ↕** (images) — rasterize whatever the decoder
  currently shows and re-encode it as a clean, legit file. The glitch state
  becomes the new base layer, ready to corrupt again — the classic corrupt →
  flatten → corrupt cycle. The transparency checkerboard is baked in below
  the glitch layer, so transparent pixels and undecoded gray dead zones
  become real, fully corruptible data. If the decoder has choked completely,
  BAKE falls back to the last frame that decoded at all. ROT is the move when
  your glitches keep streaking one way: rotate, glitch across the other axis,
  rotate back. Rotation and flips bake as they go (they have to — the bytes
  are a whole new file after re-encoding).
- **Structure awareness.** GLIT parses the file's anatomy on load — JPEG
  markers, PNG chunks, GIF blocks, RIFF chunks, ISO boxes (MP4/MOV/HEIC/HEIF/
  AVIF), PDF objects/streams/xref. Region boundaries show as white ticks on
  the data field, and the hex readout names the region under your finger
  (`scan`, `IDAT`, `mdat`, `stream 3`, `trailer`…), so you know *what* you're
  corrupting, not just *where*.
- **LOCK HEAD** protects the start of the file so it stays *openable* while
  the body gets wrecked. The default is format-aware: JPEG locks through the
  Start-Of-Scan marker, PNG through IHDR, GIF through the palette, MP3 past
  the ID3 tag. Slide it to 0 to risk the magic bytes too.
- **STRUCT LOCK** (appears for multifaceted formats) protects the ranges that
  aren't just headers but are fatal *anywhere* in the file:
  - **PDF** — the `%PDF` line and the final `startxref`→`%%EOF` trailer (a PDF
    dies from the *tail*, not the head). Everything between, including content
    streams, is yours.
  - **HEIC/HEIF/AVIF/MP4/MOV** — every non-`mdat` box (`ftyp`, `meta`, `moov`
    — the offset tables that make the file readable at all) plus `mdat`'s own
    header. The `mdat` payload, where the actual image/av data lives, stays
    fully corruptible. These formats are unopenable otherwise — one flipped
    bit in `iloc` and a HEIC is a brick.
  - **WAV/RIFF** — chunk headers and the `fmt` chunk; sample data stays open.
  Untick it to go fully feral. Strokes flow *around* locked ranges — scrub
  across the whole file and only the survivable bytes take damage.
- **PNG CRC FIX** (PNGs only): browsers refuse PNGs with bad chunk checksums,
  so this recomputes the CRCs — the corruption stays, the decoder just stops
  vetoing it. Applies to preview and export.
- `UNDO` (per stroke), `CHAOS` (spray random corruption everywhere), `RESET`
  (full reset — all the way back to the file as you opened it, through any
  number of bakes), and `EXPORT` (downloads `name.glit.ext`).
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
- **PDF** previews live in the browser's own PDF renderer (desktop
  Chrome/Firefox/Edge; iOS shows the first page at best). Browser PDF viewers
  rebuild broken xref tables, so they're forgiving — a glitched PDF that
  renders here may still choke stricter viewers, or vice versa.
- **HEIC/HEIF** only decodes in Safari (macOS/iOS) — elsewhere you'll get the
  DECODER CHOKED badge while you scrub blind; the corruption is still real,
  so export and open in a HEIC-capable viewer. AVIF previews almost anywhere.
- Corrupted PNGs without CRC FIX, and heavily corrupted MP4s, may only open in
  tolerant players (VLC, ffplay) — that's the medium.
