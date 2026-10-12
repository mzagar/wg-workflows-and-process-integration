# Branch Isolation

**Solves:** Keeping parallel branches independent and contained, so a branch's mistake, hostile input, or excessive access cannot spread to another branch or cause a protected effect \
**Used in:** [Parallel research and merge](../architectures/parallel-research-and-merge.md) \
**Requires capabilities:** Execution engine · tool access layer · state/context store · audit log \
**Related patterns:** [Parallel fan-out and join](parallel-fan-out-and-join.md) · [Proposal/execution split](proposal-execution-split.md)

## Problem

Parallel agents often share a workspace, conversation, or credentials by default. These shared resources let mistakes and hostile input influence other branches. Instead of independent findings, the workflow can get one opinion repeated several times and one compromised branch with broad access.

## Why this is hard

Isolation must balance useful context with enforceable boundaries.

- One shared workspace and broad credential are easy to set up; separation takes deliberate work.
- Too little context makes a branch guess.
- An instruction not to read another branch is not an access restriction.
- The join and merge need all findings, so isolation ends at that boundary.
- A source can contain hostile instructions that steer its reader.

## Solution

Give each branch its own brief, tools, credentials, and private workspace. A brief is the engine's instructions and supplied context; a branch is one parallel piece of work. Branches cannot communicate with each other; findings go to the join.

```text
      execution engine writes one brief per lane
            |               |               |
            v               v               v
      +-----------+   +-----------+   +-----------+
      |  lane A   |   |  lane B   |   |  lane C   |
      |  brief A  |   |  brief B  |   |  brief C  |
      | tools: 1  |   | tools: 2  |   | tools: 3  |
      | keys:  1  |   | keys:  2  |   | keys:  3  |
      +-----+-----+   +-----+-----+   +-----+-----+
            |               |               |
            x   no messages, no shared notes   x
            |               |               |
            +---- findings go to the join only ----+
```

## Invariants (must hold in any implementation)

These limits are enforced outside the branch agents.

- A branch receives only the engine-written brief as initial context, not instructions or findings from another branch or agent; it then reads its allowed source.
- Branches cannot read each other's in-progress or finished work before the join, or message each other. The join and merge receive the collected findings.
- Tools permit reading the assigned source and writing private drafts. No branch has a tool that causes a protected effect, such as sending a message, paying, or changing a target system.
- Credentials permit only read access to the branch's source and necessary private draft storage.
- Permissions, tool availability, and communication controls enforce these limits; agent instructions alone never do.
- Discard a branch's scratch workspace when it finishes. Keep returned findings and required run/audit records, not scratch leftovers for another run.
- Record branch operations and outcomes under the shared run id and a separate branch id.

## Failure modes when violated

Each missing boundary creates a specific path for harm or confusion.

- **Shared workspace or notes:** another branch treats a mistake as fact and repeats it.
- **Inter-branch messages:** a steered branch influences the others directly.
- **Protected-effect tool:** research sends, pays, or changes something without authorization.
- **Excessive credentials:** a steered branch reads or changes systems outside its source.
- **Instruction-only isolation:** boundaries disappear when the agent follows a persuasive hostile instruction.
- **Scratch workspace reused:** leftovers leak into another run.
- **Branch identity absent from records:** a reviewer cannot tell which branch performed an operation.

## Implementation approaches (illustrative, not requirements)

Implementations differ in **where isolation is enforced**. These layers combine; no row provides full isolation alone. Every implementation must meet all invariants above; the last column highlights gaps, not a complete checklist.

| Enforced at | Documented mechanism | What it provides | What the implementation must add |
|---|---|---|---|
| Identity layer | [AWS IAM roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) | Configurable permission sets; authorized role assumption produces temporary session credentials. Different roles can have different permissions | The engine assigns one read-only, source-scoped role per branch. Restrict trust and permission policies so a branch cannot assume another role or obtain broader credentials. Separate workspaces and communication controls remain necessary |
| Workload identity layer | [Kubernetes service accounts](https://kubernetes.io/docs/concepts/security/service-accounts/) | Pods authenticate as their assigned account; multiple Pods can share it. Role-based access control (RBAC) limits access to Kubernetes API resources; otherwise Pods use the namespace's default account | A dedicated least-privilege account per branch and read-only source authorization, including any external source. Add [enforced network and storage isolation](https://kubernetes.io/docs/concepts/security/multi-tenancy/), private scratch cleanup, and no access to other branches' secrets. Disable unused API-token mounts; an account alone neither separates files nor stops messages |
| Resource layer | [Bulkhead pattern (Azure Architecture Center)](https://learn.microsoft.com/en-us/azure/architecture/patterns/bulkhead) | Separates resource pools or service instances to limit cascading failures. Platform rate and resource limits can support those boundaries | Size pools and limits per branch and account for shared dependencies. Resource separation alone does not restrict data access or communication; permissions and other isolation controls must enforce those limits |

## Known uses outside agents

[Service accounts and least-privilege permissions](https://kubernetes.io/docs/concepts/security/service-accounts/) limit what a workload may access. [Bulkheads](https://learn.microsoft.com/en-us/azure/architecture/patterns/bulkhead), named after partitions in a ship's hull, separate resource pools to limit cascading failures. These controls address different risks; none alone provides every branch boundary required here.

An agent can be steered by the material it reads. Its brief is not a security boundary. Enforce access and communication limits outside the agent rather than trusting it to obey instructions. Isolation between branches does not make collected findings safe to trust at the join or merge; source-content defenses belong to the Security workstream.
