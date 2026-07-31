# Daily Activity Report

Status: Worked example

## Why this example

A daily status report drafted from the owner's own activity across several systems, delivered
to a private channel for a **post-hoc** human check. It combines untrusted free-form content,
a parallel multi-source read, one bounded generation step, and a single reversible external
write — but needs **no in-run approval gate and no durable wait**. It is the leaner sibling of
[invoice-processing](../invoice-processing/README.md): the same effect level (`EF2`) with far
less ceremony, a clean demonstration of
[DP-01 least-agentic composition](../../ra/01-foundations/design-principles.md) by
*subtraction*.

## Read it as a few-shot exemplar

```text
use-case.md   →   rationale.md   →   solution.md
  (Input)          (Reasoning)        (Architecture + Result)
```

- **[use-case.md](use-case.md)** — the business problem and requirements (Input).
- **[rationale.md](rationale.md)** — the six-step [Workflow Design Method](../../ra/08-lifecycle/workflow-design-method.md) applied (Reasoning).
- **[solution.md](solution.md)** — the resulting design, linked to the machine-readable [WDS](daily-activity-report.wera.yaml) (Architecture + Result).

## Detailed views

- [architecture.md](views/architecture.md) — logical architecture and actor responsibilities.
- [execution.md](views/execution.md) — stage-by-stage execution walkthrough.
- [sequence.md](views/sequence.md) — sequence diagram.
- [contracts-and-state.md](views/contracts-and-state.md) — key contracts and state checkpoints.

## At a glance

| Attribute | Value |
|---|---|
| Profile | `EP2` — workflow-directed + model-assisted |
| Primary pattern | `WP01` — bounded model step |
| Embedded patterns | `WP00` — deterministic baseline |
| Lifecycle envelope | *(none — no durable wait)* |
| Overlays | `OV-02` — protected effect |
| External boundaries | `XB-01`, `XB-02`, `XB-03`, `XB-05` |
| Highest effect | `EF2` — reversible write (post to a private channel) |
| Readiness / conformance | `RT1` / `CL2` |

The human review is **post-hoc and out of band**: the workflow's job ends when the draft
lands in the private channel; the owner edits and forwards it by hand.

## Contrast with invoice-processing

| | daily-activity-report | invoice-processing |
|---|---|---|
| Semantic task | generate a narrative (`ND1`) | select from a validated set (`ND3`) |
| Human role | review **after** the run (out of band) | approve **inside** the run (`WP07` / `OV-01`) |
| Durability | instant, no wait (`DUR1`) | resumable across a wait (`DUR3` / `WP08`) |
| Highest effect | `EF2` post to own channel | `EF2` unposted ERP draft |
| Readiness | `RT1` | `RT2` |

Same effect ceiling, very different ceremony — because the *impact* and the *review timing*
differ. That difference is the whole point of choosing a coordinate rather than a template.