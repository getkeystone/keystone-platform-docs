# What is public vs private

Keystone draws a deliberate line between what is open to inspection and what is
proprietary. The architecture, evaluation methodology, and design rationale are
public. The implementation is not. This page states exactly where that line
sits and how to request access to the private side for technical review.

## What is public

The following are publicly available and inspectable:

- **Platform documentation** — this site: architecture, the shared
  [substrate](architecture/substrate.md), extension design, capabilities,
  [design heritage](design/heritage.md), and sanitized
  [evaluation summaries](evaluation/index.md).
- **The evaluation framework** — keystone-verify is open source. Inspect the
  profile system, the assertion vocabulary, and the sealed-artifact
  methodology. [GitHub →](https://github.com/getkeystone/keystone-verify)
- **The published evaluation ledger** — keystone-ledger holds the sealed
  passing and failing artifacts behind every published baseline, with full
  results and metadata. [GitHub →](https://github.com/getkeystone/keystone-ledger)
- **The platform demo** — the employer-facing platform narrative at
  [getkeystone.ai/platform/](https://getkeystone.ai/platform/).

## What is private

The following are proprietary:

- **Source code** for keystone-engage, keystone-counsel, the keystone-gov
  substrate, the keystone-demo deployment, and the operator console.
- **Internal infrastructure details** — node identifiers, network topology,
  addressing, and deployment-specific configuration.
- **Internal evaluation artifacts** that have not been published to the ledger.
- **Operational and security details** — authentication configuration, secrets
  management, and monitoring specifics.

## Why it is private

Keystone is a platform under active development. The source code represents
eighteen months of independent engineering on governed AI infrastructure.

The split is intentional, not defensive. The architecture, evaluation
methodology, and design rationale are public because they are the substance of
the work — the parts worth reviewing on their merits. The implementation is
private because it is the product.

## Request access for technical review

Read-only access to the private repositories is available on request for
interview-depth technical review.

- **Intended audience** — hiring managers, staff engineers, and technical
  evaluators.
- **Scope** — read-only. Access is for review, not modification.
- **Response time** — within 24 hours.

Contact: [arnaldosepulveda.com](https://arnaldosepulveda.com) ·
[LinkedIn](https://linkedin.com/in/arnaldosepulveda)
