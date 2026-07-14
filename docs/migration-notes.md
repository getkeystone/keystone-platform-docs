# Platform Page Migration Notes

!!! danger "Internal note — not published"
    This file is excluded from the built site. It is a working checklist for
    the operator, not public documentation. Do not link to it from any public
    page and do not include it in the MkDocs navigation.

Working checklist for updating the `getkeystone.ai/platform/` page so its
evidence links point at this documentation site instead of the repositories
that are going private. Apply these edits **before** flipping repo visibility,
so no link on `/platform/` ever resolves to a 404 or a login wall.

Repositories affected:

- Going private: `keystone-engage`, `keystone-counsel`, `keystone-gov`, `keystone-demo`
- Staying public: `keystone-verify`, `keystone-ledger`
- Org profile: `github.com/getkeystone` (stays; now surfaces the two public repos plus the docs site)

---

## 1. Link replacements

Every repo link in the `/platform/` HTML gets repointed. Public repos are left
untouched. The org link stays but its landing surface changes (see §5).

| # | Current `/platform/` link | New link target | Status |
|---|---------------------------|-----------------|--------|
| 1 | `keystone-engage` repo | `/docs/extensions/engage/` | replace |
| 2 | `keystone-counsel` repo | `/docs/extensions/counsel/` | replace |
| 3 | `keystone-gov` repo | `/docs/architecture/substrate/` | replace |
| 4 | `keystone-demo` repo | `/docs/access/` | replace |
| 5 | `github.com/getkeystone/keystone-verify` | `github.com/getkeystone/keystone-verify` | keep (public) |
| 6 | `github.com/getkeystone/keystone-ledger` | `github.com/getkeystone/keystone-ledger` | keep (public) |
| 7 | `github.com/getkeystone` | `github.com/getkeystone` | keep (org profile) |

- [ ] Replace link 1 (engage → docs)
- [ ] Replace link 2 (counsel → docs)
- [ ] Replace link 3 (gov → substrate docs)
- [ ] Replace link 4 (demo → access docs)
- [ ] Confirm links 5–6 (verify, ledger) still resolve to public repos
- [ ] Confirm link 7 (org) resolves and now lists only the public surface

---

## 2. Evidence chip relabeling

Chips that pointed at private repo code now point at documentation. Relabel the
`-REPO` suffix to `-DOCS` so the wording matches the weaker (docs) signal. Chips
that still point at public repos keep their `-REPO` wording.

| Chip (current) | Was pointing at | Chip (new) | Now points at |
|----------------|-----------------|------------|---------------|
| `ENG-REPO` | keystone-engage repo | `ENG-DOCS` | `/docs/extensions/engage/` |
| `COU-REPO` | keystone-counsel repo | `COU-DOCS` | `/docs/extensions/counsel/` |
| `SUB-REPO` | keystone-gov repo | `SUB-DOCS` | `/docs/architecture/substrate/` |
| `DEMO-REPO` | keystone-demo repo | `DEMO-DOCS` | `/docs/access/` |
| `VER-REPO` | keystone-verify repo | `VER-REPO` | keystone-verify repo (unchanged) |
| `LEDGER-REPO` | keystone-ledger repo | `LEDGER-REPO` | keystone-ledger repo (unchanged) |

- [ ] Relabel `ENG-REPO` → `ENG-DOCS`
- [ ] Relabel `COU-REPO` → `COU-DOCS`
- [ ] Relabel `SUB-REPO` → `SUB-DOCS`
- [ ] Relabel `DEMO-REPO` → `DEMO-DOCS`
- [ ] Leave `VER-REPO` and `LEDGER-REPO` as-is (still public code)

---

## 3. Rewrite "Run it locally" → "Verify the evidence / request access"

The current section tells a public viewer to clone `keystone-demo` and run it
locally. That path disappears when the repo goes private. Replace the whole
section with the two-part block below: public evidence stays linkable, and the
clone instructions become a request-access note.

- [ ] Delete the clone / local-run instructions
- [ ] Drop in the replacement copy below
- [ ] Rename the section heading to **Verify the evidence**

**Replacement copy:**

> ### Verify the evidence
>
> The evaluation framework and the published eval ledger are public and
> inspectable end to end:
>
> - **keystone-verify** — the endpoint-agnostic evaluation harness: profile
>   system, assertion vocabulary, and sealed-artifact methodology.
>   [GitHub](https://github.com/getkeystone/keystone-verify)
> - **keystone-ledger** — sealed passing and failing eval artifacts with full
>   `results.json` and `run_metadata.json`.
>   [GitHub](https://github.com/getkeystone/keystone-ledger)
>
> ### Request access
>
> The platform source and a runnable deployment are proprietary. Read-only
> private repo access is available on request for hiring managers, staff
> engineers, and technical evaluators conducting interview-depth review.
> Response time: 24 hours.
> [What is public vs private, and how to request access →](/docs/access/)

---

## 4. Rewrite "Deeper inspection"

Reframe so the documentation site is the first-class public surface. Public
repos are the evidence layer (verify + ledger); architecture and extension
depth live in the docs; private code is available on request via the access
page.

- [ ] Replace the existing "Deeper inspection" list with the copy below

**Replacement copy:**

> ### Deeper inspection
>
> The documentation site is the primary public surface:
>
> - **Architecture and substrate** — [system architecture](/docs/architecture/),
>   starting with the [shared substrate model](/docs/architecture/substrate/).
> - **Extensions** — capability and design pages for
>   [engage](/docs/extensions/engage/),
>   [counsel](/docs/extensions/counsel/), and
>   [verify](/docs/extensions/verify/).
> - **Evaluation** — [methodology and published baselines](/docs/evaluation/).
>
> Public source code:
>
> - [keystone-verify](https://github.com/getkeystone/keystone-verify) — the evaluation framework.
> - [keystone-ledger](https://github.com/getkeystone/keystone-ledger) — the published eval ledger.
>
> Private source (substrate, extensions, deployment) is available for technical
> review on request. [Request access →](/docs/access/)

---

## 5. Org profile note

The `github.com/getkeystone` link stays on the page. Before repo visibility
flips, edit the org profile so its landing view reflects the new surface:

- [ ] Org profile lists only the public repos (`keystone-verify`, `keystone-ledger`)
- [ ] Org profile links to the docs site for architecture, extensions, and evaluation
- [ ] No org-profile link resolves to a now-private repo

---

## 6. Sequencing and verification

Order of operations, with a gate before repo visibility changes:

1. Deploy the docs site and confirm every `/docs/...` target above renders.
2. Apply §1–§4 edits to `/platform/` and publish.
3. Update the org profile (§5).
4. Only then flip `keystone-demo`, `keystone-counsel`, `keystone-engage`, `keystone-gov` to private, verifying after each.

Post-edit verification checklist:

- [ ] No link on `/platform/` targets `keystone-engage`, `keystone-counsel`, `keystone-gov`, or `keystone-demo`
- [ ] Every replaced link resolves to a live docs page (no 404)
- [ ] `keystone-verify` and `keystone-ledger` links still resolve to public repos
- [ ] All private-repo chips read `-DOCS`; public-repo chips still read `-REPO`
- [ ] "Verify the evidence / request access" section contains no clone/local-run instructions
- [ ] "Deeper inspection" section leads with the docs site, not repos
- [ ] Page contains no internal infrastructure identifiers, IPs, hostnames, paths, or secrets
