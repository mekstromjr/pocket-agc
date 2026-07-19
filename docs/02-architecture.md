# 02 — Architecture

How pocket-agc is put together, from the phone up. This is a *target*
architecture; Phase 0 commits to the shape, not the final implementation choices.

## The core idea: brain, face, world

Three layers, connected by one well-defined seam.

```
                        ┌──────────────────────────────┐
                        │            DSKY UI            │   "the face"
                        │  (native touchscreen app on   │
                        │   the phone/tablet)           │
                        └──────────────┬───────────────┘
                                       │ keycodes in / display+lamps out
                                       │   (channel IO packets)
        ┌──────────────────────────────┴───────────────────────────────┐
        │                       AGC CORE (yaAGC)                         │  "the brain"
        │   runs the real Colossus / Luminary binaries, cycle-accurate  │
        │   exposes channel + counter IO over a documented socket        │
        └───────┬───────────────────────────────────────────────┬──────┘
                │ counter increments in                          │ discrete/jet-driver bits out
                │ (gyros, PIPAs, radar)                          │
     ┌──────────┴──────────────────────────────────────────────┴───────┐
     │                       IO BUS / DEVICE LAYER                       │  "the world"
     │  Phase 2: software device models + a minimal spacecraft physics   │
     │  Phase 3: (conceptual) custom USB IO board bridging to real HW    │
     └───────────────────────────────────────────────────────────────────┘
```

The **seam** is the `yaAGC` IO interface: channel reads/writes and counter
increments, carried as small binary packets over a socket. *Everything* — the
DSKY, the sensors, the actuators, and eventually a USB board — is just a
peripheral speaking that protocol. This is the single most important design
decision in the project: it means the brain never knows or cares whether its
world is a phone-side simulation or a soldered circuit.

## Layer 1 — The phone host

An old, fully **jailbroken/rooted** smartphone or tablet. Root matters because we
want:

- A real POSIX userland to compile/run `yaAGC` (e.g. a chrooted or containerized
  Linux, or a native build via a toolchain like Termux — TBD in Phase 1).
- Freedom from app-store sandboxing so the AGC core, the DSKY UI, and the IO
  layer can talk over local sockets without platform interference.
- Control over the display for a proper full-screen kiosk DSKY.

Open question for Phase 1: **do the AGC core and the DSKY UI run as separate
processes on the phone (talking over localhost), or does the UI embed the core?**
The separate-process model is more faithful to the brain/face split and is the
current default assumption.

## Layer 2 — The AGC core (`yaAGC`)

We host Ron Burkey's **Virtual AGC** `yaAGC` emulator, feeding it the real
assembled Apollo binaries (`Comanche055` for the CM, `Luminary099` for the LM,
with room for other mission ropes later). `yaAGC`:

- Executes the original software cycle-accurately, including the Executive,
  Waitlist, interrupts, and the 1201/1202 alarm behavior.
- Presents **channel IO** and **counter** activity over its socket interface.

We treat `yaAGC` as a **black box we drive**, not code we fork. If we must patch
it (e.g. build tweaks for the phone toolchain), patches are kept minimal,
upstreamable, and tracked — see [`04-prior-art.md`](04-prior-art.md).

## Layer 3 — The DSKY UI (the face)

A native, full-screen touchscreen DSKY. Responsibilities:

- Render the PROG/VERB/NOUN displays, the R1/R2/R3 registers, and the full lamp
  block — including correct **flash** behavior (e.g. Verb/Noun flashing during
  data entry, alarm lamps).
- Translate touches on the 19 keys into the AGC's **keycodes** and send them as
  channel input packets.
- Consume the AGC's **display output** (relay words) and update the seven-segment
  registers and lamps accordingly.

Design intent: the DSKY UI holds **no guidance logic whatsoever**. It is a dumb,
faithful peripheral. If it computed anything about the mission, we'd be cheating.
Its entire job is: keys → channel-in, channel-out → pixels.

Reference implementation to study: Virtual AGC's own **`yaDSKY2`**. We are
building a touchscreen-native equivalent, but its behavior is our spec.

## Layer 4 — The IO bus / device layer (the world)

This is where most of the *original* engineering in pocket-agc lives. It has two
generations:

### Phase 2 — Simulated devices + minimal physics

Software models that speak the `yaAGC` IO protocol and behave like the real
spacecraft peripherals:

- **IMU / CDUs** — emit attitude-change **pulses** into the AGC's angle counters.
- **PIPAs (accelerometers)** — emit **delta-V pulses** into the velocity counters.
- **Radar** — emit range / range-rate pulses.
- **Engine + RCS** — *consume* the AGC's jet-driver / throttle output bits and
  turn them into thrust/torque in the physics model.

Behind these sits a **minimal spacecraft physics model** — just enough dynamics
(mass, thrust, gravity, attitude) to close the loop so the AGC's guidance
commands actually move a simulated vehicle, which in turn changes what the
sensors report. See [`03-io-devices.md`](03-io-devices.md) for the per-device
inventory.

The guiding rule: **devices convert between physical quantities and pulse/bit
streams; the physics model owns the quantities; the AGC owns the guidance.** Keep
those three concerns separate and the system stays honest and debuggable.

### Phase 3 — Conceptual custom USB IO board

A stretch, likely *designed but not fabricated*: a small custom PCB that connects
to the phone over USB and exposes headers/connectors for real IO — gyros,
accelerometers, radar-like sensors, thruster/engine drivers, antennas. It would
implement the *same* device-layer contract, so from the AGC core's perspective
nothing changes: the socket packets are identical whether the counters are
incremented by a physics model or by a real gyro on a bench.

This is the payoff of putting the seam at the channel/counter level: the
hardware path is a **drop-in replacement for the simulation path**, not a rewrite.

## Data-flow example: a Verb 16 monitor

To make the seam concrete, here is one round trip:

1. Crew keys `V16 N68 ENTR` on the DSKY UI. Each keypress becomes a keycode
   channel-input packet to `yaAGC`.
2. The real Luminary software interprets Verb 16 / Noun 68 and begins periodically
   writing display data out on the DSKY output channel (relay words).
3. The DSKY UI receives those output packets and lights up R1/R2/R3 with the live
   landing data — the same numbers, computed by the same code, that Apollo 11 saw.
4. Meanwhile the IO device layer is streaming IMU/PIPA/radar counter pulses into
   the AGC, so the numbers on the display *change* as the simulated vehicle moves.

No layer in that chain knows what the others are made of. That is the
architecture working as intended.
