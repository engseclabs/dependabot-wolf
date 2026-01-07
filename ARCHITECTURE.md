# Dependabot Wolf - Architecture Plan

## What It Does
GitHub Action runs on schedule (daily). It finds stuck Dependabot alerts, creates a Copilot Workspace PR to fix them.

## How It Works

```
1. GitHub Action runs (cron: daily)
2. Script finds open Dependabot alerts with no PR
3. For each stuck alert:
   - Create Copilot Workspace PR with context
   - PR description includes CVE, package, versions
   - Copilot attempts to fix the dependency conflict
```

That's it.

## Implementation

**Single File:** `.github/workflows/dependabot-wolf.yml`

```yaml
name: Dependabot Wolf
on:
  schedule:
    - cron: '0 0 * * *'  # Daily
  workflow_dispatch:     # Manual trigger

jobs:
  fix-stuck-alerts:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Find stuck Dependabot alerts
        id: find-alerts
        uses: actions/github-script@v7
        with:
          script: |
            // Get all open Dependabot alerts
            const alerts = await github.rest.dependabot.listAlertsForRepo({
              owner: context.repo.owner,
              repo: context.repo.repo,
              state: 'open'
            });

            // Filter: no associated PR = stuck
            const stuck = [];
            for (const alert of alerts.data) {
              // Check if Dependabot created a PR for this
              const prs = await github.rest.pulls.list({
                owner: context.repo.owner,
                repo: context.repo.repo,
                head: `dependabot/${alert.dependency.package.ecosystem}/${alert.dependency.package.name}`,
                state: 'all'
              });

              if (prs.data.length === 0) {
                stuck.push(alert);
              }
            }

            return stuck;

      - name: Create Copilot PR for each stuck alert
        uses: actions/github-script@v7
        with:
          script: |
            const stuck = ${{ steps.find-alerts.outputs.result }};

            for (const alert of stuck) {
              // Create branch
              const branch = `wolf/fix-${alert.security_advisory.cve_id}`;

              // Get default branch ref
              const main = await github.rest.git.getRef({
                owner: context.repo.owner,
                repo: context.repo.repo,
                ref: 'heads/main'
              });

              // Create new branch
              await github.rest.git.createRef({
                owner: context.repo.owner,
                repo: context.repo.repo,
                ref: `refs/heads/${branch}`,
                sha: main.data.object.sha
              });

              // Create PR (Copilot will populate it)
              const pr = await github.rest.pulls.create({
                owner: context.repo.owner,
                repo: context.repo.repo,
                title: `Fix ${alert.security_advisory.cve_id}: Update ${alert.dependency.package.name}`,
                head: branch,
                base: 'main',
                draft: true,
                body: `
## 🤖 Dependabot Wolf - Automated Fix Attempt

**CVE:** ${alert.security_advisory.cve_id}
**Severity:** ${alert.security_advisory.severity}
**Package:** ${alert.dependency.package.name}
**Current Version:** ${alert.dependency.manifest_path} specifies a vulnerable version
**Patched Versions:** ${alert.security_advisory.vulnerabilities[0].patched_versions}

### Problem
Dependabot could not automatically create a PR to fix this vulnerability, likely due to dependency conflicts.

### Task for Copilot
Please analyze the dependency tree and propose a fix:

1. Identify why the direct upgrade failed (check transitive dependencies)
2. Update \`${alert.dependency.manifest_path}\` with necessary version changes
3. If breaking changes are needed, update calling code
4. Explain your changes

@github-copilot workspace
                `
              });

              console.log(\`Created PR #\${pr.data.number} for \${alert.security_advisory.cve_id}\`);
            }
```

## That's The Whole Thing

**Total Files:** 1
**Total Lines:** ~80
**Dependencies:** None (just `actions/github-script`)

## What I Removed

- ❌ No Go code
- ❌ No separate CLI
- ❌ No detector/handler split
- ❌ No JSON marshaling
- ❌ No prompt templates
- ❌ No complex state management
- ❌ No API client wrappers
- ❌ No logging infrastructure

## How Copilot Fixes It

When the PR is created with `@github-copilot workspace` in the description:
1. Engineer opens the PR
2. Copilot analyzes the repo and the request
3. Engineer says "fix this" in the Copilot chat
4. Copilot proposes changes
5. Engineer reviews and merges (or closes if bad)

## Configuration

**Environment Variables:**
- `GITHUB_TOKEN` - automatically provided by Actions

**Customization:**
Change the cron schedule or add filters in the script inline.

## Next Steps

1. Create `.github/workflows/dependabot-wolf.yml`
2. Test on a repo with stuck alerts
3. Done
