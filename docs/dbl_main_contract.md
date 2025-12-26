# DBL Main Contract (v0.3.0)

Status: Draft
Scope: dbl-main projection layer built on dbl-core v0.3.0.

## Terms
- V: deterministic event stream (append-only) defined by dbl-core.
- t_index: integer index into V, with t_index = index(V) for the event itself.
- Event kinds: INTENT, DECISION, EXECUTION, PROOF (from dbl-core).
- Decision outcome: ALLOW or DENY, represented as a DECISION event (Delta).

## Non-Goals
- dbl-main MUST NOT execute tasks.
- dbl-main MUST NOT embed a kernel runtime.
- dbl-main MUST NOT rely on observational fields to derive decisions or state.
- dbl-main MUST NOT mutate prior events in V.
- dbl-main MUST NOT evaluate policy or propose DECISION events.
- dbl-main MUST NOT interpret execution success or failure.

## Responsibilities
- dbl-main MUST provide a deterministic projection of V into a finite state model:
  project_state(V) -> State
- The projection MUST be a pure function:
  - same V bytes -> same State
  - no time, randomness, IO, network, environment, caches, or mutable globals

## Deterministic projection rules
- project_state(V) MUST look only at:
  - event ordering within V
  - event.kind (INTENT, DECISION, EXECUTION, PROOF)
  - DECISION outcome (ALLOW, DENY)
- project_state(V) MUST NOT use:
  - execution trace fields (success, failure_code, exception_type, trace_digest)
  - metadata, caller ids, tenant ids, free-form payloads
  - timestamps, wall-clock time, runtime fields
Correlation_id is ignored by projection. dbl-main projects the single global stream V.

## Validity and invalidity
dbl-main MUST classify streams that violate minimal sequencing constraints as INVALID.

Minimal constraints (hard):
1) EXECUTION MUST be preceded by an effective ALLOW DECISION.
2) PROOF advances phase only if it follows EXECUTION.
3) A DECISION MUST be preceded by INTENT.
4) If both DENY and ALLOW decisions exist, the latest decision is the effective one.

Effective decision means the latest DECISION in V by index, not by timestamp.
Events outside the effective decision segment are non-reachable for phase advancement.

Notes:
- Multiple INTENT events MAY exist (re-intent), but decisions are still ordered by position.
- Multiple DECISION events MAY exist; last one wins.
- Multiple EXECUTION/PROOF events MAY exist; projection is defined on the latest reachable phase.

## Outputs
The projection output is a State with:
- phase: one of {EMPTY, INTENTED, DENIED, ALLOWED, EXECUTED, PROVEN, INVALID}
- runner_status: one of {EMPTY, INTENTED, DENIED, ALLOWED, FINALIZED, INVALID}
- t_index: last index observed or None for EMPTY

runner_status is a coarse view for runners. FINALIZED collapses EXECUTED and PROVEN.
FINALIZED means an EXECUTION or PROOF marker exists in V after an effective ALLOW and does not imply success.
Phase and runner_status are structural projections only and MUST NOT be interpreted as success.

## Versioning guarantees
- dbl-main MUST remain compatible with dbl-core v0.3.0 event kind semantics.
- Any change that alters project_state(V) output for an identical V is a breaking change.

## Security and audit stance
- V is the sole source of truth.
- External observations are advisory only and MUST NOT change decisions or projection output.
- Consumers MAY attach observational diagnostics outside of dbl-main, but MUST NOT feed them into project_state(V).
