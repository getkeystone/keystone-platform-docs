# keystone-verify

## What it does

Keystone Verify is a standalone HTTP evaluation harness. A profile describes
how to call a compatible endpoint and map its response fields. A JSONL cases
file supplies requests and declarative assertions. The runner calls the
endpoint, the judge evaluates each response, and the reporter writes structured
results and run metadata.

Verify evaluates the HTTP response surface. It does not import the system under
test or inspect its internal control flow. A passing assertion therefore
describes observed response behavior for that case, not the correctness of the
implementation mechanism behind it.

## Inputs

A run takes two explicit inputs:

- a **profile** containing the base URL, endpoint, HTTP method, timeout, and
  response-field mappings; and
- a **cases file** containing request dictionaries, categories, buckets, and
  assertions.

Profiles make the harness compatible with endpoints whose response shapes can be
mapped to Verify's model. They do not make every HTTP endpoint evaluable without
configuration.

## Current request path

    profile + cases
        |
    load and validate inputs
        |
    send each case to the profile's HTTP endpoint
        |
    measure latency and parse the JSON response
        |
    judge(case, response, profile, latency)
        |
    aggregate results
        |
    write results.json and run_metadata.json

The judge is a pure function with unit coverage. Network I/O and timing belong
to the runner; filesystem output belongs to the reporter.

## Assertion vocabulary

The current judge supports:

| Assertion | Current check |
|---|---|
| `severity` | exact mapped severity |
| `severity_in` | mapped severity belongs to an allowed set |
| `min_length` | mapped answer meets a minimum length |
| `contains` | all required substrings appear |
| `contains_any` | at least one required substring appears |
| `absent` | prohibited substrings do not appear |
| `has_citations` | mapped citations are present or absent as expected |
| `fail_closed` | mapped refusal flag matches |
| `max_latency_ms` | measured request latency stays within the case limit |

These are deterministic assertions over mapped response fields and measured
latency. The current judge does not implement a general semantic-entailment
model or verify that cited source material actually supports the answer.

## Output and retention boundary

Each run writes:

- `results.json`, containing one structured result per case; and
- `run_metadata.json`, containing the profile name, timestamp, aggregate
  counts, category and bucket summaries, and latency statistics.

The optional `--content-checksum` flag adds a SHA-256 checksum to
`run_metadata.json`. It is off by default and is documented in code as an aid
for detecting accidental modification, not as cryptographic sealing or
tamper-evidence.

Verify writes run output to the selected local directory. It does not own
historical retention or automatically publish artifacts to
[keystone-ledger](https://github.com/getkeystone/keystone-ledger). Retaining both
passing and failing runs is a repository and evaluation-process choice outside
the current CLI.

## Profiles and examples

The repository includes:

- profiles for locally served Engage and Counsel endpoints;
- vendor-neutral reference profiles for hypothetical governed endpoints; and
- a self-contained example profile that calls a public echo service.

These are worked examples of the profile contract. A profile's existence does
not establish that its target service is running, deployed, or evaluated in a
particular environment.

## Evaluation interpretation

A Verify run can test defined response behaviors such as refusal flags,
citations being present, severity values, string conditions, and latency. It
cannot by itself establish:

- that authentication or authorization policy is correct;
- that a citation semantically supports an answer;
- that logs prove a decision was justified;
- that an evaluated service is production-suitable;
- that results generalize beyond the evaluated cases and configuration; or
- that an internal result is independent validation.

A passing run belongs to the evaluated endpoint version, profile, cases, and
configuration. A failing run can identify which assertions failed in that run;
it does not establish that the evaluation methodology generally detects all
relevant defects.

## Historical artifact boundary

The current Verify CLI should not be retroactively attributed as the producer of
historical `keystone-core` artifacts without specific lineage evidence. Those
retained artifacts describe historical systems under test and evaluation
processes. Verify's current implementation should be documented from its own
runner, judge, reporter, profiles, and generated output.

## Relationship to the broader platform

Verify is external evaluation infrastructure, not a workload running on the
Engage or Counsel runtime. It can evaluate compatible HTTP endpoints through
profiles and produce structured run artifacts. Ledger separately retains
selected internal evaluation evidence and lineage.

## Source code

The framework and implementation are public at
[github.com/getkeystone/keystone-verify](https://github.com/getkeystone/keystone-verify).

## Related

- [Evaluation methodology and Ledger](../evaluation/index.md)
- [keystone-engage](engage.md)
- [keystone-counsel](counsel.md)
- [What is public vs private](../access.md)
