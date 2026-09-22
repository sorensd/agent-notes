# Estimating duration and generating a plan for agent-built software

How long will it take? With a coding agent doing the building, the honest answer is a
different shape from the one a team of people would give. This note records what was measured
on one production build (a multi-tenant clinic platform on Cloudflare Workers, 2026), how far
the measurement was from the first estimate, and a procedure for estimating the next project
that starts from the measurement rather than from habit.

---

## What was measured

The first plan for the project was written the way software plans usually are: phases, slices,
person-days. It put the foundation phase (auth with a second factor, tenancy, permissions as
data, an encrypted settings store, email, the two console shells, the module registry, tests)
at **22 agent-days**.

The foundation shipped, deployed and tested, in **about four agent-hours** of focused build.

The nine feature slices that followed (schedules and slot generation, patients, a live day
board over a Durable Object, WhatsApp plumbing with queues, a medical safety screen, a booking
flow on WhatsApp, bills and PayU payment links, ABHA linking against the ABDM sandbox contract,
an onboarding checklist with plans) took **two to five agent-hours each**, and ten slices went
from first line to deployed in **four calendar days**.

So the first estimate was off by roughly **40×** on the foundation, and the phase plan that
had been scaled from it was off by the same factor. Nothing about the work was unusually easy;
the estimate was simply measuring the wrong thing.

## Why the old numbers do not transfer

A person-day estimate bundles together things an agent does not pay for in the same currency:

- **Reading time.** A person spends a large share of a task reading code, docs and tickets.
  An agent reads a file in a fraction of a second and holds the whole project in one context.
- **Context switching and warm-up.** There is none. The agent that wrote the migration writes
  the API, the screen, the tests and the release note in one sitting.
- **Typing.** Negligible. A 400-line route file is produced in the time a person takes to
  open the editor.
- **Meetings and waiting.** None inside a slice. The waiting moves to the edges (see gates).

What the agent *does* pay for, and what the person-day estimate did not price separately:

- **Rounds of verification.** Typecheck, lint, test, deploy, check the deployed thing. Each
  round is a minute or two, and a slice takes five to fifteen of them. This is where most of
  an agent-hour goes.
- **Contract lookups.** Reading a vendor's documentation to get a request shape or a limit
  right (PayU's 25-character `txnid`; ABDM's OAEP-SHA-1 encryption) is a real cost, ten to
  thirty minutes per integration, and skipping it produces code that passes every local test
  and fails at the vendor's door.
- **Fixing what the first pass got wrong.** Roughly a third of a slice's time is spent on
  defects the agent introduced in the first pass and caught in its own tests (a race, a
  parameter limit, a wrong assumption about a helper's signature). This is normal and should
  be in the estimate, not treated as slippage.
- **Human turnaround.** Answers, credentials, reviews, and the owner testing live. These do
  not cost agent-hours but they set the calendar.

## The estimator

Estimate in **agent-hours of focused build**, per closed-loop slice, and keep calendar time
as a separate number driven by gates. Do not estimate in days or weeks of build.

### Step 1: cut the work into closed-loop slices

A slice is one increment that goes migration → API → UI → audit → tests → demo path, on both
consoles if the product has two (delivery-rules.md, "the closed-loop rule"). If a piece of work
cannot be demonstrated end to end, it is not a slice yet; cut again.

A well-cut slice has a one-sentence demo path ("raise a ₹500 bill, send the link, PayU
confirms, the cash sheet shows it"). If the demo sentence needs "and" three times, it is two
slices.

### Step 2: size each slice by its shape, not its topic

The measured costs clustered by shape:

| Shape | Agent-hours | What drives it |
|---|---|---|
| CRUD on a new table with a list and a form | 1–2 | One migration, two or three routes, one screen, one test file |
| A workflow with state and concurrency (booking, payments, holds) | 3–5 | Conditional updates, compensation, a test per race |
| A third-party integration with a documented contract | 3–5 | Reading the docs, a fake in the harness, a contract test, the encryption or signing |
| Real-time or push (Durable Object, WebSocket, queue) | 3–4 | The DO, its serialisation, the reconnecting client |
| A platform/oversight screen over existing data | 1–2 | One batched loader, one page, the "never shows patient content" test |
| A foundation (auth, tenancy, permissions, settings, shells) | 4–6 total | Larger, but it is one slice with many small parts, not twenty |

Add **one hour per external contract** the slice depends on (a payment gateway, a
government API, a messaging provider) for reading the current documentation and building a
fake that enforces it. Add **half an hour** if the slice must ship on two consoles.

### Step 3: apply the correction that the measurement demands

If you (or an agent) produced a first estimate in person-days out of habit, divide it by
somewhere between 20 and 50 to get agent-hours, then re-derive from Step 2 anyway. The
division is a sanity check, not the estimate.

If your first-pass number for a slice is under an hour, it is wrong: no closed loop with
tests and a deploy has come in under an hour on this project.

### Step 4: separate calendar time and name its drivers

Calendar time is not the sum of agent-hours. It is set by:

- **Gates that need a human**: credentials from a vendor, a sandbox account, a domain, an
  API key from a government portal, a legal or accounting answer. Each one is days to weeks,
  and the build cannot be finished without it. List every one at planning time with who
  provides it.
- **Review and live testing by the owner**: usually one round per slice; a day between
  slices if the owner is part-time.
- **Vendor turnaround**: WhatsApp template approval (hours to days), payment gateway
  activation (days), ABDM production access (weeks).
- **The agent's own rate limits or session limits**: real, and worth a line in the plan.

Write the calendar as "N agent-hours of build; calendar bounded by these gates: …", and
list the gates. A plan that gives a date without listing its gates will be wrong in the
direction of optimism, because every unlisted gate is a surprise.

### Step 5: keep a measured table and re-scale

After each slice, record the actual agent-hours next to the estimate. Re-scale the remaining
slices from the running ratio, not from the original plan. On this project the foundation's
measurement was applied to the rest of the plan the day it shipped, and the later slices
landed within the re-scaled range.

## Generating the plan

The plan document should be produced from the estimator, not written freehand. A workable
shape, used on this project as a single HTML page:

1. **Phases** with a one-line goal and the gate that opens the next phase.
2. **A table of slices per phase** with these columns, in this order: id, name, what it
   builds, the platform/oversight side, the demo path, what the tests must prove,
   agent-hours. A slice without a demo path or a test column is not planned yet.
3. **A row per gate** with the provider, what is needed, and which slices are blocked.
4. **The totals**: agent-hours per phase and overall; calendar as a range with its drivers.
5. **The measurement line**: how the numbers were derived ("P0 was estimated at 22 days and
   shipped in four hours; the rest is scaled from that"). Keep this honest and keep it
   updated; it is what lets the reader trust the rest.

When the plan is generated by an agent, ask it for the slice table first and the totals
last, and reject any slice whose demo path cannot be performed by a person with a browser.

## What to tell a stakeholder

"The build is measured in agent-hours and the calendar is set by the things we are waiting on.
Phase N is about H agent-hours; it will land within D days of the last gate opening. The gates
are: …" This is a claim that can be checked, which a person-day estimate never was.

## Failure modes seen

- **Estimating from topic prestige.** "Payments" sounds big; it was four hours, because the
  contract was documented and the fake enforced it. "Patient search" sounds small; it was
  two, because of index-backed prefix search and merge. Size by shape.
- **Treating the agent's self-corrections as slippage.** A third of every slice is fixing
  the first pass. Budget it; do not report it as a problem.
- **Forgetting the platform side.** A slice that ships the clinic screen without the
  oversight screen is half a slice and will be finished later at higher cost.
- **Letting the calendar borrow from the agent-hours.** A slice blocked on a sandbox
  credential is not "in progress"; it is waiting. Say so, and work on an unblocked slice.
