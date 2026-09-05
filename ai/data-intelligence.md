# Building data intelligence into a product

Notes from designing an analytics-and-suggestions layer for an operations system. Written
before building it, which is the point: the failure modes here are commercial rather than
technical, and the wrong shape is expensive to unwind.

Applies to anything of the form *"the platform should study itself and tell us what to
do"* — anomaly detection, recommendations, forecasting, an AI assistant over your own data.

---

## The first question is not technical

**Every question of this kind is a comparison against a baseline.** "A sudden drop in
vendor performance" means *against what that vendor normally does*. "A drop in demand"
means *against what demand normally is*. "An unusual spike" means *unusual for this time of
year*.

So before anything else: **how much history do you actually have?**

If the answer is "not much", the honest sequence is:

1. Start recording facts now, even though nothing reads them for months
2. Ship a **present-state** advisor — real, useful, and clearly scoped
3. Gate everything predictive on the history existing

A system that answers anyway is not intelligent, it is **confident**. That is worse than
silence, because **trust is spent once**: the first time it calls something a problem on
three data points and someone acts on it, nobody believes the true signal six months later.

## Separate the products hiding inside the request

"Make it data intelligent" is usually four or five different products sharing a name, with
completely different prerequisites, costs and risk:

| Layer | What it is | Prerequisite |
|---|---|---|
| Internal signals | Statistics over your own data | Your own history |
| Explanation | Turning signals into sentences, ranked | The layer above |
| Operational optimisation | Routing, scheduling, carrier choice | Records of those operations |
| External signals | News, events, market data | A paid source, and tolerance for noise |
| Autonomous action | Doing it, not suggesting it | A track record from the layers above |

Building them as one thing produces something impressive in a demo and untrustworthy in
use. Ship them in that order.

## The statistics are not AI

"Six days slower than the 90-day average" is a `SELECT`. It is faster, free, reproducible,
auditable to the row, and more reliable than any model.

**The model's job is the sentence, not the number.** Give it computed facts; let it rank by
impact and explain. If a figure appears in the output, a query produced it.

This is not purity for its own sake. A model that generates numbers will eventually
generate a plausible wrong one, and in an operations tool somebody will act on it.

## Three rules that make it trustworthy

**Every detector has a "not enough data" state.** Below a minimum sample it reports
insufficient data rather than a finding. This is the single most important rule.

**Every signal carries its evidence** — the numbers, the window, and links to the specific
rows behind it. A finding you cannot audit is a finding you cannot act on.

**Internal facts and external readings must look different on screen.** "Our margin fell
8%" comes from your database. "A tournament may be coming to this city" is a model's
reading of a web page. Same list, same weight, and people stop trusting both.

## "Auto-improving", done honestly

Not a model retraining itself in the dark. A feedback loop you can see:

- Every signal records whether it was **acted on, dismissed, or ignored**, and why
- Detector thresholds tune from that, **visibly and reversibly**
- A signal type dismissed as noise repeatedly is widened or suppressed, and says so
- Forecast accuracy is scored against what actually happened

And build **a page listing what it got wrong**. A system that only reports its successes is
one nobody should trust.

## Autonomy is earned, per class of action

| Stage | What it may do |
|---|---|
| 1 | Suggest. A person acts. The outcome is recorded. |
| 2 | Propose a **specific** change as a one-click action, pre-filled. Still a human click. |
| 3 | Named, reviewable playbooks a human enables per signal type. |
| 4 | Unattended execution of **reversible, bounded, audited** actions, with a kill switch. |

A class reaches stage 4 only with a track record from stages 1–2. Anything touching money,
capacity commitments, or contact with a customer or supplier **stays at stage 1
permanently**.

## Architecture that keeps the costs bounded

```
  business tables
        ↓  nightly + on demand
  FACT LAYER      dated, append-only, one row per metric per entity per day
        ↓
  DETECTORS       deterministic comparisons against rolling baselines
        ↓         emit signals with evidence, severity, confidence
  ENRICHMENT      external context, joined to your own exposure, clearly labelled
        ↓
  NARRATOR        ONE model call per run over the computed bundle
        ↓
  SURFACE         dashboard block, full feed, digests
        ↓
  FEEDBACK        acted / dismissed / ignored → tunes the detectors
```

- **One model call per run**, over a fact bundle — not one per record. Nightly is ~30 calls
  a month; per-record is thousands.
- The intelligence layer **never writes to a business table**. Its own tables only. A bug
  in a detector must not be able to change an order.
- It runs as a **system actor** with its own audit trail, distinguishable from a person's.
- **Kill switch and budget cap in settings**, effective without a deploy.

## The part that changes work you are doing today

An intelligence layer does not merely come *after* the rest of the system. **It is the
reason several earlier parts must record more than they need in order to operate.**

- Store the **quoted** transit time next to the **actual** one, even though nothing reads it
- Timestamp **stage transitions**, not just the current stage
- Record **who a request was waiting on**, not just that it was open

None of that is needed to ship those features. All of it is needed later, and **none of it
can be back-filled**.

So the standing question when designing anything is: *what will the analytics layer want to
have been recorded?* It costs nothing at the time and is unanswerable afterwards.

## A chat assistant over your own data

Cheapest module to build, most tempting to build first, and it still belongs after the fact
layer. Grounded on rich metrics it is genuinely useful; grounded on an empty database it
can only tell you what you already knew, while looking like it should know more.

When you do build it:

- **Role-gated on its own permission.** Asking costs money per call and summarises the whole
  business in one place — a different decision from being allowed to view one record.
- **Scoped.** It answers questions about your business and your system, and declines the
  rest. An assistant that will discuss anything is a general chatbot wearing a company logo.
- **Grounded.** It receives a computed snapshot and never produces a figure of its own. Not
  in the snapshot means "I don't have that, it's on this screen".
- **Read-only by construction.** Asked to change something, it says where to do it.
- **Conversations stored**, not ephemeral. An exchange that influences a business decision
  should be reviewable afterwards.
- **Record which provider and model answered**, so a drop in answer quality traces to a
  configuration change rather than a guess.
