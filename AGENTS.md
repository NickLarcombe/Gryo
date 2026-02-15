# Gyro: Excavator — Operations Guide

## Build & Run

This is a Swift/Metal iOS project targeting iPhone 13+ (iOS 17+).

```bash
# Open in Xcode
open Gyro.xcodeproj

# Build for device (Metal requires real hardware, not simulator)
# Xcode → Product → Run (⌘R) with iPhone connected

# Or via CLI
xcodebuild -scheme Gyro -destination 'platform=iOS,name=<device>' build
```

## Project Structure

```
src/
  App/            — SwiftUI app entry, game state machine
  Renderer/       — Metal renderer (CAMetalLayer, shaders, pipeline states)
  Physics/        — Arm kinematics, soil deformation, collision
  Input/          — Virtual sticks, gyro, BT controller
  Haptics/        — CoreHaptics engine, per-material patterns
  Audio/          — AVAudioEngine spatial audio, sound events
  Game/           — Missions, scoring, progression
  UI/             — HUD overlay, menus, settings
specs/
  physics.md      — Arm kinematics spec, soil model, force equations
  controls.md     — Input mapping, gyro calibration, controller support
  rendering.md    — Metal pipeline, terrain system, shader list
  haptics.md      — CHHapticPattern definitions per material/event
```

## Validation

```bash
# Run tests
xcodebuild test -scheme Gyro -destination 'platform=iOS Simulator,name=iPhone 15'

# Lint
swiftlint

# Profile (on device)
# Instruments → Metal System Trace (GPU)
# Instruments → Allocations (memory, target 300MB peak)
# Instruments → Time Profiler (CPU, check physics thread)
```

## Key Constraints

- **Memory:** 300MB peak target (profile with Instruments Allocations)
- **Thermal:** 30-minute sustained session on iPhone 13 without throttling
- **Physics rates:** Arm 120Hz, soil 30-60Hz, render 60Hz — NOT a single universal rate
- **Haptics:** Event-driven, not continuous. CHHapticEngine has lifecycle management (handle stops/resets)
- **Accessibility:** Every gyro function must have touch-only alternative. VoiceOver on all menus.
- **Metal:** Use CAMetalLayer directly (not MTKView). TBDR-aware: memoryless depth/stencil, proper load/store actions.

## Codex Integration

Each implementation phase has a corresponding spec in `specs/`. When using Codex:

1. Feed ONE spec at a time (max 300 lines)
2. Codex works well for: input handling, game logic, UI, scoring, settings
3. Codex needs heavy supervision for: Metal shaders, compute pipelines, physics tuning
4. Codex CANNOT do: haptic tuning (requires device testing), soil feel (requires iteration)
5. Always test on real hardware — Metal on simulator ≠ Metal on device

## Testing Notes

- Physics tests can run in simulator (pure math)
- Rendering tests MUST run on device (Metal)
- Haptic patterns MUST be tested on device (no simulator support)
- Soil deformation: visual verification required (screenshot comparison tests)
- Input: test gyro calibration in multiple phone orientations

---

*Updated: 2026-02-15*
