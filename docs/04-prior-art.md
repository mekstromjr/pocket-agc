# 04 — Prior Art, Reuse, and Licensing

pocket-agc stands on decades of preservation work by other people. This document
records what exists, what we reuse vs. build, and — critically — the **licensing
consequences** of reusing it. Get this right early; it is much cheaper than
untangling it later.

> Note: URLs and exact license versions below should be **verified in Phase 1**
> against the live sources before we distribute anything. They are recorded here
> as the research starting point, not as legal conclusions.

## The major bodies of prior art

### Virtual AGC (Ron Burkey et al.)

The center of gravity for the whole hobby. A long-running project that:

- **`yaAGC`** — a cycle-accurate AGC CPU emulator that runs the *assembled
  original binaries*.
- **`yaDSKY` / `yaDSKY2`** — DSKY emulators (our behavioral spec for the UI).
- **`yaAGS`**, **`yaTelemetry`**, and other peripheral tools.
- An **assembler** and the machinery to build the original source into runnable
  ropes.
- Extensive documentation of the CPU, the IO channels, the socket protocol, and
  the software.

**License:** GPL (verify version — historically GPLv2). This is the key
constraint: see "Licensing implications" below.

**Our stance:** *host and drive it as a black box.* Any patches we need (e.g.
build fixes for the phone toolchain) stay minimal and upstreamable.

### The original Apollo flight source (MIT Instrumentation Laboratory)

The actual guidance software, transcribed from the original hardcopy listings:

- **`Comanche055`** — Command Module, Apollo 11 (Colossus family).
- **`Luminary099`** — Lunar Module, Apollo 11 (Luminary family).
- Many other mission ropes exist (earlier/later missions, CM and LM).

Widely mirrored, including the well-known `chrislgarry/Apollo-11` GitHub
transcription and the copies within the Virtual AGC project used to build the
ropes.

**License:** the original software is **US government work / public domain**. The
*transcriptions and surrounding tooling* may carry their own licenses — verify
per source.

**Our stance:** we run these binaries unmodified. This is the "faithful, not
inspired-by" pillar made concrete.

### Hardware restorations & documentation (context, not code)

Efforts like the physical AGC restoration by curious engineers, museum
scans of the original schematics, and NASA/MIT technical reports (the "Colossus"
and "Luminary" GSOPs — Guidance System Operation Plans). These are **reference
material** for getting behavior and Verb/Noun catalogs right, not code we ship.

## Reuse vs. build — the honest split

| Concern | Source | We… |
|---|---|---|
| AGC CPU + real-time OS behavior | `yaAGC` | **reuse** (host as-is) |
| Apollo guidance/nav software | `Comanche055` / `Luminary099` | **reuse** (run unmodified) |
| DSKY *behavior* spec | `yaDSKY2` | **study & match** |
| DSKY *touchscreen UI* | — | **build** (original) |
| Simulated IO devices (IMU/PIPA/radar/RCS/engine) | — | **build** (original) |
| Spacecraft physics model | — | **build** (minimal, original) |
| Phone host integration / packaging | — | **build** (original) |
| USB IO board (Phase 3) | — | **design** (original, conceptual) |

The pattern: **reuse the irreplaceable, historically-accurate core; build the
integration, the interface, and the world.** That is both the least effort and
the most fidelity.

## Licensing implications (read this before Phase 1)

The tension: our own docs and future code are currently **MIT** (permissive), but
`yaAGC` is **GPL**. Consequences to reason about *before* we distribute a build:

- **Running `yaAGC` as a separate process** we merely launch and talk to over a
  socket is a weaker coupling than statically linking it. Whether that keeps our
  DSKY/IO code independent of the GPL depends on the details (how we distribute
  it, whether it's "mere aggregation," etc.). This needs a real read of the GPL
  and, ideally, a sanity check — **not** a guess in this doc.
- **The safe default** if we ever bundle/distribute `yaAGC` together with our code
  is to license the combined, distributable work under a **GPL-compatible**
  license.
- **The clean architecture helps us here too:** because the brain/face/world split
  keeps `yaAGC` behind a socket, our UI and device code *can* be structured as
  independent programs — which preserves options.

**Decision (tracked, not yet made):** keep this repo MIT while it is docs-only.
Revisit the license the moment Phase 1 produces a distributable artifact that
includes or links `yaAGC`. Record the final decision here and update
[`../README.md`](../README.md) and [`../LICENSE`](../LICENSE) accordingly.

## Attribution commitments

Whatever we ship will prominently credit:

- **Ron Burkey and the Virtual AGC contributors** — without `yaAGC` this project
  is years longer.
- **The MIT Instrumentation Laboratory** and the Apollo software engineers
  (including the teams led by figures like Margaret Hamilton) whose software we
  literally run.
- The broader AGC preservation and restoration community.

Standing on giants' shoulders is the whole point; say so loudly.
