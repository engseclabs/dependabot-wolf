# Dependabot Wolf 🐺

Fixes security alerts when Dependabot says **"cannot update to a non-vulnerable version."**

## The Problem

Dependabot detects a vulnerability but says:

> **"Dependabot cannot update DEPENDENCY to a non-vulnerable version"**
>
> Cannot create a pull request to update the vulnerable dependency to a secure version without breaking other dependencies in the dependency graph.

This happens when:
- Your dependency graph is complex (common in npm/RubyGems)
- Fixing the vulnerability requires updating multiple dependencies
- Those updates include major/breaking version changes
- Dependabot can't figure out how to upgrade without breaking everything

**Result:** Alert sits there. You don't know what to update or how to handle breaking changes.

## The Solution

Wolf + Copilot fix it in one shot:
1. **Wolf** analyzes the dependency graph and updates what's needed (even major versions)
2. **Copilot** handles breaking changes (updates function calls, APIs, imports, etc.)
3. You get a complete, working PR ready to review

No more stuck alerts. No more manual dependency archaeology.

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

Done! Wolf will start monitoring your alerts.

## How It Works

1. Scans for transitive dependency alerts without open PRs
2. Identifies the parent package causing the issue
3. Updates parent to latest version (even major/breaking versions)
4. Tags `@github-copilot workspace` to handle breaking changes
5. Creates draft PR with complete fix

Copilot analyzes the changes and updates your code (function calls, API usage, etc.) to work with the new version.

## FAQ

**Q: Will Copilot actually fix breaking changes in my code?**
A: Yes! Wolf tags `@github-copilot workspace` which analyzes the dependency upgrades and updates your function calls, API usage, and any breaking changes. You get a complete, working fix in one PR.

**Q: Why not just use Dependabot?**
A: Dependabot says **"cannot update to a non-vulnerable version"** when the dependency graph is complex and fixing requires multiple major version upgrades. Wolf + Copilot handle these automatically.

**Q: Will Wolf create PRs for everything?**
A: No, only for alerts where Dependabot explicitly couldn't create a PR.

**Q: Is it safe to auto-merge Wolf PRs?**
A: No, always review! Even with Copilot's help, test the changes before merging.

**Q: Does this work with npm only?**
A: Currently yes. PRs welcome for other package managers!

## Contributing

PRs welcome! Especially for:
- Support for other package managers (pip, Maven, Go modules, etc.)
- Better heuristics for identifying parent dependencies
- Improved PR descriptions

## License

MIT

---

**Note:** This is an experimental tool focused on transitive dependency vulnerabilities. Always review PRs before merging, especially for major version updates.
