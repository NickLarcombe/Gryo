# Gyro — Excavator Simulator

> Dig with your phone. Gyro-controlled excavator sim for iPhone.

## What It Is

A precision excavator simulator where your iPhone IS the controller. Tilt for bucket curl, thumbsticks for boom/stick/tracks, haptics for soil feedback. Metal renderer, real-time physics, satisfying digging.

## Status

- **Phase:** Design (post-review, pre-build)
- **Target:** iPhone 13+, iOS 17+, Landscape, Swift + Metal
- **Build tool:** Xcode 26.3 + Codex autonomous coding
- **Last Updated:** 2026-02-15

## Docs

| File | What |
|------|------|
| [DESIGN.md](./DESIGN.md) | Vision, requirements, technical approach |
| [USE_CASES.md](./USE_CASES.md) | Player scenarios and gameplay loops |
| [IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md) | Build phases and task breakdown |
| [AGENTS.md](./AGENTS.md) | Build & run guide for Codex |
| [specs/](./specs/) | Detailed specs (physics, controls, rendering) |

## Future Vehicles

The engine is designed to support multiple vehicle types. After the excavator ships and proves the market, planned expansions:

- **FPV Drone** — Recon/racing, Betaflight-style PID, gate racing + freestyle
- **Fixed Wing (Air Tractor)** — Retardant bombing runs, gyro-as-airplane tilt
- **Helicopter (Air-Crane)** — Bambi bucket water bombing, pendulum load physics
- **Campaign Mode** — Multi-vehicle wildfire firefighting (all vehicles in one scenario)
- **Fantasy Skins** — Dragon, ornithopter, broomstick (same physics engine, different flight models)

Each vehicle is a separate physics profile, not a reskin. They'll ship as paid expansions once the core excavator experience is solid.

---

*Repo pending rename from "Gryo" → "Gyro" (GitHub Settings)*
