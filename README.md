# Dependabot Wolf 🐺

**Experiment:** Can Copilot do a better job than Dependabot at fixing complex dependency vulnerabilities?

## The Problem

Dependabot detects a vulnerability but says:

> **"Dependabot cannot update DEPENDENCY to a non-vulnerable version"**
>
> Cannot create a pull request to update the vulnerable dependency to a secure version without breaking other dependencies in the dependency graph.

This happens when fixing requires updating multiple dependencies with major/breaking version changes.

## The Experiment

When Dependabot fails, **Wolf tries a simple approach:**

1. **Update the dependencies** (ignore Dependabot's constraints, just update what's needed)
2. **Ask Copilot to fix it** (tag `@github-copilot workspace` to handle breaking changes)
3. **See if it works** (maybe Copilot can resolve what Dependabot couldn't)

That's it. Let Copilot take a crack at the problem when Dependabot's native resolution fails.

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

1. Finds alerts where Dependabot couldn't create a PR
2. Updates the dependencies (simple approach: update to latest)
3. Tags `@github-copilot workspace` with instructions to fix breaking changes
4. Creates draft PR

Then you see if Copilot did better than Dependabot's native resolution.

## FAQ

**Q: Is this better than Dependabot?**
A: Maybe! That's the experiment. Dependabot's native resolution is conservative. Sometimes just updating and letting Copilot fix breaking changes works when Dependabot gives up.

**Q: Will Copilot actually fix my code?**
A: Sometimes. Copilot gets detailed instructions to analyze CHANGELOGs, find breaking changes, and update your code. Results vary.

**Q: Is it safe to auto-merge?**
A: No! This is experimental. Always review and test before merging.

**Q: Does this work with npm only?**
A: Currently yes. PRs welcome for other package managers!

## Contributing

PRs welcome! Especially for:
- Support for other package managers (pip, Maven, Go modules, etc.)
- Better prompts for Copilot
- Smarter dependency update strategies

## License

MIT

---

**Note:** This is an experiment to see if Copilot can do better than Dependabot's native dependency resolution. Always review and test PRs before merging.
