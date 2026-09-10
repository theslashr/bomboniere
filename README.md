# Bomboniere — prova rilievo

A relief viewer for Antonio Burgello's small oils on canvas, in the spirit of
Displate's Textra preview: move the pointer and the light travels across the
canvas weave and the ridges of paint.

Open `index.html` over HTTP (WebGL will not take a texture from a `file://`
image):

```bash
py -3 -m http.server 8124
```

then <http://127.0.0.1:8124/>.

## How it works

There is no scanned normal map here. The relief is recovered from each
photograph on load:

1. **Band pass.** Blur the luminance by one pixel, subtract a wider blur of it.
   The wide blur removes the composition; the one-pixel blur removes JPEG
   noise. What survives is the surface — canvas weave is roughly 2–4px at
   1280px, so it comes through and the noise does not. An earlier version
   subtracted only the wide blur, and the noise it kept showed up as red-green
   speckle in the normal map and as sparkle on the lit surface.
2. **Sobel** the result into surface normals, packed into an RGB texture.
3. **Blinn-Phong** in a fragment shader, with the light direction driven by the
   pointer, or by `DeviceOrientationEvent` on a phone.

Step 1 and 2 run once per painting on the CPU and are cached; doing the band
pass in the shader would cost about twenty texture reads per pixel per frame,
against two for a baked map. Roughly 400–900ms per painting at 1280px.

## Numbering

All 126 are in the viewer, each shown as `NN / 126`, one of a kind.

The numbers are **not** the ones in the original file names. Telegram exported
the set in two batches and restarted counting in each, so of 126 files there
were only 95 distinct `photo_N` numbers, 31 of them used by two different
paintings, and the highest was 97 — nothing would ever have been numbered 98 to
126. Calling a piece "11 / 126" on that basis would be a false claim about a
unique work.

The images are therefore stored as `n001.jpg` … `n126.jpg`, sorted by batch and
then by number within it. The file name *is* the edition number, so there is no
table to keep in step. To add or reorder anything, re-derive the whole sequence
the same way rather than renaming a file by hand.

## The sheet

The thumbnails are laid out in **columns, not grid**. Grid has to size a row
track before it can place anything in it, and an image at `width:100%`
contributes no height to that pass, so every row collapsed and the paintings
were clipped into ribbons. Columns also need a wrapper to do the scrolling: a
multi-column box with a fixed height does not grow downward, it makes more
columns and overflows sideways, so `overflow-y` on the columns themselves had
nothing to scroll.

The grid is what opens first — it is the collection, and the viewer is where a
single painting goes. `#one=1` skips straight to the viewer.

Nothing is painted over the grid. One layer sits above it carrying a mild
desaturation, masked away around the pointer, so the paintings the hand is near
keep all their colour and the rest sit back a little: 100% saturation on the
cursor, 97% one thumbnail across, 81% two away, about 63% in the far corners,
with brightness barely moving. It is about colour, not darkness.

The mask centre is two custom properties written straight from the pointer
event, so it tracks exactly — no easing, no trail, nothing to fall behind. One
style write per move on one element, rather than a filter recomputed on 126 of
them.

Two earlier attempts are worth recording as dead ends. A veil over the grid that
the cursor wiped off meant the paintings only looked right where the hand had
been, which is backwards for a page whose job is showing them — and it healed by
refilling the canvas each frame, which compounds under `source-over` rather than
settling at a target alpha, so the sheet slowly went black. Screen-blended
colour laid on top read as streaks across half the screen and buried the real
colours. Both were adding something; what was wanted was taking a little away
from everything except what you are looking at.

Fine pointers only; on a phone there is no cursor and a drag is how the grid
scrolls.

The cursor is the painter site's, lifted whole: a ring and a dot, both on the
real pointer, `mix-blend-mode: difference` so one cursor reads on a near-black
wall and on a bright painting without changing colour.

## Memory

Each painting held in the cache costs roughly 8MB — a normal map canvas and its
decoded source. Unbounded that is about a gigabyte across the full set and a
dead tab on a phone, so `CACHE_MAX` keeps only the last few and shrinks the
canvas backing store of the ones it drops.

## Two builds

- `index.html` — the one to show someone. No controls: the picture, the rail,
  and settings fixed at what looked right across the whole set. **Vedi tutte le
  126** opens every piece as a masonry sheet to jump anywhere; the rail is for
  browsing what is nearby.
- `tuning.html` — the same viewer with the sliders and the view modes back,
  for changing those settings.

## Views (tuning.html)

- **Dipinto** — the lit surface.
- **Originale** — the untouched photograph, to A/B what is being added.
- **Rilievo** — the raw normal map. This is the diagnostic: it shows what was
  extracted before any lighting is applied.

The readout under the controls gives the RMS relief for the current painting.
Higher means more material to catch the light.

## What this cannot do

- **The photograph's own lighting is baked in.** Whatever lamp was in the room
  when it was shot leaves a highlight that stays put while yours sweeps. The
  effect is additive; it is not a true relight.
- **Luminance is not height.** A dark stroke and a groove look alike to a band
  pass. In impasto the two mostly coincide, which is why this works at all, but
  hard colour edges read as ridges whether or not they are.
- **Dark paintings give weak relief.** There is not enough luminance range in
  the shadows to recover a surface from. Measured across fourteen: night scenes
  land near 5 RMS against 12 for the brightest, and no slider setting fixes it.

## URL parameters

Handy for linking a setup or taking screenshots:

| | |
|---|---|
| `#i=12` | which painting, by index |
| `#lx=-0.8&ly=-0.4` | park the light |
| `#t=0` | pin the plate flat |
| `#m=2` | view mode: 0 lit, 1 original, 2 relief (tuning.html) |
| `#all=1` | open straight to the sheet of all 126 |
