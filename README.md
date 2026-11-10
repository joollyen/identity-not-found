# identity-not-found

A webcam piece that simulates a face-mapping adversary trying to build a biometric model of the person in front of the camera, and gives that person gesture-based concealment countermeasures that damage the model and can force the mapping attempt to start over.

**Live demo:** https://joollyen.github.io/identity-not-found/ (needs a webcam; press `C` to open the terminal console in a second window)

> Demo recording: not yet recorded.

## Why I built it

Face recognition is usually discussed from the side of the system doing the recognizing. I wanted to build something from the side of the person being scanned: what a capture pipeline is trying to extract, how it degrades when the subject interferes, and what it looks like when it keeps trying anyway. It is a simulation and an art piece, not a working attack or defense tool.

## Threat model

**Adversary.** The simulated adversary is a fixed surveillance camera with a capture pipeline behind it. Its goal is to build a complete biometric descriptor of the subject (face geometry, iris, ear shape, hand geometry, and fingerprints in the fiction) and match it to an identity. It probes 25 regions of the face, hands, and upper body, runs image filters over them, and when the biometric channel fails it falls back to simulated registry lookups and file searches. Left undisturbed, it slowly completes the model, attempts a match, finds nobody, and starts again.

**Subject.** The subject's only tools are their own hands in front of the camera. They cannot touch the system, only interfere with what it captures. The question the piece explores is how much a person can degrade a capture pipeline with physical concealment alone, and how a system designed to keep trying responds to that. Everything the adversary "extracts" is simulated: the iris, ear, and fingerprint commands are scripted text, the registry is a fixed list of 39 surnames, and the hosts it queries are fictional names that are never contacted.

## How it works

MediaPipe face, hand, and pose landmarkers run in the browser on the webcam feed. The landmarks drive 25 target regions across 8 categories (face outline, eyes, mouth, skull, ears, palms, fingers, shoulders). The adversary schedules probes against those regions, and a reconstruction of the face (a 6×9 grid of 54 patches) and each hand (6 patches) is built patch by patch.

**Countermeasures and how they trigger**

| Countermeasure | Trigger | Effect on the adversary |
|---|---|---|
| Crush | Thumb and index pinch closed over a probed region (pinch measured relative to hand size, with hysteresis so it doesn't fire every frame) | That region goes offline for 20–25 s. Probes and filters on it are ejected and it cannot be probed again until it recovers. |
| Occlusion overdrive | Hand held across the face for 3 s | The adversary escalates: filters run at full rate, simulated processes crash and restart, and it leans on its non-biometric fallbacks. |
| Leave it alone | No interference for 1.4 s | The reconstruction regenerates. |

Damage accumulates. Each filter pass tears 35–60% of the patches it covers, torn patches rebuild more slowly every time, and if the model's integrity falls below 22% (after having been above about 42%) the whole reconstruction is discarded and the mapping attempt restarts from zero, with the earlier damage carried over.

**Second screen.** Press `C` to open `terminal_console_v8.html`. All the adversary's simulated terminal output, the registry sweep, and the status log are mirrored there over a `BroadcastChannel`, with `postMessage` as a fallback when the page is opened from `file://`. The camera window shows only the subject, the reconstruction, and the probe boxes.

Detailed design notes, tunable constants, and the full message protocol are in [`docs/v8-build-notes.md`](docs/v8-build-notes.md).

## Framing: concealment presentation attacks (ISO/IEC 30107)

ISO/IEC 30107-1 defines a presentation attack as a presentation to the biometric capture subsystem intended to interfere with the operation of the biometric system. It distinguishes impostor attacks (trying to be recognized as someone else) from concealer attacks (trying to avoid being recognized). In those terms, every countermeasure here is a concealment presentation attack: the subject uses their own hands as the presentation attack instrument to deny the capture subsystem a usable sample.

The piece inverts the usual point of view. Presentation attack detection (PAD) is normally the defender's job and the attacker is the person in front of the sensor. Here the person in front of the sensor is the protagonist. The simulated system does not implement PAD. It does not try to classify the interference as an attack. It only reacts to it, by escalating.

## Run it

Use the live demo link above, or locally:

```bash
git clone https://github.com/joollyen/identity-not-found.git
cd identity-not-found
python3 -m http.server 8000
# open http://localhost:8000/
```

Click **INITIALIZE OPTICAL SENSOR** and allow camera access. Opening `index.html` directly from disk also works.

| Key | Action |
|---|---|
| `C` | Open the terminal console (allow the pop-up if the browser asks) |
| `H` | Toggle the recording indicator |
| `B` | Cycle overlay opacity |
| `V` | Force a short optical dropout |
| `G` | Force overdrive for 4 s |
| `F` | Fullscreen |
| `Esc` | Stop and show the session tally |

If the MediaPipe CDN cannot be reached, the page falls back to a procedural face and hands so the simulation still runs.

## Privacy and security notes

- Everything runs locally in the browser. No images or video frames are stored, uploaded, or transmitted. There are no `fetch`, XHR, WebSocket, or storage calls in the code, and every session starts from zero.
- The only messages that leave the camera page go to the console page in the same browser, and they carry terminal text and status numbers only, never image data or landmark coordinates.
- The only network requests load the MediaPipe library from jsDelivr (pinned to `@mediapipe/tasks-vision@0.10.14`) and three models from Google's model storage. These are version-pinned but not integrity-checked.
- The hostnames shown in the simulated browser windows are fictional and are never contacted.

## Known limitations

- This is a simulation. It does not perform real biometric extraction or matching, and the countermeasures have not been tested against any real face recognition system. Whether hand occlusion defeats a production system depends on that system.
- Tracking quality depends on lighting, camera, and distance. Pinch targeting small regions such as an eye can be unreliable because thumb and index tips are the noisiest landmarks.
- Two hands maximum, hard-coded.
- Known code issue: a guard in `spawnSweep()` that was meant to cap concurrent sweep windows is left over from an earlier version and never triggers.
- Single-file app with no automated tests.

## Project contents

| Path | What it is |
|---|---|
| `index.html` | v8, the current version |
| `terminal_console_v8.html` | v8's second-screen terminal console |
| `versions/` | Earlier snapshots v1–v7 (and the v7 console) |
| `branches/holographic-v1.html` | A branch that drops the attack loop: once the face model is complete, an open hand displays it back to the subject as a rotating hologram. Notes in `docs/holographic-branch-notes.md`. |
| `branches/tracking-v1.02.html` | A later branch that removes the hologram and keeps only the face and hand mapping. Notes in `docs/tracking-branch-notes.md`. |
| `docs/` | Original build records for v8 and both branches |

## My role

This was built with AI-assisted development. I came up with the concept, tested every version on my own webcam setup, identified what failed in each round, and decided what changed in the next one.

## Next

Planned, not built:

- Subresource Integrity and a Content Security Policy for the third-party library and model loads
- A written comparison against the ISO/IEC 30107-3 testing vocabulary
- Fix the dead `spawnSweep()` guard
- Automated tests

## License

Code: MIT, see [LICENSE](LICENSE). The embedded Ubuntu Mono font is © Canonical Ltd. under the Ubuntu Font Licence 1.0, see [FONT-LICENCE.txt](FONT-LICENCE.txt).
