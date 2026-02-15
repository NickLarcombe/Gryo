# Use Cases — Gyro: Excavator

## Primary Users

1. **Casual mobile gamer** — Wants something satisfying in 5-10 minute sessions. Attracted by the novelty of gyro digging.
2. **Construction/machinery enthusiast** — Watches excavator YouTube, plays Construction Simulator. Wants authentic-feeling controls and physics.
3. **Sim hobbyist** — Wants depth, progression, skill ceiling. Will use a Bluetooth controller.

---

## Use Case 1: First Dig (Onboarding)

**Actor:** New player, first launch

**Goal:** Dig a satisfying trench within 60 seconds of opening the app

**Steps:**
1. App opens to a simple dirt lot with an excavator, no menus
2. Guided overlay shows: "Tilt phone to swing cab" → player swings
3. "Left stick: boom up/down" → player raises boom
4. "Right stick: dig" → player curls bucket into soil
5. Soil deforms, haptics fire, dirt piles up
6. "You just dug your first trench. Ready for a real job?" → transitions to first mission

**Success Criteria:**
- [ ] Player digs within 30 seconds (no menu friction)
- [ ] Haptic feedback on first bucket contact
- [ ] Visible soil deformation + spoil pile
- [ ] Total onboarding < 90 seconds

---

## Use Case 2: Trench Job (Core Loop)

**Actor:** Player with basic controls learned

**Goal:** Dig a utility trench to spec (depth, width, grade)

**Steps:**
1. Job briefing: "Dig a 1.2m deep × 0.6m wide trench, 15m long, for a water main"
2. Player positions excavator at start point (tracks + swing)
3. Digs to depth using boom/stick/bucket cycle
4. Maintains grade (visual depth indicator + haptic feedback when at target depth)
5. Loads spoil into truck (swing to truck, dump, swing back)
6. Scored on: accuracy (depth variance), efficiency (cycle time), spillage

**Success Criteria:**
- [ ] Trench depth measured and scored (±5cm tolerance)
- [ ] Cycle time tracked (dig → load → dump → return)
- [ ] Material spillage penalised
- [ ] Session length: 5-8 minutes per job

---

## Use Case 3: Tight Access Residential

**Actor:** Intermediate player

**Goal:** Dig foundations next to an existing structure without damage

**Steps:**
1. Excavator placed in constrained space (fence on left, house on right)
2. Limited swing arc — must plan dig face and dump point
3. Underground obstacles (gas line marker, existing pipe) — must dig around them
4. Overdig or contact with structure = fail
5. Precision scoring: proximity to obstacles, surface finish

**Success Criteria:**
- [ ] Collision detection with structures and underground services
- [ ] Swing arc constraints based on site geometry
- [ ] Higher scoring multiplier for clean near-miss work

---

## Use Case 4: Demolition

**Actor:** Advanced player

**Goal:** Tear down a structure efficiently and safely

**Steps:**
1. Structure assessment (weak points highlighted)
2. Sequenced demolition — top down, controlled collapse direction
3. Manage debris pile — load trucks, clear site
4. Tip-over risk from heavy swings at height
5. Scored on: speed, controlled collapse, no unplanned spread

**Success Criteria:**
- [ ] Structure has breakable components with physics
- [ ] Collapse direction matters
- [ ] Counterweight/stability a real constraint at full reach

---

## Use Case 5: Bluetooth Controller Session

**Actor:** Sim enthusiast with Xbox/PS controller

**Goal:** Play with physical sticks for precision work

**Steps:**
1. Controller connected via Bluetooth
2. Controls auto-map to ISO pattern (left stick = swing/boom, right stick = stick/bucket)
3. Triggers = track steering
4. Gyro disabled (controller provides all axes)
5. Full haptic feedback through phone (placed on surface as haptic pad)

**Success Criteria:**
- [ ] Seamless controller detection and mapping
- [ ] ISO-standard control layout
- [ ] Phone haptics still active as feedback device

---

## Use Case 6: Quick Session (Bus/Train)

**Actor:** Casual player, 5 minutes to kill

**Goal:** Satisfying dig session without commitment

**Steps:**
1. "Quick Dig" mode from main menu
2. Random site, random job type, no story context
3. Dig, score, done in 3-5 minutes
4. High score saved, optional replay

**Success Criteria:**
- [ ] < 10 seconds from app open to digging
- [ ] Self-contained 3-5 minute sessions
- [ ] Landscape grip manageable standing (no extreme tilting required)

---

## Edge Cases

- **Gyro calibration drift** — Recalibrate on double-tap or automatic baseline adjustment
- **Phone overheating** — Degrade render quality, maintain physics rate, show temp indicator
- **Accessibility** — Full stick-only mode, VoiceOver menus, haptic intensity slider, Dynamic Type
- **Controller disconnects mid-game** — Pause, offer touch controls, resume

---

*Updated: 2026-02-15*
