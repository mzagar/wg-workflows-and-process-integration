# Daily Activity Report — Rationale

Status: Worked example

*This is the [Workflow Design Method](../../ra/08-lifecycle/workflow-design-method.md)
applied step by step — the reasoning from the [use case](use-case.md) to the
[WDS](daily-activity-report.wera.yaml).*

## Step 1 — Understand the business outcome

The outcome is a drafted **"Yesterday / Today"** status report, delivered to the owner's
own private channel for a post-hoc human check. The owner then edits and forwards it; the
workflow never sends it onward.

- **Run boundary** ([INV-004](../../ra/01-foundations/architecture-invariants.md),
  [INV-005](../../ra/01-foundations/architecture-invariants.md)): the run **starts** on a
  scheduled working-day-morning timer and **ends** in exactly one of four outcomes —
  `COMPLETED`, `COMPLETED_WITH_LIMITATION` (a source was down), `ABSTAINED` (no material
  activity), `FAILED_TECHNICAL` (could not post).
- **System of record** ([INV-002](../../ra/01-foundations/architecture-invariants.md)): the
  **source systems** stay authoritative for the owner's activity. The workflow keeps only a
  run audit record (which sources were read, which draft was posted) — it is deliberately
  **not** a shadow record of the work itself (avoids [AP-07](../../ra/02-architecture-model/composition-rules.md)
  context-as-record).
- **Decision owner**: the owner, **out of band**. There is no in-run decision gate; the
  model proposes the narrative and the workflow admits it after validation.

## Step 2 — Identify the deterministic baseline ([WP00](../../ra/03-patterns/wp00-deterministic-baseline.md))

Most of the work needs **no model** ([DP-02](../../ra/01-foundations/design-principles.md)):

- read each source over a fixed time window (read-only calls);
- deduplicate, normalise, and filter noise with an explicit **drop-list / keep-list**;
- format the message and post it to the channel;
- record what was read and posted.

This deterministic floor resolves everything except the summary itself.

## Step 3 — Identify the semantic gap ([DP-03](../../ra/01-foundations/design-principles.md))

The single residual task that genuinely needs a model is: **turn a heterogeneous list of
activity items into a short, readable narrative with the right emphasis.** Grouping and
prioritising disparate activity into prose is open content generation — a flat template
produces an unreadable dump. This is the *smallest* semantic task, and it is the *only* one.

Wizard answers ([wizard-questions.md](../../ra/05-selection/wizard-questions.md)) that set
the coordinate:

- **Q01/Q02** — nondeterminism is partial; the smallest semantic task is *content
  generation* → `nondeterminism: ND1`.
- **Q03** — the **workflow** owns control flow (fixed schedule-driven sequence) →
  `control_flow_authority: workflow`.
- **Q04** — one bounded model step → `actor_topology: single_agent`.
- **Q05/Q06** — no branch or tool is model-selected; the workflow decides which sources to
  read.
- **Q07/Q08** — output is *partially* verifiable (structure, not accuracy) → `AS1`;
  abstention is acceptable → `UNC` on the semantic step.
- **Q09** — human review is **post-hoc and out of band**, not an in-run gate → no `AS4`.
- **Q10/Q11/Q12** — parallel fan-out across sources then join → `FS5`; the run completes in
  one lifetime with no wait → `DUR1`; no replayable history required.
- **Q14/Q15** — the highest effect is posting to the private channel, a **reversible write**
  → `EF2`; the post is made idempotent → `IDM`.
- **Q16/Q17** — the harvested content is **untrusted** → input containment before the model
  and boundary `XB-01`; identity/authz to read sources and post is `XB-02`; trace signals
  `XB-03`; summary faithfulness is `XB-05`.
- **Q18/Q19** — a wrong draft is trivially corrected before forwarding → `IM1`, readiness
  floor `RT1`.

## Step 4 — Choose the execution profile

Workflow owns the run; a model helps in one bounded spot → **`EP2`** (workflow-directed +
model-assisted). Not `EP4` (agent-directed): the model owns no control flow, selects no
tool, and performs no write ([DP-01](../../ra/01-foundations/design-principles.md); avoids
[AP-01](../../ra/02-architecture-model/composition-rules.md) agent-everywhere and
[AP-02](../../ra/02-architecture-model/composition-rules.md) silent whole-process agent).

**Five authorities** ([INV-006](../../ra/01-foundations/architecture-invariants.md)):

| Authority | Holder |
|---|---|
| Control-flow | Workflow runtime, schedule-driven fixed sequence |
| Decision | Model proposes narrative; workflow admits after validation |
| Action-authorisation | Workflow policy — posting only to the owner's private channel |
| Execution | Deterministic chat adapter, single-channel constrained credentials |
| State | Source systems authoritative; workflow keeps a run audit record only |

## Step 5 — Apply patterns and overlays

- **Primary pattern**: one bounded generation step under a fixed contract, validated before
  use → **[WP01](../../ra/03-patterns/wp01-bounded-model-step.md)** (Bounded Model Step),
  sitting on the **WP00** deterministic baseline.
- **Effect protection**: posting is `EF2`, which triggers
  **[OV-02](../../ra/06-overlays/workflow-overlays.md)** (protected effect). The post carries
  an idempotency key (`IDM`) so a re-run never double-posts, and a postcondition confirms it
  landed ([INV-011](../../ra/01-foundations/architecture-invariants.md)). The model never
  posts directly — a deterministic adapter does, under workflow policy `POL`
  ([INV-009](../../ra/01-foundations/architecture-invariants.md),
  [INV-010](../../ra/01-foundations/architecture-invariants.md)).
- **Untrusted input**: harvested text is contained behind validation before it reaches the
  model, and again the model's output is validated before it becomes the posted message
  ([INV-018](../../ra/01-foundations/architecture-invariants.md), `XB-01`).
- **No durable envelope**: the run is instant with no wait, so no
  [WP08](../../ra/03-patterns/wp08-durable-workflow-envelope.md) / `OV-04`. A failed run
  simply re-runs next schedule.
- **No approval gate**: review is post-hoc, so no
  [WP07](../../ra/03-patterns/wp07-human-supervised-action.md) / `OV-01`. The private channel
  *is* the review surface, but that review happens **after** the run boundary.

## Step 6 — Produce the WDS

The assembled descriptor is [daily-activity-report.wera.yaml](daily-activity-report.wera.yaml).
It sets `readiness_tier: RT1` and `conformance_target: CL2`, validates against the
[schema](../../descriptor/workflow-descriptor.schema.json), and every code resolves in
[registry.yaml](../../descriptor/registry.yaml).

## Why the least-agentic composition ([DP-01](../../ra/01-foundations/design-principles.md))

This is the leaner sibling of [invoice-processing](../invoice-processing/README.md): **same
effect level** (`EF2`, a reversible write) but far less ceremony. Invoice-processing needed
human-approval-before-effect (`AS4` → `WP07` + `OV-01`) and a resumable wait (`DUR3` →
`WP08`). This workflow needs **neither** — the review is post-hoc and the run never waits.
Adding either would be ceremony with no risk to justify it. That subtraction *is* the design.
