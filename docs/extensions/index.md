# Extensions

Keystone is three extensions running on one shared substrate. Each extension
targets a different regulated workload — conversation, retrieval, evaluation —
but none of them rebuilds governance, audit, authorization, or dispatch from
scratch. They plug into a common substrate that provides those primitives once,
so each extension is the workload logic and nothing more.

## keystone-engage

Governed conversational agent for regulated customer interaction. The served
path is a single governed agent. A multi-agent coordinator, available behind a
config flag and not the default route, runs a five-phase pipeline (Empathy,
Escalation, Engagement, Budget, Monitoring); empathy and escalation are
pre-dispatch gates and budget and monitoring are carried inline by the
coordinator, not standalone agents. Severity-tier routing escalates to a human
when the situation requires it, and every audit entry is SHA-256 hash-chained.
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
structured assertion vocabulary and writing a structured, reproducible run to
disk. Failing runs are preserved next to passing ones as evidence the
methodology finds real bugs.
It is open source.
[keystone-verify →](verify.md)

## One substrate underneath

All three extensions consume the same substrate: an agents registry, a task
state machine, a hash-chained (SHA-256) audit ledger, an event bus, query-time
authorization, and cost-aware dispatch. The substrate is what
makes this a platform rather than three separate applications.
See the [substrate model →](../architecture/substrate.md).
