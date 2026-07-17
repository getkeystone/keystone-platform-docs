# What is public vs private

Keystone's five repositories are public: the architecture, evaluation
methodology, design rationale, and the source. Deployment configuration and
infrastructure detail remain private. This page states exactly where that line
sits.

## What is public

The following are publicly available and inspectable:

- **Platform documentation** — this site: architecture, the shared
  [substrate](architecture/substrate.md), extension design, capabilities,
  [design heritage](design/heritage.md), and sanitized
  [evaluation summaries](evaluation/index.md).
- **The evaluation framework** — keystone-verify is open source. Inspect the
  profile system, the assertion vocabulary, and the structured-artifact
  reporter. [GitHub →](https://github.com/getkeystone/keystone-verify)
- **The published evaluation ledger** — keystone-ledger holds the sealed
  passing and failing artifacts behind every published baseline, with full
  results and metadata. [GitHub →](https://github.com/getkeystone/keystone-ledger)
- **The platform demo** — the employer-facing platform narrative at
  [getkeystone.ai/platform/](https://getkeystone.ai/platform/).

## What is private

The following remain private:

- **Internal infrastructure details** — node identifiers, network topology,
  addressing, and deployment-specific configuration.
- **Internal evaluation artifacts** that have not been published to the ledger.
- **Operational and security details** — authentication configuration, secrets
  management, and monitoring specifics.
- **The keystone-demo deployment and operator console.**

## Why it is private

Keystone is a platform under active development. The source code represents
independent engineering on governed AI infrastructure since 2024.

The split is intentional. The architecture, evaluation methodology, design
rationale, and source are public because they are the substance of the work,
the parts worth reviewing on their merits. Deployment and infrastructure detail
stay private because they are operational, not architectural.

## The public repositories

All five repositories are public and inspectable:

- [keystone-engage](https://github.com/getkeystone/keystone-engage) — governed conversational agent
- [keystone-counsel](https://github.com/getkeystone/keystone-counsel) — authorization-first retrieval
- [keystone-gov](https://github.com/getkeystone/keystone-gov) — governed RAG reference implementation
- [keystone-verify](https://github.com/getkeystone/keystone-verify) — evaluation framework
- [keystone-ledger](https://github.com/getkeystone/keystone-ledger) — published evaluation ledger

Contact: [arnaldosepulveda.com](https://arnaldosepulveda.com) ·
[LinkedIn](https://linkedin.com/in/arnaldosepulveda)
