# keystone-platform-docs

The public documentation surface for **Keystone Applied Intelligence**, an
independent AI engineering and R&D practice building and evaluating retrieval,
conversational AI, evaluation, and runtime-control reference systems.

The current public projects fall into four categories: workload implementations
(keystone-engage, keystone-counsel, keystone-gov), standalone evaluation
infrastructure (keystone-verify), retained evaluation evidence
(keystone-ledger), and research abstractions (the Governed Execution substrate
model). They contain related engineering mechanisms, but they are separately
composed and do not currently consume one demonstrated shared runtime.
Governed Execution is a separate runtime-governance research program that this
engineering work feeds into; Runtime Validity is Track A, its bounded public
reference implementation.

This repository is documentation, not product code. It explains the current
implementations, extension capabilities, evaluation methodology, and public/private
operational boundary alongside the public implementation repositories.

## What these docs cover

- **Architecture** — the layered model (extensions → substrate → infrastructure)
  and the design principles that carry the platform's identity.
- **Substrate model**: a research abstraction describing candidate
  governance-relevant runtime dimensions, not six shared services that every
  extension consumes.
- **Extensions** — Engage (governed conversation), Counsel (authorization-first
  retrieval), and Verify (evaluation for compatible HTTP endpoints through
  profiles).
- **Evaluation** — the methodology, versioned baselines, and the retained-artifact
  discipline behind the published numbers.
- **Design** — the contact-center heritage the governance model is built on.
- **Access** — what is public, what remains private, and the boundary between
  public source and private operational detail.

## What these docs intentionally do not expose

- Deployment-specific configuration and deployment repositories (for example,
  `keystone-demo`).
- Internal infrastructure identifiers and topology details: node identifiers,
  network topology, IP addresses, and operator paths.
- Secrets and authentication configuration.
- Unpublished internal evaluation artifacts not yet published to the eval
  ledger.
- Operational and security details not appropriate for the public surface.

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

Keystone's public technical surface includes:

1. **This documentation** — the primary explanatory surface (architecture,
   design, evaluation, access).
2. **The public implementation repositories**:
   [keystone-engage](https://github.com/getkeystone/keystone-engage),
   [keystone-counsel](https://github.com/getkeystone/keystone-counsel), and
   [keystone-gov](https://github.com/getkeystone/keystone-gov).
3. **[keystone-verify](https://github.com/getkeystone/keystone-verify)** — the
   open-source evaluation framework.
4. **[keystone-ledger](https://github.com/getkeystone/keystone-ledger)** — the
   published evaluation ledger with retained evaluation artifacts.
5. **[runtime-validity](https://github.com/getkeystone/runtime-validity)**:
   Track A, a separate bounded research implementation within Governed
   Execution.

The platform demo at [getkeystone.ai/platform/](https://getkeystone.ai/platform/)
is the employer-facing narrative; it links into these docs for architecture and
extension detail. See [`docs/migration-notes.md`](docs/migration-notes.md) for
how the platform page should reference this site.

## Validation

CI (`.github/workflows/docs-ci.yml`) runs on every push and pull request:

1. a **sanitization sweep** that fails the build if any internal identifier
   (node name, Tailnet/LAN IP, operator path) appears, and
2. `mkdocs build --strict`.

Run both locally before committing.

## Expected publish target

The docs are published at **`https://docs.getkeystone.ai/`** — a dedicated
Cloudflare Pages subdomain (chosen to avoid the `getkeystone.ai/docs/` marketing
catch-all). The platform page and org profile link to absolute
`https://docs.getkeystone.ai/...` URLs. See
[`PUBLICATION_CHECKLIST.md`](PUBLICATION_CHECKLIST.md) for the deployment steps and
the preflight (including removing internal-only files) before the first deploy.

