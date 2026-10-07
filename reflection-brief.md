# Reflection Brief — Evaluation and Observability Capstone

**Name:** Gaurav Walunje
**Date:** 2026-09-27

> Ground every answer in your own run. When a question asks for a number, file name, or line, paste
> it from your artifacts — a reviewer should be able to find it. Answers that are correct in the
> abstract but cite nothing do not meet the bar. Keep it short and specific.

---

## 0. Environment

| Field | Value |
|---|---|
| OS & version | Linux, kernel 6.6.97+ |
| Python version | Python 3.13.0 |
| Date run | 2026-09-27 |
| Ran any system live? (which) | System 1 live pipeline was attempted, but it failed with HTTP 401 Unauthorized. |

---

## 1. Validated, routed pipeline

| Evidence | Value |
|---|---|
| Passing test count | 45 passed, 3 skipped |
| Routing output file | `routing_decisions.json` was not generated because the live pipeline failed with HTTP 401; the documented routing-test fallback was captured in `routing-tests.txt`. |
| auto_approve / human_review / spot_check counts | Not available from the captured fallback artifact |

**1a. Retry boundary.** From your perturbation run (a required field removed), paste the escalation
record. How many API calls did the system make, and why is retrying a futile case worse than
escalating it?

> In `01-policy-pipeline/api-call-count.txt`, the follow-up missing-source run records
> `result=RetryFutileEscalation` and `api_call_count=1`. The underlying test also sets `endorsements`
> to null and asserts `client.call_count == 1`. The required information is genuinely absent from the
> source, so retrying cannot recover it and is less useful than escalating the case.

**1b. Reading the router.** Pick one `human_review` record from your routing output. Which of the
three signals (confidence, reviewer, integration) sent it to a human? If you had trusted the model's
confidence alone, what would have happened?

> Because `pipeline-run.txt` failed with `HTTP 401 Unauthorized`, I used the documented fallback
> in `01-policy-pipeline/routing-tests.txt`. The line
> `test_ac_04_06_high_confidence_plus_reviewer_disagreement_still_routes_to_human_review PASSED`
> shows a human-review route caused by the independent reviewer signal. The corresponding test uses
> 0.99 extractor confidence, a reviewer disagreement on `premium_amount`, and passing integration
> checks, yet the decision is `human_review`. Trusting confidence alone could have incorrectly allowed
> auto-approval despite the reviewer disagreement.

**1c. Where the aggregate lies.** Run the calibration snippet. Quote the one cell whose accuracy lags
its confidence, plus the overall figure. What does slicing by `policy_type × field` catch that a
single number hides?

> In `01-policy-pipeline/calibration-report.txt`, the `umbrella exclusions` cell has
> `n=2 conf=0.93 acc=0.00 brier=0.865`, while the overall result is
> `OVERALL brier=0.291`. The slice exposes a specific policy-type/field weakness that the overall
> Brier score alone can hide.

---

## 2. Schema-enforced two-pass extraction

| Evidence | Value |
|---|---|
| Passing test count | 25 passed |
| Document run | `02-mortgage-extraction/document-runs.txt` |
| Classified type | appraisal |

**2a. Two guarantees.** Paste your discrepancy-run output. Tool use already forces valid JSON, yet the
validator still catches a bad sum. Why are these two different guarantees? Name one error each
cannot catch.

> In `02-mortgage-extraction/document-runs.txt`, the discrepancy run reports
> `"consistent": false` with `calculated: 9642.17`, `stated: 10892.17`, and
> `delta: -1250.0`. Valid JSON guarantees structural/parsing validity; it cannot prove that numeric
> relationships are correct. The mathematical validator checks the arithmetic relationship, but it
> cannot by itself guarantee that the extracted JSON has the correct schema structure.

**2b. Refusing to fabricate.** Run on a document missing a field. Paste that field's output. Why null
instead of an invented value? Point to the schema choice that allows it.

> In `02-mortgage-extraction/document-runs.txt`, the missing-bonus run contains
> `"bonus_monthly": null`. The extraction tests also include
> `test_ac_03_03_missing_bonus_returns_none PASSED`. A nullable field allows the system to represent
> missing source information as null rather than inventing a value.

**2c. Normalization.** Quote one field where the source text and extracted value differ in format
("about 2,400 sq ft" → `2400`). Why normalize at extraction time rather than downstream?

> In `02-mortgage-extraction/document-runs.txt`, `gross_living_area_sqft` is extracted as `2400`.
> The normalization test also passed as `test_ac_03_04_informal_sqft_normalized_to_integer`.
> Normalizing during extraction puts the value into the structured form expected by downstream
> validation and use immediately.

---

## 3. Multi-source synthesis

| Evidence | Value |
|---|---|
| Passing test count | 34 passed |
| Briefing file | `03-supply-chain/investigation-runs.txt` |
| Section the conflict landed in | Contested |

**3a. Annotate, don't arbitrate.** Quote one conflicting-metric pair from your briefing — both values,
sources, dates. Give one way a reader is better served by the preserved conflict than by a single
reconciled number.

> In `03-supply-chain/investigation-runs.txt`, on-time delivery is reported as `95.0 percent —
> supplier_audit (as of 2026-04-10)` and `78.0 percent — logistics (as of 2026-04-05)`.
> Preserving both values lets the reader see the disagreement and the dates and sources behind it
> instead of hiding the disagreement inside one unexplained number.

**3b. Source goes dark.** Run with `--simulate-timeout`. Paste the part of the briefing showing the
failed source. How is "unreachable" handled differently from "nothing to report," and why does the
run still finish?

> The timeout run contains `Sources unavailable: logistics unavailable (timeout)`.
> It also places `late_shipment_count` in Incomplete with `missing source: timeout reading logistics`.
> This differs from `production_capacity_utilization`, which is incomplete because no source reported
> it. The run still finishes because the failed source is annotated and the remaining sources are
> synthesized instead of aborting the investigation.

**3c. Dates as a guardrail.** Quote two claims about the same supplier with different dates. How does
requiring a date stop a time difference from reading as a contradiction?

> The briefing reports on-time delivery as `95.0 percent` from supplier_audit on `2026-04-10` and
> `78.0 percent` from logistics on `2026-04-05`. Recording the dates makes the time context explicit,
> so the reader can distinguish different snapshots rather than treating every difference as an
> undated contradiction.

---

## 4. Synthesis

**4a. One principle.** Name the single moment in your runs (system + artifact) where *evaluate the
output, don't trust the model's word* most clearly caught something a trusting design would have
shipped.

> System 2, in `02-mortgage-extraction/document-runs.txt`, most clearly demonstrated this. The
> extracted JSON looked structurally valid, but deterministic validation marked it
> `"consistent": false` because the calculated total was `9642.17` while the stated total was
> `10892.17`. A trusting design could have accepted the stated total.

**4b. Confidence ≠ correctness.** Pick the system where this mattered most, and explain why using
something you observed.

> System 1's `calibration-report.txt` shows this directly: the `umbrella exclusions` slice had
> confidence `0.93` but observed accuracy `0.00`. The overall Brier score was `0.291`, so the slice
> exposed a problem that the aggregate alone did not make obvious.

**4c. Apply it.** Describe a real workflow where an LLM pulls structured results from messy input.
Which pattern — validated retry with escalation, independent review with deterministic routing, or
provenance-preserving conflict annotation — would you reach for first, and what would you instrument
to know when it broke?

> For a workflow that extracts structured information from messy operational documents, I would use
> validated retry with escalation first. My System 2 run showed why: structured output can still contain
> a mathematical inconsistency. I would instrument validation failures by type, retry count,
> escalation count, null-required-field count, discrepancy magnitude, and the final validation result
> so a broken extraction path is visible rather than silently shipped.
