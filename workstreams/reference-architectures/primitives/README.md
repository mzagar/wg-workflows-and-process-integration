# Reference Architecture Primitives

This experimental catalog names small workflow building blocks that patterns
and reference architectures reuse. A primitive is not an implementation, a
complete pattern, or a reference architecture. It describes one shared
workflow behavior or record that patterns can compose.

This catalog builds on terms defined by the Taxonomy workstream. It does not
redefine those terms or establish stable identifiers, normative requirements,
or a complete primitive registry.

## Primitives

| ID | Primitive | Purpose |
|---|---|---|
| [P01](P01_durable-state-checkpoint.md) | Durable state checkpoint | Preserve sufficient execution state to resume safely. |
| [P02](P02_execution-evidence-record.md) | Execution evidence record | Preserve a reconstructable record of decisions, evidence, effects, and outcomes. |

## Relationship to patterns and reference architectures

```text
Primitive → pattern → job-oriented reference architecture
```

- A **primitive** names a small shared behavior or record.
- A **pattern** solves one recurring workflow problem and owns its invariants
  and failure modes.
- A **reference architecture** composes patterns, capability roles,
  boundaries, guarantees, and exit states for a recognizable practitioner job.

Primitives do not prescribe agent count or topology. A future execution profile
or topology may compose primitives when a validated reference architecture
requires it.
