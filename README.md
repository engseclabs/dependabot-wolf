# Dependabot Wolf 🐺

When Dependabot gets stuck, Wolf shows up.

## The Problem

Dependabot detects a vulnerability but says:

> **"Dependabot cannot update DEPENDENCY to a non-vulnerable version"**

This happens with complex dependency graphs - fixing the vulnerability requires updating multiple dependencies with breaking changes, and Dependabot can't figure it out.

**Result:** Alert sits there. Nobody knows what to do.

## What Wolf Does

Wolf just finds stuck alerts and calls Copilot:

1. Finds Dependabot alerts without PRs
2. Creates one issue listing them all
3. Tags `@github-copilot workspace`

That's it. Copilot does everything else.

## Setup

### 1. Create a PAT

GITHUB_TOKEN can't read Dependabot alerts. You need a fine-grained PAT:

1. Go to **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
2. Click **Generate new token**
3. **Token name**: `dependabot-wolf`
4. **Repository access**: Select the repository
5. **Repository permissions**:
   - **Dependabot alerts**: Read-only access ⚠️ THIS IS REQUIRED
   - **Issues**: Read and write access
6. Click **Generate token** and copy it

### 2. Add Secret to Repository

1. Go to your repository **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. **Name**: `DEPENDABOT_PAT`
4. **Value**: Paste the token
5. Click **Add secret**

### 3. Add Workflow

Copy [`.github/workflows/dependabot-wolf.yml`](.github/workflows/dependabot-wolf.yml) to your repository.

### 4. Enable Dependabot

**Settings** → **Code security and analysis** → Enable **Dependabot alerts**

Done! The workflow runs daily, or trigger manually in Actions tab.

## How It Works

1. Gets all open Dependabot alerts
2. Filters for stuck alerts (no Dependabot PR exists)
3. For each stuck alert, creates an issue with complete vulnerability details
4. Attempts to assign the issue to Copilot
5. Copilot (if available) creates PRs to fix them

## What Gets Created

Each stuck alert gets a detailed issue with:

- **Full vulnerability description** including PoC and remediation steps
- **Vulnerable version range** and **patched version**
- **Whether it's a transitive or direct dependency**
- **Manifest path** (which lockfile)
- **Links to advisory and Dependabot alert**

Example: https://github.com/engseclabs/dependabot-wolf/issues/38

## Invoking Copilot

The workflow tries to assign issues to Copilot automatically. This requires:
- **GitHub Copilot Pro** or **Enterprise** subscription
- **Copilot coding agent** feature enabled

If auto-assignment doesn't work, manually assign issues to Copilot:
1. Open the issue
2. Click "Assignees" on the right
3. Search for and select "Copilot"

Copilot will then analyze the issue and create a PR.

## FAQ

**Q: What does Wolf actually do?**
A: Finds stuck Dependabot alerts, creates an issue, tags Copilot. That's it.

**Q: Will Copilot fix it?**
A: Sometimes. Always review and test any PRs Copilot creates.

**Q: Does this work with non-npm projects?**
A: Should work for any package manager. Only tested with npm.

## License

MIT
