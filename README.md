# ANPL Production Site

This repository deploys the production version of the Asphodel-Norwood Public Library website to [anpl.org](https://anpl.org).

**Source of truth:** [foxpress-design/anpl-staging](https://github.com/foxpress-design/anpl-staging)

Content is managed via the CMS at staging.anpl.org/admin. When editors click "Push to Live", a GitHub Actions workflow clones the staging repo, builds the site, and deploys it here via GitHub Pages.

Do not edit files in this repository directly. All changes should be made in the staging repo.
