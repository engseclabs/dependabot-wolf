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
2. **Creates** an issue with the vulnerability details
3. **Calls** `@github-copilot workspace` to fix it
4. **Copilot** creates a PR with the fix

That's it. Wolf doesn't try to fix anything - just creates an issue and hands it to Copilot.

## Quick Start

### 1. Add Workflow File

Copy [`.github/workflows/dependabot-wolf.yml`](.github/workflows/dependabot-wolf.yml) to your repository.

The workflow runs:
- **Daily** at midnight (via cron)
- **Manually** via the Actions tab

### 2. Enable Dependabot Alerts

If not already enabled:
1. Go to **Settings** → **Code security and analysis**
2. Enable **Dependabot alerts**

Done! Wolf will start monitoring for stuck alerts.

## How It Works

Runs daily (or on demand):

1. Checks for Dependabot alerts without PRs
2. Creates an issue for each stuck alert
3. Tags `@github-copilot workspace` with the vulnerability details
4. Copilot responds to the issue and creates a PR with the fix

Wolf just creates the issue. Copilot does everything else.

## What Copilot Sees

Each issue includes this prompt for Copilot:

```
@github-copilot workspace

Fix this security vulnerability. Dependabot couldn't figure it out -
conflicting dependencies, complex dependency graph, whatever.

Your job:
1. Figure out what needs to be updated
2. Update the dependencies (package.json, lockfiles, whatever)
3. Fix any breaking changes in the code
4. Make it work

That's it. Fix it.
```

Plus all the vulnerability details (CVE, severity, patched version, advisory link).

## FAQ

**Q: Does Wolf actually fix the vulnerability?**
A: No. Wolf just creates an issue and tags Copilot. Copilot creates the PR with the fix.

**Q: Will Copilot actually fix it?**
A: Sometimes. Copilot is pretty good at dependency upgrades and handling breaking changes. But always review and test.

**Q: Is it safe to auto-merge?**
A: No. Always review. Copilot's PRs involve dependency updates and potential breaking changes.

**Q: Does this work with non-npm projects?**
A: Should work for any package manager (pip, Maven, Gradle, Go, etc.). Only tested with npm so far.

## License

MIT
