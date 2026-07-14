# Extensions

Keystone is three extensions running on one shared substrate. Each extension
targets a different regulated workload — conversation, retrieval, evaluation —
but none of them rebuilds governance, audit, authorization, or dispatch from
scratch. They plug into a common substrate that provides those primitives once,
so each extension is the workload logic and nothing more.

## keystone-engage

Governed conversational agent for regulated customer interaction. A single
customer turn runs through a five-phase governed pipeline — Empathy,
Escalation, Engagement, Budget, Monitoring — each a registered specialist with
its own tempo and cost profile. Severity-tier routing escalates to a human when
the situation requires it, budget enforcement short-circuits a runaway session,
and every phase writes a hash-chained audit entry.
[keystone-engage →](engage.md)

## keystone-counsel

Authorization-first retrieval for legal and financial advisory content. The
authorization check is a `WHERE` clause inside the retrieval query, not a
post-retrieval filter in application code, so content the caller is not
permitted to see is never returned in the first place. The system fails closed
under insufficient authorization or insufficient confidence rather than
guessing.
[keystone-counsel →](counsel.md)

## keystone-verify

Standalone evaluation harness, endpoint-agnostic. Verify runs against any HTTP
endpoint that matches its profile contract, scoring responses against a
structured assertion vocabulary and sealing every run. Failing runs are
preserved next to passing ones as evidence the methodology finds real bugs.
It is open source.
[keystone-verify →](verify.md)

## One substrate underneath

All three extensions consume the same substrate: an agents registry, a task
state machine, a tamper-evident audit ledger, an event bus, MCP-exposed tools
with agent-scoped authorization, and cost-aware dispatch. The substrate is what
makes this a platform rather than three separate applications.
See the [substrate model →](../architecture/substrate.md).
