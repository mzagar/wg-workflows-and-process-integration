# P01: Durable State Checkpoint

**Purpose:** Preserve sufficient workflow execution state to resume safely
following interruption, restart, or a durable wait.

**Builds on taxonomy:** Workflow Execution · Workflow State · Durability

## Definition

A durable record of workflow execution state from which the workflow can resume
without silently losing progress, repeating completed work, or resetting
applicable limits.

The checkpoint records the state needed by the composition that uses it. This
may include execution position, relevant context, identity, pending wait,
attempt count, budget consumption, candidate identity, or the last gate result.

## Role in a composition

A durable state checkpoint supports recovery. It does not define the recovery
policy itself: the pattern or reference architecture using it decides whether a
resumed execution retries, reconciles an uncertain effect, remains parked,
escalates, or terminates.

## Used by

- [Durable wait](../patterns/durable-wait.md)
- [Human approval gate](../patterns/human-approval-gate.md)
- [Bounded convergence loop](../patterns/bounded-convergence-loop.md)
- [Single-agent process with human approval](../architectures/single-agent-human-approval.md)
- [Bounded autonomous remediation](../architectures/bounded-autonomous-remediation.md)
