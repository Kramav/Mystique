# Phase 0 Research — Display Approach & Vision System

Deliverable for Phase 0 of [PLAN.md](../PLAN.md). Compares the four display candidates, recommends an architecture for both the outer face display **and** the camera pass-through vision chain, and gives a rough priced BOM. All prices are USD, mid-2026, and approximate — verify at order time (this is the Phase 0 budget checkpoint; no hardware ordered yet).

---

## TL;DR recommendation

- **Face display, v1 (buildable path): flexible AMOLED wrap.** Off-the-shelf flexible AMOLED panels now ship with plug-and-play MIPI-to-HDMI driver boards (DFRobot 6.67", ~$199; Wisecoco 7.8", ~$200). Bright (450 cd/m²), high-PPI (300+), curve on one axis. Content = pre-rendered face video loops. This is the lowest-risk way to get a curved, bright, semi-realistic face on the shell.
- **Face display, realism ceiling (pursue in parallel, treat as v2): rear projection** onto a translucent face-shaped mask — the Furhat approach. It's the only option that puts the face on true 3D geometry, which is what actually sells "a real head." But in-helmet projection has two hard problems (throw distance and daylight brightness) that make it a research project, not a v1 commitment. See risks below.
- **Full 360° coverage is a Phase 3+ goal, not v1.** Wrapping the entire head in display is expensive (~$600+ in panels) and exceeds one compute board's outputs. For v1, make the **front + partial sides** a display and the **crown/back a static printed shell**. Nobody reads the back of the head as the "face."
- **Vision chain: Raspberry Pi CSI camera → uncompressed → small HDMI/DSI panel, no encoder in the path.** Measured ~34–52 ms glass-to-glass is achievable and lands in the comfortable range (Quest 3 passthrough is ~35–40 ms and people walk around in it). **Keep this on its own dedicated board and battery**, independent of the face-display compute — if the face renderer crashes, the wearer must still see.

---

## Display candidate comparison

| Approach | Realism | Resolution / PPI | Curvature | Brightness | Coverage per unit | Fragility | Rough cost | Verdict |
|---|---|---|---|---|---|---|---|---|
| **Rear projection** onto translucent mask (Furhat-style) | **Best** — face lands on real 3D geometry | Projector-limited (854×480 to 1280×720 typical pico) | N/A (mask is 3D) | Weak — needs dim ambient | Whole front face from one unit | Low (no fragile surface) | ~$100–500 projector + molded mask | **Highest realism, highest technical risk. v2.** |
| **Flexible AMOLED wrap** | Good — bright, curved, high-PPI, but reads as "screen-face" | 2400×1080 @ ~400 PPI (DFRobot); 1920×1440 @ 308 PPI (Wisecoco) | One axis only (horizontal); do **not** bend the other axis | **Strong** (450 cd/m²) | Front + partial side per panel | High (thin, bendable, easy to crack) | ~$199–200 per panel + board | **Recommended v1 face.** |
| **Tiled flat LCD facets** | Fair — angular *Prey* look, visible bezels/seams | 1080×1080 round or 1080×1920 per 5–5.5" panel | None (flat facets) | Good (350 nit) | One facet per panel | Medium | ~$50–70 per panel + ~$25 board | Robust, cheap, most "Prey-accurate," but seams and flatness hurt realism. Solid fallback. |
| **HD LED matrix (P1.5–P2)** | Poor for faces — too coarse up close | ~0.6–1 mm pixel pitch | Curvable | Very strong (outdoor-rated) | Depends on tile count | Medium | Varies | **Only** if you abandon realism for a stylized glowing face. Not recommended. |

### 1. Rear projection (Furhat approach)
How Furhat does it: a small projector sits at the top of the head and bounces the image off a **mirror** onto the back of a translucent proprietary-polymer mask; a software face engine renders expressions/gaze/lip-sync in real time; masks are magnetically swappable and the projected persona can change in a split second. This is exactly the "become anyone" behavior we want.

Why it's the realism ceiling: the face is displayed on actual face-shaped 3D geometry, so it has real depth — no other option does. Persona switching is instant and free (just change the rendered image).

Why it's risky inside a head-sized shell:
- **Throw distance.** A normal pico projector can't focus a face-sized image over the ~15–20 cm available inside a helmet. Furhat solves this with a folded mirror path and custom optics. We'd need the same mirror fold or an ultra-short-throw module — this is the single biggest engineering unknown.
- **Brightness / ambient light.** Projected faces wash out above ~100 lux. Fine indoors and at night; poor in daylight. Pico modules in our range put out ~80–300 lumens.
- Module options found: TI **DLPDLCR2000EVM** (854×480, ~$99, Pi/BeagleBone-friendly), TI **DLPDLCR3010EVM-G2** (1280×720, higher cost), or a consumer pico like the **AAXA P5** (300 lumens). The DLP2000 EVM is the cheap way to prototype the optics before committing.

### 2. Flexible AMOLED wrap — **recommended v1**
The key new fact from this research: flexible AMOLED is no longer exotic. Two hobbyist-ready options with **plug-and-play MIPI-to-HDMI driver boards**:
- **DFRobot 6.67"** flexible AMOLED, 2400×1080, 1.2 mm thick, 450 cd/m², bundled MIPI-to-HDMI board, works with Pi/Orange Pi/LattePanda — **~$199**.
- **Wisecoco 7.8"** flexible AMOLED, 1920×1440, 308 PPI, HDMI driver board — **~$200**.

Strengths: bright enough for indoors and shade, high enough PPI to read as a face at arm's length, and each panel is a standard HDMI sink so the Pi drives it with zero custom electronics. Content is just video — pre-rendered face loops play back cheaply.

Watch-outs: **curves on one axis only** (horizontal wrap around the face is fine; compound/spherical curvature is not — bending the wrong axis destroys the panel). Thin and crack-prone, so it needs a rigid backer and a protective outer window. A single panel wraps the front and a bit of each cheek; two panels get you front + sides.

### 3. Tiled flat LCD facets — fallback
Cheapest path and the most literally *Prey*-accurate (angular faceted screens). Parts are everywhere: **Waveshare 5" DSI** (720×1280, ~$58), **Waveshare 5" HDMI round** (1080×1080), **FanyiTek 5" round** (1080×1080, 350 nit), **Youritech 5.5"** (1080×1920, HDMI-to-MIPI board aimed at VR HMDs), plus **Panox** custom HDMI-to-MIPI boards. Downsides for realism: panels are flat, so a face across facets looks segmented, and there are **bezels/seams** between tiles. Good if you lean into the stylized look; weaker if you want "semi-realistic head."

### 4. HD LED matrix — not recommended
Curvable and outdoor-bright, but even P1.5 (1.5 mm pitch) is far too coarse to read as a realistic face at conversational distance. Only viable if the aesthetic goal shifts to a deliberately pixelated/glowing face.

---

## Vision system (camera pass-through)

This is a self-contained mini-project and, from a safety standpoint, the **more important** of the two systems — a face-display failure is cosmetic; a vision failure blinds the wearer inside a sealed shell.

### Latency findings
- **Target validated by VR:** OptoFidelity measured photon-to-photon passthrough at **~11 ms (Vision Pro)** and **~35–40 ms (Quest 3)**. Quest 3 is described as "noticeably high" but people walk around in it comfortably — so **under ~50 ms is the comfort zone**, and our plan's ~50 ms goal is right.
- **Achievable on a Pi:** raspivid-class uncompressed capture measures **~34–35 ms at 1280×960 and ~50–52 ms at 1080p**, glass-to-glass. A naive/default setup is **~120–200 ms** — unusable, nauseating.
- **The killer is encoding.** H.264 encode adds ~40 ms per frame. **Never put an encoder in the pass-through path** — capture straight to display, no compression, no network hop.

### Recommended vision chain
- **Camera:** Pi-ecosystem CSI camera module (CSI is far lower latency than USB webcams). Wide-FOV (~120°+); global shutter preferred to cut motion smear during head turns. Start **mono** (one center camera) — one stream, no stereo alignment headaches; revisit stereo only if depth perception proves inadequate for stairs/curbs.
- **Display:** small high-PPI DSI/HDMI panel a few cm from the eyes. Close panels need headset-style optics (Fresnel lens); a slightly farther "dashboard" panel avoids optics but gives a narrower field of view. Prototype both in Phase 2 before the shell is final.
- **Compute:** its **own dedicated board and battery**, physically separate from the face-display compute. A Pi 5, or even a Pi Zero 2 W / CM-class board doing nothing but capture→display, so nothing else can starve or crash it.

---

## Compute

- **Raspberry Pi 5 (8 GB), ~$80.** Dual micro-HDMI, drives two independent displays up to 4K@60. Use **X11 (not Wayland)** for reliable independent per-display resolutions. Enough to play back pre-rendered face video loops on 1–2 panels.
- **Key constraint:** the Pi 5's **two HDMI outputs** cap it at two HDMI-fed face panels. A third display surface needs a second Pi, a DisplayLink adapter, or DSI panels alongside HDMI. This is the real ceiling on coverage per compute board and is why full-wrap coverage is deferred.
- **Jetson Orin Nano Super (~$249)** only if we move to **real-time 3D face rendering** (rigged head, live lip-sync, gaze) rather than pre-rendered loops. 3× the price; skip it for v1 unless the video-loop approach proves too limiting.
- **Recommendation:** one Pi 5 for the face displays, a **separate** small board for vision. Pre-rendered video loops in v1; reserve Jetson + real-time 3D for a later revision.

---

## Coverage math (why 360° is deferred)

A head is roughly 55–60 cm in circumference. One 6.67" flexible panel is ~147 mm wide, so **three** panels side-by-side (~44 cm) cover front + both sides but not the full wrap — and three HDMI panels already exceed one Pi's two outputs. Full 360° coverage therefore means ~$600 in panels **plus** multiple compute boards. Projection sidesteps this for the front (one unit covers the whole face) but doesn't help the back. **v1 conclusion:** display the front + partial sides; make the crown and back a static printed shell.

---

## Rough BOM

### Path A — Flexible AMOLED face (recommended v1)
| Item | Qty | Unit | Subtotal |
|---|---|---|---|
| DFRobot 6.67" flexible AMOLED + MIPI-to-HDMI board | 2 | $199 | $398 |
| Raspberry Pi 5 8GB (face compute) | 1 | $80 | $80 |
| Pi CSI wide-FOV camera module | 1 | $30 | $30 |
| Vision compute board (Pi Zero 2 W / spare Pi) | 1 | $50 | $50 |
| Internal vision display (5" DSI panel) | 1 | $58 | $58 |
| USB-PD power bank (belt-worn, ~20 000 mAh) | 1 | $50 | $50 |
| Shell filament, mounts, fans, wiring, acrylic window | — | — | ~$120 |
| **Path A total** | | | **~$786** |

### Path B — Rear projection face (v2 / parallel research)
| Item | Qty | Unit | Subtotal |
|---|---|---|---|
| TI DLPDLCR2000EVM pico projector (prototype optics) | 1 | $99 | $99 |
| Front-surface mirror + mount (fold the throw path) | 1 | ~$30 | $30 |
| Molded translucent face mask (cast resin / vacuum-form) | — | — | ~$60 |
| Raspberry Pi 5 8GB | 1 | $80 | $80 |
| Vision subsystem (camera + board + display, as Path A) | 1 | ~$138 | $138 |
| Power, shell, mounts | — | — | ~$150 |
| **Path B total** | | | **~$557** |

Path B is cheaper on paper but the mask + mirror + throw-optics work is unpriced R&D time; that's where the real cost is.

---

## Biggest open risks (rank-order before Phase 1)

1. **Projection throw distance inside a head shell** — the make-or-break unknown for the realism path. Bench-test a DLP2000 + mirror fold onto a test mask *before* committing to projection.
2. **Vision latency under real load** — hit and hold <50 ms glass-to-glass with the actual camera+display combo, walking, before trusting the helmet on anyone.
3. **Flexible-panel curvature limit** — one-axis only; confirm the desired face wrap fits a single-axis bend before buying.
4. **Thermal** — projector or LCD driver boards + a Pi in a sealed head shell get hot; every phase needs a temperature check with fans running.
5. **Weight/balance** — keep it under ~2 kg and front-heavy is worse than evenly distributed; battery stays on the belt.

---

## Sources

Display / projection:
- [Furhat — Wikipedia](https://en.wikipedia.org/wiki/Furhat)
- [Goo Systems: rear projection for Furhat](https://goosystemsglobal.com/screen-goo-rear-projection-helps-one-of-the-worlds-most-advanced-social-robots-with-personality/)
- [TI DLPDLCR2000EVM ($99 pico projector EVM)](https://www.ti.com/tool/DLPDLCR2000EVM) · [TI DLPDLCR3010EVM-G2 (720p)](https://www.ti.com/tool/DLPDLCR3010EVM-G2)
- [DFRobot 6.67" flexible AMOLED ($199)](https://www.dfrobot.com/product-3113.html) · [CNX Software write-up](https://www.cnx-software.com/2026/05/08/6-67-inch-flexible-amoled-display-works-with-raspberry-pi-lattepanda-sbc-hdmi/)
- [Wisecoco 7.8" flexible AMOLED + HDMI board](https://www.amazon.com/wisecoco-Flexible-Bendable-Enclosure-Prototype/dp/B0C7YX94CK)
- [Waveshare 5" DSI LCD](https://www.waveshare.com/wiki/5inch_DSI_LCD) · [Waveshare 5" HDMI round 1080×1080](https://www.waveshare.com/5inch-1080x1080-lcd.htm) · [Youritech 5.5" HDMI-to-MIPI VR board](https://www.youritech.com/products/5-5-inch-10801920-1080p-hdmi-to-mipi-dsi-driver-board-for-3d-printer-vr-hmd-ar.html)

Vision / latency:
- [befinitiv — Raspberry Pi camera latency analysis](https://befinitiv.wordpress.com/2015/09/10/latency-analysis-of-the-raspberry-camera/)
- [Raspberry Pi forums — minimal camera capture latency](https://forums.raspberrypi.com/viewtopic.php?t=376251)
- [OptoFidelity via MIXED — Quest 3 vs Vision Pro passthrough latency](https://mixed-news.com/en/quest-3-vision-pro-passthrough-latency-measurement/)

Compute / prior art:
- [Raspberry Pi 5 dual-display screen configuration](https://thinkrobotics.com/blogs/learn/how-to-use-screen-configuration-on-raspberry-pi-5-a-step-by-step-guide)
- [Raspberry Pi 5 vs Jetson Orin Nano (2026 pricing)](https://thinkrobotics.com/blogs/learn/nvidia-jetson-orin-nano-vs-raspberry-pi-5-the-ultimate-edge-computing-showdown)
- [Instructables — Daft Punk helmet with programmable LED display](https://www.instructables.com/Building-a-Daft-Punk-helmet-with-programmable-LED-/)
