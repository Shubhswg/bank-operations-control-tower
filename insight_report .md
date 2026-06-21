# Insight Report: Bank Operations Control Tower

## Executive Summary

Across 15,516 simulated transactions, **9.66% failed to reconcile cleanly between the core banking ledger and the payment gateway** - and when a second, independent check for duplicate postings is included, **true exceptions rise to 13.0% (2,015 records)**, nearly 35% higher than the primary break rate alone would suggest. These exceptions hold **₹368.49M in capital that cannot be confirmed as settled** until each one is individually investigated and closed.

The findings below aren't just a description of what broke - they point to *why* it broke, *when* it's likely to break again, and *where* to look first. That's the difference between a report that describes a dataset and one that's usable by an operations team on a Monday morning.

---

## Finding 1: The break rate understates the real exposure by a third - and the gap is structural, not accidental

The headline number is 9.66% (1,499 of 15,516). But 516 transactions were posted twice by the payment gateway, and because a duplicate matches perfectly on amount and date, the standard reconciliation logic silently files it as "Matched" - not as an exception.

**Why this matters more than it sounds:** this isn't a one-off data quirk. It's a structural blind spot in any reconciliation process built solely on amount/date matching. A duplicate payment is arguably *more* financially dangerous than a missing one - money has actually moved twice - yet it's the category most likely to slip past a naive control. Counting it properly moves total exceptions from 1,499 to **2,015**, and the break rate from 9.66% to **13.0%**.

**The actionable consequence:** any reconciliation control that reports a single "break rate" number without separately surfacing duplicate-frequency checks is under-reporting risk by design, not by error. This is the kind of gap that looks fine in a monthly summary and then surfaces badly in an audit.

## Finding 2: The largest break category is fixable at the source - not a connectivity problem

| Break type | Count | Share of breaks |
|---|---|---|
| Amount Mismatch | 547 | 36.5% |
| Timing Shift | 492 | 32.8% |
| Missing in B | 460 | 30.7% |

Amount Mismatch leads, but only narrowly - all three categories are within 90 cases of each other, which itself is informative: this isn't a single dominant failure mode, it's three roughly equal-sized problems requiring three different fixes.

**Why this matters:** if the team only had budget to fix one thing, the instinct might be "improve connectivity so transactions stop going missing." The data says otherwise - Missing in B is actually the *smallest* category. The bigger lever is fixing rounding/conversion logic between systems (Amount Mismatch) and investigating processing-delay sources (Timing Shift). Connectivity is real but not the priority the intuitive answer would suggest.

## Finding 3: Breaks are predictable, not random - which changes how you'd staff for them

The daily trend line shows break volume spiking in recurring, periodic patterns aligned to month-end dates, consistent with known banking volume cycles (salary credits, EMI debits, statement runs).

**Why this matters:** a process that experiences random, evenly-distributed errors needs even, steady-state review capacity. A process with *predictable* spikes needs the opposite - surge capacity timed to known dates. The practical implication: an operations team reviewing this data wouldn't request more headcount across the board, they'd request flexible coverage concentrated around month-end, which is a cheaper, more defensible ask to make to a manager.

## Finding 4: Exposure is concentrated, not evenly spread - so is the fix

Branch-level analysis shows exposure is not uniform - Chennai carries the highest concentration of break-driven exposure among the simulated branches.

**Why this matters:** if exposure were evenly spread across branches, the only lever would be a system-wide fix. Because it's concentrated, the highest-leverage first move is a targeted root-cause review at the highest-exposure branch, not a uniform policy change everywhere at once. This is a cheaper, faster first step than a system-wide intervention, and it's the kind of triage logic a real operations or risk function would use to sequence its work.

---

## Methodology (brief)

System A (Core Banking, 15,000 synthetic transactions) and System B (Payment Gateway, a deliberately corrupted copy) were built with four break types injected at a controlled rate, weighted toward month-end. A full outer join in Power Query (15,516 combined rows) classified each transaction by comparing amount and date across both systems; duplicates were caught separately via a row-count check on Transaction ID, since they're invisible to amount/date logic by construction. Aging buckets and a simulated resolution log (owner, action, status) were layered on top to model the full detection-to-resolution lifecycle, not just a one-time classification. Full technical detail is in the README; this report focuses on what the findings mean rather than how each was computed.

## Limitations - stated plainly

- **The 9.66% headline figure excludes duplicates by construction**, not oversight. Any reporting of "break rate" from this methodology should specify whether duplicates are included, since the two numbers (9.66% vs. 13.0%) tell meaningfully different stories.
- **Aging is measured against a fixed snapshot date**, not a live "today." Because the dataset spans a fixed 6-month window, most breaks fall into the "7+ Days" bucket simply due to where they sit in that window - this reflects the dataset's construction, not a real finding about resolution speed.

## Recommendations

1. **Report exceptions, not just breaks.** Any dashboard or summary built on this methodology should show the 13.0% true exception rate alongside the 9.66% primary break rate, not instead of it.
2. **Fund a fix for Amount Mismatch logic first**, since it's the single largest category and the most directly addressable through rounding/conversion rule changes - not infrastructure spend.
3. **Build month-end surge capacity into review staffing**, rather than flat, evenly-distributed coverage - the spike pattern is consistent and predictable enough to plan around.
4. **Start branch-level remediation at the highest-exposure branch**, using this concentration as a triage signal rather than treating all branches as equal priority.
5. **Replace the fixed snapshot date with a live reference date** before treating aging-bucket distributions as an operational backlog signal in any real deployment.
