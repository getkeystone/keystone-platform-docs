# keystone-counsel

Authorization-first retrieval for legal and financial advisory content.

## What it does

Counsel answers advisory queries in regulated environments where content
authorization is not optional and partial leaks are not acceptable. The
authorization check lives in the `WHERE` clause of the retrieval query, not in a
post-retrieval filter in application code. Rows the caller is not permitted to
see never reach the application.

## Authorization model

Two dimensions determine what a query returns:

- **Caller authorization**: the caller's role resolves to the set of document
  classifications it is permitted to see. That set constrains the query.
- **Confidence threshold**: a chunk must clear a minimum similarity score to be
  returned. Below threshold, the system refuses rather than guessing.

The critical property: a bug in the orchestrator, a prompt injection, or a
hallucinated citation cannot leak content the caller is not permitted to see,
because the query never returned it. Enforcement does not depend on downstream
code behaving correctly.

## Why enforcement lives at the database layer

Most retrieval systems filter after the fact: fetch candidates, then drop the
ones the caller should not see in application code. That design has a standing
failure mode. Any component between the fetch and the filter — an orchestrator
bug, an injected instruction, a model that fabricates a citation — can surface
content that was retrieved but should have been withheld. The unauthorized rows
were in memory; something only had to fail to omit them.

Counsel closes that gap by making authorization part of the query. Unauthorized
rows are never selected, so they are never in the application's memory to leak.
There is no post-retrieval filter to bypass, because there is nothing to filter
out.

## The filter, at the layer that matters

The ACL predicate is part of the retrieval query itself. The caller's authorized
classifications constrain the candidate rows **before** similarity ranking, so an
unauthorized row is never a candidate.

The snippet below is an **illustrative shape**, not the production query. Real
column names, operators, tuned thresholds, and the full schema are omitted.

```sql
-- ILLUSTRATIVE SHAPE — not the production query.
SELECT id, content, classification
FROM   documents
WHERE  classification = ANY(:caller_authorized_classifications)  -- ACL lives here
  AND  embedding <=> :query_embedding < :distance_threshold       -- confidence as a bind param
ORDER BY embedding <=> :query_embedding
LIMIT  :k;
```

Two things to note. First, the ACL is a `WHERE` predicate bound to the caller's
authorized classifications — not a value the model or the orchestrator can
influence. Second, the confidence cutoff is a bind parameter, not a literal:
the actual threshold is tuned and not published.

## Fail-closed behavior

Counsel refuses in two cases, and both are refusals rather than guesses:

- **No authorized rows**: the caller is not permitted to see anything relevant
  to the query. The system returns a refusal and logs an escalation.
- **Below confidence threshold**: authorized rows exist but none clears the
  similarity cutoff. The system declines to compose an answer from weak matches.

In both cases the audit entry records that no content was disclosed. A refusal
is the safe outcome; a fabricated or partially-authorized answer is not.

## Request flow

```
caller identity + query
    |
    v
[authorization resolver]
    -- resolves caller's authorized classifications
    |
    v
[retrieval query with ACL WHERE clause]
    -- only authorized rows returned
    -- unauthorized rows never leave the database
    |
    v
[confidence check]
    -- if no rows or top similarity below threshold: FAIL_CLOSED
    -- audit entry: authz_denied=true, chunks_leaked=0
    |
    v
[response composition]
    -- reached only if authorized chunks passed threshold
    -- citations trace to specific authorized chunks
    |
    v
answer returned
```

## Two callers, same query, different outcomes

```
# authorized caller: results returned
→ authorized results returned (across the caller's permitted classifications)

# unauthorized caller: fails closed
→ 0 authorized results
→ FAIL_CLOSED: refusing to answer; escalation logged
→ audit_entry: authz_denied=true, chunks_leaked=0
```

The query is identical. The difference is entirely in what the caller is
authorized to retrieve, enforced before ranking.

## Published evaluation

| Baseline                      | Result                                       |
|-------------------------------|----------------------------------------------|
| keystone-core/retrieval-v1    | P@1=0.75, MRR=0.79, 8/8 ACL probes blocked   |

There is no sealed counsel evaluation baseline yet; counsel's client isolation
is covered by the cross-client retrieval regression test, which verifies denial
on both the classification-filtered and unfiltered paths. The core retrieval
baseline exercises adversarial ACL probes: eight queries crafted to retrieve
content the caller is not authorized to see, all blocked at the query layer.
Runs are executed by the endpoint-agnostic harness — see
[keystone-verify →](verify.md).

Eval artifacts: [keystone-ledger →](https://github.com/getkeystone/keystone-ledger)

## Substrate dependencies

Counsel enforces authorization at the database layer and consumes the substrate
audit chain and cost-aware dispatch. See the
[substrate model →](../architecture/substrate.md).

## Source code

The keystone-counsel source is public:
[github.com/getkeystone/keystone-counsel](https://github.com/getkeystone/keystone-counsel).
