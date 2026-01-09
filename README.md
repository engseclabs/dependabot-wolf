# Dependabot Wolf 🐺

When Dependabot gets stuck, Wolf shows up.

## The Problem

Dependabot says **"cannot update to a non-vulnerable version"** when dependency graphs are too complex. Alert sits there. Nobody fixes it.

## What Wolf Does

1. Finds stuck Dependabot alerts (no PR exists)
2. Creates an issue with vulnerability details
3. Assigns it to Copilot

Copilot attempts the fix and creates a PR. Might work, might not - but at least it tries.

## Setup

### 1. Create a PAT

1. **Settings** → **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
2. **Generate new token**
3. **Repository access**: Select your repo
4. **Permissions**:
   - Actions: Read and write
   - Contents: Read and write
   - Dependabot alerts: Read-only
   - Issues: Read and write
   - Pull requests: Read and write
5. Copy token

[Why these permissions?](https://github.com/orgs/community/discussions/164267)

### 2. Add Secret

1. **Settings** → **Secrets and variables** → **Actions**
2. **New repository secret**
3. Name: `DEPENDABOT_PAT`, Value: your token

### 3. Add Workflow

Copy [`.github/workflows/dependabot-wolf.yml`](.github/workflows/dependabot-wolf.yml) to your repo.

### 4. Enable Dependabot

**Settings** → **Code security and analysis** → Enable **Dependabot alerts**

Done. Runs daily at midnight, or manually via Actions tab.

## Example Issue

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

Assigned to Copilot. Copilot creates a PR to fix it.

## Notes

- Always review PRs before merging
- Tested with npm, should work with other package managers

## License

MIT
