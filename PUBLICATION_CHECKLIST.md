# Publication checklist — keystone-platform-docs

Operational checklist for taking this docs repo public. Nothing here changes
remote state; it documents the decisions and the exact steps to run when you
choose to go live.

## Canonical naming decision

**Canonical repo: `keystone-platform-docs`.**

An earlier local-only draft exists at `keystone-docs-public` (an 8-page MVP). This
repo supersedes it: it is the expanded, spec-complete site (architecture overview,
substrate deep-dive, three extension pages, evaluation, heritage, access, about,
plus an internal migration note). `keystone-docs-public` should be treated as an
abandoned local draft — do **not** create a remote for it. Keep or delete it
locally; it is not part of the public surface.

## Remote creation (run later, only when authorized)

Create the public remote from this repo, without deploying anything:

```bash
cd keystone-platform-docs        # the local clone of this repo
gh repo create getkeystone/keystone-platform-docs \
  --public \
  --description "Keystone Applied Intelligence — public platform documentation" \
  --source=. --remote=origin
# then, when ready:
git push -u origin main
```

## Deployment options

| Option | How | Notes |
|---|---|---|
| **GitHub Pages** | `mkdocs gh-deploy` from CI, or a Pages deploy workflow | Simplest; serves at `<org>.github.io/<repo>/` or a custom domain via CNAME. |
| **Cloudflare Pages** | Connect the repo; build command `mkdocs build`, output dir `site` | Matches how the marketing/personal sites are already hosted; one place for DNS. |

`.github/workflows/docs-ci.yml` here is **validation only** (strict build +
sanitization gate). It intentionally does not deploy — wire deployment in as a
separate, deliberate step.

## URL target — decided: `docs.getkeystone.ai` (Option A)

The docs are served from a dedicated Cloudflare Pages subdomain,
`https://docs.getkeystone.ai/`. This was chosen over path routing at
`getkeystone.ai/docs/` because a dedicated deployment has **zero catch-all risk**
(the current `getkeystone.ai/docs/*` is a marketing soft-200) and is the fastest
operator-safe path. The `/platform/` and org-profile links have been repointed to
absolute `https://docs.getkeystone.ai/...` URLs (held commits).

`site_url` in `mkdocs.yml` is set to `https://docs.getkeystone.ai/`. The full
cutover steps are in `DOCS_DOMAIN_CUTOVER_PLAN.md` (release-ops working area).

## Preflight — before the first public push

- [ ] **Remove the internal file `TECHNICAL_REVIEW_ACCESS_POLICY.md`** from the
      public repo (it is an internal ops doc). Either `git rm --cached` it and add
      it to `.gitignore`, or move it to a private location. It must not ship in a
      public repo.
- [ ] Sanitization sweep returns zero matches (CI enforces this; run locally too).
- [ ] `mkdocs build --strict` passes.
- [ ] `.venv/` and `site/` are not tracked (both are gitignored).
- [ ] `docs/migration-notes.md` is excluded from the built site (it is, via
      `exclude_docs`) — confirm `site/migration-notes/` does not exist after build.
- [ ] No links to `keystone-engage`, `keystone-counsel`, `keystone-gov`, or
      `keystone-demo` as public GitHub destinations in any published page.
- [ ] `site_url` in `mkdocs.yml` matches the chosen deploy base.

## Post-deploy verification

- [ ] Every page in the nav renders; no 500s.
- [ ] Every internal link resolves (no 404 within the docs).
- [ ] A nonexistent path (e.g. `/docs/nope`) returns a real 404 (the branded
      `404.html`), **not** a soft-200 homepage/catch-all.
- [ ] Re-run the sanitization sweep against the deployed HTML: zero matches.
- [ ] The `/platform/` page's docs links resolve to real pages here.
- [ ] One other person loads the site and confirms it renders.

## 404 handling

Material ships a branded `404.html` automatically (verified in the local build).
No custom `docs/404.md` is added, because a content page would build to `/404/`
and would not be the server's 404 handler — the theme's `404.html` already is.
Customize by overriding the theme's `404.html` template if branded copy is wanted.
