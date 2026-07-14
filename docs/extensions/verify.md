# keystone-verify

Endpoint-agnostic evaluation harness for governed AI systems. Verify runs
against any HTTP endpoint that satisfies a structured profile and scores the
responses against an assertion vocabulary. Every run is sealed, and failing runs
are preserved next to the passing runs that replaced them.

## What it does

Verify turns "does this system behave correctly under governance" into a
repeatable, artifact-producing measurement. It is not tied to any single model,
extension, or deployment: it evaluates behavior at the network boundary, so the
same harness measures a conversational agent, an authorization-first retrieval
service, or any other governed endpoint without changing the harness itself.

The loop is deliberately small:

1. Declare a **profile** — the endpoint contract, the request template, the
   assertion vocabulary, and where cases come from.
2. Write **cases** — inputs paired with expectations.
3. Point the harness at a running endpoint and execute.
4. Read the **sealed run** — per-case results plus run metadata, written to disk
   as durable evidence.

Because the profile carries everything endpoint-specific, the judge logic stays
generic and the same evaluation discipline applies uniformly across every
extension on the platform.

## Endpoint-agnostic by design

Verify evaluates over HTTP. It does not import the system under test, share a
process with it, or reach into its internals. It substitutes each case into the
profile's request template, sends the request, and evaluates the response that
comes back over the wire.

```
profile (endpoint contract + assertions + case source)
    |
    v
[case iterator]        reads cases from the declared source
    |
    v
[executor]             renders each case into the request template,
    |                  calls the endpoint, captures response + timing +
    |                  reported cost fields
    v
[judge]                pure-function evaluators applied per assertion class
    |
    v
[reporter]             writes results.json and run_metadata.json,
    |                  then seals the run
    v
sealed run directory
```

This boundary is the point. The thing that gets measured is exactly the thing a
caller would hit in production — governance decisions, refusals, citations, and
cost included — not a mocked stand-in that behaves differently from the deployed
service.

## Profiles

A profile is the declarative contract that makes one endpoint measurable. It
specifies:

- **Endpoint** — where to send requests.
- **Request template** — how a case's fields render into a request body.
- **Case source** — where the case set is read from.
- **Assertion vocabulary** — which classes of check apply, and their
  expectations.

Swapping the profile is what re-points the harness from one endpoint to another.
The executor, judge, and reporter are unchanged; only the profile differs. This
is why a single evaluation framework can hold every extension to the same
standard.

## Assertion vocabulary

The judge is a set of pure-function evaluators grouped into five assertion
classes. A case can carry assertions from any combination of them.

| Class          | What it checks                                                                    |
|----------------|----------------------------------------------------------------------------------|
| **Literal**    | Exact-match checks: string equality, JSON-path value equality.                   |
| **Structural** | The response conforms to an expected schema or shape.                             |
| **Semantic**   | Meaning-level checks: citations resolve to the expected sources; the answer entails the expected claim. |
| **Governance** | The response's governance decision falls in the expected set (allow / review / deny). |
| **Cost**       | Per-call or per-run cost stays under a declared threshold.                        |

The governance and cost classes are what make this an evaluation vocabulary for
*governed* AI rather than a generic API test runner. A response can be fluent,
well-formed, and still wrong because it allowed something it should have denied
or spent more than its budget permitted. Those are first-class pass/fail
criteria here, not afterthoughts.

## Sealed runs, preserved failures

A run produces a sealed directory: an immutable record of what was measured,
against which cases, with what result. Each sealed run carries per-case outcomes
(`results.json`), run-level metadata such as latency and reported cost
(`run_metadata.json`), the case set, and the judge configuration.

The discipline that matters most: **a failing run is not deleted when a passing
run replaces it.** Both are kept, side by side. The failing run is retained as
evidence that the methodology surfaces real defects rather than rubber-stamping
whatever the system happens to do.

Two published examples from the evaluation ledger illustrate the pattern:

```
agent-v0   sealed, FAILING   66 cases    surfaced 4 real bugs
agent-v1   sealed, PASSING    186 cases   558 executions, 0 failures
```

The `agent-v0` run failed, and it failed usefully: it found four genuine bugs.
Those were fixed, and `agent-v1` became the canonical passing baseline. The
failing run stays in the ledger next to the passing one. A methodology that only
ever produces green checkmarks has not been shown to detect anything; a
preserved red run is the proof that it can.

## Reference profiles

The platform ships reference profiles for its own extensions as worked examples
of the contract:

- **Engage profile** — targets the governed conversational-agent endpoint.
- **Counsel profile** — targets the authorization-first retrieval endpoint.

They are illustrations of a conforming profile, not the limit of what the
harness can evaluate. Any endpoint that satisfies a profile can be measured the
same way.

## Why this matters for governed AI

Governed systems make claims that are expensive to be wrong about: this request
was authorized, this content was permitted, this action stayed within budget,
this answer is grounded in a real source. Those claims need to be *tested at the
boundary and recorded*, not asserted in a README.

Verify gives that testing a fixed shape:

- **Behavior is measured where it is served** — over HTTP, on the same surface a
  caller uses.
- **Governance is a pass/fail dimension** — allow/review/deny decisions and cost
  ceilings are checked directly, alongside correctness.
- **Every measurement is durable** — sealed runs are artifacts, not console
  output that scrolls away.
- **Failures are evidence, not noise** — preserved failing runs demonstrate the
  method has teeth.

This turns "we evaluate our system" from a statement into a set of inspectable,
reproducible artifacts.

## What is public here

This page documents the **methodology**: the endpoint-agnostic harness model,
the profile contract, the assertion vocabulary, and the sealed-run discipline
with preserved failures. That methodology is public because it is the substance
of the approach.

Access to the keystone-verify **framework and implementation** is handled under
the platform access policy. See
[What is public vs private](../access.md) for what is available and how a
technical evaluator can request access.

## Related

- [Evaluation methodology and ledger →](../evaluation/index.md)
- [keystone-engage →](engage.md)
- [keystone-counsel →](counsel.md)
- [What is public vs private →](../access.md)
