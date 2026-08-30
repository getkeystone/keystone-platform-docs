# Platform page migration notes

!!! failure "Superseded plan"
    This plan proposed making several Keystone implementation repositories
    private. That decision was reversed and the plan was not executed as
    written.

!!! danger "Repository note, excluded from the built site"
    This file records a superseded documentation decision. It is not an active
    operator checklist and should not be linked from public navigation.

## Current state

The public implementation and evidence repositories include:

- `keystone-gov`
- `keystone-engage`
- `keystone-counsel`
- `keystone-verify`
- `keystone-ledger`
- `runtime-validity`

Deployment configuration remains a separate concern. Repository visibility does
not establish deployment state, production suitability, or the existence of one
composed shared runtime.

See [What is public vs private](access.md) for the maintained visibility
description.

## Why the original checklist was retired

The original checklist would have replaced public source links with
documentation links and introduced a request-access process. Once the
implementation repositories remained public, those steps no longer matched the
intended public surface.

The checklist also used terminology that is no longer accepted:

- Verify is a standalone HTTP evaluation harness for compatible endpoints
  through profiles, not the owner of historical artifact retention.
- Ledger retains selected internal evaluation artifacts and lineage; it does
  not provide a general cryptographic sealing mechanism.
- Passing internal evaluation belongs to an evaluated commit, cases, and
  configuration and is not independent validation.
- The workload repositories contain separately composed mechanisms rather than
  consuming one demonstrated shared runtime.

## Historical decision

The superseded plan was:

1. redirect implementation links to documentation;
2. relabel repository evidence links as documentation links;
3. replace local-run instructions with a private-access workflow; and
4. change repository visibility after those documentation changes.

Because the visibility decision was reversed, none of those steps should be
executed from this note.

Future visibility or deployment changes require a new decision record based on
the repository state at that time. They should not reactivate this historical
checklist.
