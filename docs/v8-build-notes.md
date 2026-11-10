# IDENTITY UNKNOWN — Prototype v8

> **Note:** this is the original build record, kept as written. File names refer to the original working folder. In this repo: `prototype_v8.html` is `index.html`, `console terminals/terminal_console_v8.html` is `terminal_console_v8.html`, `prototype_v1.html`–`v7.html` are in `versions/`, the holographic branch `prototype_v1.html` is `branches/holographic-v1.html`, and `ver_1.02.html` is `branches/tracking-v1.02.html`. The `.claude/` settings file mentioned below is not included. Reference screenshots and screencasts are not included.

**Handoff / build record.** Everything that is finished and working in the v8 pair, written so
the two files can be lifted into a new folder and picked up cold.

- `prototype_v8.html` — the rig. 362 KB, 2,976 lines, zero dependencies on disk.
- `console terminals/terminal_console_v8.html` — the detached terminal wall. 36 KB, 870 lines.

---

## 1. What the piece is

A webcam-driven surveillance fiction. The camera is a fixed installation pointed at the subject.
A machine behind it is trying to reconstruct their biometric topology and commit it to an
identity, and it never succeeds. The subject can fight back with two gestures.

Two halves of one loop:

- **Leave it alone and it heals.** With no interference the reconstruction stops losing ground
  and starts taking it — more patches under construction at once, faster knitting, dead ground
  actually recovering. Left long enough it reaches a complete descriptor, commits to a registry
  match, finds nobody at all, and then turns its own filter bank on the model it just built.
- **Reach in and it breaks.** Closing thumb and index over a specific assembly crushes it. The
  picture drops for a fraction of a second, leaving only the amber wire and the red record of
  everything already torn off it, and that region is dark for 20–25 seconds. Nothing probes it,
  no filter runs on it, and the host's own recalibration is refused by hardware.

The governing idea of v8 specifically: **the frame belongs to the subject.** Every readout, every
terminal, every ticker was a reason to look somewhere other than the face, so all of it was moved
off the rig and onto a second screen. What is left on the rig is the subject, the reconstruction,
the probe boxes, and a recording light.

---

## 2. Files to move

### Required

| File | What it is |
|---|---|
| `prototype_v8.html` | The rig. Self-contained: embedded Ubuntu Mono TTF (229 KB base64 on line 10), all CSS, all JS inline. |
| `console terminals/terminal_console_v8.html` | The console. Self-contained, no simulation of its own — pure mirror. |

The rig opens the console by **relative path** (`window.open('terminal_console_v8.html', …)`). If
you keep the console in a subfolder in the new project, update that path at
`prototype_v8.html` → `openConsole()`, or put both files side by side.

### Optional, worth taking

| Path | Contents |
|---|---|
| `reference_screenshots/` | 9 loose stills + `facial_recognition/` (incl. `iris_detection/`), `hand_recognition/`, `reference_cycle_filters/` (9), `terminal_organization/` (4), `new/` (2). Visual reference the look was built against. |
| `reference_screencasts/` | 3 `.webm` captures + `random_terminal_sequence/`. |
| `.claude/settings.local.json` | Permission allowlist for the headless-Chrome + fake-video-capture screenshot harness used to test this without a real webcam. Paths inside are stale session dirs — rewrite them. |

### Not needed

`prototype_v1.html` … `prototype_v7.html`, `console terminals/terminal_console_v7.html`. Each is a
complete standalone snapshot; keep them archived if you want the lineage, but nothing in v8
references them.

---

## 3. Running it

1. Open `prototype_v8.html`. Works from `file://` or a local server.
2. Click **`> INITIALIZE OPTICAL SENSOR`**. Grants camera, loads models, boot log fills in.
3. Press **`[C]`** to pop the console onto a second monitor.

**Network:** the rig pulls MediaPipe from CDN at boot — `@mediapipe/tasks-vision@0.10.14` bundle +
WASM from jsDelivr, and three `.task` models from `storage.googleapis.com/mediapipe-models`
(face_landmarker float16, hand_landmarker float16, pose_landmarker_lite float16). If the CDN is
unreachable the rig sets `S.degraded` and falls back to `synthTrack()` — a procedural face and two
procedural hands that keep the whole simulation running with no camera intelligence at all. Boot
does not fail. **If the new project needs to run offline, vendoring those four assets is the one
outstanding dependency task.**

**Camera constraints** are all `ideal` (1280×720 @60, `facingMode:'user'`), never `min` — a hard
`min` on frameRate makes `getUserMedia` throw `OverconstrainedError` on cameras that can't meet it,
which denies the sensor outright instead of just running slower. There's a bare-request retry
underneath that.

**Console transports:** `BroadcastChannel('identity_unknown_v8')` is the good path — same origin,
any number of listeners, doesn't care who opened whom. But a `file://` page gets an opaque origin,
and two opaque origins are not the same origin, so on a double-clicked file the channel silently
connects to nothing. Second path: if the console was opened with `[C]`, the rig holds its window
handle and `postMessage`s it directly; the console answers through `window.opener`. Both are wired
and both are fed; duplicates arriving on the other path are dropped by sequence number.

---

## 4. Interaction map

### Gestures (rig)

| Gesture | Effect |
|---|---|
| **Pinch** — thumb + index close over an assembly | Crushes it. Picture drops 150–300 ms to wire-only, that region goes dark 20–25 s. Everything pointed at it dies: filters truncated, seal released, probes ejected `ASSEMBLY CRUSHED BY SUBJECT`. Host opens a `/dev/sensor/<lbl> — FAULT` window that tries to recalibrate and is refused by its own hardware. |
| **Pinch on nothing** | Registers as a miss — small amber shock ring, no crush. The reach reads as aimed rather than random. |
| **Hand across face ≥ 3 s** | Overdrive. Filter bank goes flat out, terminals lose the thread — some kernel-panic and reboot, some are closed outright and reopened as a fresh session. The blocked region itself becomes a filter target: the bank keeps hammering the place where the face was, through the hand. |
| **Stop interfering ≥ 1.4 s** | Regeneration eases in over ~800 ms (and drops in ~120 ms the moment a hand moves). |

Pinch ratio is measured against the hand's own span (wrist → middle knuckle) so it works the same
close to the lens and across the room. Hysteresis (`0.34` close / `0.52` open) keeps a hand hovering
at the threshold from firing every frame. The grip midpoint gets its own 52 ms easing on top of the
hand easing — thumb and index tips are the two noisiest landmarks the model produces, their midpoint
is noisier still, and this gesture aims at things the size of an iris.

### Keys — rig

| Key | Action |
|---|---|
| `C` | Open / focus the terminal console. Opened from a keypress so the browser treats it as user-initiated and doesn't eat it as a popup. |
| `H` | Toggle the HUD (the REC line). |
| `B` | Cycle overlay ink: 100% → 45% → off. |
| `V` | Force a 700 ms optical dropout. |
| `G` | Force overdrive for 4 s without holding a hand up. |
| `F` | Fullscreen. |
| `Esc` | Halt. Stops tracks, kills all terminals, prints the final tally card. |

### Keys — console

| Key | Action | Also |
|---|---|---|
| `T` | Tile | button |
| `C` | Cascade | button |
| `S` | Stack (one column, sorted by category — the reading layout) | button |
| `X` | Clear dead | button |
| `D` | Toggle the dock | button |
| `F` | Fullscreen | button |

Plus per-window: drag bar to move, drag any of 8 edges/corners to resize, wheel on bar (or
alt+wheel) to scale text 60–280%, double-click bar to pin (pinned windows are skipped by
TILE/CASCADE/STACK), double-click a corner to reset size, `✕` to close — which tells the rig to
close it too, so the two stay in step.

Console toggles: **FOLLOW** (auto-place new windows), **AUTOSCROLL** (stick to bottom of scrollback),
**KEEP DEAD** (leave killed windows on the wall as dimmed corpses), **DOCK**.

---

## 5. Rig architecture

Single `requestAnimationFrame` loop, one canvas, one `<video>` held offscreen at 1×1 px.

```
frame()
  ├ fps / dt / smoothing constants
  ├ inferTick()        pick 1–2 detectors by weighted overdue-ness
  ├ easeHands()        slot-stable, velocity-carried hand smoothing
  ├ rebuildROIs()      25 target rects, exponentially smoothed
  ├ pruneLocks / pruneCrush
  ├ pinchTick / occlusionTick / regenTick
  ├ filterChurn + thrashTick     (overdrive only)
  ├ reconProgress ×3 → applyReconDamage → debrisTick
  ├ scheduler()        spawn probes, browsers, fs claws, ambient dropouts
  ├ job.update() ×N  ·  rosterTick()  ·  TBUS.tick()
  └ RENDER: bg → video (mirrored, graded) → filters → face recon → hand recon
            → debris → boxes → crush FX → pinch ring
```

### Regions

25 targets across 8 categories: `gestalt`, `ocular`, `oral`, `cranial`, `aural`, `manus`, `digit`,
`frame`. Each carries a label and a plausible command line (`iris.extract --eye=L --bits=2048`,
`otometry.scan --canal --helix`, `minutiae.lift --finger=1`, …). The three `frame` targets
(acromion ×2, cervical) are `fbOnly` — they only become available when there is no resolvable face.

ROI rects are built from MediaPipe landmark index sets with per-region padding (iris 0.22, eye 0.34,
ear 0.22, everything else 0.14) and smoothed toward the new measurement each frame. There's a hard
guard rejecting any rect larger than 42% of the viewport — that's the fix for the huge empty
rectangles v2 produced. Each ROI also stamps `at`, the last moment it was genuinely real: the pinch
test needs it, because two fingers closing over an eye can cost the landmarker that eye for a frame
or two, and the grab must not evaporate because of that.

### Reconstruction

The green/amber topology is never simply "on." It's assembled patch by patch and torn back off
wherever a filter is currently chewing.

- Face: 6×9 = **54 patches**, landmarks assigned to spatial cells of the face bbox once, on first sight.
- Hands: **6 patches** each (palm + one per digit).
- Four parallel `Float32Array`s per reconstruction: `built`, `damaged` (bright red tear, 900 ms decay),
  `scar` (red residue, 7 s), `necro` (dead tissue, 26 s and barely fading), plus `tears` counting how
  many times each patch has been taken.
- A filter never wipes everything it covers — it tears a random 35–60% of the patches under it
  (×1.5 in overdrive). That's what leaves the topology permanently half-built rather than
  permanently empty.
- Necrosis drags rebuild speed by up to 72%: ground that's been torn off before knits back slower
  every time.
- Below 22% integrity (having once been above ~42%) the whole generation is scrapped and restarts
  from zero — but the scars and necrosis carry over onto the new generation.
- Torn patches throw physical debris shards (max 220, velocity + gravity + decay) so a tear reads as
  violent instead of as a fade.

Visibility is the quiet trick: `VIS_LIVE = 0.16` under a working sensor — a faint amber tracing you
have to look for — easing to `VIS_VOID = 1.0` when the optical channel drops. The reconstruction only
becomes the picture once there's nothing else in the room. The **wound layer is exempt**: red stays
legible at ≥72% no matter what the sensor is doing, because the damage is the record.

The wound render is the most detailed drawing in the file: dashed necrosis borders with animated
dash offset, stipple on the worst patches, torn seams jittered along the rip edge, `x3` tear counters,
double-hatched holes, a ghost of the mesh that used to be there, and embers at the corners the patch
let go of.

### Corruption filters

Seven, all clipped strictly inside the probed rect, all working through a downscaled offscreen
canvas capped at 95,000 px:

`MOSAIC` · `DISSOLVE` · `INTERLACE` · `DATAMOSH` · `BLOOM` · `GRID` · `GLYPH`

`GLYPH` re-renders the region as terminal characters from a pre-baked atlas (`drawImage` is far
cheaper than `fillText`) and is barred from any rect over 46,000 px — on a whole-face rect it would
blank the subject entirely. A cascade fires 1–3 kinds normally, 2–4 in overdrive. Per-frame filter
budget scales with measured fps: 7 / 5 / 3 / 2.

`SLITSCAN` was cut in v7 — the rolling vertical smear read as a liquid ripple across the eyes, nose
and mouth, and ran often enough to become the signature of the whole piece rather than one
corruption among several. Displacement is `DATAMOSH`'s job now, and it tears in hard bands instead
of waving.

### Rules that make it a system rather than a loop

- **Three-second exposure rule.** No probe may sit under a live filter for more than 3 s before it's
  saturated and thrown out. Exposure accrues faster with more overlapping filters (`×(1+0.6(n−1))`)
  and ×1.7 in overdrive, and recovers slowly at 0.75×dt when clear. Any filter whose rect merely
  *touches* the probe counts — a pass sitting above the iris is enough to saturate it. The slow
  ocular passes feel this most, which is the point.
- **One eye at a time.** The iris reader is singular hardware. While an ocular job is live, every
  other ocular target is off the table, left and right alike, and after it ejects there's a
  1.8–3.6 s cooldown before the other eye is even a candidate. So the reader visibly *moves* between
  eyes rather than flickering. Ocular targets are also weighted *under* the ordinary weight in the
  spawn bag: the rig wants the iris more than anything, it just can't have it very often.
- **Crushed ≠ sealed.** A *sealed* region is the host locking the subject out (host countermeasure,
  10–15 s, max 3 concurrent). A *crushed* region is the opposite: the subject reached into the frame
  and closed a hand on a scanner. The host doesn't get to argue — nothing may probe it, no filter may
  run on it, overdrive can't reach through it, and the patches underneath are finally left in peace.
  **A crushed region is also a healing one.**
- **Long passes get re-corrupted mid-flight.** Any probe over 2 s keeps having fresh cascades thrown
  at it every 0.65–1.3 s (0.26–0.6 s in overdrive) — the host isn't corrupting once and walking away.

### The other doors

The rig isn't a face scanner. It's a machine with one question, and when the biometrics refuse it,
it goes at the question from any other angle it has:

- **`openBrowser()`** — 12 fictional hosts (`civil-registry.eu`, `passport-db.gov`,
  `breach-dump.onion`, …), a query built from the partial descriptor, a URL bar with a spinner, and a
  refusal: 403, 429, `robots.txt :: Disallow: /identity`, CAPTCHA `[ cannot comply ]`, 451,
  connection reset. Then mirrors, then blacklisting. Max 2 concurrent.
- **`openCrawl()`** — `find` / `cat` / `mount` / `grep` down 8 root paths looking for a name.
  Permission denied, I/O error, "device is not the subject", `setuid(0)` refused —
  *"this door is not ours."* Max 2 concurrent.

Both run on a slower clock (4.5–11 s, or 1.4–3 s in overdrive) and both are opened immediately the
moment overdrive starts, because that's when it stops trusting the biometric channel entirely. With
no face in frame it leans on them much harder.

Plus `spawnSweep()` when there is nothing biometric in frame at all — a full raster sweep that finds
none of it, keeping the system visibly alive while the subject is out of shot.

### The registry sweep and the final match

`rosterTick()` runs a fake civil-registry compare every 70–150 ms against a 39-surname roster.
Each row goes out on the bus twice: once **pending**, once resolved **FALSE** or **PARTIAL MATCH**
(4.5% chance). A partial opens a `deepcmp` window that aligns geometry, reports 38–71% landmark
overlap, notes the source topology is only 11–44% built, and rejects the candidate.

If the subject leaves it alone long enough to reach **93% integrity**, `attemptMatch()` fires: the
first complete descriptor, queried against 11–44 registries at threshold .94, then .61, then —
*"descriptor is well-formed and matches nobody."* Then it takes it out on the model: cascades on
six facial regions and collapses the topology it just finished building. Cooldown 12 s.

### Detection scheduling

`RATE = {face:40, hand:28, pose:220}` ms. The hand is the fastest of the three, because it's the only
one the subject is actively steering; pose is demoted to a fallback that only matters when there's no
face. Up to **two** detectors may run per frame when fps > 48, and the hand gets a 1.6× weight on
overdue-ness so it wins ties outright.

Hand slots are kept stable by nearest-wrist matching before anything else touches them — the
landmarker doesn't promise the same hand returns at the same index, and a swap mid-gesture would drop
a pinch. Each slot then eases toward the newest sample with a 46 ms time constant and carries 35% of
the sample velocity forward, which both smooths jitter and covers the gap between detections instead
of holding still through it. Slots survive 260 ms without a detection.

---

## 6. The mirror bus

The most portable piece of the build. In v7 this was a second way to read the terminals; in v8 it's
the only way. Costs nothing when no console is listening — nothing is serialised until something says
`hello`, and the rig runs perfectly happily blind into it.

**Channel:** `identity_unknown_v8`. Every message carries `n`, a monotonic sequence number, so a
duplicate arriving on the second transport is dropped rather than written twice.

### Rig → console

| `k` | Payload | Meaning |
|---|---|---|
| `snapshot` | `terms[]`, `ticks[]` | Answer to `hello`. Every live terminal with up to 220 lines of scrollback, plus the last 14 status lines. A console opening mid-session never starts on an empty screen. |
| `open` | `t` (full term snapshot) | New terminal. |
| `write` | `id`, `text`, `cls` | One line. `cls` ∈ `dimx warn ice fail panic num eq hit url deny stamp mark`. |
| `prog` | `id`, `v` (0–1) | Progress bar. |
| `title` | `id`, `v` | Retitle. |
| `url` | `id`, `url`, `spin` | Browser URL bar + spinner frame. |
| `cls` | `id`, `c`, `on` | Toggle a state class (`repelled`, `panic`, …). |
| `glitch` | `id`, `ms` | Timed visual glitch. |
| `wipe` | `id` | Buffer wiped by supervisor. |
| `kill` | `id` | Process gone. |
| `roster` | `r` `{id, nm, st, ct}` | Registry row. `st` ∈ `pending` `no` `hit`. Second message updates the row the first created, so the list reads as a ledger and not as double vision. |
| `tick` | `s` | One status line. |
| `state` | `rage`, `void`, `fps`, `jobs`, `crush`, `locks` | ~10 Hz status. |

### Console → rig

| `k` | Meaning |
|---|---|
| `hello` | On load, and re-offered every 1.8 s until something answers. Triggers `snapshot`. |
| `ping` | Keepalive, every 1.8 s. |
| `bye` | On `beforeunload`. |
| `close` | Operator closed a mirrored window; the rig kills the real process. |

**No geometry on the wire at all.** The rig has no idea how big any of these are and doesn't need
one; the console sizes and places its own. (The console still accepts and ignores `geom` so a v7 rig
can talk to it.)

**Liveness timeouts are deliberately long** — 120 s on the rig, 14 s to grey the console's link
light. A browser throttles timers in a tab it considers hidden, and a console parked on a second
monitor can easily miss several pings in a row. Broadcasting to nobody costs a postMessage into the
void; cutting a live console off costs the whole point of the window.

---

## 7. Console features

Full window manager for the mirrored wall:

- Move, 8-handle resize, wheel-scale, pin, close, reset — all per window.
- **TILE** (square-ish grid), **CASCADE** (staggered, uniform), **STACK** (columns sorted by
  category, the reading layout). Pinned and dead windows are excluded.
- Auto-placement deals arriving windows onto a coarse grid, nudged 22/18 px each full pass so a busy
  rig doesn't stack every window on one spot.
- **1,400 lines of real scrollback per window** (vs 220 held on the rig). Autoscroll sticks to the
  bottom only when you're already at the bottom — if you've scrolled up to read something, a new line
  arriving must not yank it out from under you. Same rule in both dock lists.
- Windows open appreciably larger than the rig ever drew them — 400×250 default, 520×330 compute,
  540×320 browser. That's what the second screen is for.
- Compute windows get a phosphor border and a bright title bar; browser windows get a URL bar.
- A rig kernel-panic **wipes** the rig's buffer; here the history is the value, so the wipe is
  recorded as a dashed rule rather than performed.
- **The dock** (`[D]`) — the two readouts taken off the rig's frame. `INDEX SWEEP / CIVIL REGISTRY`
  (500 rows, struck-through FALSE in dark red, amber glow on PARTIAL MATCH) and `STATUS LOG`
  (400 lines, newest highlighted). Toggling it reflows any window parked in the 302 px strip back
  inside the wall.
- **Three idle states**, told apart on the card: nothing on the channel at all; a rig that has
  answered but hasn't opened a terminal yet (still on its boot screen, or the subject is out of
  shot); and a wall with windows on it, where the card gets out of the way.
- Status bar: link state, window count, total lines, rig fps. The top border goes amber when the rig
  reports overdrive.

---

## 8. Tunables

All at the top of the `<script type="module">` block unless noted.

### Timing (ms)

| Constant | Value | Meaning |
|---|---|---|
| `T_INJECT` | 90–190 | Probe injection |
| `T_PROBE` | 620–1500 | Normal probe duration |
| `T_PROBE_OC` | 1700–2800 | Ocular probe — the slow greedy one |
| `OCULAR_COOLDOWN` | 1800–3600 | Before the single iris path can point at the other eye |
| `T_EJECT` | 170–300 | Ejection |
| `SPAWN_GAP` | 110–330 | Between probe spawns (×0.45 in overdrive) |
| `LOCK_MS` | 10000–15000 | Host seal duration |
| `CRUSH_MS` | 20000–25000 | Crushed assembly offline |
| `CRUSH_FLASH` | 150–300 | Wire-only blackout on impact |
| `PINCH_REARM` | 420 | Before the same hand can crush again |
| `EXPOSURE_MS` | 3000 | Max time under a live filter |
| `OCCLUDE_MS` | 3000 | Hand-over-face hold to trip overdrive |
| `RAGE_TAIL_MS` | 2600 | Overdrive runs on after the hand drops |
| `REGEN_AFTER` | 1400 | Calm before regeneration kicks in |

### Reconstruction

| Constant | Value |
|---|---|
| `FACE_GRID_X` × `FACE_GRID_Y` | 6 × 9 = 54 patches |
| `HAND_PATCHES` | 6 |
| `BUILD_HEADS` / `BUILD_MS` | 5 simultaneous / 85 ms per patch |
| `TEAR_FRAC` | 0.35–0.6 of covered patches |
| `DAMAGE_MS` / `SCAR_MS` / `NECRO_MS` | 900 / 7000 / 26000 |
| `NECRO_DRAG` | 0.72 |
| `COLLAPSE_AT` | 0.22 integrity |
| `MATCH_AT` | 0.93 integrity |
| `VIS_LIVE` / `VIS_VOID` | 0.16 / 1.0 |
| `REGEN_HEADS` / `REGEN_SPEED` / `REGEN_HEAL` / `REGEN_VIS` | +7 / ×1.5 / ×2.4 / +0.10 |

### Gesture

| Constant | Value |
|---|---|
| `PINCH_ON` / `PINCH_OFF` | 0.34 / 0.52 of hand span |
| `PINCH_REACH` | 12 px slop around a rect |
| `PINCH_EASE` | 52 ms |
| `OCCLUDE_FRAC` | 0.30 hand/face overlap |
| `HAND_EASE` / `HAND_LEAD` / `HAND_HOLD` | 46 ms / 0.35 / 260 ms |

### Budgets

`MAX_TERMS` 14 · `MAX_LOCKS` 3 · `MAX_BROWSE` 2 · `MAX_CRAWL` 2 · `MAX_DEBRIS` 220 ·
`DPR_CAP` 1.5 · `MAX_FX_PX` 95,000 · `MIRROR_SCROLLBACK` 220 (rig) / `SCROLLBACK` 1,400 (console) ·
`EXOTIC_BAND` 0.01–0.09, re-rolled every 30–60 s so the compute terminal's own frequency drifts.

### Overlay ink

`setBoxInk(m)` drives all overlay alpha from one multiplier — edges 0.13m (barely an edge), corner
brackets 0.42m (these carry the read), labels 0.40m, scan sweep 0.16m, reticles 0.55m. `[B]` cycles
`[1, 0.45, 0]`. Raise `BOX_A_MUL` to bring the whole overlay back up; drop it to make the boxes
almost subliminal.

### Palettes

`phosphor #4dff88` (weight 44) · `cyan #00e5ff` (26) · `crimson #ff2d3f` (18) · `amber #ffb000` (12),
drawn from a weighted bag. Background `#040705`. Reconstruction amber is its own ramp:
`edgeDim 255,196,54` → `edgeHot 255,222,120` → `node 255,238,180`, wounds
`tear 255,52,64` → `scar 206,32,44` → `necro 122,14,24` → `ember 255,110,70`.

---

## 9. What v8 changed from v7

### Removed from the rig

| Gone | Why |
|---|---|
| **All terminal windows** | Every window was a claim on the rig's rectangle, and the rectangle belongs to the subject. `Term` is now purely the process — title, palette, scrollback, progress, state flags. It owns no element, has no position or size, and is never drawn. Nothing about the simulation changed; the same processes run on the same schedule and say the same things, they simply say them somewhere else. |
| **Twelve of thirteen HUD rows** | Six rows top-left, six top-right, a clock, a key legend. All of it was true and none of it was the point. What survives is the recording light and running time — the one thing on a surveillance frame that's supposed to be there. |
| **The bottom ticker** | 26 seconds a lap meant that if you looked away you lost the line. Now a dock list you read at your own pace. Call sites unchanged, so nothing else had to learn a new habit. |
| **The civil-registry panel** | Moved to the dock. Still runs at exactly the same rate against exactly the same registry. |
| **The face-ID / mesh panel** | It was a second copy of something already on screen at full size, over the actual face. With the corner cleared it's the frame that carries it. The subsystem-fault behaviour went with it — a panel that crashes is only interesting if there's a panel. |
| **Scanline raster, crawl band, vignette, bloom** | The picture is no longer lit by something that isn't in the room. |
| **Occlusion countdown, overdrive banner, blackout caption, regeneration readout** | All describing something already visible. Overdrive is not a subtle event — the filter bank going flat out *is* the announcement. |
| **The link canvas** | Went with the windows it joined. |
| **Window grab/carry, two-hand pull-open** | No windows in the frame, so the pinch means one thing again. Only the 52 ms smoothing constant survives from that machinery, because aiming at an iris still needs it. |
| **The DOM write queue** | Existed to keep ten windows' worth of `insertBefore` churn from eating the frame. With nothing to insert into, a write is a push and a postMessage. |

### Kept — half the picture

Sensor grain, at roughly half its old weight. A real sensor has some, and a perfectly clean frame
reads as a render rather than as a camera. It's the only post layer left, and it does not move the
picture.

### Added

- The console became the **only** readout, not a second one — and gained the dock, three idle states,
  a `snapshot` handshake with real scrollback for late joins, and window pinning.
- `MAX_TERMS` raised 10 → 14. Ten was a screen-space budget; the console has a whole screen and lays
  out its own wall. It's still a cap, because each one is a running simulated process.
- Sequence-numbered dedup across both transports.
- Roster and ticker moved onto the bus as first-class message kinds.

---

## 10. Known rough edges

1. **Dead guard in `spawnSweep()`** — `if(document.querySelectorAll('.term').length>3) return;` is a
   leftover from when terminals had DOM elements on the rig. In v8 there are none, so it always
   evaluates 0 and never fires. Harmless, but it means the intended cap on concurrent sweep windows
   isn't in effect. Swap for `openTerms.filter(t=>!t.gone && t.cat==='sweep').length>3`.
2. **`Term.slam()` is now just `kill()`** — the rig has no closing animation left to play, so the
   distinction only exists for the console, which plays its own. Fine as is; worth collapsing if you
   simplify the class.
3. **CDN dependency at boot** (see §3). The degraded fallback is genuinely good, but it is a fallback.
4. **The console path is relative** (see §2).
5. **No persistence.** Every session starts from zero; nothing is written anywhere. `Esc` prints the
   tally to canvas and that's the end of it.
6. **Two hands max**, hard-coded throughout (`S.pinch` is a 2-tuple, `HAND_SLOTS` is length 2,
   targets go `_0`/`_1`).
7. **`.claude/settings.local.json` paths are stale** session directories from an older run.

---

## 11. Notes for the new project

The pieces here that generalise, roughly in order of how much work they'd save:

- **The mirror bus** (§6) — a complete, transport-redundant, late-join-safe one-way mirroring
  protocol between two browser pages that works on `file://`. ~70 lines on each side. Nothing about
  it is specific to this piece.
- **The console** — a genuinely usable floating window manager with tiling, scrollback, and
  pinning, that holds no state of its own. Point it at a different message vocabulary and it's a
  general-purpose process wall.
- **The reconstruction model** — four decaying damage layers over a patch grid, with rebuild speed
  dragged by accumulated necrosis. It's a good general shape for "a thing being built and unbuilt at
  once," and it's ~120 lines.
- **The hand pipeline** — slot-stable nearest-wrist matching, velocity-carried easing, span-normalised
  pinch with hysteresis. This is the part that took the most iteration to stop looking like a
  staircase, and it's independent of everything else in the file.
- **The exposure/cooldown/singular-hardware rules** (§5) — the reason the piece reads as a system
  with constraints rather than a loop with randomness. Worth keeping the *shape* of even if the
  fiction changes entirely.

---

*Record of prototype v8, completed 2 Aug 2026.*
