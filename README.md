# keystone-platform-docs

The public documentation surface for **Keystone Applied Intelligence** — a
governed AI platform for regulated enterprises: three extensions on one shared
substrate, with governance, authorization, and evaluation designed in from the
first commit.

This repository is documentation, not product code. It explains the platform's
architecture, extension capabilities, evaluation methodology, and access policy
in enough depth for a serious technical reader to evaluate the work — without
exposing the implementation.

## What these docs cover

- **Architecture** — the layered model (extensions → substrate → infrastructure)
  and the design principles that carry the platform's identity.
- **Substrate** — the six shared surfaces every extension consumes.
- **Extensions** — Engage (governed conversation), Counsel (authorization-first
  retrieval), and Verify (endpoint-agnostic evaluation).
- **Evaluation** — the methodology, versioned baselines, and the sealed-artifact
  discipline behind the published numbers.
- **Design** — the contact-center heritage the governance model is built on.
- **Access** — what is public, what is private, and how to request read-only
  technical review.

## What these docs intentionally do not expose

- Source code for the proprietary repositories: `keystone-engage`,
  `keystone-counsel`, `keystone-gov` (substrate implementation), and
  `keystone-demo` (deployment).
- Internal infrastructure: node identifiers, network topology, IP addresses,
  operator paths, secrets, and deployment-specific configuration.
- Internal evaluation artifacts not published to the eval ledger.

Sanitization is a first-class constraint here: no page contains an internal
hostname, IP, private path, or secret.

## Run locally

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

mkdocs serve      # http://127.0.0.1:8000
mkdocs build      # renders the static site to ./site
```

The theme is MkDocs Material; configuration is in `mkdocs.yml`.

## How this fits the Keystone public surface

Keystone's public surface has three parts:

1. **This documentation** — the primary explanatory surface (architecture,
   design, evaluation, access).
2. **[keystone-verify](https://github.com/getkeystone/keystone-verify)** — the
   open-source evaluation framework.
3. **[keystone-ledger](https://github.com/getkeystone/keystone-ledger)** — the
   published evaluation ledger with sealed artifacts.

The platform demo at [getkeystone.ai/platform/](https://getkeystone.ai/platform/)
is the employer-facing narrative; it links into these docs for architecture and
extension detail. See [`docs/migration-notes.md`](docs/migration-notes.md) for
how the platform page should reference this site.
