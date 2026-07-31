# Daily Activity Report — Execution

Status: Worked example

*This is a **detailed view** of the [solution](../solution.md). It walks the run stage by
stage, naming the work performed and the accepted transition plus the authority that permits
it.*

## Stage walkthrough

Control-flow authority stays with the workflow runtime throughout; the model only proposes.

| Stage | Work performed | Accepted transition and authority |
|---|---|---|
| 1. Scheduled trigger | `TRG` fires on the working-day-morning timer. | Enter the run; workflow runtime opens the lifecycle ([INV-004][architecture-invariants]). |
| 2. Fan out to sources | `PAR` fans out to the configured sources with a join (all, with a deadline) — `FS5`. | Start parallel reads; workflow runtime (control-flow). |
| 3. Read each source | `TRO` reads issue-tracker, chat, meeting/calendar, and mail activity over the time window, read-only. | Collect raw activity; source adapters (least privilege). A source that fails or times out is noted, not fatal → contributes to `COMPLETED_WITH_LIMITATION`. |
| 4. Dedupe, window, filter | `BRL` deduplicates, applies the time window, and runs the drop-list / keep-list filter to remove noise. | Produce a clean activity set; deterministic baseline (`WP00`). |
| 5. Assemble context | `CAS` packages a bounded, classification-tagged context for the model — the untrusted text is contained here before it reaches the model (`XB-01`, [INV-018][architecture-invariants]). | Build the model input; workflow runtime (state authority). |
| 6. Abstain check | `UNC` decides whether there is material activity to report. | If none → terminal `ABSTAINED` (a short note, not a padded draft); else continue. Workflow runtime; abstention is available to the semantic step ([CR-019](../../../ra/02-architecture-model/composition-rules.md)). |
| 7. Draft the narrative | `GEN` proposes the "Yesterday / Today" narrative from the assembled context. | Produce a draft (untrusted proposal); generation model (proposal only, `WP01`). |
| 8. Validate the draft | `DVL` checks the draft against its contract — required sections, length bound, and that no harvested text is treated as an instruction — and `SCH` confirms schema conformance. | Accept a valid draft, else terminal `FAILED_TECHNICAL`; deterministic validation ([INV-009][architecture-invariants]). |
| 9. Authorise the post | `POL` permits posting **only** to the owner's private channel. | Authorise one destination; policy and authorisation. |
| 10. Post the draft | Following `OV-02` propose→validate→authorise→execute→confirm→reconcile, the chat adapter posts as `TRW` at `EF2` with an `IDM` key; a postcondition confirms exactly one message landed. | Draft posted and confirmed; chat adapter under authorisation; reconcile not blind-retry ([INV-011][architecture-invariants]). |
| 11. Record and finish | `AUD` records the sources read and the draft posted; `FSK` closes the run. | Reach a terminal outcome; workflow runtime (state authority). |

The run ends in one of `COMPLETED`, `COMPLETED_WITH_LIMITATION`, `ABSTAINED`, or
`FAILED_TECHNICAL` ([INV-005][architecture-invariants]). No path silently drops: an
unpostable draft surfaces as `FAILED_TECHNICAL` for the next run to make visible.

## Least-agentic path

Every stage except stage 7 is deterministic. The model is confined to a single `GEN` call
over context the workflow has already assembled, and its output cannot become the posted
message until `DVL` and `SCH` accept it. The following deterministic controls carry the run:

- parallel read of each source over a fixed time window (`TRO`, `PAR`);
- dedupe, windowing, and drop-list / keep-list filtering (`BRL`);
- bounded, classification-tagged context assembly (`CAS`);
- the abstention check when there is nothing material (`UNC`);
- draft validation and schema conformance (`DVL`, `SCH`);
- single-destination posting policy (`POL`);
- idempotent, confirmed posting (`IDM`, `TRW`);
- the run audit record (`AUD`).

The model closes only the narrative gap, never the whole task
([DP-01](../../../ra/01-foundations/design-principles.md)). This is the intended default —
adding an in-run approval gate or a durable wait would be ceremony with no risk to justify
it (avoids `AP-01` agent-everywhere).

## Related views

- [architecture.md](architecture.md) — logical architecture and actors.
- [sequence.md](sequence.md) — interaction sequence.
- [contracts-and-state.md](contracts-and-state.md) — contracts and state checkpoints.

<!-- link definitions -->
[architecture-invariants]: ../../../ra/01-foundations/architecture-invariants.md