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

GITHUB_TOKEN can't read Dependabot alerts. Create a fine-grained PAT:

1. Go to **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
2. Click **Generate new token**
3. **Token name**: `dependabot-wolf`
4. **Repository access**: Select the repository
5. **Repository permissions**:
   - **Actions**: Read and write
   - **Contents**: Read and write
   - **Dependabot alerts**: Read-only
   - **Issues**: Read and write
   - **Pull requests**: Read and write
   - **Metadata**: Read-only (auto-selected)
6. Click **Generate token** and copy it

**Why these permissions?** To assign issues to Copilot and let it create PRs to fix vulnerabilities. [More details](https://github.com/orgs/community/discussions/164267).

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

Each stuck alert gets an issue with the Dependabot data:

```
Fix CVE-2025-64756 in `glob`

**Severity:** high
**Package:** `glob`
**Ecosystem:** npm
**Manifest:** `package-lock.json`
**Vulnerable range:** >= 10.2.0, < 10.5.0
**Patched version:** 10.5.0

**Dependabot alert:** https://github.com/.../dependabot/21
**Advisory:** https://github.com/advisories/GHSA-...
```

Wolf just passes the Dependabot data to Copilot. Copilot figures out what to do.

## Invoking Copilot

The workflow automatically assigns issues to Copilot. Just click the "Assign to Copilot" button in the issue sidebar.

## FAQ

**Q: What does Wolf actually do?**
A: Finds stuck Dependabot alerts, creates an issue, assigns it to Copilot. That's it.

**Q: Will Copilot actually fix it?**
A: Maybe, maybe not. Copilot might not figure it out either - but at least it'll try harder than Dependabot. When Dependabot says "cannot update," it just gives up. Copilot will actually attempt to update dependencies, fix breaking changes, and create a PR. Worth a shot.

**Q: Should I trust Copilot's fixes?**
A: No. Always review and test any PRs Copilot creates. These involve dependency updates and potential breaking changes.

**Q: Does this work with non-npm projects?**
A: Should work for any package manager. Only tested with npm.

## License

MIT
