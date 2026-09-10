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

## Views

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
| `#m=2` | view mode: 0 lit, 1 original, 2 relief |
