# Evaluation Methodology

Evaluation at Keystone is a first-class artifact, not a QA afterthought. Each
published metric listed here comes from a recorded evaluation run with
retained artifacts in the ledger, traceable back to the exact cases and
endpoint that generated it. Passing and failing runs live side by side in the
same ledger.

These are internal evaluations. A PASS is a result at the evaluated commit and
does not establish generality, portability, production suitability, or
independent validation.

## Principles

1. **Evaluation is a first-class output.** Baselines are versioned, published,
   and cited, treated with the same rigor as the code they measure.
2. **Failing runs are preserved, not deleted.** A retained failing run can show
   that a particular evaluation configuration detected specific defects in the
   evaluated implementation, and it stays in the ledger next to the run that
   fixed it.
3. **For the baselines listed here, each published metric maps to a retained
   artifact in the public ledger.** A number in a table maps to a specific run,
   its cases, and its raw results, with no orphan claims.
4. **Governance is reported alongside quality.** Accuracy without a governance
   figure is half a result. Severity, fail-closed behavior, and citation presence
   are first-class assertions, not footnotes.
5. **The current harness is profile-driven** and can target compatible HTTP
   endpoints through profiles.

## Versioned baselines

Baselines follow the naming convention `keystone-{component}/{type}-v{n}`:

- **component**: the system under test (`core`, `engage`, `counsel`).
- **type**: the evaluation family (`retrieval`, `agent`).
- **v{n}**: the version, incremented when the case set or method changes.

Versioning is what makes a baseline citable. `keystone-core/agent-v1` names a
fixed set of cases, an assertion vocabulary, and a retained result artifact,
not "the latest run of the agent tests." When the case set changes, the
version increments and the prior baseline stays addressable.

## Published example baselines

The following are published examples from the evaluation ledger. Each row maps
to a retained artifact with its full case set and raw results.

| Identifier                    | Type              | Summary                                                 | Status         |
|-------------------------------|-------------------|---------------------------------------------------------|----------------|
| keystone-core/retrieval-v1    | retrieval         | P@1 0.75, MRR 0.79, 8/8 adversarial ACL blocked, fail-closed 5/6 (83%) | mixed          |
| keystone-core/agent-v0        | agent             | 186 cases; 9 failing cases, 4 root-cause defects        | retained failing |
| keystone-core/agent-v1        | agent (canonical) | 186 cases, 558 executions; 153 strict passes, 33 characterization cases (after 4 defects fixed) | passing        |
| keystone-engage/agent-v1      | engage baseline   | 100/100 (regression 70, architecture 25, edge 5)       | passing        |

The retrieval baseline reports access-control behavior as a first-class metric:
all eight adversarial cases that attempt to retrieve out-of-scope records are
blocked, and the system fails closed on 5 of 6 (83%) of the ambiguous cases in
the retained 2026-04-11 baseline. The single miss (FC-005) has a demo-grade
domain-scope guard merged 2026-05-17, with re-verification not yet recorded.
Selected governance-related behaviors are included as explicit assertions in
the evaluation.

## Retained failing runs

An earlier 66-case pre-run (`keystone-core/agent-v0-pre`) passed; expanding the
evaluation to 186 spec-compliant cases is what surfaced defects the smaller run
had missed. That expanded run, `keystone-core/agent-v0`, produced nine failing
cases traced to four distinct root-cause implementation defects. It is preserved
as a retained artifact next to the passing `keystone-core/agent-v1` baseline.
The failing run is not an embarrassment. In this lineage, the retained failing
run shows that the evaluation configuration detected specific implementation
defects in the evaluated system. It is evidence about the evaluation method,
not a proof of the system's substantive correctness, safety, or compliance.

This is a discipline the contact-center industry built long ago for compliance
and quality management: bad calls were not hidden, they were analyzed. Keystone
applies the same discipline to model behavior. See
[contact-center heritage](../design/heritage.md) for where this practice comes
from.

## The published ledger

The evaluation ledger is published at
[keystone-ledger](https://github.com/getkeystone/keystone-ledger). It holds the
retained artifacts (case sets, run metadata, and raw results) that back every
figure in the table above. The retained artifacts allow a reviewer to inspect
the recorded cases, run metadata, and raw results associated with a published
metric. Reproducing a historical execution additionally depends on
reconstructing the relevant software, models, data, and environment.

## The evaluation framework

[keystone-verify](../extensions/verify.md) is the current standalone HTTP
evaluation harness for compatible endpoints. It is open source and targets
those endpoints through evaluation profiles. The historical
`keystone-core/*` artifacts listed above are retained in keystone-ledger and
should not be assumed to have been produced by the current Verify CLI. The
current framework provides:

- A **profile system** (JSON) for declaring endpoint connection details and
  mapping response fields into the judge model.
- A separate **cases file** (JSONL) containing request dictionaries and
  deterministic assertions.
- A **pure-function judge engine** for severity, string conditions, minimum
  length, citation presence, fail-closed flags, and latency limits. It does not
  implement general semantic entailment or source-support verification.
- A **structured-artifact reporter** that writes `results.json` and
  `run_metadata.json` to a selected local directory.
- A **CLI** for running evaluations locally or in CI.

The reporter does not own historical retention or automatically publish output
to keystone-ledger. Retaining both passing and failing runs is a separate
repository and evaluation-process choice.

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
convention only. These are historical artifact identifiers retained in
keystone-ledger; keystone-core itself is not the public evidence repository.
