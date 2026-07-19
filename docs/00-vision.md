# 00 — Vision

## The hook

The line gets repeated at every science-museum gift shop: *"There's more
computing power in your smartphone than NASA used to land on the Moon."* It is
true, and it is also the least interesting thing you can say about the Apollo
Guidance Computer.

The AGC had roughly **2K words of erasable memory** and **36K words of fixed
(rope) memory**, ran at about **1 MHz**, and did on the order of tens of
thousands of additions per second. A modern phone dwarfs it by a factor of
millions on every raw axis. And yet the AGC did something the raw numbers hide:
it ran a **priority-scheduled, interrupt-driven, fault-tolerant real-time
operating system** that flew human beings to another world and recovered
gracefully — mid-descent — when it was overloaded (the famous **1201/1202**
program alarms during Apollo 11).

So pocket-agc inverts the museum cliché. Instead of "look how weak the AGC was,"
the project says:

> "Your phone has all this power. Let's spend it on the one job the AGC actually
> had — and discover that the *compute* was never the hard part. The **real-time
> IO fabric and the software discipline** were."

## What we are building

A jailbroken phone (or tablet) that is a **drop-in replacement for the AGC**:

- **On the screen:** a native **DSKY** — the Verb/Noun numeric console. You
  operate it exactly as the crew did: `VERB 16 NOUN 68 ENTR` and so on.
- **Under the screen:** the **real Apollo flight software** running on a
  cycle-accurate AGC emulator. Not a reimplementation — the actual assembled
  Colossus/Luminary binaries.
- **Beneath that:** an **IO layer** that feeds the AGC its world — first as a
  software simulation of the spacecraft's sensors and actuators, later
  (conceptually) through a custom **USB IO board** that could wire to real gyros,
  radar, thrusters, and antennas.

## Goals

- **G1 — It runs the real thing.** The unmodified Apollo binaries execute on the
  phone and respond to DSKY input faithfully.
- **G2 — It feels real.** The DSKY looks and behaves like the hardware: the
  seven-segment registers, the status/warning lamps, the key-release and
  operator-error logic, the flashing Verb/Noun during data entry.
- **G3 — It flies a scenario.** A simulated IMU + engine + RCS model lets the AGC
  actually *guide* something — e.g. run a powered descent program end to end.
- **G4 — It documents the journey.** Each phase is explained well enough that a
  curious viewer/reader learns how the AGC really worked. Education is a
  first-class deliverable, not a byproduct.

## Non-goals (at least at first)

- **N1 — Not building an AGC CPU emulator from scratch.** Virtual AGC's `yaAGC`
  already does this superbly. Reimplementing it would be years of effort for a
  strictly worse result. We *host* it.
- **N2 — Not fabricating hardware (yet).** The custom USB IO board is designed and
  conceptualized, and possibly laid out — but fabrication is explicitly out of
  scope until/unless the software side proves the concept.
- **N3 — Not a full flight simulator.** We simulate *just enough* of the
  spacecraft physics to exercise the AGC honestly. We are not competing with
  Orbiter or KSP; the AGC is the star, the physics is a supporting actor.
- **N4 — Not a general "space calculator."** Fidelity to the actual AGC — its
  quirks, its alarms, its exact Verb/Noun catalog — matters more than adding
  features the real machine never had.

## Definition of "done" for the vision

pocket-agc has succeeded when someone can hold a phone, punch a real Apollo
descent sequence into a real-looking DSKY, watch a simulated Lunar Module
respond to the AGC's guidance, and — if they yank a simulated radar switch at the
wrong moment — see a **1202** alarm and recover from it the way the crew did.

That is the demo. Everything in the [roadmap](roadmap.md) exists to reach it.

## The video-series angle

This is deliberately structured as a **series of demonstrable milestones**, so it
doubles as source material for videos (if Michael chooses to make them). The
natural episode breaks map onto the roadmap phases:

- "I put the Moon-landing computer's *real* software on a junk phone."
- "Building the DSKY — the Apollo interface — as a touchscreen app."
- "Teaching the phone to think it has gyroscopes and a rocket engine."
- "Could a $10 circuit board let this phone fly a real spacecraft?"

The point of writing the docs this thoroughly *now* is that good research is what
makes those stories accurate instead of hand-wavy.
