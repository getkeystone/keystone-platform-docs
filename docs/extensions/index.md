# Extensions

The Keystone engineering platform includes two workload reference
implementations and a standalone evaluation harness:

- **keystone-engage** implements governed conversational-agent workflows.
- **keystone-counsel** implements authorization-first retrieval.
- **keystone-verify** evaluates compatible HTTP endpoints through profiles.

These repositories contain related engineering mechanisms, but they are
separately composed implementations. They do not currently consume one composed
Keystone runtime.

## keystone-engage

Engage's default served path uses a single `EngageOrchestrator`. An optional
multi-agent Coordinator is available behind configuration and is not the default
route. Engage also contains severity routing, corpus-scope authorization,
task-state mechanisms, OpenTelemetry instrumentation, and an unkeyed SHA-256
audit chain.

Some implemented mechanisms are outside the normal served path. NATS JetStream
belongs to the optional multi-agent and event integration, and public deployment
repositories do not establish it as a shared deployed service. Budget, tempo,
model, token, and cost fields exist, but real cost-based selection and complete
budget enforcement are not demonstrated end to end.

[keystone-engage](engage.md)

## keystone-counsel

Counsel's served path checks role and classification access, applies
client-relationship constraints during retrieval, handles confidence separately,
and generates locally when authorized evidence meets the configured threshold.
Records excluded by the retrieval predicates are not returned through that
query.

Client isolation is implemented and regression-tested, but the currently
published corpus contains only global records with `client_id NULL`. This is not
evidence of a production multi-client corpus. Counsel has no dedicated
evaluation baseline retained in keystone-ledger.

[keystone-counsel](counsel.md)

## keystone-verify

Verify is a standalone HTTP evaluation harness, not a workload running on the
Engage or Counsel runtime. Profiles describe how to call and interpret
compatible endpoints. The current CLI writes structured results and run
metadata.

Verify does not own historical Ledger retention, and the current CLI should not
be attributed as the producer of historical `keystone-core` artifacts without
specific lineage evidence. A passing internal run describes the evaluated
commit, cases, and configuration; it is not independent validation or evidence
of general production suitability.

[keystone-verify](verify.md)

## Relationship to the architecture

The repositories provide engineering examples relevant to shared-substrate
design concerns such as authorization, task state, evidence, and resource
metadata. Their mechanisms have different coverage and are not one universal
audit, authorization, event, or dispatch implementation.

The broader [system architecture](../architecture/index.md) documents those
boundaries. The [substrate model](../architecture/substrate.md) is a research
abstraction whose candidate dimensions remain hypotheses rather than a complete
ontology or an implemented service stack.
