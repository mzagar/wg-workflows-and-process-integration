# Parallel Fan-out and Join

**Solves:** Running independent parts of a task at the same time and collecting them under a fixed rule, so a slow, failed, or missing part cannot hang the run or go unnoticed \
**Used in:** [Parallel research and merge](../architectures/parallel-research-and-merge.md) \
**Requires capabilities:** Execution engine · state/context store · audit log \
**Related patterns:** [Branch isolation](branch-isolation.md) · [Durable wait](durable-wait.md)

## Problem

A task has parts that do not need each other's results, so they can run at the same time. The workflow must decide how long to wait, what happens when a part fails, and how the result states what is missing. Without rules fixed in advance, a run can hang or quietly return less than requested.

## Why this is hard

Parallel work creates several competing needs.

- Waiting for everyone can hang on a stuck part; taking whatever arrived can silently omit important work.
- Parts finish at different times, and a restart must not lose or repeat saved work.
- Different result shapes make collection unreliable.
- A coordinator that adds parts during execution can multiply cost.
- Readers can mistake a partial result for a complete one.

## Solution

Fix the plan, run the branches, then collect their terminal outcomes at a join. A branch is one independent part; the join collects results and states, rather than interpreting the findings.

```text
     plan, fixed before the first branch starts
     A (required)   B (required)   C (optional)
            |              |              |
            v              v              v
            A              B              C      run at the same time,
            |              |              |      each with its own limits
            +--------------+--------------+
                           |
              JOIN: every branch done, failed, or skipped
                           |
          +----------------+------------------+
          |                                   |
   all succeeded                        C missing, A and B ok
   -> complete                          -> partial, and says so

   A or B failed -> failed, or explicit handoff to a person
```

## Invariants (must hold in any implementation)

These rules apply regardless of runtime or deployment.

- The branches, their work, and their required-or-optional status are fixed before the first branch starts. Branches cannot add branches or change the plan.
- Every branch has a time limit and cost limit; the whole run also has limits. Retries and recovery cannot reset them.
- The join waits for a terminal outcome from every planned branch: done, failed, or explicitly skipped. Reaching a limit counts as failure; retry handling is bounded by the saved plan.
- The joined output names every planned branch and its terminal state.
- Fewer successful branches than planned always means a partial result. A failed or skipped required branch cannot produce a partial success: the run fails or records an explicit handoff.
- Each completed result is saved durably before the branch is marked done. Recovery keeps these results and does not rerun finished branches.
- Each result has the shape required by the plan; any other shape counts as failure.
- Results arriving after the join are recorded and ignored.

## Failure modes when violated

Each shortcut breaks one of those rules in a visible way.

- **Plan changes:** new branches keep discovering work and increasing the bill.
- **Limits absent or reset:** a stuck source or repeated restart consumes time and money indefinitely.
- **Join runs early:** slow but important sources are excluded before their deadlines.
- **Branch states omitted:** readers assume every source was consulted when one was unavailable.
- **Partial status omitted:** an incomplete answer is treated as complete.
- **Required failure treated as partial success:** the result lacks an essential source but still proceeds.
- **Done recorded before durable result:** a crash loses work already claimed complete; rerunning it adds cost or duplicates findings.
- **Result shapes differ:** collection drops or misreads findings.
- **Late results included:** released or approved content changes without a new decision.

## Implementation approaches (illustrative, not requirements)

Implementations differ in **where parallel execution and the join rule are enforced**. Each row supplies part of the solution. Every implementation must meet all invariants above; the last column highlights gaps, not a complete checklist.

| Enforced at | Documented mechanism | What it provides | What the implementation must add |
|---|---|---|---|
| Orchestration service | [AWS Step Functions Parallel state](https://docs.aws.amazon.com/step-functions/latest/dg/state-parallel.html) | Runs defined branches as concurrently as possible and waits for all to finish. An unhandled branch error fails the Parallel state and stops its branch execution, but not necessarily work already running | Branch deadlines and cost limits, run limits, required-or-optional handling, shape checks, terminal states, and partial labeling. Configure retries inside branches and durable run state: [Standard workflows persist state; Express does not](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html). Activity workers need cooperative stopping; running Lambda functions cannot be stopped. Record and ignore late results |
| Messaging and integration layer | [Spring Integration scatter-gather](https://docs.spring.io/spring-integration/reference/scatter-gather.html) | Routes requests to recipients and aggregates correlated replies under a release rule. A configured finite gather timeout bounds waiting for the aggregate reply, not necessarily synchronous recipient execution; timeout does not itself produce a partial result | Asynchronous recipient execution, saved branch/reply state and restart-safe dispatch, branch/run limits, failure replies, fixed required-or-optional release rules, shape checks, and partial labeling. Retain completion records to reject and record late replies; durable reply storage alone is not recovery logic |
| Agent framework | [LangGraph Graph API](https://docs.langchain.com/oss/python/langgraph/graph-api) | Runs scheduled nodes in parallel within a graph step; reducers combine updates to shared state. Static edges support fixed fan-out; reducers alone are not a terminal-state join | A fixed plan and explicit join over every planned branch, required-or-optional failure handling, budgets, shape checks, and partial labeling. Configure a [persistent checkpointer and crash-safe writes](https://docs.langchain.com/oss/python/langgraph/checkpointers) for finished results; in-memory checkpoints do not survive restart. Per-attempt timeouts do not replace branch/run budgets |

## Known uses outside agents

[Scatter-Gather](https://www.enterpriseintegrationpatterns.com/patterns/messaging/BroadcastAggregate.html) sends a request to several recipients and combines their replies into one message. Its loan-broker example asks banks for quotes and selects the best offer. Step Functions Parallel states provide a similar fan-out and wait structure in a workflow. These are precedents for the mechanism, not proof of this pattern's failure, completeness, or recovery guarantees.

The agent-specific concern is that generated findings may contain unsupported or incomplete claims even when a branch succeeds. The join reports missing branches. In Parallel Research and Merge, the later merge also labels every claim with its branch; that label records claimed origin, not correctness.
