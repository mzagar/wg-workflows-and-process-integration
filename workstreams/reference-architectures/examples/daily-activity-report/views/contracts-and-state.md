# Daily Activity Report — Contracts and State

Status: Worked example

*This is a **detailed view** of the [solution](../solution.md). It defines the key contracts
exchanged between components and the authoritative state the workflow records.*

## Key contracts

- **Canonical activity item** — the normalised representation of one piece of activity after
  a source is read and filtered. Produced from a `TRO` read once `BRL` dedupes and windows
  it; each item carries provenance back to its source system. The raw source text remains
  **untrusted** (`XB-01`); the canonical item is the trusted internal form.
- **Bounded generation context** — the input handed to the model (`CAS`). It contains the
  filtered, classification-tagged activity set for the window and a fixed instruction to
  produce a "Yesterday / Today" narrative. It is bounded in size and scope; the untrusted
  activity text is contained here and is data to summarise, never instructions to follow
  ([INV-018][architecture-invariants], [DP-03](../../../ra/01-foundations/design-principles.md)).
- **Draft narrative** — the model's output (`GEN`): a short report proposal, optionally a
  `UNC` abstention when there is nothing material. It is a **proposal only** and holds no
  authority until `DVL` and `SCH` accept it against the draft contract (required sections,
  length bound, no executed instructions).
- **Post intent** — the exact message payload assembled from the accepted draft to post one
  copy. It carries an `IDM` idempotency key (`run-id + report-date + channel-id`) and is
  executed as a `TRW` write at `EF2` under single-channel constrained credentials. It targets
  the owner's private channel and nothing else.
- **Run audit record** — the record of which sources were read (and which were unavailable)
  and which draft was posted, with the terminal outcome. It is the workflow's own history,
  **not** a record of the owner's work — the source systems remain authoritative for that
  ([INV-002][architecture-invariants]).

## Authoritative state checkpoints

The workflow state store records the run audit history (`AUD`); it links to (does not
duplicate) the source systems' activity ([INV-002][architecture-invariants]) and is not a
shadow record of the owner's work (avoids `AP-07`).

| Checkpoint | State recorded |
|---|---|
| Run opened | `TRG` context; the report date and time window. |
| Sources read | Which sources responded, which were unavailable; the raw activity counts per source (`TRO`). |
| Baseline resolved | The deduped, windowed, filtered activity set after `BRL` and `CAS`. |
| Abstain decision | Whether material activity was found (`UNC`); if not, the `ABSTAINED` terminal outcome. |
| Draft validated | The accepted draft after `GEN` + `DVL` + `SCH`, or the `FAILED_TECHNICAL` outcome on rejection. |
| Post confirmed | The posted message identity; `IDM` key; confirm/reconcile status of the `TRW` effect. |
| Run closed | Sources-read summary and the terminal outcome (`AUD`, `FSK`). |

There is **no durable-wait checkpoint**: the run completes in one lifetime (`DUR1`). A run
that fails mid-way is not resumed — it simply re-runs on the next schedule, which is safe
because the post is idempotent per report-date.

## Idempotency rule

The post is keyed on `run-id + report-date + channel-id`. A retried or re-scheduled run for
the **same report-date** must not create a second message: the idempotency key ensures at
most one report per day lands in the channel ([INV-011][architecture-invariants],
[CR-005](../../../ra/02-architecture-model/composition-rules.md)). If the post result is
ambiguous (the adapter is unsure whether the message landed), the workflow **reconciles**
against the channel rather than blind-retrying (avoids `AP-05`). This is the one rule that
turns "runs must never silently drop **or** double-post" from the use case into a concrete
control.

## Related views

- [architecture.md](architecture.md) — logical architecture and actors.
- [execution.md](execution.md) — stage-by-stage walkthrough.
- [sequence.md](sequence.md) — interaction sequence.

<!-- link definitions -->
[architecture-invariants]: ../../../ra/01-foundations/architecture-invariants.md