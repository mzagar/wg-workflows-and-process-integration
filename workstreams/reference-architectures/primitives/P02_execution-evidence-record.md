# P02: Execution Evidence Record

**Purpose:** Preserve a reconstructable record of the decisions, evidence,
effects, and outcomes associated with one workflow execution.

**Builds on taxonomy:** Workflow Execution · Workflow State · Handoff

## Definition

An execution record that links a workflow run to the evidence needed to explain
what happened and why. The record may include candidate or proposal identity,
authorization or gate evidence, external-effect evidence, attempt history,
escalation or handoff information, and terminal outcome.

The record is distinct from runtime traces and metrics. It is the workflow-level
evidence required to reconstruct the execution and its decisions.

## Role in a composition

An execution evidence record supports accountability and recovery analysis. It
does not replace the durable state checkpoint used to resume the workflow,
although an implementation may store both through the same underlying mechanism.

## Used by

- [Proposal/execution split](../patterns/proposal-execution-split.md)
- [Human approval gate](../patterns/human-approval-gate.md)
- [Bounded convergence loop](../patterns/bounded-convergence-loop.md)
- [Deterministic acceptance gate](../patterns/deterministic-acceptance-gate.md)
- [Single-agent process with human approval](../architectures/single-agent-human-approval.md)
- [Bounded autonomous remediation](../architectures/bounded-autonomous-remediation.md)
