# IDENTITY UNKNOWN — Prototype ver.1 · Holographic Branch

> **Note:** this is the original build record, kept as written. File names refer to the original working folder. In this repo: `prototype_v8.html` is `index.html`, `console terminals/terminal_console_v8.html` is `terminal_console_v8.html`, `prototype_v1.html`–`v7.html` are in `versions/`, the holographic branch `prototype_v1.html` is `branches/holographic-v1.html`, and `ver_1.02.html` is `branches/tracking-v1.02.html`. The `.claude/` settings file mentioned below is not included. Reference screenshots and screencasts are not included.

**Handoff / build record.** `prototype_v1.html` — 2,043 lines, 75 KB, one file, no assets on disk.

Descended from the v8 pair (`PROTOTYPE_V8_HANDOFF.md`), but it is a different piece. v8 was a
machine tearing a face apart. This one builds the face, finishes, gets out of the way, and then
hands the model back to the subject.

---

## 1. What changed from v8

### Cut — every effect that fired while it scanned the face

| Gone | Was |
|---|---|
| **The seven corruption filters** | `MOSAIC` `DISSOLVE` `INTERLACE` `DATAMOSH` `BLOOM` `GRID` `GLYPH`, the offscreen downscale canvas, the per-frame filter budget, the cascade scheduler, the re-corruption of long passes. |
| **The damage model** | `damaged` / `scar` / `necro` layers, tear counters, torn seams, stipple, embers, the mesh ghost, `NECRO_DRAG`, `COLLAPSE_AT`, and the 220-shard debris system. |
| **Probes and the host** | 25 ROI targets, the probe lifecycle, the three-second exposure rule, one-eye-at-a-time ocular hardware, seals, crush, the pinch gesture, optical dropouts, overdrive. |
| **The other doors** | `openBrowser()`, `openCrawl()`, `spawnSweep()`, the 39-surname registry, `attemptMatch()`. |
| **The console** | Terminals, the `identity_unknown_v8` mirror bus, both transports, the window manager. There is no second screen and nothing is serialised anywhere. |

The reconstruction is now only the honest half of the v8 loop: a thing being assembled, which
finishes.

### Kept

Patch-by-patch reconstruction. Slot-stable nearest-wrist hand matching with velocity-carried
easing. Span-normalised gesture reads. The amber/crimson ink. A frame with almost nothing on it —
a recording light and one status line.

### New — the projection

Once the topology is complete, an open hand held into frame is given the model back: a rotating
hologram of the subject's own facial structure, standing over the palm.

---

## 2. Running it

Open `prototype_v1.html` and click **`> INITIALIZE OPTICAL SENSOR`**. Works from `file://` or a
local server.

`?synth=1` skips the camera and the network entirely and runs the procedural track — the whole
piece, demonstrable on a machine with no webcam. Useful for showing it; also how it was tested.

**Network at boot:** `@mediapipe/tasks-vision@0.10.14` from jsDelivr plus two `.task` models from
`storage.googleapis.com/mediapipe-models` (face_landmarker float16, hand_landmarker float16).
Unreachable CDN sets `S.degraded` and falls back to the procedural track. Boot does not fail.
Vendoring those three assets is the one outstanding dependency task if this has to run offline.

Camera constraints are all `ideal`, never `min`, with a bare-request retry underneath — a hard
`min` on frameRate makes `getUserMedia` throw `OverconstrainedError` on cameras that cannot meet
it, which denies the sensor outright instead of just running slower.

---

## 3. The three states

### Mapping

First clean sight of a face fixes the topology and the reconstruction starts. 54 patches (6 × 9
over the face bbox), five under construction at once, 165 ms each — roughly **2.5 seconds** end to
end. Every landmark is assigned to a patch once, on first sight, so an edge only draws when both
of its endpoints' patches are built. The mesh knits rather than fades in.

Visible only in this phase: the acquisition bracket, the scan line, `TOPOLOGY nn%`, the
under-construction patch cells, and the progress rule.

Lose the face for **3.4 s** and the topology starts coming apart over 2.2 s; walk back in and it
re-knits from zero. `[R]` forces that by hand.

### Mapped

At 100 % the read is finished, so the drawing gets out of the subject's way: mesh ink eases from
1.0 down to **0.115**, contour ink to **0.165**, over about 600 ms. The face stays mapped, but you
now have to look for it. This is the point of the phase — the topology is a trace over the
subject, not a thing covering them.

### Projection

Open a hand — three of four fingers extended, fingers spread past 0.62 of the hand's own span —
and hold it for **520 ms**. A ring gathers over the palm, a bright dot carries the model along an
arc from the face to the hand, and the projection materialises over 980 ms.

| | Anchor | Scale | Steering |
|---|---|---|---|
| **One hand** | 3.3 spans above the palm, clear of the fingertips | 1.7 × hand span | Sweep the hand sideways |
| **Two hands** | Midpoint, lifted by half the separation | 0.46 × separation — **pull apart to grow** | Raise one palm and drop the other; it turns like a wheel |

Close the hand or drop it out of frame and it fades in 460 ms. `[G]` forces one at screen centre
for six seconds without a hand, for demonstration.

**There is no back of a head.** The landmarker only ever saw the front, so the projection does not
spin — it *turns*: an autonomous sweep of ±0.68 rad that the subject can push to ±1.24 with their
hands, which drifts back to the sweep over ~2.6 s when they hold still.

The model is not a photograph. It eases toward the live face every frame it can see one, so the
projection turns when the subject turns and moves when they speak.

---

## 4. The face mesh

Two inks, always: **amber** for the tessellation, **crimson** for the features — face oval, both
eyes, both brows, outer and inner lip rings, the nose ridge and base, and an iris ring with a hot
centre in each eye.

The topology comes from one of two places:

- **Canonical.** If MediaPipe loads, `adoptTopology()` takes `FaceLandmarker.FACE_LANDMARKS_TESSELATION`
  (2,556 directed connections → **1,322 unique edges**) and its contour sets. Authoritative; no
  index guessing anywhere.
  MediaPipe's connection lists are **not consistently wound** — an eye arrives as an upper lid and
  a lower lid pointing at each other, so following `start → end` snaps every ring into two
  half-arcs. `chainLoops()` walks them undirected and from both ends of the seed, which returns
  the oval as one ring of 36, each eye as one of 16, LIPS as two of 20 (outer and inner) and each
  brow as the two open 5-point arcs they actually are.
- **Derived.** No library, or a procedural subject: `deriveMesh()` runs Bowyer–Watson over the
  landmark cloud and culls it back to the face — alpha-shape cull at 11.5 % of the face diagonal,
  then point-in-polygon so nothing sits outside the rim, across an eye, or over the mouth opening.
  Coincident landmarks (the iris quads) get a deterministic hundredth-of-a-pixel jitter, and a
  degenerate triangle is treated as *bad* rather than skipped, or it survives forever throwing an
  edge across the face. ~1,245 edges. Runs once, at most every 400 ms while it is failing.

Either way an edge is a pair of landmark *indices*, so it deforms correctly forever after.

Drawing is bucketed: seven build-level buckets means the whole mesh is seven strokes per frame
instead of thirteen hundred.

---

## 5. The hands

Slot-stable nearest-wrist matching runs before anything else touches them — the landmarker does
not promise the same hand returns at the same index, and a swap mid-gesture would drop the
projection. Each slot eases toward the newest sample with a 46 ms time constant and carries 35 %
of the sample velocity forward, which both smooths jitter and covers the gap between detections
instead of holding still through it. A slot survives 260 ms without a detection.

The fast mapping re-runs **every single time a hand enters frame** — bones knit outward from the
wrist, staggered, complete in about 280 ms. On top of that:

- **A scan corona** — three concentric arc rings standing off the palm, counter-rotating, with
  tick marks. This is the read from the second reference image.
- **An arrival flare** that snaps outward as the hand lands.
- **A re-scan sweep** across the hand whose speed scales with how fast the hand is travelling, and
  which lights the joints it crosses. Move faster, the rig works harder to hold you.
- **Motion echoes** — up to seven ghost skeletons at 42 ms intervals, above 40 % of full speed.
- **Fingertip trails**, brightening with speed, crimson on the thumb.
- Crimson palm web, fingertip rings, an oriented reticle, and a live `Δ / SPR` readout that becomes
  `MAP nn%` while the hand is still knitting.
- Once the topology is complete, an open hand is told what it is for: a pulsing `HOLD`.

---

## 6. The projection, drawn

In order, most of it under `globalCompositeOperation = 'lighter'`:

1. **The volume** — 18 vertical light shafts with 6 % overlap, each with its own falloff. A single
   filled polygon has two hard diagonal edges and reads as a grey wedge instead of as light.
2. **The emitter** — a hot core in the palm, white through amber.
3. **Base rings** — four concentric plates squashed to 0.26, counter-rotating, with 24 tick marks
   each. The plate the projection stands on.
4. **The mesh**, depth-bucketed into seven bands. Turned side-on, a triangulation derived in the
   plane throws slivers across the frame, so anything stretched past a fifth of the head is
   dropped — it is not structure, it is the projection seen edge-on. Contours get the same guard
   at 0.30.
5. **The contours**, crimson, depth-shaded.
6. **The irises**, brightest things in the frame.
7. **The sweep plane** — a soft lens, tapering to nothing at the crown and the chin, lighting the
   vertices it passes.
8. **Motes** rising through the volume.
9. **Glyph columns** with leader lines, kept in the upper half because the lower flanks are where
   the subject's own hands are.
10. **The transfer arc**, once, at birth: face → palm.

Nothing is printed below the base plate. That is where the hand is. The cradle mode is said once,
on the banner, when the projection commits.

---

## 7. Keys

| Key | Action |
|---|---|
| `H` | Toggle the HUD |
| `B` | Cycle overlay ink 100 % → 45 % → off |
| `N` | Toggle sensor grain |
| `R` | Reset the reconstruction and re-map |
| `G` | Force a projection at screen centre for 6 s, no hand needed |
| `F` | Fullscreen |
| `Esc` | Halt. Stops tracks and prints the tally card. |

---

## 8. Tunables

All at the top of the `<script type="module">` block.

### Timing (ms)

| Constant | Value | Meaning |
|---|---|---|
| `RATE` | face 34 / hand 22 | Detector cadence. Hand is fastest — it is the only thing being steered. |
| `BUILD_MS` / `BUILD_HEADS` | 165 / 5 | Per patch / simultaneous |
| `LOST_GRACE` / `DECAY_MS` | 3400 / 2200 | Before the topology lets go / how long that takes |
| `FADE_MS` | 1500 | Ink easing once complete |
| `HAND_EASE` / `HAND_LEAD` / `HAND_HOLD` | 46 / 0.35 / 260 | The hand pipeline |
| `BONE_MS` / `BONE_STAGGER` | 46 / 26 | The fast mapping |
| `ECHO_GAP` / `ECHO_MAX` / `TRAIL_MAX` | 42 / 7 / 14 | Motion |
| `HOLO_ARM_MS` / `HOLO_IN_MS` / `HOLO_OUT_MS` | 520 / 980 / 460 | Commit / materialise / release |

### Look

| Constant | Value |
|---|---|
| `FACE_GRID_X` × `FACE_GRID_Y` | 6 × 9 = 54 patches |
| `MESH_A_MAP` → `MESH_A_DONE` | 1.00 → **0.115** |
| `CONT_A_MAP` → `CONT_A_DONE` | 1.00 → **0.165** |
| `HOLO_SPIN` / `HOLO_STEER` | 0.30 rad/s sweep / 0.0042 rad per px |
| `OPEN_MIN` | 3 extended fingers (plus spread > 0.62 span) |
| `FAST_V` | 900 px/s counts as moving |
| `DPR_CAP` | 1.75 |

### Palette

Amber `dim 255,178,40` → `hot 255,214,96` → `node 255,240,190`.
Crimson `line 255,58,72` → `hot 255,126,112` → `deep 188,26,40`. Background `#05070a`.

`[B]` drives all overlay alpha through `S.ink`.

---

## 9. Known rough edges

1. **CDN dependency at boot.** The procedural fallback is genuinely good, but it is a fallback.
2. **The gesture thresholds are set from the procedural track and from geometry**, not from a real
   hand in a real room. `OPEN_MIN` (3 of 4 fingers) and the 0.62 spread guard are the two numbers
   most likely to want a nudge once it has been stood in front of. `window.__IU.S.hands[0]` reports
   `ext` and `spread` live.
3. **The canonical mesh has not been seen on a real face here** — no webcam in this environment.
   Its contour chaining is verified (36 / 16 / 16 / 5+5 / 20+20 loops, exactly right) and the
   render path is shared with the derived mesh, which is verified visually. The remaining risk is
   cosmetic, not structural.
4. **Two hands max**, hard-coded throughout (`HAND_SLOTS` is length 2).
5. **No persistence.** Every session starts from zero. `Esc` prints the tally and that is the end.
6. **The nose contour is still hand-written** (`C_NOSE`, `C_NOSTR`) — MediaPipe does not export a
   nose set separately, only inside the 124-connection `FACE_LANDMARKS_CONTOURS`.

---

## 10. Debug handle

`window.__IU` exposes `S`, `REC`, `HOLO`, `banner()`, a `topology` getter (`canonical` |
`derived`), a `contours` getter (loop sizes and closure), and `feed()` / `feedHands()` /
`synthFace()` / `synthHands()` — which is how the whole thing was driven without standing in front
of the lens.

---

*Record of prototype ver.1, 5 Aug 2026.*
