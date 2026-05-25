# Dependabot Wolf 🐺

When Dependabot gets stuck, Wolf shows up.

<img width="959" height="399" alt="image" src="https://github.com/user-attachments/assets/d000fe9e-c187-40cb-b2aa-77611bd6d45e" />

> [!WARNING]
> **Deprecated as of April 2026.** GitHub now supports [assigning Dependabot alerts directly to AI agents](https://github.blog/changelog/2026-04-07-dependabot-alerts-are-now-assignable-to-ai-agents-for-remediation/) (Copilot, Claude, or Codex) from the alert detail page. The agent opens a draft PR directly — no issue, no PAT, no workflow needed. It also covers the "no patched version available" case Wolf was built for. Use that instead.
>
> The workflow has been removed; the README is kept for historical context.

## The Problem

Dependabot says **"cannot update to a non-vulnerable version"** when dependency graphs are too complex. Alert sits there. Nobody fixes it.

## What Wolf Does

1. Finds stuck Dependabot alerts (no PR exists)
2. Creates an issue with vulnerability details
3. Assigns it to Copilot

Copilot attempts the fix and creates a PR. Might work, might not - but at least it tries.

## Notes

- Always review PRs before merging
- Tested with npm and Go, should work with other package managers
- "If I'm curt with you it's because time is a factor." - Mr. Wolf

## License

MIT
