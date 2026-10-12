# Parallel Research and Merge

**The job it serves:** Answering one request by researching independent sources at the same time, then merging the findings into one result with a record of where each claim came from and what is missing \
**Built from patterns:** [Parallel fan-out and join](../patterns/parallel-fan-out-and-join.md) · [Branch isolation](../patterns/branch-isolation.md) · [Durable wait](../patterns/durable-wait.md) · [Human approval gate](../patterns/human-approval-gate.md) · [Proposal/execution split](../patterns/proposal-execution-split.md) \
**Requires capabilities:** Execution engine · two or more agents · tool access layer · merge step · state/context store · audit log · human interaction surface (when a person approves) \
**Audience:** Engineers and architects building parallel research workflows · reviewers checking the design · people reading or approving the results

## Status

Draft for working group review. The 13 decisions in the appendix are fixed for this draft; they are not yet approved WG guidance.

## Purpose

This architecture answers a request that needs findings from independent sources: split the research, run the parts at the same time, and merge them into one result. Every claim names the branch it came from, and the result says when a branch is missing. It adds a parallel research job to the existing approval and remediation architectures, exercising parallel execution, failure recovery, tool access, and optional human review (charter 3A–3D and 3F).

Sources may be separate, speed may matter, or one reader may not hold all the material. Model capacity changes, so the checklist tests independence, not job size or agent count.

**Reading paths.** For the idea and reasons, read this section and the worked scenario. For fit, read *Is this your architecture?* For implementation guidance, read *Structure* through *Composition considerations*. The appendix records the fixed decisions and their reasons.

### The idea in plain words

Picture a research manager who needs a market report. She writes a brief for each of three analysts, each reading a different source. They work at the same time and do not compare notes. When each has returned or reached a deadline, she combines their findings and marks who found what. A rule set before they started decides whether a missing analyst means a partial report or a stop.

Here, agents are the analysts and the execution engine is the manager.

### Words used here

These explanations use the [workflow taxonomy](../../taxonomy/terms.md); branch, brief, join, and merge step are local terms, not proposed taxonomy additions.

| Word | Meaning |
|---|---|
| Business task | The request or problem the workflow handles, as in the existing architectures; not a separate taxonomy entry |
| Activity | A bounded piece of work with defined inputs and outputs, tracked to completion |
| Execution engine | The part that runs the workflow's activities and remembers its state |
| Control flow | The rules for ordering, branching, and waiting for activities |
| Branch | One parallel piece of work, done by one agent in its own lane |
| Brief | The instructions and context the engine gives a branch; the branch then reads its allowed source |
| Plan | The taxonomy's structured guide to execution, restricted here to a branch list fixed before work starts, including each branch's source and required-or-optional status |
| Join | The activity that collects every branch's terminal state and available findings after deadlines and retries |
| Merge step | The activity that combines the collected findings into one result |
| Handoff | An explicit transfer of responsibility, context, state, or authority to another participant or workflow |
| Protected effect | An action that changes the real world, such as a payment, a message, or a change in a system |

## Is this your architecture? (checklist)

Bring a use case and follow these questions before designing the workflow.

1. **Does the research break into parts that do not need each other's results?** Yes: continue. No: use a sequence of activities, not this parallel composition.
2. **Do the parts examine different material, such as different sources, systems, or document sets?** Yes: continue. No: use one research activity; reading the same material in the same way does not require this composition.
3. **Is it known which sources are essential and which may be missing from a useful answer?** Yes: continue; the plan will mark their branches required or optional. No: settle that requirement before choosing this architecture. If every source is essential, all branches are required.
4. **Will the findings lead to a protected effect?** Yes, with human approval required: this architecture supplies the merged proposal to [Single-Agent Process with Human Approval](single-agent-human-approval.md); that workflow owns authorization and execution. Yes, without human approval: the research may fit, but this architecture does not cover authorization or execution; use a separate effect-controlling composition. No: this architecture fits; continue, with optional human review of the result.

The checklist also helps assess an existing workflow. Check its invariants, exit states, and boundaries below; record any gaps.

## Variants

No variants are currently defined. These nearby jobs are not variants:

- **Dependent steps** need an ordinary sequence, not parallel research.
- **Independent checking of findings** is a different job, not defined here.
- **Fleet-wide remediation** has branches that change targets. It needs its own effect-controlling composition; see [Bounded autonomous remediation](bounded-autonomous-remediation.md).
- **Agents choosing at runtime who runs next** do not follow the fixed plan required here.

## Structure

The diagram shows capability roles and their relationships, not the reasoning inside an agent.

```text
                         business task
                              |
                    +---------v----------+
                    |  Execution engine  |  builds the plan and writes
                    +---+-----+-----+----+  one brief per branch
                        |     |     |
                  brief |     |     | brief
                        v     v     v
                  +---------+ +---------+ +---------+
                  | Agent A | | Agent B | | Agent C |  each branch:
                  | source 1| | source 2| | source 3|  own lane, read-only,
                  +----+----+ +----+----+ +----+----+  time and cost limit
                       |           |           |
                       +-----------+-----------+
                                   | findings in the agreed shape
                                   | (or a recorded failure)
                    +--------------v--------------+
                    |  Execution engine: JOIN     |  every branch terminal;
                    +--------------+--------------+  apply the plan's rule
                                   | findings + state of every branch
                            +------v------+
                            | Merge step  |  one result, every claim
                            +------+------+  labeled with its branch
                                   | merged result (complete or partial)
                    +--------------v--------------+
                    |  Execution engine: CHECK    |  unlabeled claims removed
                    +--------------+--------------+
                                   | checked result
                                   v
                      released, or sent to a person
                      (optional human gate; any protected effect
                       belongs to another architecture)

        Workflow state is saved so a run can resume.
        Audit records share one run id, with an id for each branch.
```

| Capability | Responsibility in this architecture | Charter basis |
|---|---|---|
| Execution engine / orchestrator | Builds the plan, starts branches, enforces limits, runs the join and label check, and resumes after restart | 3A, 3B |
| Branch agents | Each researches one source and returns findings in the agreed shape; internals unspecified | 3A, 3C |
| Tool / capability access layer | Gives each branch read-only access to its own source and private draft storage, not protected-effect tools | 3C |
| Merge step | Combines findings into one labeled result; rules or an agent, depending on whether judgment is needed | 3A |
| State / context store | Keeps the plan, branch states and findings, retries, and consumed budgets across restart | 3B |
| Audit log / system of record | Records the plan, branch outcomes, merge, check, and decisions under one run id | 3B |
| Human interaction surface | When a person approves, shows the exact result, labels, and completeness status and captures the attributed decision | 3D |

The codes refer to [charter scope 3A–3F](../../../charter/charter.md#3-scope). These are logical roles, not separate required services. A platform may combine them while enforcing their boundaries.

## Pattern composition

These patterns supply the controls used at each boundary.

| Pattern | Where it sits | Why it is required here |
|---|---|---|
| [Parallel fan-out and join](../patterns/parallel-fan-out-and-join.md) | From the plan to the merge | A fixed plan and bounded join prevent a slow branch hanging the run or a missing branch going unnoticed |
| [Branch isolation](../patterns/branch-isolation.md) | Around each branch | Keeps research independent and unable to cause protected effects |
| [Durable wait](../patterns/durable-wait.md) | While waiting for branches and at any human gate | Saved wait position, context, identity, and deadlines survive restart |
| [Human approval gate](../patterns/human-approval-gate.md), when a person approves | After the merge and label check | The identified, authorized approver decides on the exact immutable result, including its completeness status |
| [Proposal/execution split](../patterns/proposal-execution-split.md), when an action follows | At the handoff to the effect-controlling workflow | The merged result is proposal data; only that workflow's executor acts after authorization |

**Invalid compositions.** These shortcuts remove a required guarantee:

- A branch has a tool or credential that can cause a protected effect.
- Branches can read each other's work or message each other.
- The join has no deadline or silently skips a missing branch.
- The plan changes during the run, for example when a branch starts more branches.

## The agent and workflow-control boundary

Branch agents may read their allowed source, reason about it, write private drafts, and return findings in the agreed shape or report failure. A merge agent may combine supplied findings, show disagreements, and label every claim. It receives no protected-effect capability.

Workflow control owns the plan, branch count, briefs, limits, scheduling, stopping, retries, join rule, label check, saved state, human gate, handoff, and audit records. No agent can cause a protected effect. If one follows the result, a separate effect-controlling workflow owns authorization and execution.

## Choosing deterministic and model-driven activities

Use deterministic activities for the guarantees: building the plan from a task template, enforcing limits and retries, joining terminal branch states, checking completeness and labels, marking partial results, and recording outcomes. Here, deterministic means consistent behavior for the same inputs, state, and history, as in the taxonomy. Predictability, testability, and enforceability matter, not cost alone.

Use model reasoning for open-ended research and a merge that needs judgment. Use rules when the merge only collects, sorts, or ranks by stated criteria. A model does not choose the split, branch count, or next activity.

## Exit states

Every run ends in a recorded state; pending research or approval is not an exit.

| Exit state | Reached when | Recorded as |
|---|---|---|
| Completed | Every planned branch succeeded and the checked result was released, after approval if required | Plan, branch results, merge, label check, decision if any, release |
| Partial | Every required branch succeeded, at least one optional branch failed or was skipped, and the checked result was released with that gap stated, after approval if required | The same records, plus missing branches and reasons |
| Failed | A required branch failed after one retry, required work exhausted its budget, or planning, merge, check, storage, or release could not complete within the run's limits | Failure class, every branch's state, consumed budget, last saved state |
| Escalated | Responsibility was handed to a person or another workflow instead of releasing a result | Handoff target, reason, and state at handoff |
| Rejected | A person explicitly declined the result at the gate | Decision, authorized approver identity, time, exact version reviewed |
| Expired | No decision arrived by the gate deadline and no escalation continued the run | Timeout and escalation trail |
| Cancelled | An authorized requester or operator stopped the run before release | Who, why, and each branch's state |

## Worked scenario: a market landscape report

This example sharpens the inventory's [Market landscape report](../../critical-use-cases/use-case-inventory.md#research--intel). Its stated risk is outdated competitive data feeding a strategic decision. The inventory also includes critique before writing; that extra stage is not covered here. The branch plan, failure, retry, and recovery below illustrate this architecture, not a verified deployment.

1. **Admit and plan** *(execution engine, state store).* A strategy team requests a market report. A task template produces three briefs: regulatory filings (required), news and web (required), and company data (optional). The engine saves the plan and limits before starting work.
2. **Start branches** *(engine, agents, tool access layer).* Three agents start together with separate briefs, private workspaces, source-scoped read tools, and time and cost limits.
3. **Research** *(agents, tool access layer).* Each returns findings in the agreed shape, including sources and retrieval dates. The engine saves each completed result.
4. **Join** *(engine, state store).* Branches reach a successful or failed terminal state within their limits. Company data fails, is retried once within the saved budgets, and fails again. It is optional, so the engine permits a partial result.
5. **Merge** *(merge agent).* The agent combines the saved findings. Every claim names its branch. Disagreements between filings and news appear side by side.
6. **Label check** *(engine).* A rule removes unlabeled claims and records the removals. The result says: "Partial: company data missing." This check does not verify the truth of labels or claims.
7. **Human approval** *(human interaction surface, engine).* An identified, authorized analyst sees and approves or rejects that exact immutable version. The decision is recorded before the run resumes; silence never means approval.
8. **Release and record** *(engine, audit log).* The approved partial report is released once per run. The record joins the plan, branch outcomes, merge, check, decision, and release. Any later protected effect belongs to another workflow.

### The run over time: where durability lives

Saved progress survives process death; finished branches are not repeated.

```text
 process lifetime 1           process lifetime 2             process lifetime 3
|-----------------------|    |----------------------------|  |--------------------|
 plan -> start branches       reload run; keep saved         decision arrives;
 filings done (saved)         filings; recover unfinished    reload run; record
 news still running           branches within saved budgets  decision; release;
                              -> join -> merge -> check      record partial exit
   [process dies]             -> PARKED at the gate
                                    (hours or days pass)
```

### If the process dies during activity N

Recovery uses recorded state and never resets the plan, retry count, or budget.

| If the process dies during... | What happens on restart | What makes it safe |
|---|---|---|
| 1. Admit and plan | Redeliver the request and reload or rebuild the plan | Stable request identity; save the plan before dispatch |
| 2. Start branches | Reconcile dispatch state before starting unfinished branches | Stable branch ids and deduplicated dispatch; no second active copy |
| 3. Research | Keep durably saved results; recover unfinished work from its brief | No protected effects; duplicate reads can still cost money or see newer data, so attempts and budgets remain recorded |
| 4. Join | Reload terminal states and reapply the plan's rule | Saved rule, deadlines, retry counts, and outcomes |
| 5. Merge | Rerun from saved findings if no completed merge was saved | No protected effect; a different draft is acceptable before review |
| 6. Label check | Reapply the same rule to the saved draft | Repeatable check; record removals with the draft version |
| 7. Human approval | Reload the parked run and its exact review version | Saved wait, deadline, and decision; duplicate or late events cannot release twice |
| 8. Release and record | Reconcile release, then retry unfinished recording | Release and audit append are idempotent per run and activity |

## Composition considerations

Combining the patterns creates these concerns beyond any one pattern.

- **The join sees everything.** The join and merge receive all findings. Handle their output as sensitively as the most sensitive branch.
- **Labels are not proof.** A label records claimed origin; the check proves its presence, not that the branch supplied or supported the claim.
- **Partial approval binds to a version.** An approver sees the missing-branch banner and approves exactly that version. Edits require a new approval.
- **Branch limits are not a run limit.** Include research, retries, merge, and review handling in the overall budget.
- **Late findings cannot change an approved result.** Record and ignore them after the join.

## Out of scope (handled elsewhere)

This architecture leaves these jobs and controls to other compositions or workstreams.

- **Model-chosen splits or agents choosing who runs next.** The plan here is fixed.
- **Independent checking of merged findings.** A separate job, not defined here.
- **Coordination across teams or companies.** Not defined here.
- **Identity, credential issuance, and delegation.** Identity & Trust WG; source-scoped access is a requirement here, not an identity standard.
- **Hostile source content and data-exfiltration controls.** Security WG; isolation limits cross-branch exposure but does not make findings safe to trust at the merge.
- **Finding quality.** Accuracy & Reliability WG.
- **Traces, metrics, and telemetry.** Observability WG; the audit log here is a system of record.

## Validation record

This is a first-pass fit assessment against the [inventory](../../critical-use-cases/use-case-inventory.md), using its [12-column schema](../../critical-use-cases/schema-agentic-use-cases.md), not verified implementation conformance. `Validated` means architectural fit, as in the [validation guide](../docs/ra-use-case-validation-guide.md); the Use Cases workstream should confirm these assessments. `Partial fit` retains the draft's limited-coverage label, not the guide's narrower formal `Partial` result. No label has been upgraded.

| Scenario | Source | Result | Notes |
|---|---|---|---|
| Supplier bid evaluation | [Inventory, Supply chain](../../critical-use-cases/use-case-inventory.md#supply-chain) | Validated | The inventory describes independent bids scored concurrently and merged, with an approval gate. It calls this the natural design; one branch per bid and the pre-run branch count are not established |
| Architecture decision record (ADR) drafting | [Inventory, SDLC](../../critical-use-cases/use-case-inventory.md#sdlc) | Validated | Earlier-record retrieval and options generation run in parallel; a writer combines both into a draft for team review, with a content approval gate |
| Bug triage from logs | [Inventory, Engineering](../../critical-use-cases/use-case-inventory.md#engineering) | Candidate | Parallel log analysis and code search feed a synthesis agent. The workflow files a Jira ticket, but who writes it and whether writing is separated from research are unknown |
| Cross-brand product recommendation | [Inventory, Marketing](../../critical-use-cases/use-case-inventory.md#marketing) | Validated | Separate catalog agents feed synthesis and ranking, with fixed coordination and no mandatory human gate. Instant agent execution does not establish restart behavior |
| Market landscape report | [Inventory, Research & intel](../../critical-use-cases/use-case-inventory.md#research--intel) | Partial fit | Parallel research and synthesis with content approval fit. Critique before writing is an additional stage not covered here; its independence is not established |
| Case law research brief | [Inventory, Legal](../../critical-use-cases/use-case-inventory.md#legal) | Partial fit | Parallel searches feed relevance synthesis, then citation checking and sign-off. The additional citation-checking stage is not covered here; its independence is not established |
| Content repurposing pipeline | [Inventory, Marketing](../../critical-use-cases/use-case-inventory.md#marketing) | Partial fit | Parallel outputs from the same blog post meet at a review queue. Only the parallel pattern fits; this is content generation, not independent-source research and evidence merging |
| Closed-loop dependency & CVE remediation (fleet-wide) | [Inventory, SDLC](../../critical-use-cases/use-case-inventory.md#sdlc) | No fit | Repository branches write and test fixes and open PRs in convergence loops, conflicting with the read-only research boundary |
| Service deployment orchestration | [Inventory, Platform engineering](../../critical-use-cases/use-case-inventory.md#platform-engineering) | No fit | A concurrent monitor can abort deployment execution. Deployment and rollback are not research and merge; the inventory also records runtime AI routing |

## Open questions

None. The fixed choices and their reasons are recorded below.

## Appendix: Design decisions

### Decisions taken, and why

These are design choices for this draft, not claims of empirical optimality.

| Decision | Why |
|---|---|
| The execution engine runs a fixed plan. Agents never decide who runs next | Limits and records must be enforced outside the agents. Of 44 inventory rows classified as multi-agent, 38 have `Fixed/scripted` coordination, 2 have `AI-coordinated, routing`, and 4 leave it unspecified. These classifications do not prove use of this fixed-plan architecture |
| The plan is built by rules from a template for the type of task, before work starts. A model does not choose the number or kind of branches | Keeps runs predictable and auditable, and cost bounded |
| No branch can start more branches | Prevents runaway cost |
| Branches cannot see each other's work until the join | Keeps findings independent and limits cross-branch influence; branches can run at the same time |
| Branches are read-only | Research cannot cause a protected effect; the same boundary as the proposal/execution split |
| Each branch is required or optional, set in the plan | Failure handling is decided before the run, not improvised during it |
| One retry per failed branch, then the plan's rule applies | A bounded design default limits repeated work and cost; it is not evidence that most failures are temporary or that one retry is optimal |
| A result built from fewer branches than planned is always marked partial | A silent gap looks like a complete answer |
| Disagreements between branches are shown side by side, never resolved silently | A silent choice looks like a fact; the reader should decide |
| Every claim carries the label of its branch. Unlabeled claims are removed | Readers can trace claimed origin; the check is a simple rule, not proof of correctness |
| The join and the merge are separate steps | The join is mechanical; the merge may need judgment, so its output is a draft |
| The checklist tests whether the parts are independent, not how big the job is | Capacity changes with models and is not a stable routing test; independence applies regardless of the reason for splitting |
| A result that arrives after the join is ignored and recorded | Approved content remains identical to the recorded version |
