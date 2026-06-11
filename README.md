# SRE-Logsmith

> *Label it. Close it. Logged.*

Automatically generates separate log files from closed GitHub issues, organized by label.

---

## What is SRE-Logsmith?

**SRE-Logsmith** is a lightweight GitHub Action that automatically generates and maintains separate log files based on your issue labels. When an issue is closed, SRE-Logsmith reads its label and updates the corresponding log file — keeping your defect log, conflict log, and changelog always up to date without any manual work.

Built for teams who want a simple, label-driven way to track what went wrong, what was resolved, and what changed — all from within GitHub Issues itself.

---

## Features

- 🐛 **Defect Log** — tracks all closed issues labeled `defect`
- ⚠️ **Conflict Log** — tracks all closed issues labeled `conflict`
- 📋 **Changelog** — tracks all closed issues labeled `changelog`
- 👤 **Issue opener tracking** — shows who opened each issue with a clickable `@username`
- 🔗 **Linked issue IDs** — every entry links back to the original GitHub issue
- ⚠️ **Unlabeled issue warnings** — warns in the Actions log if a closed issue has no matching label
- ⚡ **Zero configuration** — works out of the box with default labels

---

## How It Works
1. Create an issue and assign it a label (`defect`, `conflict`, or `changelog`)
2. Close the issue once resolved
3. SRE-Logsmith automatically updates the corresponding log file
4. Changes are committed directly to your repository

---

## Usage

```yaml
name: Generate Logs

on:
  issues:
    types: [closed]
  workflow_dispatch:

jobs:
  update-logs:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write

    steps:
      - uses: actions/checkout@v4
      - uses: nidqija/SRE-Logsmith@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Log Output Format

Each log file entry follows this format:
---

## Labels

Create these labels in your repository under **Issues → Labels**:

| Label | Color | Description |
|---|---|---|
| `defect` | `#d73a4a` | Something isn't working |
| `conflict` | `#e4e669` | Conflicting changes |
| `changelog` | `#0075ca` | Track changes |

---

## Generated Files

| File | Label | Description |
|---|---|---|
| `DEFECT_LOG.md` | `defect` | Tracks bugs and broken functionality |
| `CONFLICT_LOG.md` | `conflict` | Tracks merge conflicts and resolution history |
| `CHANGELOG.md` | `changelog` | Tracks general changes and updates |

---

## Requirements

- GitHub Actions enabled on your repository
- Issues feature enabled
- Labels created before closing issues

---

## License

MIT © [nidqija](https://github.com/nidqija)