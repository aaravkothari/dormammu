# Dispatch

Parallel AI phone calls for phone-gated bottlenecks. Built at SpeedHacks (a16z speedrun, Oct 11 2026).

## Thesis

A lot of real-world tasks are bottlenecked behind a phone call: getting quotes,
booking an appointment, negotiating or canceling a bill. Each one is solvable by
a single AI caller, but the real leverage is running many of them **at once** —
one request in, N calls out, results aggregated back.

## Tracks (demo scope)

1. **Comparison shopping** — dispatch simultaneous calls to multiple vendors
   (movers, insurance, contractors), come back with a ranked quote table.
2. **Appointment scheduling** — call phone-only booking lines and lock in a slot.
3. **Bill negotiation / cancellation** — call a vendor (or, for the demo, our own
   controlled test line) to negotiate a rate or cancel.

## Architecture (placeholder — no implementation yet)

```
router/        task router: plain-language request -> N per-target call scripts
callers/       per-call conversation agent (Dial-backed), one call session per target
aggregator/    normalizes heterogeneous call outcomes into one comparable result
dashboard/     live view of concurrent call status + aggregated results
test-lines/    our own controlled IVR test line(s) for safe/rehearsable demo calls
```

## Sponsor tool

Built on **Dial** (a16z speedrun) — gives the caller agents real phone numbers to
place calls, navigate IVR trees, and talk to a human rep.

## Status

Barebones template only. No implementation yet.
