# Implementation Plan — Gyro: Excavator

## Architecture

```
┌─────────────────────────────────────────┐
│  App Layer (SwiftUI menus, game states) │
├─────────────────────────────────────────┤
│  Game Logic (missions, scoring, progression) │
├──────────┬──────────┬───────────────────┤
│  Input   │  Physics │  Rendering        │
│  System  │  Engine  │  (Metal)          │
│          │          │                   │
│ Sticks   │ Arm IK   │ Terrain mesh      │
│ Gyro     │ Soil def │ Vehicle model     │
│ BT Ctrl  │ Tracks   │ Shadows           │
│ Haptics  │ Collision│ Particles         │
├──────────┴──────────┴───────────────────┤
│  Audio Engine (AVAudioEngine)           │
└─────────────────────────────────────────┘
```

**Tick rates (unified 60Hz — mixed rates cause sync bugs):**
- Input polling: 60Hz (gyro via CMDeviceMotion, not raw gyro)
- All physics (arm + soil): 60Hz (semi-implicit Euler with damping)
- Rendering: 60Hz, adaptive down to 30Hz under thermal load
- Audio: Event-driven
- Haptics: Transients via CHHapticEngine, continuous via AVAudioEngine haptic sync

## Phase 1: Renderer + Input (Weeks 1-3)

Get something on screen with working controls.

- [ ] Metal renderer boilerplate (CAMetalLayer, command queue, triple-buffer)
- [ ] Camera system (third-person, follow excavator)
- [ ] Flat terrain with basic texture
- [ ] Simple excavator mesh (box placeholder or low-poly model)
- [ ] Virtual thumbstick system (floating origin, dead zones, expo curves)
- [ ] Gyro input (CMDeviceMotion with sensor fusion, xArbitraryZVertical ref frame, 60Hz polling)
- [ ] Bluetooth controller detection + mapping

**Milestone:** Render a textured plane with a box you can move using sticks + gyro.

## Phase 2: Excavator Arm (Weeks 3-5)

The core mechanic — articulated arm that feels heavy.

- [ ] Boom + stick + bucket as serial linkage (forward kinematics)
- [ ] Joint angle limits (min/max per joint, realistic ranges)
- [ ] Hydraulic speed simulation (joints move at limited rate, slower under load)
- [ ] Arm physics at 60Hz (semi-implicit Euler with damping)
- [ ] ISO control mapping: left stick = swing+boom, right stick = stick+bucket, toggle button = track mode
- [ ] Gyro mirrors right stick Y for bucket curl (CMDeviceMotion, xArbitraryZVertical ref frame)
- [ ] Counterweight / centre-of-gravity tracking
- [ ] Tip-over detection and physics

**Milestone:** Move boom/stick/bucket with sticks, see arm respond with weight and joint limits.

## Phase 3: Soil + Digging (Weeks 5-8)

Make digging satisfying.

- [ ] Heightmap terrain system (GPU-side vertex buffer)
- [ ] Stamp-based deformation (bucket shape subtracted on contact)
- [ ] Material types: topsoil, clay, gravel, rock (different resistance curves)
- [ ] Bucket-soil force model (penetration resistance, breakout, curl force)
- [ ] Mass accumulation in bucket (affects arm dynamics)
- [ ] Spoil pile generation (pre-computed shapes stamped on dump)
- [ ] Volume tracking (hole ≈ pile × swell factor)
- [ ] Angle-of-repose post-processing (trench walls collapse if too steep)
- [ ] Soil deformation on CPU (parallel heightmap — no GPU→CPU readback, breaks TBDR)
- [ ] Terrain collision uses same CPU heightmap as deformation (no sync issues)

**Milestone:** Dig a trench in clay, see walls, pile spoil, feel the depth.

## Phase 4: Haptics + Audio (Weeks 8-10)

Feel and hear the machine.

- [ ] CoreHaptics engine setup (CHHapticEngine lifecycle, stoppedHandler, resetHandler)
- [ ] Per-material haptic patterns (topsoil, clay, gravel, rock strike)
- [ ] Boom stress haptics, track rumble, counterweight shift
- [ ] Hydraulic audio (strain under load, idle, boom movement)
- [ ] Bucket impact sounds per material
- [ ] Engine audio (RPM-mapped, load-reactive)
- [ ] Spatial audio positioning (AVAudioEngine 3D)
- [ ] Haptic intensity slider in settings
- [ ] Battery/thermal haptic budget management

**Milestone:** Close your eyes and know what material you're digging by feel + sound alone.

## Phase 5: Missions + Progression (Weeks 10-13)

Turn the toy into a game.

- [ ] Mission framework (briefing → gameplay → scoring → results)
- [ ] Trench jobs (target depth/width/length, scored on accuracy + speed)
- [ ] Loading jobs (dig face → bucket → truck, cycle time scoring)
- [ ] Foundation work (precision dig to plans)
- [ ] Demolition (breakable structures, sequenced teardown)
- [ ] Tight-access scenarios (constrained swing, obstacle avoidance)
- [ ] Progression system (unlock harder sites, new soil types, tighter tolerances)
- [ ] Quick Dig mode (random site, 3-5 min sessions)
- [ ] High score tracking

**Milestone:** Playable game with 15+ missions across 4 difficulty tiers.

## Phase 6: Polish + Ship (Weeks 13-16)

- [ ] PBR materials and proper excavator model
- [ ] Shadow cascades, dust particles, environment props
- [ ] Onboarding flow (guided first dig, < 90 seconds)
- [ ] Settings (gyro sensitivity, stick dead zones, haptic intensity, controller config)
- [ ] Accessibility audit (VoiceOver menus, touch-only controls, Dynamic Type)
- [ ] Thermal profiling on iPhone 13 (30-minute sustained session)
- [ ] Memory profiling (target 300MB peak)
- [ ] App Store metadata (screenshots, preview video, description, age rating 4+)
- [ ] Privacy Nutrition Label ("Data Not Collected")
- [ ] TestFlight beta

**Milestone:** App Store submission.

## Timeline Summary

| Phase | Weeks | What |
|-------|-------|------|
| 1. Renderer + Input | 1-4 | Metal boilerplate, sticks, gyro, camera |
| 2. Excavator Arm | 4-7 | Articulated arm, joint physics, controls |
| 3. Soil + Digging | 7-12 | Heightmap deformation, materials, forces |
| 4. Haptics + Audio | 12-15 | CoreHaptics patterns, cabin audio, spatial |
| 5. Missions | 15-19 | Game loop, progression, scoring, Free Dig |
| 6. Polish + Ship | 19-24 | Art, onboarding, accessibility, App Store |

**Total: ~24 weeks (6 months) to App Store submission.**

*Round 2 reviewers unanimously said 16 weeks was fantasy. Codex specialist estimated 24-30 weeks. Metal renderer alone needs 4+ weeks for stability. Soil deformation is the hardest phase.*

## Codex Notes

- Each phase = 1-2 CODEX_PROMPT.md files, max 300 lines each
- Metal renderer needs ~60-80% hand-written code (Codex can't reliably generate Metal pipelines)
- Soil deformation compute shader = manual work (Codex task boundary)
- Codex sweet spot: mission logic, UI, scoring, settings, input handling
- Test each Codex output on device before proceeding (simulator ≠ real Metal performance)

---

*Updated: 2026-02-15*
