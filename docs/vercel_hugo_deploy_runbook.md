# Vercel Hugo Deploy Runbook For FCAJ Workshop

Last updated: 2026-07-09

This runbook explains how to manually deploy the FCAJ Hugo workshop site to
Vercel. The goal is not only to publish the site, but also to understand each
step so the deployment can be repeated later without guessing.

This document is an internal technical guide. It is stored under `docs/`, so it
is not rendered as a public Hugo workshop page by default.

## 1. Deployment Pipeline

The workshop is a Hugo static site. The deployment flow is:

```text
Markdown content + Hugo theme
-> Hugo build
-> public/
-> GitHub repository
-> Vercel build
-> Production URL for FCAJ review
```

What each part means:

| Step | Meaning |
| --- | --- |
| Markdown content | Workshop pages under `content/`. |
| Hugo theme | The site layout under `themes/hugo-theme-learn`. |
| Hugo build | Converts Markdown and static assets into HTML/CSS/JS. |
| `public/` | Generated static output. Do not edit this manually. |
| GitHub | Source repository that Vercel reads from. |
| Vercel | Builds the site and serves it on a public URL. |

## 2. Working Directory

Use the workshop repository, not the SOC AI application repository:

```bash
cd /home/tran_thien/fcaj_project/fcj-workshop-ThienIT84
```

Confirm that this is a Hugo site:

```bash
ls
```

Expected important files and folders:

```text
config.toml
content/
static/
themes/
docs/
.gitmodules
```

## 3. Preflight Checks

Run these before building or committing anything:

```bash
hugo version
git status --short
git submodule status
```

Why these checks matter:

| Command | Why it matters |
| --- | --- |
| `hugo version` | Confirms Hugo is installed and shows which version builds locally. |
| `git status --short` | Shows which files are modified or untracked before commit. |
| `git submodule status` | Confirms the Hugo theme submodule is available. |

The local build has been validated with Hugo:

```text
0.123.7 extended
```

Use the same Hugo version on Vercel unless there is a reason to upgrade.

## 4. Sensitive File Warning

Do not run this blindly:

```bash
git add .
```

This repository currently contains local/raw files that should not be published
unless they are intentionally reviewed and masked first. Examples:

```text
static/picture aws/
private-evidence/
5.6-Cleanup-content.tar.gz
crop_images.py
raw screenshots
temporary archives
```

Safe rule:

```text
Only add the exact files and folders that are ready for public review.
```

For cleanup images, prefer the curated publish-ready folder:

```text
static/images/5-Workshop/5.6-Cleanup/socai-cleanup/
```

Before commit, open the images and confirm that sensitive data is masked:

- AWS account ID
- IAM user or email
- public IP address if not meant to be shown
- ARNs that should stay private
- secrets or credentials
- private evidence paths

## 5. Local Preview

Use local preview when you want to read the site in a browser before deploying:

```bash
hugo server -D --bind 127.0.0.1 --baseURL http://localhost:1313/
```

Open:

```text
http://localhost:1313/
```

Useful pages to check:

```text
http://localhost:1313/
http://localhost:1313/vi/
http://localhost:1313/5-workshop/5.6-cleanup/
http://localhost:1313/vi/5-workshop/5.6-cleanup/
```

What to verify:

- The menu loads.
- English and Vietnamese pages load.
- Cleanup screenshots display correctly.
- Search still works.
- No page shows raw sensitive information.

Stop the local server with `Ctrl+C` when finished.

## 6. Local Production Build Check

Local preview is useful, but production deployment uses a generated static
folder. Build it once before pushing:

```bash
HUGO_BASEURL="https://example.vercel.app/"
hugo --minify --baseURL "$HUGO_BASEURL" --destination /tmp/fcj-vercel-check
```

Why this matters:

- It proves Hugo can generate the production output.
- It catches broken Markdown, missing theme files, or missing images.
- It lets you inspect the generated files without touching the repository's
  tracked `public/` folder.

Expected result: Hugo prints a build summary and exits without error.

## 7. Check Generated Output

Confirm that the cleanup pages exist in the generated output:

```bash
find /tmp/fcj-vercel-check -path '*5.6-cleanup*' -type f
```

Confirm that the old sample domain is not embedded in generated HTML:

```bash
rg "workshop-sample.fcjuni.com" /tmp/fcj-vercel-check
```

Expected result:

```text
No output
```

If this command returns matches, the site is still generating links to the old
sample domain. Fix this before deploying by using the correct `HUGO_BASEURL`.

Check that cleanup images were emitted:

```bash
find /tmp/fcj-vercel-check/images/5-Workshop/5.6-Cleanup/socai-cleanup -type f | wc -l
```

Expected result: a non-zero number. In the current cleanup content, the expected
count is 36 images.

## 8. Safe Git Commit Flow

First inspect changes:

```bash
git status --short
```

Add only reviewed content. Example:

```bash
git add content/5-Workshop/5.6-Cleanup/_index.md
git add content/5-Workshop/5.6-Cleanup/_index.vi.md
git add static/images/5-Workshop/5.6-Cleanup/socai-cleanup/
git add docs/vercel_hugo_deploy_runbook.md
```

If you intentionally changed proposal or other workshop pages, add those files
explicitly too. Do not add unrelated raw files.

Review the staged list:

```bash
git status --short
git diff --cached --stat
```

Commit:

```bash
git commit -m "docs: add cleanup workshop content and deploy guide"
```

Push to GitHub:

```bash
git push origin main
```

If your branch is not `main`, confirm the active branch:

```bash
git branch --show-current
```

Then push that branch or merge it into `main`, depending on how the GitHub repo
is managed.

## 9. Vercel Project Setup

Open the Vercel Dashboard and import the GitHub repository.

Use these settings:

| Setting | Value |
| --- | --- |
| Framework Preset | `Other` or `Hugo` |
| Root Directory | Repository root |
| Install Command | `git submodule update --init --recursive` |
| Build Command | `hugo --minify --baseURL "$HUGO_BASEURL"` |
| Output Directory | `public` |

Set environment variables:

```text
HUGO_VERSION=0.123.7
HUGO_BASEURL=https://<vercel-production-domain>/
```

Important notes:

- Keep the trailing slash in `HUGO_BASEURL`.
- Use the final production domain, not the old sample domain.
- If you add a custom domain later, update `HUGO_BASEURL` and redeploy.

## 10. First Deploy Flow

For the first deployment, you may not know the Vercel production domain yet.
Use this flow:

1. Import the GitHub repository into Vercel.
2. Use the build settings above.
3. Deploy once.
4. Copy the generated production domain, for example:

   ```text
   https://fcj-workshop-thienit84.vercel.app/
   ```

5. Go to Vercel Project Settings -> Environment Variables.
6. Set:

   ```text
   HUGO_BASEURL=https://fcj-workshop-thienit84.vercel.app/
   ```

7. Redeploy the latest production deployment.

This second deploy is important because Hugo uses `baseURL` when generating
language switch links, canonical links, and some absolute URLs.

## 11. Post-Deploy Verification

Open these pages from the Vercel production URL:

```text
/
/vi/
/5-workshop/5.6-cleanup/
/vi/5-workshop/5.6-cleanup/
```

Verify:

- [ ] Home page loads.
- [ ] Vietnamese home page loads.
- [ ] Cleanup English page loads.
- [ ] Cleanup Vietnamese page loads.
- [ ] Cleanup screenshots display.
- [ ] Search works.
- [ ] Language switch keeps you on the Vercel domain.
- [ ] No link sends reviewers to `workshop-sample.fcjuni.com`.
- [ ] No raw evidence folder is accessible from the public site.
- [ ] No unmasked sensitive screenshot is visible.

Optional command-line checks:

```bash
curl -I https://<vercel-production-domain>/
curl -I https://<vercel-production-domain>/vi/
curl -I https://<vercel-production-domain>/5-workshop/5.6-cleanup/
curl -I https://<vercel-production-domain>/vi/5-workshop/5.6-cleanup/
```

Expected status:

```text
HTTP 200
```

## 12. What To Send To FCAJ

Send a short package:

```text
Production site: https://<vercel-production-domain>/
GitHub repository: https://github.com/<owner>/<repo>
Suggested review path: /5-workshop/5.6-cleanup/
Notes: Hugo static workshop deployed on Vercel from GitHub.
```

If the reviewers should start in Vietnamese, send:

```text
Vietnamese entry: https://<vercel-production-domain>/vi/
```

## 13. Troubleshooting

### Theme missing or layout broken

Symptom:

```text
module "hugo-theme-learn" not found
```

Cause: the theme submodule was not initialized.

Fix locally:

```bash
git submodule update --init --recursive
```

Fix on Vercel: confirm Install Command is:

```bash
git submodule update --init --recursive
```

### English/Vietnamese switch goes to the wrong domain

Symptom: clicking language switch opens `workshop-sample.fcjuni.com`.

Cause: `HUGO_BASEURL` is missing or incorrect during build.

Fix:

```text
Set HUGO_BASEURL=https://<vercel-production-domain>/
Redeploy production.
```

### Images do not show

Possible causes:

- The image file was not committed.
- The Markdown path is wrong.
- The file exists only in a local raw folder.
- The image name has a typo or case mismatch.

Check locally:

```bash
find static/images/5-Workshop/5.6-Cleanup/socai-cleanup -type f | sort
rg "socai-cleanup" content/5-Workshop/5.6-Cleanup
```

Then commit the missing publish-ready image if it is safe to publish.

### Vercel build fails because of Hugo version

Symptom: Vercel build works differently from local build.

Fix: set this environment variable in Vercel:

```text
HUGO_VERSION=0.123.7
```

Redeploy after saving the environment variable.

### Deploy still shows old content

Possible causes:

- You pushed to a different branch.
- Vercel is connected to another branch.
- Vercel reused build cache.
- The file was not committed.

Checks:

```bash
git branch --show-current
git log --oneline -5
git status --short
```

Fix on Vercel:

```text
Project -> Deployments -> Redeploy -> Redeploy without Build Cache
```

### Sensitive file was published

If a sensitive file appears on the public site:

1. Remove it from the Hugo publish path, especially under `static/`.
2. Commit and push the removal.
3. Redeploy on Vercel.
4. If the information is truly sensitive, rotate the exposed credential or
   secret. Removing the screenshot is not enough if a real secret was visible.

## 14. Final Readiness Checklist

Before sending the link to FCAJ:

- [ ] Local Hugo build passes.
- [ ] Vercel production deployment passes.
- [ ] `HUGO_BASEURL` uses the final Vercel or custom domain.
- [ ] EN and VI pages load.
- [ ] Cleanup pages load with images.
- [ ] Search works.
- [ ] No old sample domain appears in generated links.
- [ ] No raw or unmasked evidence is public.
- [ ] GitHub repository contains only reviewed deploy-ready files.

Once this checklist passes, the site is ready to submit for review.
