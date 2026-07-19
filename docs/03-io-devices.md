# 03 — IO Device Inventory

The AGC's "world" — every category of device it exchanged signals with, and what
faithfully *simulating* each one requires. This is the research backlog for
Phase 2, and the connector wishlist for the Phase 3 board.

Each device is described by the same template:

- **Direction** — does the AGC *read* it (sensor) or *drive* it (actuator)?
- **AGC mechanism** — channel bits vs. counter increments.
- **What it represents physically.**
- **Simulation notes** — what the software model must do, and where the hard parts
  are.
- **Fidelity risk** — how likely we are to get it subtly wrong, and why it matters.

> Exact channel/counter assignments are deliberately left as "confirm in Phase 1"
> — they come from the Virtual AGC docs and the original source, and writing
> guesses here would just create authoritative-looking errors.

---

## Sensors (AGC reads)

### IMU — Inertial Measurement Unit (via CDUs)

- **Direction:** in.
- **AGC mechanism:** angle **counter increments** through the Coupling Data Units
  (CDUs), one per gimbal axis.
- **Physically:** a gimballed, gyro-stabilized platform holding an inertial
  reference. As the spacecraft rotates, the gimbal angles change; the CDUs
  digitize those changes into pulse streams.
- **Simulation notes:** the model takes the current spacecraft attitude from the
  physics layer and emits the *difference* since last update as counter pulses.
  Must handle gimbal-angle wraparound and the **gimbal-lock** geometry (which
  drives the DSKY's GIMBAL LOCK lamp).
- **Fidelity risk:** high. Attitude is the backbone of guidance; a sign error or a
  wrong pulse weight corrupts everything downstream. Also the source of the
  authentic "align the platform" (P51/P52) procedures.

### PIPAs — Pulsed Integrating Pendulous Accelerometers

- **Direction:** in.
- **AGC mechanism:** velocity **counter increments** (one counter per axis).
- **Physically:** accelerometers that emit one pulse per fixed increment of
  velocity ("delta-V"). The AGC integrates velocity by *counting pulses* — it
  never sees acceleration directly.
- **Simulation notes:** the model integrates the physics layer's acceleration and
  emits a pulse each time accumulated delta-V crosses the quantum. This is the
  cleanest example of "hand the AGC pulses, not floats."
- **Fidelity risk:** high, and *educational gold*: it viscerally shows that the
  AGC's navigation is literally pulse-counting.

### Rendezvous / Landing Radar

- **Direction:** in.
- **AGC mechanism:** range and range-rate **counter increments**; plus discrete
  status bits on input channels.
- **Physically:** the LM's landing radar (altitude + velocity over the surface)
  and rendezvous radar (range/range-rate to the CM).
- **Simulation notes:** derive range/altitude from the physics model's geometry and
  emit pulses; model the discrete "data good" flags. **This is the device behind
  the Apollo 11 1202 alarm** — a rendezvous-radar switch position flooded the AGC
  with counter activity. Simulating that switch is a deliberate demo goal.
- **Fidelity risk:** medium on the numbers, high on the *timing/interrupt load*
  (which is the whole point of the 1202 story).

### Optics / Sextant / Star sightings

- **Direction:** in (angles) + interaction via the "Mark" buttons.
- **AGC mechanism:** CDU-style angle counters for the optics shaft/trunnion, plus
  a **Mark** discrete on an input channel.
- **Physically:** the astronaut sights a star through the sextant/telescope and
  presses Mark to snapshot the angles for platform alignment.
- **Simulation notes:** later-phase; needed for authentic P51/P52 alignment. Can be
  stubbed early with canned star data.
- **Fidelity risk:** low priority, medium complexity. Nice-to-have for a "full"
  experience.

### Discrete switches & flags

- **Direction:** in.
- **AGC mechanism:** individual **bits on input channels**.
- **Physically:** cockpit switches, engine-arm discretes, mode selects, the
  rendezvous-radar switch, etc.
- **Simulation notes:** a straightforward map of virtual switches to input-channel
  bits. Cheap to build, high narrative value (flipping switches is filmable).
- **Fidelity risk:** low individually; the risk is *knowing which bit is which*.

---

## Actuators (AGC drives)

### Reaction Control System (RCS) jets

- **Direction:** out.
- **AGC mechanism:** **jet-driver bits** on output channels — one bit per thruster
  (or thruster group), pulsed on/off.
- **Physically:** small attitude-control thrusters fired in short bursts by the
  **Digital Autopilot (DAP)** to hold or change attitude.
- **Simulation notes:** the device layer consumes the jet bits and applies
  torque/thrust impulses to the physics model. The DAP's *pulse pattern* is real
  AGC software — we just render its effect. Watching the jets chatter to hold
  attitude is a great visualization.
- **Fidelity risk:** medium. Torque mapping and thruster geometry must be
  plausible or the DAP will "fight" a wrong plant model.

### Main / descent / ascent engine (throttle + on/off + gimbal)

- **Direction:** out.
- **AGC mechanism:** engine on/off **discretes**, throttle command (the LM descent
  engine was throttleable), and **TVC/gimbal-trim** counter outputs.
- **Physically:** the big engines. The LM descent engine's throttle is central to
  the powered-descent demo.
- **Simulation notes:** consume throttle/on-off/gimbal outputs; apply thrust and
  thrust-vector to the physics model. This closes the powered-descent loop
  (G3 in the vision).
- **Fidelity risk:** high — this is the device that makes "fly a descent" real.

### DSKY (yes, it's an IO device too)

- **Direction:** both.
- **AGC mechanism:** keycodes in on an input channel; display **relay words** and
  lamp bits out on output channels.
- **Physically:** the crew console.
- **Simulation notes:** handled specially as the **UI layer**, not a
  physics device — see [`02-architecture.md`](02-architecture.md). Listed here for
  completeness because architecturally it rides the exact same channel seam.
- **Fidelity risk:** high on *feel* (flash timing, key-release logic), low on
  physics (there is none).

### Downlink / Uplink telemetry & antennas

- **Direction:** both (mostly a data stream).
- **AGC mechanism:** telemetry output on dedicated channels at a fixed cadence;
  uplink input (ground can poke erasable memory).
- **Physically:** the S-band / comm link to Mission Control; steerable antennas.
- **Simulation notes:** low priority for flying, high value for a "Mission Control
  view" — decode the downlink to drive an external display, like the real ground
  consoles. Antenna *pointing* would be a Phase 3 hardware flourish.
- **Fidelity risk:** low priority; the downlink format is well documented.

---

## Prioritization for Phase 2

To hit **G3 (fly a scenario)** with the least surface area, build in this order:

1. **IMU/CDU (attitude in)** and **PIPA (velocity in)** — without these the AGC is
   blind.
2. **RCS jets (attitude out)** — closes the attitude control loop with the DAP.
3. **Descent engine (thrust out)** + **landing radar (altitude in)** — closes the
   powered-descent loop; enables the P63/P64 demo.
4. **Discrete switches** — cheap, and unlocks the **1202 alarm** demo via the
   rendezvous-radar switch.
5. Everything else (optics, telemetry, antennas) — polish and later episodes.

The unifying implementation contract, restated: **a device is a small process
that maps between the physics model's quantities and the AGC's pulses/bits over
the `yaAGC` socket.** Build one well (PIPA is the cleanest) and the rest are
variations on it.
