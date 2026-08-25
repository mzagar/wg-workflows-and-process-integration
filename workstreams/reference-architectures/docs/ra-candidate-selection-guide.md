# Reference Architecture Candidate Selection Guide

This guide helps contributors turn a candidate from the
[RA candidate backlog](ra-candidate-backlog.md), or a proposed grouping of
scenarios from the [Use Case Inventory](../../critical-use-cases/use-case-inventory.md),
into a validated job-oriented reference architecture.

For repository contribution rules, templates, required section order, and
self-review, follow [CONTRIBUTING.md](../CONTRIBUTING.md). If this guide and
the contribution guide differ, the contribution guide takes precedence.

## Contents

- [Related working map](#related-working-map)
- [Operating model](#operating-model)
- [1. Select and frame a candidate](#1-select-and-frame-a-candidate)
- [2. Choose validation scenarios](#2-choose-validation-scenarios)
- [3. Identify composition and genuine gaps](#3-identify-composition-and-genuine-gaps)
- [4. Draft and validate the RA](#4-draft-and-validate-the-ra)
- [5. Open, review, and re-evaluate](#5-open-review-and-re-evaluate)
- [Quick checklist](#quick-checklist)
- [Validate one RA against one use case](#validate-one-ra-against-one-use-case)

## Related working map

The [RA candidate backlog](ra-candidate-backlog.md) is a working map of current
use-case cohorts, counts, existing RA fit, and future RA candidates. It is not
a conformance classification or a fixed roadmap. Use it as a starting point,
or propose a new cohort from the Use Case Inventory when the map does not fit.

## Operating model

```text
Use-case inventory or real scenarios
        ↓
Select and frame a practitioner job
        ↓
Choose contrasting validation scenarios
        ↓
Compose existing patterns and identify genuine gaps
        ↓
Draft a focused RA
        ↓
Validate, review, merge
        ↓
Re-evaluate the affected scenarios and select the next candidate
```

The primary catalog entry point is a recognizable practitioner job, not agent
count, a topology, a vendor, or a framework.

```text
Good RA candidate:
  Human-approved operation
  Bounded autonomous remediation

Not an RA candidate by itself:
  Single-agent
  Multi-agent
  Fan-out
  Supervisor/worker
  A particular product or framework
```

Agent count and topology are implementation choices unless the job genuinely
requires a particular authority or coordination structure.

## 1. Select and frame a candidate

1. Start with a real use-case cohort or a set of related scenarios.
2. State the practitioner job in one sentence:

   ```text
   [Verb]-ing [a recognizable job] while preserving [the defining boundary or guarantee].
   ```

3. Identify the closest existing RA and explain why it does not already fit.
4. Record a provisional name, scope, explicit non-goals, and expected authority
   boundary.

Questions to answer:

```text
Is this a practitioner job rather than a technical structure?
Does it have a stable problem and guarantee set?
Does it add a boundary not covered by an existing RA?
Would a reader know when to choose it and when not to?
```

## 2. Choose validation scenarios

For a new job-oriented RA, prefer at least two contrasting, evidence-backed
scenarios when available.

- Prefer scenarios that differ by trigger, industry, or operating context but
  share the proposed job and authority boundary.
- Record an expected result before drafting: `strong fit`, `partial fit`, or
  `no fit`.
- If a useful scenario is outside the inventory, record it as an external
  candidate. Do not count it as inventory evidence until it becomes a reviewed
  inventory row.

The scenarios should test the proposed RA rather than merely illustrate one
industry example.

## 3. Identify composition and genuine gaps

1. Read existing patterns before proposing a new concept.
2. Map the candidate job to existing patterns, capability roles, boundaries,
   guarantees, and exit states.
3. Keep agent count and topology as implementation choices unless the job
   genuinely requires a particular authority or coordination structure.
4. When a possible new pattern appears, record it as an explicit open question
   or candidate composition concern inside the RA.
5. Do not create a separate pattern entry until scenario validation demonstrates
   the same independently reusable problem, invariant set, and failure modes
   outside that one job.

## 4. Draft and validate the RA

Draft from [the architecture template](../architectures/TEMPLATE.md). Retain
its core section order and make the following visible rather than implicit:

```text
- authority and protected-effect boundaries;
- deterministic versus model-driven work;
- recovery, retries, waits, and explicit terminal outcomes;
- how patterns compose for this job;
- what evidence each validation scenario provides.
```

Keep capability roles vendor-neutral. Reuse patterns by linking to them rather
than copying their full invariants and failure modes.

Walk each selected scenario through normal, failure, restart, timeout, and
escalation paths where relevant. Record each result in the RA's Validation
record as `strong fit`, `partial fit`, or `no fit`.

## 5. Open, review, and re-evaluate

Keep the PR focused on one RA and only the patterns directly required by it.
The PR description should state:

```text
- scope;
- practitioner job;
- patterns composed;
- validation scenarios and their results;
- open questions and deliberately deferred work.
```

After merge, re-evaluate the affected use-case cohort and nearby borderline
scenarios. Update the candidate backlog with `strong fit`, `partial fit`, and
`no fit` evidence before selecting the next RA.

When choosing the next candidate, consider more than cohort size:

| Criterion | Question |
|---|---|
| Cohort size | How many real use cases may fit? |
| Job coherence | Is this one recognizable practitioner job? |
| Distinct guarantees | Does it add boundaries not covered by existing RAs? |
| Validation readiness | Are contrasting scenarios available? |
| Reuse | Can it compose current patterns without speculative new ones? |

## Validate one RA against one use case

To validate an existing or proposed RA against one use case and produce a
small GitHub Issue or PR, follow the
[RA use-case validation guide](ra-use-case-validation-guide.md).

## Quick checklist

Before opening an RA draft PR, confirm:

- [ ] The candidate is a practitioner job, not agent count or a topology.
- [ ] The job has a one-sentence description and explicit non-goals.
- [ ] The closest existing RA is identified and does not already cover it.
- [ ] Contrasting validation scenarios are selected when available.
- [ ] Existing patterns are reused where they fit.
- [ ] Any possible new pattern is an open question or candidate composition
      concern until independently reusable.
- [ ] Authority, effects, recovery, and terminal outcomes are explicit.
- [ ] Capability roles are vendor-neutral.
- [ ] Cross-WG concerns are marked out of scope rather than redefined.
- [ ] The affected cohort will be re-evaluated after merge.
