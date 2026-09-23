# Exploring Fabric data agents

A hands-on guide to the interesting part: **how the agent decides where to send a question.**

Setup is in [`README.md`](README.md). This assumes `InsuranceAgent` is published.

---

## The idea

Three sources, one dataset. None of them is "best" — each can do something the others can't.

| Source | Language | Can do | Can't do |
|---|---|---|---|
| `InsuranceSM` | DAX | Aggregate, rank, trend | Reach fields not in the model |
| `RF_Lakehouse` | T-SQL | CTEs, windows, NULL checks, row lists | — |
| `InsuranceOntology_ManualGen` | Graph | Multi-hop traversal | **Aggregate anything** |

Most agent demos have one source, so routing never matters. Here it's the whole point.

---

## 1. Watch it route

Ask these back to back and check which source it picks each time.

**→ Semantic model**
```
What are total estimated loss, approved payout, and loss ratio by claim type?
```
Pure aggregation. Should go to DAX and never touch the other two.

**→ Lakehouse**
```
Which claims have estimated losses greater than their policy coverage amount?
```
This compares a row against a threshold on a *related* table. The semantic model
can't express it. Watch it fall through to SQL.

**→ Ontology**
```
Which adjuster handled claim CN500042, and what policy covers the asset involved?
```
Two hops from one known record. An aggregate query here would be the wrong tool.

**The point:** same data, three engines, and the agent chose. Ask it *why* it picked
one — the instructions tell it to state its source.

---

## 2. Try to break it

This is where you learn how much the configuration is doing.

### Invented categories
```
How many claims are closed?
```
There is no `closed` status. A correctly configured agent says so and offers
`paid`/`denied` instead of quietly returning a number.

Statuses are: `filed`, `under_review`, `investigation`, `approved`, `paid`, `denied`.

### The NULL trap
```
What's the total approved payout?
Now how many claims have no approved amount at all?
```
`approved_amount` is NULL until a claim is paid, and `SUM` skips NULLs — so the first
number silently excludes unpaid claims. The agent should say how it handled that.

Ask it directly: *"Does that total include unpaid claims?"*

### The fan-out trap
```
For each claim, how many events occurred and what was the total event cost?
```
Joining `claims` to `claim_events` multiplies rows. Aggregate the events first or the
counts inflate. This is exactly what the few-shot examples teach — see below.

### Ambiguity
```
Which adjuster worked on claim CN500107?
```
There are two answers. `claims.adjuster_id` is who was *assigned*;
`claim_events.adjuster_id` is who *performed the work*. A good agent asks or states
which one it used.

### Out of scope
```
What's the average customer satisfaction score by office?
```
No such column. It should decline, not invent one.

---

## 3. See what the configuration is worth

The most instructive experiment in this repo:

1. Note the answer to the fan-out question above.
2. Remove `example-queries.json` from the lakehouse data source.
3. Republish and ask it again.

Without the 15 few-shot pairs the agent tends to join naively and report inflated
totals — confidently. Put them back and it aggregates in a CTE first.

Same model, same data, same question. The difference is entirely configuration.

Worth trying the same with `topics.md` and the per-source instructions.

---

## 4. Configuration layers

Four places to steer behaviour, broad to narrow:

| Layer | File | Controls |
|---|---|---|
| Agent instructions | `agent-instructions.md` | Routing, fallback, output format |
| Source instructions | `data-sources/*/instructions.md` | Dialect and semantics for one source |
| Topics | `1-lakehouse-*/topics.md` | Guidance triggered by subject matter |
| Example queries | `1-lakehouse-*/example-queries.json` | Concrete join paths and patterns |

Rough rule: **rules** go in instructions, **patterns** go in examples. If the agent
keeps making the same structural mistake, a few-shot example fixes it faster than more
prose.

---

## 5. Make it yours

Small changes with visible effects:

- **Add a measure** to `InsuranceSM`, then update
  `2-semantic-model-InsuranceSM/instructions.md` to mention it. Watch it get used.
- **Add a topic** for fraud signals (SIU review events, claims above coverage on recent
  policies) and see questions start routing differently.
- **Tighten the output rules** — force every answer to state row counts and date range.
- **Remove a source** entirely and watch the routing degrade. Instructive.

---

## Reference: full question set

Grouped by intended route. All return real results on the quickstart data.

**Semantic model**
- What are total estimated loss, approved payout, and loss ratio by claim type?
- What's the monthly trend in claims filed in Q1 2026?
- Which offices have the highest approved payouts?
- Which claim types have an average estimated loss above the overall average?
- How many claims are in each status?

**Lakehouse**
- Which claims have estimated losses greater than their policy coverage amount?
- Which adjusters are closest to or over their maximum active-claim capacity?
- Which insured assets have no recorded last inspection date?
- Which claim events are still incomplete, with their claim and adjuster?
- For each claim, how many events occurred and what was their total cost?
- How many claims and how much estimated loss per policyholder state?

**Ontology**
- Which adjuster handled claim CN500042?
- What policy covers the asset involved in claim CN500042, and who holds it?
- What events occurred on claim CN500107, and who performed them?
- Which office is that adjuster based at?

**Stress tests**
- How many claims are closed?
- What's the average customer satisfaction score by office?
- Does the approved payout total include unpaid claims?

> Claim numbers run `CN500001`–`CN500400` on the quickstart data. If you loaded the
> workshop dataset instead, check `claims` for valid numbers first.
