# Operational Intelligence Before Applied AI

Keystone's engineering work does not assume that every operational problem
needs AI. Before choosing retrieval, automation, conversational AI, or another
model-assisted mechanism, the upstream questions are:

- What is actually happening in the operation?
- What evidence supports that conclusion?
- What can the available evidence legitimately measure?
- What remains unknown?
- What competing explanations fit the evidence?
- Where is the supported operational constraint?
- What interventions could address it?
- Is Applied AI actually the best intervention?

[Support Operations Intelligence](https://github.com/arnaldosepulveda/support-operations-intelligence)
is a public research and portfolio project maintained by Arnaldo Sepulveda. It
is part of the broader Keystone portfolio of applied work and provides a bridge
between operational diagnosis and Applied AI implementation.

## From evidence to intervention

```text
operational evidence
        ↓
source semantics
        ↓
analytical contract
        ↓
baseline
        ↓
competing explanations
        ↓
workflow diagnosis
        ↓
intervention selection
        ↓
process / software / retrieval / automation / AI
        ↓
evaluation
        ↓
operational and business decision
```

Applied AI enters only when the operational diagnosis and comparison of
interventions justify it. The sequence is a reasoning discipline, not a claim
that Keystone currently implements the entire evidence-to-business-outcome
lifecycle.

## Relationship to Keystone

```text
Support Operations Intelligence
    What should change?
    Is AI warranted?

            ↓ if warranted

Keystone Applied AI Engineering
    How should the intervention be implemented?

            ↓

Evaluation / Verify / Ledger
    Did the mechanism behave as intended?
    What evidence supports the result?

            ↓ when stronger runtime governance is relevant

Governed Execution
    Is the intended consequence still justified
    at execution time?
```

Support Operations Intelligence is not an implemented component of the
Keystone platform. It is an upstream research and portfolio project that
complements Keystone's applied engineering work. The workload implementations,
Verify, Ledger, Governed Execution, and Runtime Validity retain their documented
boundaries; together they should not be read as one completed or universally
validated production platform.

## Possible interventions

An operational diagnosis may lead to:

- better data;
- process change;
- policy change;
- routing change;
- deterministic software;
- search or retrieval;
- automation;
- Applied AI;
- no intervention until stronger evidence exists.

A conclusion that AI is not warranted is a valid outcome.

## Current status

Support Operations Intelligence is under active development. Current public
work includes:

- a deliberately small canonical Case abstraction;
- explicit evidence and claim boundaries;
- source-validation work using real operational datasets;
- whole-file structural and raw-identifier validation of a 7.47-million-row
  Calgary 311 artifact.

This work does not yet establish a completed source adapter, workflow diagnosis,
causal operational finding, AI intervention, live operational improvement, or
realized business or financial outcome.

Continue with the public
[Support Operations Intelligence repository](https://github.com/arnaldosepulveda/support-operations-intelligence).
