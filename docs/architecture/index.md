# System architecture

Keystone Applied Intelligence is an engineering platform composed of several
public implementations and supporting evidence repositories. The current public
work should not be interpreted as one fully composed runtime shared by every
workload.

The architecture uses a **shared-substrate design direction**: authorization,
task state, audit evidence, evaluation, and resource metadata are treated as
runtime concerns that can be composed into stronger common services over time.
Today, however, those mechanisms exist at different levels of maturity and are
not uniformly implemented across the public repositories.

## Current engineering view

```text
┌──────────────────────────────────────────────────────────────────┐
│  Workload implementations                                       │
│                                                                  │
│  keystone-gov        keystone-engage        keystone-counsel     │
│  governed RAG        conversational agent    authorization-first  │
│                                              retrieval            │
└──────────────────────────────────────────────────────────────────┘
             │                  │                    │
             │ implementation-specific runtime mechanisms          │
             ▼                  ▼                    ▼
┌──────────────────────────────────────────────────────────────────┐
│  Runtime mechanisms                                              │
│                                                                  │
│  authorization       task state         audit/evidence           │
│  retrieval controls  orchestration      resource metadata        │
│  human routing       optional events    observability            │
└──────────────────────────────────────────────────────────────────┘

                   external evaluation
                           │
                           ▼
                 ┌────────────────────┐
                 │  keystone-verify  │
                 │  HTTP evaluation  │
                 │  harness          │
                 └────────────────────┘
                           │
                           ▼
                 ┌────────────────────┐
                 │  keystone-ledger  │
                 │  retained public  │
                 │  evaluation       │
                 │  lineage          │
                 └────────────────────┘
```

This diagram is an engineering inventory, not a claim that the mechanisms form
one deployed service.

## Implemented workload boundaries

### keystone-gov

`keystone-gov` is a governed RAG reference implementation. Its served query path
includes authorization-aware retrieval, PostgreSQL full-text and pgvector
retrieval, evidence thresholds, local generation, and per-record keyed
HMAC-SHA256 audit integrity.

Its authorization predicates are applied during retrieval rather than only
after records are returned. This is an implementation property of this workload,
not a platform-wide guarantee.

### keystone-engage

`keystone-engage` provides the current conversational-agent workload. The
normal served path is a single governed agent. A separate multi-agent
Coordinator is implemented behind configuration and is not the default route.

The implementation includes task-state mechanisms, severity-based human
routing, audit records, observability instrumentation, and optional NATS
JetStream integration for the multi-agent/event path.

NATS is not required by the normal served request path and should not be read as
a currently deployed shared event service across the platform.

Dispatch and audit schemas include tempo, budget, model, token, and cost-related
fields. Real cost-based model selection and end-to-end budget enforcement have
not yet been demonstrated.

### keystone-counsel

`keystone-counsel` implements authorization-first retrieval using role and
classification constraints together with client isolation in the database
query.

The client-isolation mechanism is implemented and regression-tested. The
published corpus used for the current example does not establish a full
production multi-client deployment, and there is no dedicated Counsel evaluation
baseline in the public ledger.

### keystone-verify

`keystone-verify` is external evaluation infrastructure, not a workload running
inside the Keystone runtime.

It is a standalone harness that can evaluate compatible HTTP endpoints through
structured profiles and cases and write structured result and run-metadata
artifacts.

Historical `keystone-core/*` evaluation artifacts in
[keystone-ledger](https://github.com/getkeystone/keystone-ledger) predate the
current Verify CLI and should not be assumed to have been produced by it.

## Shared-substrate mechanisms

The public implementations contain several related mechanisms that motivate the
shared-substrate design.

### Authorization and retrieval controls

Authorization is enforced differently by workload.

`keystone-gov` applies role, domain, and jurisdiction constraints during
retrieval. `keystone-counsel` applies classification and client-isolation
predicates in its retrieval query. `keystone-engage` applies its own
corpus-scope authorization logic.

These mechanisms support a design direction in which authority is checked before
governed retrieval or action, but they are not evidence of one shared
authorization service.

### Task state

Engage contains an explicit task-state implementation with validated
transitions and mechanisms for heartbeat, stuck-task handling, takeover, and
rescheduling.

Not every state or lifecycle mechanism is exercised by the normal served path.
The implementation therefore supports evaluation of these mechanisms but does
not establish distributed workflow guarantees.

### Audit and evidence

Audit implementations differ across workloads.

Engage and Counsel use unkeyed SHA-256 hash chaining. `keystone-gov` uses a
keyed HMAC-SHA256 integrity value per record with different coverage and
verification semantics.

These mechanisms provide implementation-specific integrity and lineage
properties. They do not establish semantic correctness, authorization validity,
or one common platform audit service.

### Events and observability

Engage contains optional NATS JetStream integration for the multi-agent/event
path and OpenTelemetry instrumentation.

The NATS path is off by default and is not evidence that a shared event bus is
deployed across Keystone workloads.

### Cost and resource metadata

Runtime schemas contain budget, tempo, model, token, and cost-related fields.
These are implemented data and interface mechanisms.

Active cost-based model selection, complete budget enforcement, and
platform-wide resource governance remain future engineering or evaluation work.

## Engineering architecture and research architecture

The engineering platform and the Governed Execution research architecture are
related but not interchangeable.

The controlling Governed Execution architecture separates:

1. **Control plane**: authority, policy, admissibility, placement, budget, and
   release decisions.
2. **Execution plane**: models, retrieval, tools, delegation, and workflows.
3. **Evidence plane**: decisions, authorizations, actions, evaluations, failures,
   and outcomes.
4. **Action boundary**: the boundary at which output may create an external
   consequence.

The public Keystone implementations provide engineering mechanisms that can be
used to investigate parts of this architecture. They do not demonstrate a
complete Governed Execution Runtime.

The candidate substrate dimensions of Identity, Task state, Tempo, Cost,
Currency, and Fidelity are research hypotheses, not a complete ontology and not
a list of implemented Keystone services.

See the [substrate model](substrate.md) for that research abstraction.

## Design principles

The following principles describe the direction of the engineering work without
asserting that every workload currently implements every control.

1. **Authority should be checked before governed consequence.**
   Authorization and admissibility should be evaluated before the system crosses
   a consequential boundary, rather than inferred from model output afterward.

2. **Evidence should accompany consequential decisions.**
   Decisions, authorizations, actions, failures, and evaluations should produce
   evidence sufficient for later reconstruction. Current implementations provide
   partial mechanisms toward that goal; complete external reconstructability is
   not yet established.

3. **Failure behavior should be explicit.**
   Where a workload defines a fail-closed condition, ambiguous or unauthorized
   states should produce an explicit refusal, hold, or escalation rather than an
   unsupported best-effort result.

4. **Evaluation should preserve failures as well as successes.**
   Passing and failing internal evaluation artifacts are retained so changes can
   be compared against prior observed behavior. A retained failing run shows what
   that evaluation configuration detected; it does not prove general system
   safety or correctness.

5. **Orchestration and governance are distinct concerns.**
   Orchestration determines how work proceeds. Governance determines whether the
   intended consequence remains justified to proceed.

6. **Implementation claims should remain bounded.**
   Implemented code, served behavior, retained evaluation results, deployment
   state, and research hypotheses are separate categories and should be reported
   separately.

## Related

- [The substrate model](substrate.md): the research abstraction and candidate
  substrate dimensions
- [Extensions overview](../extensions/index.md): the current workload
  implementations and evaluation harness
- [Evaluation methodology](../evaluation/index.md): retained internal evaluation
  evidence and its limits
- [What is public vs private](../access.md): repository access and boundaries
