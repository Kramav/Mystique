# Mystique — Full-Head Display Mask

A wearable helmet that covers the entire head and displays semi-realistic, animated faces/heads on its outer surface. Visual reference: the full-head masks/helmets from *Prey* (2017) — sleek, sealed, slightly retro-futuristic — but the surface is a display that can become anyone.

This is a plan for Opus to **research first, then build in stages**. Do not skip the research phase; the display-technology decision drives everything else (shell geometry, power, compute, software).

## Goal

- Full-head coverage (front face, sides, ideally crown/back).
- Displays a library of semi-realistic faces/heads that can idle (blink, breathe, micro-movement), switch identities, and eventually lip-sync to the wearer's voice.
- **Wearer vision via outward-facing cameras** feeding a low-latency display inside the helmet — no eye holes or view strips breaking up the outer display surface.
- Wearable: the wearer can see, breathe, and hear; total head weight target under ~2 kg; battery worn on the body, not the head.

## Phase 0 — Research & decide the display approach (the critical decision)

Research each candidate and produce a short comparison (cost, realism, resolution, curvature, weight, power, fragility). Recommend one primary + one fallback.

1. **Rear projection onto a translucent face shell** — a pico projector inside the helmet projects an animated face onto a molded translucent mask (this is how Furhat Robotics achieves realistic robot faces). Best realism per dollar because the face is projected onto actual 3D face geometry; weak in bright light; front-of-face only.
2. **Tiled flat LCD panels** — several 5–7" high-PPI rectangular LCDs (HDMI/MIPI-DSI driver boards) arranged as facets around a helmet shell, *Prey*-style angular aesthetic. Robust and cheap; faces read as "face on a screen" rather than a 3D head; bezels between panels.
3. **Flexible AMOLED panels** — curved wraparound surface, the most futuristic look. Research current hobbyist availability (driver boards for flexible panels are the hard part), cost, and fragility.
4. **High-density LED matrices (P1.5–P2)** — curvable and bright, but likely too low-resolution for "semi-realistic." Evaluate only as the fallback aesthetic.

Also research these prior-art projects and note what they got right/wrong: Furhat Robotics (rear-projected face), Qudi Mask, Daft Punk helmet replicas with LED/LCD builds, cosplay "digital eyes" builds, and any wraparound-screen helmet builds on Hackaday/YouTube.

In parallel, research the **camera pass-through vision system** (this is its own mini-project):

- Glass-to-glass latency budget: aim under ~80 ms, ideally ~50 ms — above that, walking gets nauseating. Research which camera + compute + display chains hobbyists have actually measured at that level (Pi camera via CSI is far lower-latency than USB webcams; avoid anything that pipes through an encoder).
- Camera choice: wide-FOV (~120°+) module(s), global-shutter preferred for head motion; decide mono (one center camera, simpler) vs. stereo (depth cues, harder alignment).
- Internal display: small high-PPI panel a few cm from the eyes needs optics (Fresnel lens, headset-style) vs. a slightly farther "dashboard" screen needing none. Note what FOV the wearer actually gets in each setup.
- Failure mode: if the vision system dies the wearer is blind inside the helmet — this hardens the quick-release requirement and argues for the vision chain being on its own power/compute path, independent of the face displays.

**Deliverable:** `docs/research.md` with the comparison table, a recommended architecture (display tech **and** vision chain), and a rough BOM with prices.

## Phase 1 — Bench prototype (no helmet yet)

- One display unit (chosen tech) driven by the chosen compute board — likely Raspberry Pi 5 or an Orange Pi/Jetson if rendering load demands it. Confirm the board can drive the *final* number of displays before committing.
- Render a single animated face: idle loop with blinking and small head movement. Two viable content pipelines to evaluate:
  - **Real-time 3D**: Godot/Unity with a rigged head (MetaHuman-derived or similar) — flexible but heavy on embedded hardware.
  - **Pre-rendered video loops**: render face states (idle/blink/talk/transition) offline, play back and crossfade on-device — much cheaper at runtime, likely the right call for v1.
- **Deliverable:** a face blinking on a bench display, face-switch on a button press, measured power draw.

## Phase 2 — Shell design

- 3D-scan or measure the wearer's head; design the helmet shell in CAD (Fusion 360/Onshape) around the chosen display layout, with a padded liner and quick-release.
- Build the camera pass-through rig from the Phase 0 research **before** the shell is final: camera(s) on a mock frame + internal display, measure real glass-to-glass latency, and walk around with it. Shell geometry (eye relief, camera placement, fan clearance) depends on what this test settles.
- Camera placement: at outer-eye height so the view feels natural, recessed/disguised so lenses don't break the face illusion (behind a dark acrylic window works and doubles as protection).
- Ventilation (small fans), mic placement, cable routing to a belt/backpack pack.
- **Deliverable:** printed shell that mounts one display, gives camera vision good enough to walk a room confidently, and is comfortable for 20+ minutes.

## Phase 3 — Full integration

- All displays mounted and driven; face content spans/coordinates across them.
- Belt-worn battery (USB-PD power bank sized from Phase 1 measurements), single umbilical to the helmet.
- Face library: 3–5 identities, transition animation between them ("shapeshift" morph), control via phone (simple web UI served from the Pi).
- **Deliverable:** wearable v1 demo — walk around, switch faces from a phone.

## Phase 4 — Stretch

- Mic-driven lip-sync (audio → viseme mapping on-device).
- Eye contact: face's gaze follows nearby people via the external camera.
- Face capture pipeline: photograph a real person → generate a displayable face for the library.

## Constraints & safety (apply to every phase)

- Weight on head < ~2 kg, no battery cells on the head, everything hot or heavy goes to the belt pack.
- Wearer must be able to remove the helmet unassisted in seconds — non-negotiable, since a vision-system failure leaves the wearer blind inside the shell. Keep the camera→internal-display chain on its own power/compute path so a face-display crash never takes out vision.
- Never wear it near stairs, traffic, or drops until the vision system has hours of proven reliability.
- Thermal check every phase: LCD driver boards and projectors in an enclosed shell get hot fast.
- Budget checkpoint at end of Phase 0 before any hardware is ordered.
