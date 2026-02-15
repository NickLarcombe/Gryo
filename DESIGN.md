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

### Should Have (v1.0 if time permits)
- [ ] Free Dig / sandbox mode (no objectives, just dig — parents and kids love this)
- [ ] Leaderboards (cycle time, precision scoring — the #1 missing retention hook)
- [ ] Float function (boom drops under gravity for grading — operators say this is fundamental)
- [ ] Simplified "kid mode" controls (tilt-only, bigger targets, more forgiving)

### Nice to Have (v1.x)
- [ ] Track deformation on soft ground
- [ ] Auxiliary hydraulics (thumb clamp, tilt rotator, breaker)
- [ ] Travel-lock / arm-tuck mechanic for repositioning
- [ ] Night operations with work lights
- [ ] Replay/recording system

### v2.0+ (Future Vehicles)
- [ ] FPV Drone (recon/racing)
- [ ] Fixed Wing Air Tractor (retardant bombing)
- [ ] Helicopter Air-Crane (water bombing)
- [ ] Campaign Mode (multi-vehicle wildfire)

## Non-Goals (v1.0)

- **NOT a flight sim** — No aircraft at launch
- **NOT sandbox-only** — Structured missions with scoring + Free Dig sandbox mode
- **NOT multiplayer** — Single-player first, network architecture not required at launch
- **NOT 240Hz everything** — Per-system tick rates (see Technical Approach)
- **NOT <150MB** — Budget 300MB realistically

## Controls — ISO Pattern

Pure ISO controls with gyro enhancement and a track toggle.

### Digging Mode (default)
| Function | Input |
|----------|-------|
| Swing (cab rotation) | Left stick X |
| Boom up/down | Left stick Y |
| Stick in/out | Right stick X |
| Bucket curl/dump | Right stick Y **+ Gyro tilt** (tilt phone = curl bucket) |

### Track Mode (toggle button)
| Function | Input |
|----------|-------|
| Left track | Left stick Y |
| Right track | Right stick Y |
| Differential steering | Push sticks unevenly |

A toggle button switches between dig mode and track mode. Simple, predictable, no context-sensitive magic.

### Gyro (Hero Mechanic)
- **Gyro tilt = bucket curl/dump** — mirrors right stick Y, the signature "tilt to scoop" feel
- Uses `CMAttitudeReferenceFrame.xArbitraryZVertical` for proper landscape orientation
- Sensor fusion via `CMDeviceMotion` (gyro + accelerometer + magnetometer) — raw gyro drifts
- **60Hz polling** with interpolation to 120Hz physics (120Hz polling murders battery)
- Recalibrate: double-tap or automatic baseline adjustment
- Sensitivity adjustable in settings
- **Always optional** — right stick Y does the same thing. Gyro enhances, never required.

### BT Controller (MFi/Xbox/PS)
Full ISO mapping: left stick = swing+boom, right stick = stick+bucket, triggers = tracks. Gyro disabled when controller connected.

### Accessibility
- Every gyro function has touch-only equivalent (right stick Y)
- VoiceOver-accessible menus and purchase flows
- Switch Control and button remapping support
- Haptic intensity slider
- Dynamic Type in all UI

## Haptic Design

Core Haptics (CHHapticEngine) is a PRIMARY feedback channel, not polish.

| Soil Type | Haptic Feel |
|-----------|-------------|
| Topsoil | Low intensity continuous, gradual onset, low sharpness (~0.2) |
| Clay | High intensity sustained, slow sharpness ramp UP, sudden release snap |
| Gravel | Stochastic transients (20-80ms intervals), medium sharpness (0.5-0.7) |
| Rock strike | Sharp transient, sharpness 0.9+, intensity 1.0, 50ms decaying ring |

Additional patterns: boom stress, track rumble, counterweight shift, engine load.

**Frequency band:** 150-250Hz (Taptic Engine resonant frequency). Below 100Hz feels mushy and fights the hardware. Use transient events for impacts, `AVAudioEngine` with haptic sync for continuous backgrounds (CoreHaptics is for transients, not sustained rumble).

**Thermal budget:** Sustained haptic budget is 20-50mW max (not 50-200mW — that will thermal throttle in 3-5 minutes). Monitor `thermalStateChangeHandler` and scale intensity dynamically. Sharpness cap at 0.8 for max-intensity patterns (0.9+ triggers accessibility warnings).

## Soil Deformation — Technical Approach

**Representation:** Heightmap terrain with stamp-based deformation.
- Bucket shape stamps subtracted on contact
- Angle-of-repose post-processing at 30-60Hz (walls collapse if too steep)
- Pre-computed pile shapes for dumping (stamped onto heightmap)
- Volume tracking: hole volume ≈ pile volume × 1.25 (swell factor)

**NOT doing:** Voxel terrain, volumetric soil, SPH particles, real-time continuum mechanics.

**Physics tick rates (revised from round 2 review):**
- All physics: **60Hz unified** (semi-implicit Euler with damping — RK4 is overkill on mobile, and mixed rates cause sync nightmares where arm clips through un-updated soil)
- Soil deformation: 60Hz (same tick as arm physics — eliminates temporal lag)
- Rendering: 60Hz
- Haptics: Event-driven transients from physics, continuous via AVAudioEngine haptic sync

**Bucket-soil interaction forces:**
- Penetration resistance (function of soil type, depth, angle)
- Breakout force
- Curl force under load
- Mass accumulation in bucket (affects CG, arm dynamics)

## Rendering

- **Metal** with CAMetalLayer (not MTKView — need manual frame pacing)
- **TBDR-aware:** memoryless depth/stencil, proper load/store actions, pass merging
- **Shadows:** 2-cascade shadow maps
- **Terrain:** Heightmap with CPU-side deformation (parallel CPU heightmap for collision — GPU→CPU readback breaks TBDR pipeline and causes frame drops)
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

- **9+** (demolition content = destruction themes, even abstract. 4+ will be rejected per App Store guidelines)
- Demolition targets = abstract/industrial only (no recognizable buildings, schools, hospitals)

---

---

## Audio Design

- **Cabin acoustics first** — muffled external sounds, seat creaks, control clicks, dashboard rattles. Player is IN the cab, not watching from outside.
- **Dynamic range compression** — iPhone speakers are terrible. Compress for mobile output, not studio monitors.
- **Essential sounds:** Hydraulic strain, relief valves, backup beeper, track clunks, engine RPM, bucket impacts per material, swing motor whine.
- **Audio mixing hierarchy** — when multiple hydraulics activate simultaneously, prioritise player's active input. Clear feedback > realism.
- **UI sounds** — touch feedback on controls (silent UI feels broken on mobile).
- **2D fallback** — graceful degradation if AVAudioEngine 3D init fails.

---

*Informed by 48 independent expert reviews across 2 rounds (2026-02-15). Full synthesis in project files.*
*Updated: 2026-02-15*
