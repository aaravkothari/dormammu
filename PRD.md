# Dormammu — PRD

**Built at:** SpeedHacks (a16z speedrun hackathon), Oct 11 2026, SF Tech Week
**Brief:** "Give people their time back — remove a meaningful bottleneck in how people work, learn, or access services."

## 1. Problem

Huge amounts of real-world adult life run through phone calls that are individually
low-value but collectively enormous time sinks: comparison shopping for a service,
booking something phone-gated, negotiating or canceling a bill, setting up new
accounts. Nobody does all of them well because each one means holding, navigating
an IVR, and talking to a human — repeated across every vendor you need to reach.

This bottleneck spikes hardest at life transitions: moving to a new place, starting
college, turning 18, starting a new job. In a single week you might need insurance
quotes, movers, a new ISP, a doctor, a dentist, a bank, and to cancel whatever you
had at the old address — each one its own phone call, usually during business hours,
usually on hold.

## 2. Product

Dormammu is an agentic calling system. One request from the user ("I'm moving next
month, handle it") triages everything that needs a phone call, spawns N calls
**simultaneously**, and aggregates the results back into one place.

**Target user:** general-purpose — anyone, not a narrow persona. The strongest
demo wedge is life-transition moments (moving, turning 18, starting school/a job)
because the number of simultaneous phone-gated tasks peaks there, but the mechanic
works for anyone's ongoing bills/appointments/subscriptions too.

**Without Dormammu:** see appendix — money and time leak out continuously through
tasks that are individually not worth a person's attention but collectively add up,
because the only path today is doing each call yourself, sequentially.

## 3. Scope for today (all three tracks)

1. **Comparison shopping** — dispatch simultaneous calls to multiple vendors
   (movers, insurance, ISPs, etc.), return a ranked quote table.
2. **Appointment scheduling** — call phone-only booking lines (doctor, DMV-adjacent
   services, repair shops), check the user's calendar, lock in a time.
3. **Bill negotiation / cancellation** — call a vendor to negotiate a rate or cancel
   an old service.

All three share the same core loop: triage → dispatch N calls in parallel → per-call
agent executes its objective → aggregator normalizes results → user sees one
combined outcome.

## 4. Demo mechanic

Submission is a **recorded video**, not a live on-stage call. This removes the
live-call risk flagged in earlier review (hold times, real-time consent law,
IVR unpredictability eating a timed demo slot) — calls can be placed ahead of time
against our own controlled test line(s) and/or real low-risk targets, recorded, and
edited into the submission. Still show the parallel-dispatch moment on screen
(N calls, concurrent status, results landing) since that's the actual thesis.

## 5. Core services

| Need | Choice | Why |
|---|---|---|
| Calling / telephony | **Dial** (getdial.ai) | Sponsor tool, gives agents real phone numbers, MCP server + SDK already connected to this repo. Likely already bundles STT/TTS — confirm via docs.getdial.ai before adding a separate voice stack. |
| STT/TTS (if Dial's own pipeline isn't sufficient) | **Ultravox** or **ElevenLabs Conversational AI** | Near-complete speech-to-speech pipelines (not just raw STT or TTS), smallest integration surface for a 12hr build. Deepgram+Aura-2 as the "boring but reliable" fallback. |
| Calendar | **Google Calendar API** | Obvious choice for scheduling track. Use a Desktop-app OAuth client for fastest local setup; add team Google accounts as OAuth test users to avoid `403: access_denied`; keep `credentials.json` named exactly that per Google's sample code. |
| Mock vendor data (comparison shopping) | **Our own fixture data + our own test phone line(s)**, not a third-party sandbox | Full control, zero live-dependency risk, demoable and rehearsable. A hand-built JSON of 4-5 vendor quotes read back via a simple IVR/TTS stub we control. |
| Agent-performance analytics / continuous improvement | **Keewano (KeewanoDB)** | Core MVP feature — see §6. |

## 6. Keewano integration (core MVP, not a stretch goal)

Keewano's current product (KeewanoDB, launched Sept 2026) is a general event-sequence
database for AI agents, not gaming-locked despite the "gaming analytics" label —
agents are first-class entities in its data model.

**What it does for Dormammu:**
- Every call (each caller-agent, each turn, each tool-call, each outcome) is logged
  to KeewanoDB as an ordered event sequence per entity (one entity per call/agent run).
- Use **Keewano MCP for Agents** to query those sequences for patterns: which event
  sequences precede a failed negotiation vs. a successful one, which IVR navigation
  paths dead-end, which call scripts get hung up on.
- Feed those patterns back into the per-track call scripts/prompts — this is the
  "continually improve the agentic stack" loop: Keewano powers the analysis step,
  we close the loop by updating prompts/logic based on what it surfaces.
- Honest scope note: Keewano analyzes and surfaces patterns, it does not
  auto-rewrite prompts. The improvement loop is Keewano-informed, not fully
  autonomous — state this plainly in the pitch rather than overclaiming.

**Setup:** install `@getdial/sdk`-equivalent via Keewano's TypeScript SDK
(`Keewano.init({ apiKey })`), ship call events via SDK or REST, query via MCP/Lua.
No hackathon-specific quickstart page found for Keewano (unlike Dial's) — check
their booth/Discord at the event for a demo API key or sandbox dataset.

## 7. Competitive landscape

- **Pine AI (19pine.ai)** — closest direct competitor. AI voice agent that calls
  Comcast/Verizon/AT&T/etc. to negotiate/cancel. $25M Series A. Self-reported 93%
  success rate, $3M saved aggregate (treat as their claim, not verified fact).
  Scoped to billing/subscriptions only, sequential.
- **Billshark, Trim, Rocky** — established bill-negotiation apps, mostly human- or
  semi-automated, not full voice-agent-led.
- **DoNotPay** — operating under a Jan 2025 FTC final order over "AI lawyer"
  marketing claims. Cautionary note: do not overclaim capabilities/success rates
  in Dormammu's own pitch, given regulatory scrutiny of this exact category.
- **Wajo AI ("Fo")** — general task assistant combining calendar + calling + quotes,
  human-in-the-loop fallback. Architecturally closest to our multi-track scope.

**Differentiation:** every competitor found is single-purpose and sequential (one
call at a time, one domain — usually bills). Dormammu's fan-out mechanic — N
simultaneous calls across arbitrary phone-gated tasks, aggregated back into one
result — was not found in any competitor. That is the core unique claim.

## 8. Open questions / risks

- Confirm Dial's own voice pipeline (STT/TTS/telephony) is sufficient before adding
  a second voice vendor — check docs.getdial.ai first.
- Verify any third-party mock-data APIs (e.g. insurance quote sandboxes) are
  actually reachable before depending on them; prefer our own fixtures.
- Keewano hackathon-day setup (API key, sandbox dataset) unconfirmed — check with
  Keewano reps at the event.
- Recording-consent rules apply even for a pre-recorded demo if real vendors/humans
  are called and included in the submitted video — get verbal consent on-call or
  use our own test line's human to stay safe.

## Appendix: without Dormammu

- Full price on every renewal because negotiating means holding, so people don't.
- Comparison shopping means calling 3-5 places yourself over several days.
- Phone-gated bookings mean calling during business hours or giving up and taking
  whatever's most convenient instead of best.
- Old subscriptions stay active for months because canceling requires a call people
  keep putting off.
- Nobody tracks all of this against actual spend, so overpayments sit unnoticed
  across many separate bills.
- Net effect: money and time leak out continuously, in amounts too small
  individually to act on, but large in aggregate — especially at life transitions
  when many of these hit at once.
