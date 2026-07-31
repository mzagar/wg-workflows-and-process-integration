# Daily Activity Report — Sequence

Status: Worked example

*This is a **detailed view** of the [solution](../solution.md). It shows the interaction
sequence across participants, including the abstain and post branches.*

## Interaction sequence

```mermaid
sequenceDiagram
    participant TM as Scheduler / timer
    participant WR as Workflow runtime
    participant SR as Source adapters
    participant GM as Generation model
    participant VP as Validation and policy
    participant CA as Chat adapter
    participant OW as Owner (out of band)

    TM->>WR: Working-day-morning trigger (TRG)
    WR->>SR: Read sources in parallel (PAR TRO)
    SR-->>WR: Raw activity (some sources may be unavailable)
    WR->>WR: Dedupe, window, filter (BRL)
    WR->>WR: Assemble bounded context (CAS XB-01)

    alt No material activity
        WR->>WR: Abstain (UNC) -> ABSTAINED note
    else Material activity
        WR->>GM: Draft Yesterday / Today (GEN)
        GM-->>WR: Draft narrative or abstain (UNC)
        WR->>VP: Validate draft and schema (DVL SCH)
        VP-->>WR: Valid draft or FAILED_TECHNICAL
        WR->>VP: Authorise private-channel post (POL)
        VP-->>WR: Permitted (owner's channel only)
        WR->>CA: Post draft with idempotency (TRW IDM)
        CA-->>WR: One message confirmed (or reconcile)
        WR->>WR: Record sources and draft (AUD FSK)
    end

    Note over WR,OW: Run boundary ends here.
    OW->>OW: Read draft in private channel, edit, forward to managers
```

## Important boundary

Two boundaries define this design.

**The model never posts.** The generation model returns its draft to the workflow runtime,
which validates it (`DVL`, `SCH`), authorises a single destination (`POL`), and only then
instructs the chat adapter to post it as a reversible `EF2` write with an idempotency key.
This preserves the `OV-02` propose→validate→authorise→execute→confirm→reconcile separation
and keeps all five authorities away from the model
([INV-009](../../../ra/01-foundations/architecture-invariants.md),
[INV-010](../../../ra/01-foundations/architecture-invariants.md)).

**The human review is outside the run.** The workflow's terminal outcome is *draft posted*.
The owner then reads, edits, and forwards it — but that happens **after** the run boundary,
in the private channel, with no workflow step waiting on it. That is exactly why this design
carries no in-run approval gate (`WP07` / `OV-01`) and no durable wait (`WP08` / `OV-04`):
the review is real, but it is out of band.

## Related views

- [architecture.md](architecture.md) — logical architecture and actors.
- [execution.md](execution.md) — stage-by-stage walkthrough.
- [contracts-and-state.md](contracts-and-state.md) — contracts and state checkpoints.