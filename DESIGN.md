# Design Document — Gyro: Excavator

## Vision

The most satisfying excavator experience on mobile. Your phone IS the machine — tilt to curl the bucket, feel the soil through haptics, hear hydraulics strain under load. Not an arcade game with an excavator skin. A sim that teaches real dig cycles and rewards efficiency.

**One vehicle, done right.** Ship the excavator. Prove the market. Expand later.

## Core Design Pillars

1. **Phone-as-controller** — Gyro, sticks, and haptics replace a joystick cab
2. **Satisfying digging** — Soil that resists, deforms, and piles up convincingly
3. **Skill ceiling** — Easy to dig a hole, hard to run efficient cycles
4. **Feel over fidelity** — Not a training sim, but every input should feel heavy and deliberate

## Requirements

### Must Have (v1.0)
- [ ] Articulated excavator arm (boom + stick + bucket) with joint limits
- [ ] ISO-pattern controls adapted for phone (see Controls section)
- [ ] Soil deformation — heightmap-based with stamp deformation + angle-of-repose
- [ ] Material types: topsoil (easy), clay (sticky), gravel (resistive), rock (can't dig)
- [ ] Haptic feedback per material — distinct CoreHaptics patterns for each soil type
- [ ] Spoil piles with volume conservation (dirt out = dirt in pile, minus swell)
- [ ] Counterweight/tip-over physics — overreach with loaded bucket = consequences
- [ ] Progressive difficulty: open-site bulk → utility trenching → tight-access residential → demolition
- [ ] Metal renderer (custom, not SceneKit) — shadows, PBR, terrain deformation
- [ ] Spatial audio — hydraulic strain, engine load, bucket impact, soil scraping
- [ ] Bluetooth controller support (MFi/Xbox/PS)
- [ ] Full touch-only alternative for every gyro function (accessibility)
- [ ] VoiceOver-accessible menus and purchase flows

### Nice to Have (v1.x)
- [ ] Track deformation on soft ground
- [ ] Float function (boom drops under gravity for grading)
- [ ] Auxiliary hydraulics (thumb clamp, tilt rotator, breaker)
- [ ] Travel-lock / arm-tuck mechanic for repositioning
- [ ] Night operations with work lights
- [ ] Replay/recording system
- [ ] Leaderboards (cycle time, precision scoring)

### v2.0+ (Future Vehicles)
- [ ] FPV Drone (recon/racing)
- [ ] Fixed Wing Air Tractor (retardant bombing)
- [ ] Helicopter Air-Crane (water bombing)
- [ ] Campaign Mode (multi-vehicle wildfire)

## Non-Goals (v1.0)

- **NOT a flight sim** — No aircraft at launch
- **NOT a sandbox/creative game** — Structured missions with scoring
- **NOT multiplayer** — Single-player first, network architecture not required at launch
- **NOT 240Hz everything** — Per-system tick rates (see Technical Approach)
- **NOT <150MB** — Budget 300MB realistically

## Controls — Adapted ISO Pattern

Real excavators use ISO controls. We adapt for phone:

| Function | Real ISO | Gyro Adaptation |
|----------|----------|-----------------|
| Swing (cab rotation) | Left stick L/R | Left stick X-axis |
| Boom up/down | Left stick U/D | Left stick Y-axis |
| Stick in/out | Right stick L/R | Right stick X-axis |
| Bucket curl/dump | Right stick U/D | **Gyro tilt forward/back** (tilt phone = curl bucket — the signature mechanic) |
| Track L/R | Foot pedals | Dedicated track mode toggle OR context-sensitive |

**Key decisions:**
- **Gyro for bucket curl** — this is the hero mechanic. Tilting to scoop dirt is what makes this feel different from every other mobile game
- Swing stays on left stick X — it's used constantly during dig cycles
- Every gyro function has a touch-only alternative (accessibility requirement — App Store will reject without this)
- Haptic feedback on bucket contact reinforces the gyro→dig connection
- Gyro sensitivity adjustable + recalibrate on double-tap

**Note from expert review:** Reviewers flagged that bucket curl is high-frequency input and gyro may cause fatigue/interference. Counter: bucket curl during the actual dig stroke is a brief, deliberate motion (not constant micro-adjustment). The tilt-to-scoop gesture is the entire USP. Test extensively on device and provide stick fallback.

## Haptic Design

Core Haptics (CHHapticEngine) is a PRIMARY feedback channel, not polish.

| Soil Type | Haptic Feel |
|-----------|-------------|
| Topsoil | Low intensity continuous, gradual onset, low sharpness (~0.2) |
| Clay | High intensity sustained, slow sharpness ramp UP, sudden release snap |
| Gravel | Stochastic transients (20-80ms intervals), medium sharpness (0.5-0.7) |
| Rock strike | Sharp transient, sharpness 0.9+, intensity 1.0, 50ms decaying ring |

Additional patterns: boom stress, track rumble, counterweight shift, engine load.

**Frequency band:** 30-80Hz (heavy machinery feel). Reserve higher bands for future vehicles.

**Thermal budget:** Haptic engine draws 50-200mW sustained. Include in thermal management.

## Soil Deformation — Technical Approach

**Representation:** Heightmap terrain with stamp-based deformation.
- Bucket shape stamps subtracted on contact
- Angle-of-repose post-processing at 30-60Hz (walls collapse if too steep)
- Pre-computed pile shapes for dumping (stamped onto heightmap)
- Volume tracking: hole volume ≈ pile volume × 1.25 (swell factor)

**NOT doing:** Voxel terrain, volumetric soil, SPH particles, real-time continuum mechanics.

**Physics tick rates:**
- Rigid body / arm kinematics: 120Hz (RK4 or semi-implicit Euler)
- Soil deformation: 30-60Hz (GPU compute shader)
- Rendering: 60Hz
- Haptics: Event-driven from physics, CHHapticEngine handles timing

**Bucket-soil interaction forces:**
- Penetration resistance (function of soil type, depth, angle)
- Breakout force
- Curl force under load
- Mass accumulation in bucket (affects CG, arm dynamics)

## Rendering

- **Metal** with CAMetalLayer (not MTKView — need manual frame pacing)
- **TBDR-aware:** memoryless depth/stencil, proper load/store actions, pass merging
- **Shadows:** 2-cascade shadow maps
- **Terrain:** Heightmap with GPU compute deformation, readback for collision
- **Particles:** Dust on dig, soil chunks, engine exhaust (GPU-driven, budgeted)
- **Memory budget:** 300MB target. iPhone 13+ (A15) minimum.
- **Thermal cascade:** Degrade render resolution → shadow quality → particle count. Physics rate stays fixed.

## Monetization (Preliminary)

- **$6.99 premium** (one-time purchase, no IAP to unlock core content)
- Cosmetic skins as optional IAP ($1.99-2.99)
- Future vehicle expansions as separate paid apps or large IAP ($6.99-9.99 each)
- No battle pass, no energy system, no ads
- Sim audiences pay for quality — respect that

## Age Rating

- 4+ (simulation, no fire/destruction of recognizable structures at launch)
- Demolition targets = abstract/industrial only

---

*Informed by 23 independent expert reviews (2026-02-15). Full synthesis in project files.*
*Updated: 2026-02-15*
