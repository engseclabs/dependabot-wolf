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
   - **Actions**: Read-only
   - **Contents**: Read-only
   - **Dependabot alerts**: Read-only ⚠️
   - **Issues**: Read and write ⚠️
   - **Pull requests**: Read and write
   - **Metadata**: Read-only (auto-selected)
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

Each stuck alert gets a simple issue with just what Copilot needs:

**For transitive dependencies:**
```
Fix CVE-2025-64756 by updating `glob` to version 10.5.0 or higher.

This is a transitive dependency in `package-lock.json`.
You'll need to update the parent dependency that brings it in.

Alert: https://github.com/.../dependabot/21
```

**For direct dependencies:**
```
Update `package-name` to version 1.2.3 or higher to fix CVE-2025-XXXXX.

Manifest: `package.json`

Alert: https://github.com/.../dependabot/21
```

## Invoking Copilot

The workflow automatically assigns issues to Copilot. Just click the "Assign to Copilot" button in the issue sidebar.

## FAQ

**Q: What does Wolf actually do?**
A: Finds stuck Dependabot alerts, creates an issue, tags Copilot. That's it.

**Q: Will Copilot fix it?**
A: Sometimes. Always review and test any PRs Copilot creates.

**Q: Does this work with non-npm projects?**
A: Should work for any package manager. Only tested with npm.

## License

MIT
