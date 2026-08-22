# Daily Recruitment-Assessment Briefing — Day 1

**Date:** 2026-08-22 · **Country track:** Tanzania · **Framework:** IPSAS
**Level:** 1 (entry/mid blended, foundational) · **Version code:** `TZ-L1-2026-08-22-v1`
**Boss Level today:** No (Tanzania's first Boss Level lands at Level 6)
**Companion artifact:** `mock-exam.html` (single-file, offline, printable — see below)

---

## 1. Why this scenario

Level 1 opens the Tanzania track. The organization is a multi-country donor-funded
NGO; the scenario is a real EU-funded agricultural livelihoods project running out of
a Tanzania country office, paid in EUR by the donor, spent in TZS through a pooled
project bank account. It is deliberately narrow — one donor, one project, one
reporting period — because Level 1 must isolate foundational reasoning without
burying it under interdependency. Ambiguity and traps come from evidence quality, not
volume.

## 2. Concepts tested

**Entry-level concept — IPSAS 23 non-exchange revenue recognition.** Cash received
from a donor is not automatically income. Under IPSAS 23, a grant received subject to
stipulations (conditions the entity must satisfy or return the funds) is recognized as
a liability (deferred/unearned grant income) and released to revenue only as and when
the conditions are satisfied — typically as eligible expenditure is incurred against
the grant's budget lines. Candidates must distinguish cash received, income earned,
and liability remaining, and tie each to a source document.

**Mid-level concept (introduced as a stretch section, not required for a passing
entry-level score) — donor-specific FX method and available-to-commit.** The donor's
grant agreement specifies its own exchange-rate basis (a monthly operational rate) for
converting EUR budget and reported TZS expenditure, which will differ from the actual
bank conversion rate on the tranche receipt. This creates a realized/unrealized FX gap
between the ledger (actual bank rate) and the donor report (donor rate), and it moves
the EUR-equivalent "available-to-commit" balance independently of any new spending —
a classic source of an over-commitment nobody actually caused by overspending.

## 3. Mechanics embedded (see PROGRESSION.md for what carries to Day 2)

Deferred grant income · pooled bank & anti-commingling controls · analytical
dimensions (Donor/Cost Unit, Project, Cost Center, Budget Line, Funding
Agreement/Tranche) · staff imprest vs. expense · donor-specific FX method · FX effect
on available-to-commit · ex-post (line-item eligibility) budget control.

Deliberately **not** introduced yet: intercompany, consolidation, management fee/ICR,
fleet & mileage, MEAL/narrative triangulation, forecasting/EAC. These enter from
Tanzania Level 3 onward per the continuity log.

## 4. Scoring rubric (100 points, 9 dimensions)

| Dimension | Points | Minimum competency threshold |
|---|---|---|
| Accounting & framework knowledge (IPSAS 23) | 18 | 10/18 (56%) |
| Numerical accuracy | 14 | 7/14 |
| Reconciliation & systems thinking (bank ↔ ledger ↔ donor report) | 14 | 7/14 |
| Evidence discipline (cites Evidence IDs, shows workings) | 12 | 6/12 |
| Attention to detail (cut-off, dimensions, dates) | 10 | 4/10 |
| Prioritization & materiality | 10 | 4/10 |
| Cognitive flexibility (Section E revision) | 8 | 3/8 |
| Donor/tax/compliance judgment | 8 | 3/8 |
| Communication (Section F memo) | 6 | 2/6 |

**Pass gate:** total ≥ 60/100 **and** every dimension at or above its minimum **and**
zero critical-fail responses. A high total cannot buy back a critical fail or a missed
threshold — this stops polished, generic prose from masking a dangerous accounting
error.

## 5. Red flags to watch for while grading

- Candidate recognizes the full EUR 62,000 tranche as income on receipt (premature
  recognition — ignores IPSAS 23 stipulations).
- Candidate proposes clearing the unsupported TZS 4,100,000 prior-quarter advance to
  expense "to tidy the balance" without requesting supporting documentation —
  **critical fail** if stated as a recommended action rather than flagged as a risk
  needing evidence.
- Candidate treats the staff imprest (EV-06) as a recognized project expense rather
  than a receivable/advance pending retirement.
- Candidate declares the pooled-account transfer in EV-04 as commingling/fraud without
  first identifying what evidence would confirm or refute it (premature conclusion,
  scored as a judgment-signal miss even though the concern itself is reasonable).
- Candidate ignores the Section E evidence pack entirely, or revises Section C's
  conclusion with no citation to the new evidence (unjustified answer-changing).
- Candidate uses the bank-actual FX rate for the donor report line, or the donor
  operational rate for the ledger line (wrong-basis FX application).

## 6. Expected reasoning signals (what "good" looks like)

- Explicitly separates cash-received, income-recognized, and liability-remaining, each
  with an Evidence ID and a one-line calculation.
- Treats the EV-04 pooled-account transfer as an open question in Section C, names the
  specific missing evidence, and only concludes once Section E supplies it —
  demonstrating evidence discipline and inhibition of premature conclusions.
- Flags the unsupported advance as a control risk requiring documentation before any
  clearing entry, not as a balance to be zeroed for tidiness.
- Recomputes available-to-commit using the donor's stated FX method and notices the
  reduction is FX-driven, not a spending overrun — a reconciliation/systems-thinking
  signal linking donor report ↔ ledger ↔ cash flow.
- In Section F, communicates the FX-driven available-to-commit change to a
  non-finance program manager in plain language, without hiding the deferred-income
  mechanics behind jargon.

## 7. Remote-assessment design improvement introduced today

Today's exam adds a **locally-persisted, single-attempt evidence-reveal gate**: the
Section E "second evidence pack" is hidden behind a button the candidate can only
click after submitting a locked-in Section C conclusion (the textarea becomes
read-only via JS on reveal). This prevents a candidate from silently reading ahead,
answering with hindsight, and only pretending to revise — it forces the "before" and
"after" positions to exist as two distinct, timestamped, locally-stored artifacts that
an evaluator can compare side by side, which is far harder for a generic AI-assisted
answer to fake convincingly because it requires committing to an early, incomplete-
information judgment first.

## 8. Candidate-facing artifact

See `mock-exam.html` in this folder — single file, no network calls, works offline,
printable (candidate view and evaluator pack print separately), answers persist to
`localStorage` only and are never transmitted, with an on-page export/copy function.
