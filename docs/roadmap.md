# Roadmap

Phased plan. Each phase is scoped to end in a **demonstrable, filmable moment**.
Phases are intentionally ordered so that each one produces something real before
the next begins — no six-months-of-nothing valleys.

Status legend: `[ ]` not started · `[~]` in progress · `[x]` done.

---

## Phase 0 — Vision & Research  `[~]`

*Goal: know the machine well enough to build faithfully, and commit to an
architecture.*

- [x] Write the vision, non-goals, and definition of done ([00-vision.md](00-vision.md)).
- [x] Write the AGC primer: DSKY, channels, counters, executive ([01-agc-primer.md](01-agc-primer.md)).
- [x] Sketch the brain/face/world architecture ([02-architecture.md](02-architecture.md)).
- [x] Inventory the IO devices and their fidelity risks ([03-io-devices.md](03-io-devices.md)).
- [x] Catalog prior art and settle the licensing approach ([04-prior-art.md](04-prior-art.md)).
- [ ] Build `yaAGC` and `yaDSKY2` on a *desktop* first; run `Luminary099`; confirm
      the channel/counter/socket protocol hands-on. (De-risks Phase 1.)
- [ ] Pick the target phone(s) and confirm a viable rooted Linux userland path.
- [ ] Verify every "confirm in Phase 1" note in the docs against real sources
      (exact channel numbers, keycodes, license versions).

**Demo:** the real Luminary software running on a laptop, driven by a keyboard,
displaying live data — proof the core works before we fight the phone.

---

## Phase 1 — It runs on the phone  `[ ]`

*Goal: the real Apollo software running on a jailbroken phone, driven by a native
DSKY.*

- [ ] Get `yaAGC` compiling and running on the target phone's userland.
- [ ] Build the native full-screen **DSKY UI** (PROG/VERB/NOUN, R1–R3, lamp block).
- [ ] Wire the DSKY UI to `yaAGC` over the local socket: keycodes in, display/lamps
      out. No guidance logic in the UI.
- [ ] Reproduce authentic DSKY behavior: Verb/Noun flash on entry, KEY REL, RSET,
      OPR ERR, COMP ACTY.
- [ ] Run a self-contained program that needs no external IO (e.g. a display/
      monitor Verb) end-to-end on the phone.

**Demo:** "I put the Moon-landing computer's *real* software on a junk phone" —
punch a Verb/Noun sequence into the touchscreen DSKY and watch the real software
respond.

---

## Phase 2 — It thinks it has a spacecraft  `[ ]`

*Goal: simulated IO + minimal physics so the AGC can actually guide something.*

- [ ] Define the device-layer contract (a device = a process mapping physics
      quantities ↔ AGC pulses/bits over the socket).
- [ ] Build the cleanest device first: **PIPA** (velocity-in via counter pulses).
- [ ] Build **IMU/CDU** (attitude-in) and a **minimal physics model** (mass,
      thrust, gravity, attitude).
- [ ] Build **RCS jets** (attitude-out) and close the DAP attitude loop.
- [ ] Build the **descent engine** (thrust-out) and **landing radar** (altitude-in).
- [ ] Fly a **powered descent** (P63/P64) end to end against the simulation.
- [ ] Add virtual **discrete switches**, including the rendezvous-radar switch, and
      reproduce a **1201/1202** alarm + recovery.

**Demo:** "Watch a phone fly a Lunar Module to the surface — and survive the same
computer overload that nearly aborted Apollo 11."

---

## Phase 3 — Could it fly real hardware?  `[ ]`  *(stretch)*

*Goal: design (not necessarily fabricate) a bridge to physical IO.*

- [ ] Specify a **custom USB IO board**: connectors for gyros, accelerometers,
      radar-like sensors, engine/RCS drivers, antennas.
- [ ] Show the board implements the **same device-layer contract**, so it is a
      drop-in replacement for the Phase 2 simulation.
- [ ] Schematic capture and possibly PCB layout.
- [ ] (Aspirational, explicitly optional) bench-test a single real sensor (e.g. a
      MEMS gyro) incrementing a real AGC counter.

**Demo:** "Here's the circuit board that could let this phone fly an actual
spacecraft — and why the software was designed so that this is a *drop-in*, not a
rewrite."

---

## Cross-cutting, all phases

- **Fidelity log:** whenever we discover the real AGC did something surprising,
  record it in `research/`. These are both correctness anchors and great story
  beats.
- **Attribution:** keep credits to Virtual AGC / MIT / the preservation community
  current and prominent (see [04-prior-art.md](04-prior-art.md)).
- **Keep the seam clean:** resist any temptation to let the DSKY UI or a device
  "shortcut" through the physics or guidance. The whole design depends on the
  channel/counter boundary staying pure.
