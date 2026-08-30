# keystone-counsel

## What it does

Keystone Counsel is an authorization-first retrieval reference implementation
for legal, financial, and compliance content. Its served FastAPI endpoint checks
which requested document classifications an advisor may access, retrieves from
the authorized set under a client-relationship constraint, applies a confidence
condition, generates locally when sufficient evidence is available, and records
request and response events.

This is permission enforcement around retrieval. It does not by itself establish
that a resulting advisory decision is justified. Permission governance asks who
is allowed to do what. Decision justification asks why this particular decision
was appropriate for this context, evidence, affected party, and consequence
level.

## Authorization model

Counsel applies two authorization dimensions and one separate retrieval-quality
condition:

- **Role and classification.** An in-process access matrix maps each advisor
  role and document classification to access for any client, access for the
  advisor's own clients, or denial. Unknown advisors and denied combinations
  return no authorized classification.
- **Client relationship.** Global records are eligible for an authorized caller.
  A client-specific record is eligible only when its client identifier matches
  the client context supplied to retrieval. With no client context, only global
  records are eligible.
- **Confidence.** After authorized retrieval, the best similarity result must
  meet the configured confidence threshold. Confidence is a retrieval-quality
  condition, not an authorization decision.

The current application registers demo advisor profiles in process. The
implementation therefore does not establish production authentication, correct
caller identity, or correctness of the access policy for a particular
deployment.

## Why authorization is applied during retrieval

On the PostgreSQL path, classification and client predicates constrain the rows
eligible for vector ranking. Records excluded by those predicates are not
returned through that retrieval query. On the in-memory development and test
path, equivalent filtering runs before cosine scoring.

This placement reduces the set of records exposed to later generation code. It
does not establish the absence of other data paths, complete system
confidentiality, semantic correctness, or correct authorization inputs.

## Retrieval predicate

The PostgreSQL implementation has a classification-filtered branch and a branch
without a classification filter. Both apply the client constraint. The
following is an illustrative shape of the filtered branch:

    SELECT content, classification, client_id
    FROM chunks
    WHERE classification = ANY(:authorized_classifications)
      AND (client_id IS NULL OR client_id = :caller_client_id)
    ORDER BY embedding <=> :query_embedding
    LIMIT :k;

The actual code uses positional bind parameters and selects additional provenance
fields. When no caller client is supplied, it uses `client_id IS NULL` rather
than comparing against a missing value. PostgreSQL computes similarity as
`1 - (embedding <=> query_embedding)`.

## Served request flow

The `/counsel` endpoint currently performs this sequence:

    open request audit entry
        |
    resolve requested classifications or the default advisory set
        |
    call authorize_retrieval for each classification
        |
    if none are authorized: return a fail-closed response
        |
    retrieve using authorized classifications and caller client context
        |
    if retrieval is unavailable, empty, below confidence, or generation fails:
    return a fail-closed response
        |
    generate locally from retrieved context and build citations
        |
    close the audit entry and return the response

When only some requested classifications are authorized, the endpoint continues
with the authorized subset and records the denied classifications. Local
generation is reached only after retrieval produces at least one result and the
best similarity score meets the configured threshold.

## Fail-closed behavior

The served path returns `fail_closed=true` for defined conditions including:

- an invalid requested classification;
- no authorized requested classifications;
- an unavailable or unready retrieval pipeline;
- embedding failure;
- no records returned from the authorized retrieval set;
- a best similarity score below the confidence threshold; and
- local generation failure.

The no-authorization response contains no citations. Some retrieval-quality
failures may include provenance for the authorized chunks that were retrieved,
even though no generated answer is returned. These behaviors are application
responses for defined code paths, not proof of complete system confidentiality.

## Cross-client regression

A prior defect allowed the branch without a classification filter to omit the
client boundary. The current in-memory regression tests create records for two
clients plus one global record and exercise both retrieval branches:

- with a classification filter, each caller receives its own client record and
  the global record, but not the other client's record;
- without a classification filter, the other client's record remains excluded;
  and
- with no client context, only the global record is returned.

The PostgreSQL implementation contains the corresponding client predicate in
both SQL branches. The regression test directly executes the in-memory store; it
does not run against a live PostgreSQL service.

Client isolation is therefore implemented and regression-tested. The currently
published corpus is entirely global: the corpus loader creates chunks without a
client identifier, leaving `client_id` as `NULL`. The published corpus does not
exercise a real multi-client dataset and is not evidence of a production
multi-client deployment.

## Evaluation status

There is no dedicated Counsel evaluation baseline retained in
[keystone-ledger](https://github.com/getkeystone/keystone-ledger). Historical
retrieval metrics elsewhere in the Ledger belong to an earlier governed
retrieval system under test and are not Counsel results.

Counsel contains local evaluation cases and describes a retrieval evaluation in
its own repository, but those materials are not a Counsel artifact retained in
the public Ledger. The cross-client regression tests are the current repository
evidence for client isolation.

The current [keystone-verify](verify.md) CLI is a standalone HTTP evaluation
harness. It uses profiles to call compatible endpoints and writes structured
results and run metadata. That current tool should not be attributed as the
producer of historical `keystone-core` artifacts without specific lineage
evidence.

## Audit boundary

Counsel's JSONL and PostgreSQL audit backends use an unkeyed SHA-256 hash chain.
The `AuditEntry` model includes `prev_hash` and `curr_hash`; its hash covers the
timestamp, event type, actor, payload, and previous hash. The verifier recomputes
the chain and reports malformed records, hash mismatches, or previous-hash
mismatches.

This mechanism can support detection of changes relative to trusted prior state.
An actor able to rewrite an unanchored store may potentially recompute later
hashes. The chain does not provide immutable history, independent witnessing,
semantic correctness, or proof that an authorization decision was valid.

The served request flow records events such as `request.received`,
`authorization.checked`, `authorization.denied_all`, and `response.generated`.
The implementation does not define the previously documented
`authz_denied` or `chunks_leaked` audit fields.

## Relationship to the broader platform

Counsel is a separately composed reference implementation within the Keystone
engineering platform. Its authorization, retrieval, generation, and audit
mechanisms are implemented in the Counsel repository. They are engineering
examples relevant to the broader [system architecture](../architecture/index.md)
and [substrate research abstraction](../architecture/substrate.md), not evidence
that Counsel consumes one composed platform audit or dispatch service.

## Source code

The implementation is public at
[github.com/getkeystone/keystone-counsel](https://github.com/getkeystone/keystone-counsel).
