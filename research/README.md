# research/

Raw material, not polished docs. This is the scratch space for the Phase 0
research backlog: reference tables, source links, and a running **fidelity log**
of surprising things the real AGC did. Content here is allowed to be messy,
half-verified, and contradictory — the polished conclusions graduate into
`../docs/`.

## To confirm (open research items)

These are the "confirm in Phase 1" flags scattered through the docs, collected:

- [ ] **Channel map.** Exact input/output channel numbers for: DSKY keyboard in,
      DSKY display out (relay-word format), RCS jet drivers, engine discretes,
      discrete switches. Source: Virtual AGC docs + original source listings.
- [ ] **Counter map.** Which erasable counters take pulses from CDUs, PIPAs, and
      radar, and the **pulse weight** (physical units per pulse) for each.
- [ ] **Keycodes.** The 5-bit keycode for each of the 19 DSKY keys.
- [ ] **Lamp semantics.** For every caution/warning + status lamp: what drives it,
      and steady-vs-flash behavior.
- [ ] **`yaAGC` socket protocol.** The exact packet format for channel IO and
      counter increments, and how a peripheral connects.
- [ ] **License versions.** Confirm the GPL version on Virtual AGC and the
      provenance/license of each source mirror we use.
- [ ] **Verb/Noun catalog.** The subset needed for the powered-descent demo
      (P63/P64) and the monitor Verbs used in Phase 1.

## Candidate primary sources (verify URLs in Phase 1)

- **Virtual AGC** — Ron Burkey's project: the emulator, `yaDSKY2`, the assembler,
  and its extensive documentation of the CPU and IO.
- **Original Apollo source** — `Comanche055` (CM) and `Luminary099` (LM)
  transcriptions (e.g. the `chrislgarry/Apollo-11` mirror and the copies within
  Virtual AGC).
- **GSOPs** — the MIT "Colossus" and "Luminary" Guidance System Operation Plans,
  and related NASA/MIT technical reports.
- **AGC hardware restoration** writeups and schematic scans — for behavioral and
  timing sanity checks.

## Fidelity log

Append surprising, verified facts here as we find them — one bullet each, with a
source. These anchor correctness *and* make good story beats.

- _(none yet — Phase 0 in progress)_
