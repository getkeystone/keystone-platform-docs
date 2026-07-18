# What is public vs private

Keystone's five repositories are public: the architecture, evaluation
methodology, design rationale, and the source. Deployment configuration and
infrastructure detail remain private. This page states exactly where that line
sits.

## What is public

The following are publicly available and inspectable:

- **Platform documentation** — this site: architecture, the shared
  [substrate](architecture/substrate.md), extension design, capabilities,
  [design heritage](design/heritage.md), and
  [evaluation summaries](evaluation/index.md).
- **The evaluation framework** — keystone-verify. Inspect the profile system,
  the assertion vocabulary, and the judge engine.
  [GitHub →](https://github.com/getkeystone/keystone-verify)
- **The published evaluation ledger** — keystone-ledger holds the artifacts
  behind every published baseline, with results and metadata.
  [GitHub →](https://github.com/getkeystone/keystone-ledger)
- **The conversational extension** — keystone-engage. Governed conversational
  agent source.
  [GitHub →](https://github.com/getkeystone/keystone-engage)
- **The retrieval extension** — keystone-counsel. Authorization-first retrieval
  source, including the client-isolation enforcement.
  [GitHub →](https://github.com/getkeystone/keystone-counsel)
- **The governed RAG reference implementation** — keystone-gov. Query-time
  RBAC, fail-closed gating, and audit trail source.
  [GitHub →](https://github.com/getkeystone/keystone-gov)
- **The platform demo** — the employer-facing platform narrative at
  [getkeystone.ai/platform/](https://getkeystone.ai/platform/).

## What is private

The following remain private:

- **keystone-demo deployment configuration** — Docker Compose, Caddy, initdb
  migrations, and corpus management. This is a separate private repository.
- **Internal infrastructure details** — node identifiers, network topology,
  addressing, and deployment-specific configuration.
- **Internal evaluation artifacts** that have not yet been published to the
  ledger.
- **Operational and security details** — authentication configuration, secrets
  management, and monitoring specifics.

## Why it is private

What remains private is deployment-specific: infrastructure identifiers,
operational configuration, and internal artifacts that are not yet part of the
published evaluation ledger. The architecture, the evaluation methodology, and
the extension source are public because they are the substance of the work.
What stays private is operational detail that would not help a technical
reviewer and could complicate deployment security if disclosed.

Contact: [arnaldosepulveda.com](https://arnaldosepulveda.com) ·
[LinkedIn](https://linkedin.com/in/arnaldosepulveda)
