# About Keystone Applied Intelligence

Keystone is governed AI infrastructure for regulated and high-consequence
environments. It is three extensions running on shared runtime services, with
governance, authorization, and evaluation designed in from the first commit
rather than bolted on afterward.

The distinction is deliberate. Most LLM-era systems treat governance as a
wrapper: a content filter in front of a model, an audit log written after the
fact, an access-control check bolted onto retrieval once the product already
works. Keystone inverts that. Authorization fails closed at the database layer,
high-risk interactions gate on severity-tier human review, and every action
lands in a hash-chained (SHA-256) audit record whose chain is re-verifiable on
replay by `verify_chain`. The controls
are structural, not advisory — the kind of precondition a bank, an insurer, or a
legal team would require before putting any system in front of real users and
regulators.

The operational rigor is not new. It is the discipline the contact-center
industry already built for compliance reasons over the pre-LLM decades of
conversational AI, rebuilt here for the LLM substrate. See
[the contact-center heritage](design/heritage.md) for the pattern-by-pattern
mapping.

## How to navigate these docs

Start with [Architecture](architecture/index.md) for the layered model and the
[substrate model](architecture/substrate.md), which covers both the shared
runtime and the research abstraction.
[Extensions](extensions/index.md) covers the three capabilities built on that
shared runtime — [keystone-engage](extensions/engage.md) (governed conversational
agent), [keystone-counsel](extensions/counsel.md) (authorization-first
retrieval), and [keystone-verify](extensions/verify.md) (the standalone
evaluation harness). [Evaluation](evaluation/index.md) explains the
methodology and the published baselines, including failing runs sealed
alongside passing ones. [Design](design/heritage.md) traces where the
governance decisions come from. [Access](access.md) states plainly what is
public (all five repositories) and what remains private (deployment
configuration and infrastructure detail).

## Where to go next

For the employer-facing platform narrative and a walkthrough of the extensions
in context, see the platform demo at
[getkeystone.ai/platform/](https://getkeystone.ai/platform/).

Keystone is independent engineering, not a company pitch. For technical review,
hiring conversations, or a deeper look at the private implementation, reach the
builder at [arnaldosepulveda.com](https://arnaldosepulveda.com) or on
[LinkedIn](https://linkedin.com/in/arnaldosepulveda).
