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

1. Finds Dependabot alerts without PRs
2. Creates one issue with all stuck alerts
3. Tags `@github-copilot workspace fix these vulnerabilities`
4. Copilot creates PRs

Wolf does nothing else. Just finds stuck alerts and calls Copilot.

## What Copilot Sees

One issue listing all stuck alerts:

```
Dependabot got stuck. Fix it.

- **CVE-2025-64756** (high): `glob` - https://github.com/advisories/GHSA-...
- **CVE-2024-XXXXX** (medium): `package-name` - https://github.com/advisories/...

@github-copilot workspace fix these vulnerabilities
```

That's it. Copilot figures out the rest.

## FAQ

**Q: What does Wolf actually do?**
A: Finds stuck Dependabot alerts, creates an issue, tags Copilot. That's it.

**Q: Will Copilot fix it?**
A: Sometimes. Always review and test any PRs Copilot creates.

**Q: Does this work with non-npm projects?**
A: Should work for any package manager. Only tested with npm.

## License

MIT
