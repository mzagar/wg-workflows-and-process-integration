# Reference Architecture Candidate Backlog

> **Status:** Working map. Candidate cohorts are hypotheses for RA workstream
> discussion, not conformance classifications or approved Reference
> Architectures. Update this document as scenario validation strengthens,
> narrows, or disproves a candidate fit.

This backlog maps scenarios from the [Use Case Inventory](../../critical-use-cases/use-case-inventory.md)
to current and potential job-oriented reference architectures. Use it with the
[RA Candidate Selection Guide](ra-candidate-selection-guide.md): this document
shows possible starting points; the guide explains how to validate a candidate
before drafting an RA.

## Contents

- [Counting method](#counting-method)
- [High-level cohort map](#high-level-cohort-map)
- [Detailed primary cohort mapping](#detailed-primary-cohort-mapping)
- [Candidate details](#candidate-details)
  - [Selected next candidate](#selected-next-candidate)
  - [Future candidates](#future-candidates)

## Counting method

The inventory currently has **88 use-case rows**. Each row is assigned to one
primary job cohort so counts remain meaningful. A row may still have secondary
characteristics such as approval, fan-out, or escalation.

## High-level cohort map

| Cohort | Count | RA status |
|---|---:|---|
| Human-approved operation | **39 / 88** | Existing RA — candidates |
| Bounded autonomous remediation | **1 / 88** | Existing RA — candidate |
| Fleet-wide remediation parent | **1 / 88** | Partial |
| Exception Detection and Escalation | **10 / 88** | Selected next candidate |
| Evidence-grounded briefing and delivery | **11 / 88** | Future candidate |
| Assessment and operational routing | **8 / 88** | Future candidate |
| Forecasting, recommendation, or planning output | **7 / 88** | Future candidate |
| Research, aggregation, or synthesis | **4 / 88** | No current RA |
| Autonomous operational action | **7 / 88** | No current RA |

```text
39 + 1 + 1 + 10 + 11 + 8 + 7 + 4 + 7 = 88 use cases
```

> **Source of truth:** The detailed mapping below defines cohort membership and
> counts. The high-level map is its summary.

| Fit | Meaning |
|---|---|
| Validated | The RA's Validation record documents this scenario as a fit. |
| Candidate | The cohort assignment is plausible but has not yet been validated in an RA walkthrough. |
| Partial | Some composition fits; additional scope, pattern, or parent composition is needed. |
| No current RA | No current RA candidate is sufficiently coherent yet. |

## Detailed primary cohort mapping

Each use case appears once. In the table, blank cohort and shared-reason cells
inherit the value from the row above. A row-specific note applies only to that
one row; it is not inherited by later rows. This keeps one reviewable table
while making the groups visible.

| Primary cohort | Use case | Fit | Shared reason | Row-specific note |
|---|---|---|---|---|
| Human-approved operation | Outreach sequence drafting | Candidate | These use cases include a person who must approve a draft or proposed action before the workflow sends, writes, publishes, provisions, or otherwise applies it. They are candidates for Human-approved Operation, not validated fits yet: a walkthrough must show that the person approves the exact version that will be used, the agent cannot bypass the approval, a workflow-controlled component performs the approved effect, and the decision is recorded with the result. | |
|  | Quote & proposal generation | Candidate | | |
|  | Content repurposing pipeline | Candidate | | |
|  | Draft response generation | Candidate | | |
|  | FAQ & knowledge base update | Candidate | | |
|  | Invoice processing & coding | Candidate | | |
|  | Expense report review | Candidate | | |
|  | Monthly financial narrative | Candidate | | |
|  | Dependency upgrade scanning | Candidate | | |
|  | Resume screening | Candidate | | |
|  | Interview question generation | Candidate | | |
|  | Market landscape report | Candidate | | |
|  | Prior authorization drafting | Candidate | | |
|  | Clinical documentation assist | Candidate | | |
|  | Medication refill triage | Candidate | | |
|  | Coding & billing audit | Candidate | | |
|  | Discharge summary generation | Candidate | | |
|  | Access request fulfillment | Candidate | | |
|  | Runbook-guided remediation | Candidate |  | A human must confirm before the workflow performs a destructive action, so this is a candidate for Human-approved Operation. A walkthrough must still confirm that the person approves the exact action, the agent cannot bypass approval, and a workflow-controlled component performs the action. |
|  | Requirements analysis & gap detection | Candidate | | |
|  | Test case generation | Candidate | | |
|  | Code documentation generation | Candidate | | |
|  | Security vulnerability scan triage | Candidate | | |
|  | Release notes generation | Candidate | | |
|  | Architecture decision record (ADR) drafting | Candidate | | |
|  | Gated multi-stage delivery pipeline | Candidate | | |
|  | Inventory reorder automation | Candidate | | |
|  | Supplier bid evaluation | Candidate | | |
|  | Production schedule adjustment | Candidate | | |
|  | Contract review & red-lining | Candidate | | |
|  | NDA intake & tracking | Candidate | | |
|  | Legal hold notification | Candidate | | |
|  | Case law research brief | Candidate | | |
|  | Assignment feedback drafting | Candidate | | |
|  | Accreditation evidence packaging | Candidate | | |
|  | Listing description generation | Candidate | | |
|  | Lease abstraction | Candidate | | |
|  | Cloud resource provisioning on demand | Candidate | | |
|  | IDP self-service request fulfillment | Candidate | | |
| Bounded autonomous remediation | Autonomous maintenance run ("night shift") | Candidate | This use case gives the agent one small, pre-defined repair task, such as a dependency update or lint fix, in an isolated work area. The workflow checks each proposed change with deterministic tests or rules. If the change does not pass, it may retry only within configured limits; if it passes, the workflow performs the limited effect of opening a pull request. A human is involved only when the task cannot safely continue, such as after repeated failure or an out-of-scope change. | |
| Fleet-wide remediation parent | Closed-loop dependency & CVE remediation (fleet-wide) | Partial | Each affected repository can use Bounded Autonomous Remediation as its own child workflow. The fleet-wide parent workflow is only a partial fit because it must also find affected repositories, start and coordinate many child runs, limit or group pull requests to match human review capacity, and combine the results into one fleet-wide outcome. Those parent-level concerns are not yet covered by the current RA. | |
| Exception Detection and Escalation | Churn risk detection | Candidate | These use cases detect a condition that may need attention, collect the relevant signals and assessment, and create an escalation package for the human team responsible for the next decision. Creating and recording that handoff is part of the workflow. The human team's assessment, decision, and any consequential response are outside this workflow's scope. For example, the workflow may open an NOC escalation, but the NOC—not the workflow—decides whether to change a firewall rule. | |
|  | Brand protection & fraud monitoring | Candidate | | |
|  | Regulatory change tracking | Candidate | | |
|  | Care gap identification | Candidate | | |
|  | Alert noise reduction & grouping | Candidate | | |
|  | Network anomaly detection | Candidate | | |
|  | SSL/cert expiry monitoring | Candidate | | |
|  | SLA breach prediction | Candidate | | |
|  | Supplier risk monitoring | Candidate | | |
|  | At-risk student identification | Candidate | | |
| Research, aggregation, or synthesis | Competitor monitoring | No current RA | These use cases often collect information from several sources and combine the results, sometimes in parallel. That shared technical structure is not enough to define one Reference Architecture: the jobs, audiences, effects, and required guarantees are still too different. A future fan-out or synthesis pattern may help several of them, but there is no single job-oriented RA for this group yet. | |
|  | Cross-brand product recommendation | No current RA | | |
|  | PR review & summarization | No current RA | | |
|  | Bug triage from logs | No current RA | | |
| Autonomous operational action | B2B wholesale order assistant | No current RA | These use cases autonomously perform an external operational action, such as sending a message, creating an account, issuing a label, changing an allocation, or deploying a service. They are not Human-approved Operation fits because no person approves each action before it happens. They are not Bounded Autonomous Remediation fits because they do not describe a bounded repair loop with an independent acceptance gate. The group may later split into more specific jobs, but it is too broad for one RA today. | |
|  | Appointment no-show follow-up | No current RA | | |
|  | Onboarding task creation | No current RA | | |
|  | Returns & reverse logistics triage | No current RA | | |
|  | Maintenance request triage | No current RA | | |
|  | Automotive forecast-to-replan | No current RA | | |
|  | Service deployment orchestration | No current RA | | |
| Evidence-grounded briefing and delivery | Meeting prep brief | Candidate | These use cases read information from source systems, create a summary, report, or briefing from that information, and send the result to a known low-impact destination, such as a private channel or a stakeholder inbox. The source systems remain the authoritative record; the generated output is only a derived artifact. The workflow does not make or apply a consequential decision based on the briefing. This is a possible future RA, but its shared guarantees still need validation. | |
|  | Campaign performance digest | Candidate | | |
|  | On-call incident summary | Candidate | | |
|  | Pulse survey analysis | Candidate | | |
|  | Earnings call summarization | Candidate | | |
|  | News monitoring & digest | Candidate | | |
|  | Patient survey analysis | Candidate | | |
|  | Incident post-mortem drafting | Candidate | | |
|  | Sprint retrospective synthesis | Candidate | | |
|  | Course content summarization | Candidate | | |
|  | Rent roll & NOI reporting | Candidate | | |
| Assessment and operational routing | Lead research & enrichment | Candidate | These use cases assess incoming information and produce a routine operational result, such as assigning a ticket, enriching a record, creating a follow-up task, or sending a scheduled reminder. This group may overlap with Exception Detection and Escalation: the key question is whether the workflow performs ordinary routing or identifies an exception that must be handed to a human team for the next decision. That boundary must be tested before this group becomes a separate RA. | |
|  | Ticket triage & routing | Candidate | | |
|  | Vendor contract renewal alerts | Candidate | | |
|  | Patch compliance reporting | Candidate | | |
|  | Flaky test detection & reporting | Candidate | | |
|  | Compliance deadline tracking | Candidate | | |
|  | Entity & subsidiary monitoring | Candidate | | |
|  | Showing feedback aggregation | Candidate | | |
| Forecasting, recommendation, or planning output | SEO keyword gap analysis | Candidate | These use cases produce a forecast, recommendation, or plan, such as a demand projection, route suggestion, learning path, or price range. Some deliver advice for a person to consider, while others write a recommendation or projection into a planning system. Because those outcomes have different authority and effect boundaries, this group is not yet one clear Reference Architecture. It needs further scenario validation before it is split into more specific jobs. | |
|  | Demand forecasting | Candidate | | |
|  | Shipment delay detection & response | Candidate | | |
|  | Route & carrier optimization | Candidate | | |
|  | Personalized learning path generation | Candidate | | |
|  | Comparable sales (CMA) analysis | Candidate | | |
|  | Platform health & capacity reporting | Candidate | | |

```text
39 + 1 + 1 + 10 + 4 + 7 + 11 + 8 + 7 = 88 use cases
```
## Candidate details

### Selected next candidate

| Candidate | Cohort | Job | Why selected now | Validation scenarios |
|---|---:|---|---|---|
| Exception Detection and Escalation | **10 / 88** | Detect and assess an exception, then route evidence to a human-owned process when the workflow cannot safely resolve it autonomously. | Clear boundary distinct from current RAs: the workflow creates and records an escalation package, but the human-owned process makes the consequential decision outside this workflow's scope. | Network anomaly detection; Regulatory change tracking. |

**Candidate pattern to test in the RA:** `Evidence-backed escalation`.

Create a separate pattern only if both scenarios demonstrate the same reusable
problem, invariant set, and failure modes. Otherwise it remains
composition-level RA guidance.

### Future candidates

| Candidate | Cohort | Job | Why not selected now | Next validation step |
|---|---:|---|---|---|
| Evidence-grounded briefing and delivery | **11 / 88** | Produce a derived briefing from source evidence and deliver it to a low-impact destination. | Shared guarantees still need validation. | Test Meeting prep brief and On-call incident summary. The external Daily Activity Report is also a strong future scenario. |
| Assessment and operational routing | **8 / 88** | Assess incoming information and route a routine operational result. | May overlap with Exception Detection and Escalation. | Use Ticket triage and Patch compliance reporting to distinguish ordinary routing from exception handoff. |
| Forecasting, recommendation, or planning output | **7 / 88** | Produce an advisory forecast, recommendation, or plan. | Rows have different authority and effect boundaries. | Refine this cohort into narrower candidate jobs before drafting an RA. |
