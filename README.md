# pocket-agc

> "There's more computing power in your phone than was used to go to the Moon."
> Fine. Then let's make a phone *actually fly the mission.*

**pocket-agc** is a project to turn an old, fully jailbroken smartphone into a
faithful, drop-in stand-in for the **Apollo Guidance Computer (AGC)** — the
computer that guided the Command and Lunar Modules to the Moon and back.

The phone runs the *real, unmodified Apollo flight software* on a cycle-accurate
AGC emulator. On top of that sits a native **DSKY** (Display & Keyboard) — the
green numeric console the astronauts actually used — so you interact with it the
way Buzz Aldrin did: by punching **Verb** and **Noun** codes. Underneath, the
project progressively replaces the emulator's fake sensors and actuators with
*simulated* IO devices (IMU, radar, engines, RCS thrusters), and — as a stretch
goal — a *conceptual custom USB IO board* that could drive real hardware.

This repository is the **vision, research, and design home** for the project. It
is documentation-first on purpose: the hard part of this project is not writing
code, it is *understanding a 1960s real-time avionics computer well enough to
honor it.* Code lands in later phases (see the [roadmap](docs/roadmap.md)).

---

## Why this is achievable (the good news)

Almost every hard piece already exists as open source. The project is mostly
*integration and IO faithfulness*, not reinvention:

- **The flight software is real and public.** MIT's original Apollo 11 source —
  `Comanche055` (Command Module) and `Luminary099` (Lunar Module) — has been
  transcribed from the original listings and is publicly available. It is US
  government work.
- **A cycle-accurate CPU emulator exists.** Ron Burkey's **Virtual AGC** project
  provides `yaAGC`, which runs the *assembled original binaries*, plus `yaDSKY2`,
  a working DSKY emulator. It is GPL-licensed.
- **The IO model is documented.** The AGC talked to the spacecraft through
  numbered **input/output channels** and **counter increments**. `yaAGC` exposes
  exactly this over a documented socket protocol — which is the clean seam where
  our DSKY, our simulated devices, and (someday) real hardware plug in.

So the reframing that makes this tractable:

> We are **not** re-implementing the AGC. We are **hosting the real AGC brain on
> a phone**, giving it a great DSKY, and steadily swapping its fake world for a
> simulated — then physical — one.

See [`docs/04-prior-art.md`](docs/04-prior-art.md) for what we reuse vs. build,
and the licensing implications.

## Project pillars

1. **Faithful, not "inspired by."** Run the real binaries. Honor the real Verb/
   Noun interface, the real channel IO, the real 1201/1202 alarm behavior.
2. **The phone is a host, the DSKY is a peripheral.** Clean separation between
   the AGC core and everything that talks to it — the same boundary a hardware
   IO board would sit on.
3. **Simulate the world before wiring it.** Every physical device (gyro, radar,
   engine) gets a software model first. Hardware is the last mile, not the first.
4. **Every milestone is a story.** The phases are structured so each one is a
   demonstrable, filmable "it works now" moment.

## Repository layout

```
README.md              This file — the pitch and the map.
docs/
  00-vision.md         Goals, non-goals, and the "why" in depth.
  01-agc-primer.md     How the real AGC worked: DSKY, channels, counters, the executive.
  02-architecture.md   The pocket-agc stack: phone host, AGC core, DSKY UI, IO bus, future PCB.
  03-io-devices.md     Inventory of every IO device the AGC talked to and what simulating it needs.
  04-prior-art.md      Virtual AGC, MIT source, licenses — what we reuse and what we build.
  roadmap.md           Phase 0..3 milestones. Each phase ≈ one video.
research/              Scratch notes, reference tables, links. Raw material, not polished docs.
```

## Status

**Phase 0 — Vision & Research.** This repo currently holds the concept, the
architecture sketch, and the research needed before any code is written. Nothing
here flies yet. See the [roadmap](docs/roadmap.md) for what comes next.

## License

The original Apollo flight software is US government work (public domain).
Virtual AGC components are GPL-licensed. This repository's *own* content (docs,
and future original code) is under the [MIT License](LICENSE) for now — with the
explicit caveat that once Phase 1 links against GPL'd `yaAGC`, the combined,
distributable work will likely need a GPL-compatible license. That decision is
tracked in [`docs/04-prior-art.md`](docs/04-prior-art.md).
