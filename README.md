# Dependabot Wolf 🐺

Automatically creates fix PRs for Dependabot security alerts that Dependabot can't fix itself.

## The Problem

Dependabot is great at detecting vulnerabilities, but sometimes it can't create a PR to fix them:

- **Transitive dependencies**: Vulnerability is deep in your dependency tree (e.g., `glob` through `rimraf`)
- **Complex upgrades**: Fix requires updating parent dependencies
- **Breaking changes**: Teams close PRs without understanding the security context

**Result:** Alerts pile up, vulnerabilities remain unfixed.

## The Solution

Dependabot Wolf automatically:
1. Finds security alerts without open PRs
2. Identifies which parent dependency needs updating
3. Creates a draft PR with the fix + full CVE context
4. Tags `@github-copilot workspace` for AI-assisted review

**Your team gets:** A complete PR with context, ready to review and merge.

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

## Example

This repo includes a real vulnerability to demonstrate how Wolf works.

**The scenario:**
```
rimraf@5.0.0
  └─> glob@10.4.5 (VULNERABLE - CVE-2025-64756)
```

**The problem:**
- `glob` has a command injection vulnerability
- It's a transitive dependency (not directly in `package.json`)
- Dependabot detects it but can't create a PR

**Wolf's fix:**
```diff
  "devDependencies": {
-   "rimraf": "5.0.0"
+   "rimraf": "^6.1.2"
  }
```

**Result:** rimraf@6.1.2 brings in glob@13.0.0 (patched) ✅

See [PR #31](https://github.com/engseclabs/dependabot-wolf/pull/31) for the actual fix Wolf created.

## Real-World Example

This pattern matches [playlab PR #2234](https://github.com/playlab-education/playlab/pull/2234):
- Vulnerability: `glob` via `@sentry/vite-plugin`
- Fix: Update `@sentry/vite-plugin` from 3.3.1 to 4.6.1
- Dependabot couldn't figure this out automatically

## How It Works

```mermaid
graph LR
    A[Daily Scan] --> B{Stuck Alerts?}
    B -->|Yes| C[Identify Parent Dependency]
    C --> D[Update to Latest Version]
    D --> E[Create Draft PR]
    E --> F[Tag Copilot for Review]
    B -->|No| G[Done]
```

**Technical details:**
1. Scans Dependabot alerts via GitHub API
2. Filters for alerts without open PRs
3. Analyzes `package-lock.json` to find parent dependencies
4. Updates parent to latest version (e.g., `rimraf@latest`)
5. Commits fix and creates draft PR with CVE context

## FAQ

**Q: Why not just use Dependabot?**
A: Dependabot can't always figure out complex transitive dependency fixes. Wolf handles these cases.

**Q: Will Wolf create PRs for everything?**
A: No, only for alerts that don't have an open Dependabot PR.

**Q: Is it safe to auto-merge Wolf PRs?**
A: No! These are draft PRs for review. Always test breaking changes, especially major version bumps.

**Q: Does this work with npm only?**
A: Currently yes. PRs welcome for other package managers!

**Q: What if Wolf can't fix an alert?**
A: It will skip it or create a PR explaining what needs manual attention.

## Contributing

PRs welcome! Especially for:
- Support for other package managers (pip, Maven, Go modules, etc.)
- Better heuristics for identifying parent dependencies
- Improved PR descriptions

## License

MIT

---

**Note:** This is an experimental tool. Always review PRs before merging, especially for major version updates.
