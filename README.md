# Dependabot Wolf 🐺

Automatically fixes transitive dependency vulnerabilities that Dependabot can't handle.

## The Problem

Dependabot detects a vulnerability deep in your dependency tree but says **"cannot create a PR"** because:
- The vulnerable package isn't in your `package.json` (it's transitive)
- Fixing it requires a major version bump of the parent dependency
- That major version may have breaking changes

**Result:** Alert sits there. Your team doesn't know which package to update or how to handle breaking changes.

## The Solution

Wolf + Copilot fix it automatically:
1. **Wolf** identifies the parent dependency and updates it (even major versions)
2. **Copilot** gets tagged to handle any breaking changes (update function calls, APIs, etc.)
3. You get a draft PR with the complete fix—one-shot, ready to review

No more "cannot create a PR" alerts blocking your security fixes.

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
A: Yes! Wolf tags `@github-copilot workspace` which analyzes the dependency upgrade and updates your function calls, API usage, and any breaking changes. You get a complete, working fix in one PR.

**Q: Why not just use Dependabot?**
A: Dependabot says "cannot create a PR" for transitive dependencies that require major version upgrades. Wolf + Copilot handle these automatically.

**Q: Will Wolf create PRs for everything?**
A: No, only transitive dependency alerts where Dependabot couldn't create a PR.

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
