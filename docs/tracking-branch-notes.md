# IDENTITY UNKNOWN — ver.1.02 · Tracking Branch

> **Note:** this is the original build record, kept as written. File names refer to the original working folder. In this repo: `prototype_v8.html` is `index.html`, `console terminals/terminal_console_v8.html` is `terminal_console_v8.html`, `prototype_v1.html`–`v7.html` are in `versions/`, the holographic branch `prototype_v1.html` is `branches/holographic-v1.html`, and `ver_1.02.html` is `branches/tracking-v1.02.html`. The `.claude/` settings file mentioned below is not included. Reference screenshots and screencasts are not included.

**Handoff / build record.** `ver_1.02.html` — one file, no assets on disk.

Lineage: v8 (`PROTOTYPE_V8_HANDOFF.md`) → ver.1 (`PROTOTYPE_V1_HANDOFF.md`) → here.
`prototype_v1.html` is kept as the record of the projection branch; nothing in ver.1.02 references it.

---

## 1. What changed

### Cut — the projection, entire

Everything that let the subject hold the model over their palm is gone: the hologram, the cradle
gesture, the arming ring, the light cone, the base plates, the sweep plane, the motes, the glyph
columns, the transfer arc, the `HOLO` state machine, the captured 3D model, and the yaw/pitch/roll
steering. `captureModel()`, `holoTick()`, `drawHolo()`, `seedMotes()`, `tickMotes()` and
`seedGlyphs()` are deleted rather than disabled — about 420 lines. The `[G]` key went with them.

An open hand no longer means anything. The `ext` / `spread` read survives only because it drives
the size of the scan corona, and it is reported as a number rather than acted on.

### Kept exactly

**The ink.** Mesh rests at `MESH_A_DONE = 0.115`, contours at `CONT_A_DONE = 0.165`, and still
only after the whole face is mapped. Hand overlay weight is unchanged. Nothing about the
transparency was touched.

Also unchanged: the patch-by-patch build, the canonical/derived topology split, the fast hand
mapping and everything layered on it, and the fact that no effect fires while the face is read.

---

## 2. What the tracking does now

The tracking *is* the piece, so it was rebuilt rather than tuned.

### The 1€ filter

Both the face and the hands run every landmark through a 1€ filter (Casiez, Roussel & Vogel). A
fixed time constant — ver.1's `HAND_EASE = 46 ms`, and nothing at all on the face — has to pick
one of two things: smooth enough to kill the jitter of a still subject, or fast enough not to lag
a moving one. It cannot be both, and ver.1 picked smooth, which is why a quick turn of the head
dragged behind.

The 1€ filter widens its own cutoff with the measured speed of each coordinate. Held still, the
cutoff collapses and the mesh is glass-steady. Moving, the cutoff opens and the lag nearly
disappears. The derivative that drives it is itself filtered, so the speed estimate is not just
noise.

| | `MIN_CUT` (Hz at rest) | `BETA` (speed → cutoff) | `D_CUT` |
|---|---|---|---|
| Face | 1.5 | 0.016 | 1.2 |
| Hand | 2.4 | 0.035 | 1.4 |

Lower `MIN_CUT` = steadier when still. Higher `BETA` = less lag when moving.

**The face had no smoothing at all in ver.1** — raw landmarks, straight from the detector, held
until the next one. That is where most of the visible jitter was coming from.

### The predictive lead

A detector running at ~45 Hz under a display running at 60 Hz means the landmarks sit perfectly
still for a frame and then jump. Carrying the filtered derivative forward across the remainder of
that gap turns a 45 Hz detector into 60 Hz of motion. Capped (`FACE_LEAD_CAP` 40 ms,
`HAND_LEAD_CAP` 34 ms) so a subject who stops dead does not overshoot.

`[L]` cycles it **NORMAL → HIGH → OFF**, live, on the frame. That is deliberate: it is the one
change here best judged by feel.

### Cadence

`RATE` dropped to **24 ms face / 15 ms hand**, and `adaptRates()` backs both off (×1.3 / ×1.75 /
×2.4) as the frame rate falls, because a stuttering 30 fps tracks worse than a clean 60 fps on
slightly staler samples.

`advance()` fixed a quiet loss: `next = now + rate` snaps the effective cadence to whole frames,
so a 24 ms request became a 33 ms reality at 60 fps — a quarter of the detections thrown away.
Advancing the *target* instead lets it alternate 17/33 and average what was actually asked for.
**Measured face rate went 30 Hz → 46 Hz with no other change.**

### Predictive slot matching

Hand slots are now matched against where a hand is *going* — the filtered wrist plus its
derivative over the elapsed gap — rather than where it last was. The gate also widens with the
slot's own speed. A hand crossing the frame moves a long way between two detections, and matching
on a stale wrist is how a slot gets stolen by the other hand mid-sweep, which drops the whole
knitted skeleton and starts it over.

### Jump guards

A hand that leaves the frame and returns somewhere else is not a hand that moved very fast.
Filtering across that produces an enormous derivative, which the lead then throws further — a
whip-crack, and a fingertip trail streaked across the frame. Both the face and the hands now
detect a jump (>40 % / >34 % of the short screen edge) and **snap**: filter reset, derivative
zeroed, trails cleared, and for a hand, `born` reset so the skeleton knits itself again from the
wrist. You can see it happen — the readout drops to `MAP 0%` and climbs.

### Made legible

- **Face motion echoes** — up to 5 traces at 46 ms, above 520 px/s, drawn at `mA × 0.42 × a²`,
  which at the resting ink is about 0.05 alpha. A whisper. Every other edge only.
- **The ink lifts by at most `FACE_MOTION_A = 0.030`** with speed. The tracker engaging, not a
  brightness change — it is capped well under the resting level so the transparency reads the same.
- **The HUD reports the tracker**: `60 FPS · FACE 46Hz · MANUS 58Hz ×2`, and `LEAD` when it is not
  at default. The hand readout's `SPR` became `TRK`, the wrist speed in px/s.

---

## 3. Measured

The procedural track is closed-form, so at any instant the true face position is known exactly.
Comparing it against what the rig is *drawing*, over 26 s, in pixels, split by whether the face is
moving faster than 400 px/s:

| Lead | Fast: median | Fast: p95 | Still: median |
|---|---|---|---|
| OFF (filter only) | 5.4 px | 19.4 px | 0.9 px |
| **NORMAL (shipped)** | 5.4 px | **11.6 px** | 0.7 px |
| HIGH (1.6×) | 5.4 px | 9.2 px | 0.7 px |

The lead cuts **worst-case** lag — the moments of peak acceleration, which is exactly where a
tracker feels like it is dragging — by about 40 %, and leaves the median alone.

Two honest caveats. The procedural track has **no capture latency**: its "detection" is of the
current instant, where a real camera adds 30–60 ms of exposure and inference that the lead also
helps hide, so the real-world gain should be larger than this table. And it has **no sensor
noise**, which is the thing a high lead amplifies into jitter — which is why NORMAL ships and HIGH
is a key rather than the default.

---

## 4. Keys

| Key | Action |
|---|---|
| `H` | Toggle the HUD |
| `B` | Cycle overlay ink 100 % → 45 % → off |
| `N` | Toggle sensor grain |
| `R` | Reset the reconstruction and re-map (also resets the tracker) |
| `L` | **Predictive lead: NORMAL → HIGH → OFF** |
| `F` | Fullscreen |
| `Esc` | Halt |

`?synth=1` runs the whole piece with no camera and no network. Its procedural face now drifts
slowly and then **turns fast every ~4.8 s** — the drift tests the filter's stillness, the turn
tests its lag. A subject who only ever drifts proves nothing about a tracker.

---

## 5. Tuning it on a real camera

`window.__IU.tune({...})` retunes the filter live, mid-motion, without a reload:

```js
__IU.tune({ handBeta: 0.05 })   // hands lagging on fast moves → raise
__IU.tune({ faceMinCut: 1.0 })  // face shimmering when still → lower
```

Keys are `faceMinCut faceBeta faceDCut handMinCut handBeta handDCut faceLead handLead`.
`__IU.TRK` reports `faceHz`, `handHz`, `faceSpeed`, `lead` and the echo count.

The order that works: get stillness right with `MIN_CUT` first, holding still; then get motion
right with `BETA`, moving; then decide the lead with `[L]`.

---

## 6. Known rough edges

1. **CDN dependency at boot** — unchanged from ver.1. The procedural fallback is good, but it is a
   fallback.
2. **The numbers above are from the procedural track**, not from a real face — no webcam in this
   environment. See the caveats in §3.
3. **`RATE` is mutated in place** by `adaptRates()`; `RATE_FLOOR` holds the requested values.
4. **Two hands max**, hard-coded.
5. The canonical mesh has still not been seen on a real face here. Its contour chaining is verified
   (36 / 16 / 16 / 5+5 / 20+20), and it shares a render path with the derived mesh, which is
   verified visually.

---

*Record of ver.1.02, 5 Aug 2026.*
