# ANPL Production Site

This repository deploys the production version of the Asphodel-Norwood Public Library website to [anpl.org](https://anpl.org).

**Source of truth:** [foxpress-design/anpl-staging](https://github.com/foxpress-design/anpl-staging)

Content is managed via the CMS at [staging.anpl.org/admin](https://staging.anpl.org/admin). When editors click "Push to Live", a GitHub Actions workflow clones the staging repo, builds the site, and deploys it here via GitHub Pages.

Do not edit files in this repository directly. All changes should be made in the staging repo.

---

## How deploys work

1. Staging repo sends a `repository_dispatch` event (event type: `deploy`) to this repo.
2. The `deploy` workflow checks out the staging `main` branch (using `STAGING_PAT`).
3. The site URL is patched from `staging.anpl.org` to `anpl.org` via sed.
4. `npm ci` installs dependencies from `package-lock.json`.
5. `npx astro build` produces a static site in `dist/`.
6. `actions/deploy-pages` pushes `dist/` to GitHub Pages.
7. A notification email is sent to Philip and Trish via Resend.

To trigger a deploy manually (without using the CMS button):

```bash
gh api repos/foxpress-design/anpl-production/dispatches -f event_type=deploy
```

---

## Secrets required

| Secret | Purpose |
|---|---|
| `STAGING_PAT` | Fine-grained PAT with read access to foxpress-design/anpl-staging |
| `RESEND_API_KEY` | Deploy notification emails |

---

## Troubleshooting

**`npm ci` fails with "Missing" or "Invalid" lock file errors**
The `package-lock.json` in the staging repo is out of date. In the staging repo, run:
```bash
npm install --package-lock-only
git add package-lock.json
git commit -m "regenerate package-lock.json for prod deploy"
```

**Pages are 310-byte redirect files instead of real HTML**
The production Build step must NOT set `KEYSTATIC_STORAGE_KIND=github`. If that variable is present, Astro's middleware auth gate redirects all pages during static pre-rendering. Remove it from the workflow's Build env.

**Deploy succeeds but changes aren't showing on anpl.org**
GitHub Pages CDN (Fastly) caches aggressively. Wait a few minutes, or check the `last-modified` header:
```bash
curl -sI https://anpl.org/programs/ | grep last-modified
```
