# GYRO Excavator — Round 2 Review Synthesis (25/25 Complete)

## Executive Summary

25 expert reviewers tore apart the excavator-only design. The scope reduction from 4 vehicles to 1 was universally praised, but the excavator itself still has significant technical and business challenges. The core concept (tilt-to-scoop with haptic soil feedback) genuinely excites people — the problems are in execution, pricing, and content depth.

---

## CRITICAL FIXES (Applied to design docs)

These were flagged as showstoppers and have been incorporated:

### 1. Haptic Frequency Band Was Wrong
- **Problem:** Design said 30-80Hz. Taptic Engine resonates at 150-250Hz. Below 100Hz is mushy.
- **Fix:** Redesigned to 150-250Hz. Transient events via CoreHaptics, continuous feedback via AVAudioEngine haptic sync. Sharpness cap 0.8 (not 0.9+).
- **Source:** CoreHaptics Engineer (#8)

### 2. CMMotionManager Reference Frame Breaks in Landscape
- **Problem:** `deviceMotion.attitude` gives orientation relative to Earth, not how user holds phone. Forward tilt becomes right tilt in landscape.
- **Fix:** Use `CMAttitudeReferenceFrame.xArbitraryZVertical` with proper initial calibration and sensor fusion (gyro+accelerometer+magnetometer via `CMDeviceMotion`).
- **Source:** iOS Input Developer (#25)

### 3. GPU→CPU Readback Breaks TBDR Pipeline
- **Problem:** Reading terrain deformation results back from GPU causes pipeline bubbles, frame drops to ~20fps.
- **Fix:** CPU-side parallel heightmap for both deformation and collision. No GPU readback.
- **Source:** Metal/GPU Engineer (#6)

### 4. Mixed Tick Rates Cause Sync Bugs
- **Problem:** Arm at 120Hz, soil at 30Hz = 2-4 frame lag. Bucket clips through un-updated soil.
- **Fix:** Unified 60Hz for all physics (arm + soil). Semi-implicit Euler with damping (RK4 is overkill).
- **Source:** Physics Engineer (#19), Engine Architect (#1), QA Lead (#18)

### 5. 120Hz Motion Polling Murders Battery
- **Problem:** Apple docs recommend 60Hz max for motion apps.
- **Fix:** 60Hz polling with interpolation. More than adequate for bucket curl.
- **Source:** iOS Input Developer (#25)

### 6. Age Rating Must Be 9+ (Not 4+)
- **Problem:** Demolition content = destruction themes. 4+ explicitly prohibits this.
- **Fix:** Changed to 9+.
- **Source:** App Store Reviewer (#9)

### 7. Controls Simplified to Pure ISO + Track Toggle
- **Problem:** Original had ambiguous track controls ("context-sensitive").
- **Fix:** Dig mode (ISO: left=swing+boom, right=stick+bucket) with toggle button for track mode.
- **Source:** Nick's directive + Excavator Operator (#5)

---

## CONSENSUS THEMES

### What Excites People (Positive)
- **Tilt-to-scoop is genuinely novel** — Parent (7/10), Kid (7/10), casual gamer all say "cool concept"
- **Per-material haptics** are a real differentiator if executed well
- **Focused scope** — every reviewer praised "one vehicle done right" vs round 1's 4-vehicle sprawl
- **No IAP/no ads** — parents love the safety of a premium contained experience
- **Cabin audio** opportunity — muffled sounds, hydraulic strain, seat creaks (Audio Director)

### What Kills It (Negative)

**Business:**
- $6.99 premium is brutal vs free competitors with 100M+ downloads
- 1 excavator vs Construction Sim 4's 40+ vehicles at same price
- iPhone-only, landscape-only limits addressable market severely
- No retention hooks → D30 predicted 3-8%
- YouTuber: "one machine = one video" — zero content creator appeal
- Revenue projection: $10-50K/year (monetization analyst)

**Technical:**
- 16-week timeline was fantasy → revised to 24 weeks
- Memory budget questioned (Metal engineer says ~150MB real after iOS reserves)
- Forward kinematics may not be enough — physics engineer wants inverse kinematics with hydraulic system modeling
- Soil heightmaps can't represent overhangs/undercuts (geotechnical, physics)
- No concrete particle budgets for VFX (dust, chunks, exhaust)

**Content:**
- 15 missions = ~6 hours, then nothing
- No Free Dig / sandbox mode (parents and kids want this most)
- No leaderboards = no score-chasing motivation
- Scoring on accuracy+efficiency+spillage may be cognitively overloaded
- Demolition without hydraulic breakers is "amateur hour" (demolition contractor)

**Accessibility:**
- Still needs: colorblind modes, high-contrast, Switch Control, button remapping, audio navigation
- "Planned" accessibility features = instant App Store rejection

---

## REVIEWER ROSTER & RATINGS

| # | Reviewer | Sentiment | Key Insight |
|---|----------|-----------|-------------|
| 1 | Engine Architect | Harsh | Mixed tick rates = sync disaster |
| 2 | UX Specialist | Harsh | Dual-stick + gyro = cognitive overload |
| 3 | Game Producer | Very harsh | "Engineering exercise masquerading as business" |
| 4 | Geotechnical Engineer | Very harsh | 4 soil types is "kindergarten-level" |
| 5 | Excavator Operator (15yr) | Harsh | Gyro bucket curl still wrong, missing float |
| 6 | Metal/GPU Engineer | Very harsh | GPU readback = TBDR death, memory fantasy |
| 7 | Monetization Analyst | Very harsh | Projects $10-50K/year, "revenue suicide" |
| 8 | CoreHaptics Engineer | Very harsh | Frequency band completely wrong |
| 9 | App Store Reviewer | Harsh | 85% rejection chance as-is |
| 10 | Retention Analyst | Harsh | D30: 3-8%, "designed for abandonment" |
| 11 | ASO/Marketing | Very harsh | Name is SEO suicide, platform restrictions |
| 12 | Audio Director | Harsh | No cabin acoustics, no dynamic range strategy |
| 13 | Codex Specialist | Harsh | 24-30 weeks realistic, not 16 |
| 14 | Construction YouTuber | Harsh | "One machine = one video max" |
| 15 | 10-Year-Old Kid | Positive (7/10) | Loves tilt-to-scoop, but "just digging holes" |
| 16 | Casual Gamer | Very harsh | "Tilt phone on the bus? Are you insane?" |
| 17 | Accessibility Advocate | Very harsh | Zero accessibility for disabled users |
| 18 | QA Lead | Very harsh | "QA death trap," predicts 50+ critical bugs |
| 19 | Physics Sim Engineer | Harsh | Needs inverse kinematics, unified tick rate |
| 20 | VFX Artist | Harsh | Zero particle budgets, no overdraw strategy |
| 21 | Training Sim Dev | Harsh | Training value 2/10 (but design says "not training sim") |
| 22 | Game Designer | Harsh | No meaningful progression rewards |
| 23 | Demolition Contractor | Very harsh | Bucket-only demo is "amateur hour" |
| 24 | Parent (7yo excavator fan) | Positive (7/10) | Would buy, wants kid mode + free play |
| 25 | iOS Input Developer | Harsh | CMMotionManager ref frame = showstopper |

---

## REMAINING OPEN QUESTIONS

1. **Pricing:** $6.99 premium vs freemium with tier unlocks? Every business reviewer says premium is death.
2. **Inverse kinematics:** Is forward kinematics good enough for "feel over fidelity" or do we need hydraulic modeling?
3. **Memory:** Is 300MB achievable or do we need to target 200MB?
4. **Heightmap limitations:** Accept no overhangs, or hybrid approach?
5. **Content depth:** 15 missions enough, or need procedural generation / Free Dig mode at launch?
6. **Name:** "Gyro" still returns Greek food. Alternatives?

---

*25/25 reviews complete. Synthesis compiled 2026-02-15.*
