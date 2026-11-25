# Modal data files · `sfg-modal/1`

Chapter 8 of the Sound Microscope (`analyze.html#modal`) can import a `.json` file with a mode list from a simulation. Use **import .json**, drop the file on the lab, or paste the JSON into "paste a mode list". **export .json** saves the current table and knobs in the same format. **load the Rhodes N30 · D3 file** loads the worked example in `data/rhodes_N30_D3.sfg.json`.

To start a new file, press **download template** (or take `data/template.sfg.json`). It is a filled-in example with an `instructions` block that explains each step, so it can be handed to a person, a script or an AI model together with the solver output, and a `figure.spec` block that describes the drawing to make from the STEP file.

## Fields

| field | unit | meaning |
|---|---|---|
| `format` | | `"sfg-modal/1"` |
| `name` | | shown in the readout |
| `reference.deformation_mm` | mm | each mode is scaled so the **reference point** moves this much (the "1 mm" in the FE viewer) |
| `reference.point` | | where that is. For a pickup, use the tine tip so the levels are what the pickup sees |
| `hammer.impulse_mNs` | mN·s | hammer impulse J. A half-sine force of peak F and length T gives J = 2FT/π |
| `hammer.contact_ms` | ms | hammer contact time (half-sine) |
| `damping.T60_1k_s`, `damping.T60_8k_s` | s | Rayleigh damping, used only for modes without their own `T60` |
| `listen` | | `"displacement"`, `"velocity"` (pickup) or `"acceleration"` (air) |
| `duration_s` | s | length of the synthesised note |
| `strikes` | | optional list of strike points: `[{ "id": "12 mm", "label": "12 mm from the block face", "color": "#ff4f00", "default": true }, …]`. `label` becomes the button's tooltip, `color` (hex) its dot, matching the arrows in the figure |
| `modes[].f` | Hz | mode frequency (required) |
| `modes[].E` | J | strain (= kinetic) energy of the mode at the reference scale. Gives k = 2E/d² and m = k/ω² |
| `modes[].w` | – | motion of the strike point relative to the reference point, **signed**. Either a number, or an object with one value per strike id |
| `modes[].T60` | s | optional decay time for this mode (overrides the Rayleigh knobs) |
| `modes[].label`, `modes[].mode` | | optional name and original mode number, shown in the table |
| `figure.src` | | optional drawing of the part: a path relative to `analyze.html` (e.g. `data/my_part.webp`), an `https://` address or a `data:image/…` URI. Shown in the lab when the file loads |
| `figure.alt`, `figure.caption` | | text alternative (a full sentence for readers who cannot see the image) and the caption under it |
| `figure.spec` | | ignored by the page: the style brief for making the drawing (see below) |
| `modes[].fundamental` | | optional `true` on the mode that is the note: the static screen uses it as ×1 for the harmonic grid (otherwise the first mode) |

Unknown fields are ignored, so an exporter can add its own (for example `group`, `source` or `instructions`).

## From a finite-element solve

With mass-normalised mode shapes φ (φᵀMφ = 1), a listening point *p* and a strike point *s*:

```
m_i = 1 / φ_i(p)²            effective mass seen at p
E_i = ½ · m_i · ω_i² · d²    energy at the reference deformation d
w_i = φ_i(s) / φ_i(p)        signed
```

The page then gives each mode a starting velocity at *p* of `v = J · w · H(ω) / m` (H is the half-sine hammer spectrum). That is exact modal superposition for a point impulse, so the synthesised note is the FE response at the pickup point.

Modes in which the pickup point barely moves (φ(p) ≈ 0) have a huge effective mass and come out silent. Leave them out of the file.

## Minimal example

```json
{
 "format": "sfg-modal/1",
 "name": "my part",
 "reference": { "deformation_mm": 1, "point": "tine tip" },
 "hammer": { "impulse_mNs": 2, "contact_ms": 0.4 },
 "damping": { "T60_1k_s": 4, "T60_8k_s": 0.4 },
 "listen": "velocity",
 "duration_s": 4,
 "modes": [
  { "f": 1161, "E": 0.012, "w": 1 },
  { "f": 2249, "E": 0.05, "w": 0.4, "T60": 1.2, "label": "2nd bending" }
 ]
}
```

## The figure

Each file can carry one drawing of what was simulated, made from the STEP file, in the style of `data/rhodes_N30_D3.webp`. The full brief is in `figure.spec` in the template; in short:

| | |
|---|---|
| canvas | 2000 × 1200 px, WebP or PNG under 300 kB, flat background `#ececea` |
| view | true isometric, slightly from below and in front so the struck face shows; model in the middle ~70 % of the width |
| shading | flat light-grey solids (`#d6d6d2` / `#bdbdb9`, edges `#9a9a96`), rubber `#2b2b2b`, the struck vibrating part darkest `#3a3a3a` |
| kicker | top left, IBM Plex Mono 22 px uppercase `#6d6d6a`: `PART · NOTE · ASSEMBLY` |
| title | Archivo 700, 64 px, `#111`: "where the hammer strikes" |
| callouts | IBM Plex Mono 18 px uppercase with straight 1.5 px leader lines, key dimensions after ` · `, fixed parts marked `(fixed)` |
| datum | dotted line at the zero of the strike distances, e.g. "block face = 0" |
| strikes | one arrow per strike id in the hammer's direction, coloured with `strikes[].color` (grey `#6d6d6a`, default orange `#ff4f00`, blue `#3a5fc4`), distance label under each in the same colour |
| footer | two lines of Archivo 600, 22 px, saying what pushes where and which strike is the reference; bottom right "isometric · not to scale for arrows" |

Save it next to the `.json` file and set `figure.src`.
