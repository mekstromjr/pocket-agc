# 01 — AGC Primer

A working understanding of the real Apollo Guidance Computer, focused on the
parts pocket-agc has to honor. This is not a full history — see
[`04-prior-art.md`](04-prior-art.md) for authoritative sources. Treat exact
channel/counter numbers here as *illustrative*; the Virtual AGC documentation and
the original source listings are the ground truth we validate against in Phase 1.

## What the AGC was

The AGC was a real-time flight computer built by the **MIT Instrumentation
Laboratory**. One flew in the Command Module (CM), one in the Lunar Module (LM).
Rough shape of the machine:

- **Word size:** 15 data bits + 1 parity bit (a 16-bit word). Numbers are
  ones-complement.
- **Erasable memory (RAM):** ~2,048 words of magnetic-core memory.
- **Fixed memory (ROM):** ~36,864 words of **core rope** memory — the program was
  literally *woven* by hand; a wire threading a core meant a 1.
- **Clock:** ~1.024 MHz (derived from a 2.048 MHz master). Basic instructions took
  a small number of ~11.7 µs memory-cycle "MCTs."
- **Architecture:** accumulator-based, with a small, clever instruction set
  extended by "interpreter" pseudo-instructions for vector/matrix and
  double-precision math (guidance needs trigonometry the raw ISA can't do
  directly).

The two things that matter most for us are **the DSKY** (how humans talked to it)
and **the IO channel/counter model** (how the spacecraft talked to it).

## The DSKY (Display & Keyboard)

The DSKY is the crew interface — the icon of the whole program. pocket-agc's
front end *is* a DSKY, so we model it precisely.

### The keyboard

Nineteen keys:

- Digits `0`–`9`
- `+` and `−` (sign entry)
- `VERB` and `NOUN` (the two-letter language, below)
- `CLR` (clear a data-entry field)
- `PRO` / `PROCEED` (proceed / accept — also standby when held)
- `KEY REL` (key release — hand the displays back to the running program)
- `ENTR` (enter / execute the assembled Verb-Noun command)
- `RSET` (reset — acknowledge/clear caution & warning conditions)

### The displays

- **PROG** — 2 digits — the current major mode / program number (e.g. `P63`
  braking phase of powered descent).
- **VERB** — 2 digits — the action.
- **NOUN** — 2 digits — the operand/data the action applies to.
- **R1 / R2 / R3** — three signed 5-digit registers — the actual numeric data
  (position, velocity, angles, time, etc.), each with a sign.

### The status / caution & warning lamps

A block of indicator lights, including: **COMP ACTY** (computer activity),
**UPLINK ACTY**, **TEMP**, **NO ATT** (no attitude), **GIMBAL LOCK**, **STBY**
(standby), **PROG** (program alarm), **KEY REL**, **RESTART**, **OPR ERR**
(operator error), **TRACKER**, and mission-context lamps like **VEL** / **ALT**.
Getting these right — *when they flash vs. glow steady* — is much of what makes a
DSKY feel authentic.

### The Verb/Noun language

You don't type commands as text. You compose them:

- **Verb** = what to do (e.g. `V16` = "monitor/display in decimal, continuously").
- **Noun** = what to do it to (e.g. `N68` = a specific set of landing data).
- **Enter** assembles and runs it.

Data entry flashes the Verb/Noun display to prompt you; you key digits into R1/R2/
R3, pressing `ENTR` between fields; `KEY REL` gives control back; `RSET` clears
alarms. Reproducing this state machine faithfully is a core Phase 1 task.

## How the AGC talked to the spacecraft: channels & counters

This is the seam of the whole project, so it gets the most attention.

The AGC did **not** have a memory-mapped IO bus like a modern microcontroller.
Instead it had two distinct mechanisms:

### 1. IO channels

A set of dedicated **input channels** and **output channels**, addressed by small
channel numbers and manipulated with special instructions (`READ`, `WRITE`,
`ROR`, `RAND`, etc.). Conceptually:

- **Output channels** drive things: DSKY lamps and the relay word that latches
  digits into the display; RCS jet-driver bits; discrete commands.
- **Input channels** read things: DSKY keycodes; discrete switches and flags from
  around the spacecraft.

Illustrative examples (validate against Virtual AGC in Phase 1):

- DSKY **keyboard input** arrives on an input channel as a **5-bit keycode**.
- DSKY **display output** goes out on an output channel using a "relay word" that
  selects *which* display element is being updated and supplies its digits/sign.
- **RCS jet commands** are driven by bits on a pair of output channels.

### 2. Counters (incremental IO)

Separate from channels, the AGC had hardware **counter registers** in erasable
memory that external pulses could **increment or decrement** asynchronously.
This is how *rate-like* and *integrating* sensors fed the computer:

- **Gyros / CDUs (Coupling Data Units):** attitude changes arrive as streams of
  pulses that bump angle counters.
- **PIPAs (accelerometers):** each velocity increment ("delta-V") is a pulse that
  bumps a velocity counter. The AGC integrates by counting.
- **Radar:** range/range-rate arrive as counter pulses.

For pocket-agc this is the key insight: **our simulated sensors don't hand the
AGC a float. They hand it a stream of increment pulses**, exactly like the real
CDUs and PIPAs — and the AGC's own software integrates them. Fidelity here is
what separates "authentic" from "faking it."

### Interrupts and the executive

The AGC ran a small **real-time OS**: an **Executive** that scheduled "jobs" by
priority, and a **Waitlist** for short time-driven tasks. Timing and IO events
raise **interrupts**. When too many high-rate jobs pile up and the Executive runs
out of core sets to allocate, it throws a **1201/1202** alarm, **restarts the
software**, and *keeps flying the essential jobs* — which is exactly what saved
the Apollo 11 landing when a misconfigured rendezvous radar flooded the CPU.

Reproducing this overload-and-recover behavior (because we run the real software
on a faithful CPU model) is one of the most compelling demos pocket-agc can show.

## Why the emulator matters more than the phone

Everything above is *behavior of the real machine*. `yaAGC` reproduces it because
it executes the real binaries on a cycle-accurate model of the real CPU. That is
why pocket-agc hosts `yaAGC` rather than writing new guidance code: the alarms,
the timing, the Verb/Noun catalog, the quirks — we get them *for free and
correct* because it is literally the same software the astronauts flew.

Our job is the **two ends of the channel/counter fabric**: the DSKY on one side,
and the simulated (then physical) devices on the other. That is what
[`02-architecture.md`](02-architecture.md) lays out.
