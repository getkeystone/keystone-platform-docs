# About Keystone Applied Intelligence

Keystone Applied Intelligence is an independent AI engineering and R&D practice
focused on enterprise AI systems where authorization, evidence, evaluation,
failure handling, and operational state matter alongside model capability.

The public engineering work includes separately composed implementations for
governed retrieval, conversational-agent workflows, authorization-aware
retrieval, evaluation, and runtime-governance research. These repositories
contain related runtime mechanisms, but they should not be interpreted as one
fully composed production runtime.

The engineering approach is influenced by more than a decade of work with
enterprise contact-center systems, where routing, escalation, access control,
observability, incident handling, and controlled change are operational
requirements rather than optional features. Keystone applies those engineering
habits to LLM-based systems while treating the limits of the current
implementations explicitly.

Examples include query-time authorization before retrieval, fail-closed behavior
under selected conditions, severity-based human routing in the served Engage
path, implementation-specific audit mechanisms, retained evaluation artifacts,
and explicit task-state mechanisms. These capabilities are not uniform across
all workloads, and their presence in one implementation should not be read as
evidence that every Keystone component provides the same control.

See [the contact-center heritage](design/heritage.md) for the engineering
patterns that motivated parts of the platform design.

## How to navigate these docs

Start with [Architecture](architecture/index.md) for the engineering architecture
and the [substrate model](architecture/substrate.md) for the separate research
abstraction.

[Extensions](extensions/index.md) covers the current public capabilities:

- [keystone-engage](extensions/engage.md): governed conversational-agent
  engineering, with a served single-agent path and an optional experimental
  multi-agent path.
- [keystone-counsel](extensions/counsel.md): authorization-first retrieval with
  classification and client-isolation controls.
- [keystone-verify](extensions/verify.md): a standalone evaluation harness for
  compatible HTTP endpoints.

[Evaluation](evaluation/index.md) documents the published internal baselines and
the retained passing and failing artifacts available through
[keystone-ledger](https://github.com/getkeystone/keystone-ledger). These results
are evidence about the evaluated implementations at their recorded commits; they
are not independent validation or proof of production suitability.

[Design](design/heritage.md) explains some of the engineering lineage behind the
work. [Access](access.md) describes which implementation and evidence surfaces
are public and which deployment and infrastructure details remain private.

Runtime-governance research is developed separately under **Governed
Execution**. Its bounded public reference implementation is
[Runtime Validity (Track A)](https://github.com/getkeystone/runtime-validity).

## Where to go next

For the employer-facing engineering narrative and a walkthrough of the current
platform work, see
[getkeystone.ai/platform/](https://getkeystone.ai/platform/).

Keystone is independent engineering and research work. For technical review or
hiring conversations, see
[arnaldosepulveda.com](https://arnaldosepulveda.com) or
[LinkedIn](https://linkedin.com/in/arnaldosepulveda).
