# Branch Isolation

**Solves:** Keep parallel research branches independent. Prevent direct cross-branch access and messages, and prevent protected effects \
**Used in:** [Parallel research and merge](../architectures/parallel-research-and-merge.md) \
**Requires capabilities:** Execution engine · tool access layer · state/context store · audit log \
**Related patterns:** [Parallel fan-out and join](parallel-fan-out-and-join.md) · [Proposal/execution split](proposal-execution-split.md)

## Problem

Parallel agents often share a workspace, conversation, or credential. Shared access lets a branch influence another branch directly. Several independent views can become one repeated view.

## Why this is hard

- Sharing is easy, and many systems share by default.
- Restrictions must remain effective when an agent follows hostile instructions.
- A branch needs enough context to work without access to other branches.
- The join and merge receive all findings. Hostile source content can reach them through those findings; branch isolation does not stop that path.

## Solution

Give each branch its own brief, tools, credentials, and private workspace. Briefs contain the engine's instructions and initial context. The lanes do not share notes, credentials, or communication channels. Findings go to the join.

```mermaid
flowchart LR
    subgraph A[Lane A]
        a["Branch A<br/>brief, tools, private workspace<br/>source 1 read access"]
    end
    subgraph B[Lane B]
        b["Branch B<br/>brief, tools, private workspace<br/>source 2 read access"]
    end
    subgraph C[Lane C]
        c["Branch C<br/>brief, tools, private workspace<br/>source 3 read access"]
    end
    a & b & c -- findings --> join[Join]
    join -- collected findings --> merge("Merge<br/>rules or model")
    classDef lane fill:#fffbf5,stroke:#fdba74,color:#7c2d12
    classDef workflow fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef mergeStep fill:#f3f4f6,stroke:#6b7280,color:#374151
    class a,b,c lane
    class join workflow
    class merge mergeStep
```

The branches can share controlled infrastructure. They cannot share research context or access each other's work. The join uses rules; the merge can use rules or model reasoning. The merge treats returned findings as untrusted input and has no protected-effect tools.

## Invariants (must hold in any implementation)

- The engine writes each branch's brief. This is its only initial context; it then reads its permitted source.
- A branch cannot read another branch's work or message it. Only the join and merge receive the collected findings.
- Tools permit reading the assigned source and writing private drafts. They cannot send, pay, or change a protected target.
- Credentials permit only the source reads and private draft storage needed by that branch.
- Permissions, tool availability, and communication restrictions enforce these rules. Instructions alone do not.
- Remove scratch workspaces when branches finish. Keep returned findings and required workflow/audit records, not scratch leftovers for another run.
- Record operations under one run id and a separate branch id.

## Failure modes when violated

- **Another branch controls the brief:** one branch can steer another before research starts.
- **Shared notes or messages:** an error or hostile instruction can influence other branches directly.
- **Protected-effect tool:** research can send, pay, or change a target without authorization.
- **Credentials too broad:** a steered branch reaches systems outside its task.
- **Prompt-only restriction:** the agent can follow an instruction that breaks the supposed boundary.
- **Workspace reused:** old content leaks into another run.
- **Branch id absent:** a reviewer cannot identify which branch performed an operation.

## Implementation approaches (illustrative, not requirements)

Implementations differ in **where isolation is enforced**. Combine layers as needed. Each row supplies only part of the pattern; identity or resource separation alone is not full isolation.

| Enforced at | Documented mechanism | What it provides | What the implementation must add |
|---|---|---|---|
| Identity | [AWS IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) | Permission sets and temporary session credentials | The engine assigns a read-only, source-scoped role per branch. Restrict trust and permission policies to prevent other-role access or broader credentials. Separate files and communication remain necessary |
| Workload identity | [Kubernetes service accounts](https://kubernetes.io/docs/concepts/security/service-accounts/) | A Pod uses its assigned account; several Pods can share one. Role-based access control (RBAC) limits Kubernetes API access | A dedicated account per branch and suitable source authorization. Add [enforced network/storage boundaries](https://kubernetes.io/docs/concepts/security/multi-tenancy/), private scratch cleanup, and no other branch's secrets. Disable unused API-token mounts. RBAC alone does not isolate the workload |
| Resources | [Bulkhead pattern (Azure)](https://learn.microsoft.com/en-us/azure/architecture/patterns/bulkhead) | Separate resource pools limit cascading failures | Size limits per branch and account for shared dependencies. Permissions and communication controls must restrict access; resource limits alone do not |

## Known uses outside agents

Service identities and least privilege limit access. Bulkheads separate resources to limit cascading failures. These controls address different risks; no single control gives every branch boundary.

An agent's instructions are not an access boundary. Enforce restrictions outside the agent. Isolation does not prove that findings are safe or true. Defenses for hostile source content belong to the Security workstream.
