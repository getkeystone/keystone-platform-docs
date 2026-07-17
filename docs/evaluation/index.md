# Evaluation Methodology

Evaluation at Keystone is a first-class artifact, not a QA afterthought. Every
published metric is produced by a repeatable run, sealed to a durable
artifact in the ledger, and traceable back to the exact cases and endpoint that generated it.
Passing and failing runs live side by side in the same ledger.

## Principles

1. **Evaluation is a first-class output.** Baselines are versioned, published,
   and cited — treated with the same rigor as the code they measure.
2. **Failing runs are preserved, not deleted.** A run that surfaces real bugs is
   evidence the method works, and it stays in the ledger next to the run that
   fixed it.
3. **Every published metric is traceable to a sealed artifact.** A number in a
   table maps to a specific run, its cases, and its raw results — no orphan
   claims.
4. **Governance is reported alongside quality.** Accuracy without a governance
   figure is half a result. Severity, fail-closed behavior, and citation presence
   are first-class assertions, not footnotes.
5. **The framework is reusable, not Keystone-specific.** The same harness runs
   against any HTTP endpoint, so the methodology outlives any single system.

## Versioned baselines

Baselines follow the naming convention `keystone-{component}/{type}-v{n}`:

- **component** — the system under test (`core`, `engage`, `counsel`).
- **type** — the evaluation family (`retrieval`, `agent`).
- **v{n}** — the version, incremented when the case set or method changes.

Versioning is what makes a baseline citable. `keystone-core/agent-v1` names a
fixed set of cases, an assertion vocabulary, and a sealed result — not "the
latest run of the agent tests." When the case set changes, the version
increments and the prior baseline stays addressable.

## Published example baselines

The following are published examples from the evaluation ledger. Each row maps
to a sealed artifact with its full case set and raw results.

| Identifier                    | Type              | Summary                                                 | Status         |
|-------------------------------|-------------------|---------------------------------------------------------|----------------|
| keystone-core/retrieval-v1    | retrieval         | P@1 0.75, MRR 0.79, 8/8 adversarial ACL blocked, FC 83% | passing        |
| keystone-core/agent-v0        | agent             | 66 cases, 4 real bugs surfaced                          | sealed failing |
| keystone-core/agent-v1        | agent (canonical) | 186 cases, 558 executions, 0 failures                  | passing        |
| keystone-engage/agent-v1      | engage baseline   | 100/100 (regression 70, architecture 25, edge 5)       | passing        |

The retrieval baseline reports access-control behavior as a first-class metric:
all eight adversarial cases that attempt to retrieve out-of-scope records are
blocked, and the system fails closed 83% of the time under ambiguity. Governance
is measured, not assumed.

## Sealed failing runs

`keystone-core/agent-v0` found four real system bugs. It is preserved as a
sealed artifact next to the passing `keystone-core/agent-v1` baseline. The
failing run is not an embarrassment — it is proof that the evaluation method
finds real defects before they reach production.

This is a discipline the contact-center industry built long ago for compliance
and quality management: bad calls were not hidden, they were analyzed. Keystone
applies the same discipline to model behavior. See
[contact-center heritage](../design/heritage.md) for where this practice comes
from.

## The published ledger

The evaluation ledger is published at
[keystone-ledger](https://github.com/getkeystone/keystone-ledger). It holds the
sealed artifacts — case sets, run metadata, and raw results — that back every
figure in the table above. Because the artifacts are versioned and preserved, a
published metric can be re-derived from its source at any time.

## The evaluation framework

[keystone-verify](../extensions/verify.md) is the standalone framework that
produces these artifacts. It is open source, endpoint-agnostic, and runs against
any HTTP endpoint. The framework provides:

- A **profile system** (JSON) for declaring endpoints, request templates, and
  assertion vocabularies.
- A **pure-function judge engine** with assertion types: literal, structural,
  semantic, and governance.
- A **structured-artifact reporter** that writes machine-readable results and run
  metadata, preserving both failing and passing runs.
- A **CLI** for running evaluations locally or in CI.

The [keystone-verify page](../extensions/verify.md) covers the full
methodology and assertion model. Source:
[github.com/getkeystone/keystone-verify](https://github.com/getkeystone/keystone-verify).

## Historical naming

The ledger was formerly named `keystone-kdat`. Historical KDAT identifiers map
to the current versioned names:

| Historical | Versioned                  |
|------------|----------------------------|
| KDAT-001B  | keystone-core/retrieval-v1 |
| KDAT-002C  | keystone-core/agent-v0     |
| KDAT-002D  | keystone-core/agent-v1     |

Old references remain valid through this map; new baselines use the versioned
convention only.
