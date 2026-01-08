# Dependabot Wolf 🐺

When Dependabot gets stuck, Wolf shows up.

## The Problem

Dependabot detects a vulnerability but says:

> **"Dependabot cannot update DEPENDENCY to a non-vulnerable version"**

This happens with complex dependency graphs - fixing the vulnerability requires updating multiple dependencies with breaking changes, and Dependabot can't figure it out.

**Result:** Alert sits there. Nobody knows what to do.

## What Wolf Does

Wolf handles it:

1. **Finds** the stuck alerts (no Dependabot PR exists)
2. **Creates** a PR with the vulnerability details
3. **Calls** `@github-copilot workspace` to fix it

That's it. Wolf doesn't try to fix anything - just hands the problem to Copilot and gets out of the way.

## Quick Start

### 1. Create a Personal Access Token (PAT)

GitHub's default `GITHUB_TOKEN` can't read Dependabot alerts, so you need a PAT:

1. Go to **GitHub Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
2. Click **Generate new token**
3. Configure:
   - **Repository access**: Select your repository
   - **Permissions**:
     - Contents: **Read and write**
     - Pull requests: **Read and write**
     - Security events: **Read-only**
4. Generate and copy the token

### 2. Add PAT as Repository Secret

1. Go to your repo's **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `DEPENDABOT_PAT`
4. Value: Paste your token
5. Click **Add secret**

### 3. Add Workflow File

Copy [`.github/workflows/dependabot-wolf.yml`](.github/workflows/dependabot-wolf.yml) to your repository.

The workflow runs:
- **Daily** at midnight (via cron)
- **Manually** via the Actions tab

### 4. Enable Dependabot Alerts

If not already enabled:
1. Go to **Settings** → **Code security and analysis**
2. Enable **Dependabot alerts**

Done! Wolf will start monitoring for stuck alerts.

## How It Works

Runs daily (or on demand):

1. Checks for Dependabot alerts without PRs
2. Creates a branch + draft PR for each stuck alert
3. Tags `@github-copilot workspace` with the vulnerability details
4. Copilot figures out what to update and fixes any breaking changes

Wolf just creates the PR. Copilot does the work.

## FAQ

**Q: Does Wolf actually fix the vulnerability?**
A: No. Wolf just creates a PR and hands it to Copilot. Copilot figures out what needs updating and fixes the code.

**Q: Will Copilot actually fix it?**
A: Sometimes. Copilot is pretty good at dependency upgrades and handling breaking changes. But always review and test.

**Q: Is it safe to auto-merge?**
A: No. Always review. These PRs involve dependency updates and potential breaking changes.

**Q: Does this work with non-npm projects?**
A: Currently no - only npm. PRs welcome for other package managers!

## Contributing

PRs welcome for:
- Other package managers (pip, Maven, Gradle, Go, etc.)
- Better Copilot prompts
- Improvements to the workflow

## License

MIT

---

**Note:** Wolf doesn't fix anything - it just shows up when Dependabot is stuck and hands the problem to Copilot. Always review and test before merging.
