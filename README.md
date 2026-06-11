# SRE-Logsmith
> *Label it. Close it. Logged.*

Automatically generates separate log files from closed GitHub issues, organized by label.

---

## What is SRE-Logsmith?

**SRE-Logsmith** is a lightweight GitHub Action that automatically generates and maintains separate log files based on your issue labels. When an issue is closed, SRE-Logsmith reads its label and updates the corresponding log file — keeping your defect log, conflict log, and changelog always up to date without any manual work.

Built for teams who want a simple, label-driven way to track what went wrong, what was resolved, and what changed — all from within GitHub Issues itself.

---

## Features

- 🐛 **Defect Log** — tracks all closed issues labeled `content defect`, `documentation defect`, or `agreement defect`
- ⚠️ **Conflict Log** — tracks all closed issues labeled `conflict`
- 📋 **Changelog** — tracks all closed issues labeled `changelog`
- 👤 **Issue opener tracking** — shows who opened each issue with a clickable `@username`
- 🔗 **Linked issue IDs** — every entry links back to the original GitHub issue
- ⚠️ **Unlabeled issue warnings** — warns in the Actions log if a closed issue has no matching label
- ⚡ **Zero configuration** — works out of the box with default labels

---

## How It Works

1. Create an issue and assign it a label (`content defect`, `documentation defect`, `agreement defect`, `conflict`, or `changelog`)
2. Close the issue once resolved
3. SRE-Logsmith automatically updates the corresponding log file
4. Changes are committed directly to your repository

---

## Setup

### Step 1 — Add the Workflow

1. Go to your newly created repository → **Actions** tab
2. Click **set up a workflow yourself**
3. Rename the file to `changelog.yml`
4. Clear the default content and paste the following workflow:

```yaml
name: Generate Changelog
on:
  issues:
    types: [closed]
  workflow_dispatch:
jobs:
  update-changelog:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate Defect Log
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          REPO: ${{ github.repository }}
        run: |
          echo "# 🐛 Defect Log" > DEFECT_LOG.md
          echo "" >> DEFECT_LOG.md

          echo "## 📝 Content Defects" >> DEFECT_LOG.md
          echo "" >> DEFECT_LOG.md
          CONTENT=$(gh issue list \
            --repo "$REPO" \
            --state closed \
            --label "content defect" \
            --limit 100 \
            --json number,title,author,closedAt \
            | jq -r '.[] | "- [#\(.number)](https://github.com/'"$REPO"'/issues/\(.number)) \(.title) — opened by @\(.author.login) _(closed \(.closedAt[:10]))_"')
          if [ -z "$CONTENT" ]; then
            echo "_No content defects logged yet._" >> DEFECT_LOG.md
          else
            echo "$CONTENT" >> DEFECT_LOG.md
          fi

          echo "" >> DEFECT_LOG.md

          echo "## 📄 Documentation Defects" >> DEFECT_LOG.md
          echo "" >> DEFECT_LOG.md
          DOCS=$(gh issue list \
            --repo "$REPO" \
            --state closed \
            --label "documentation defect" \
            --limit 100 \
            --json number,title,author,closedAt \
            | jq -r '.[] | "- [#\(.number)](https://github.com/'"$REPO"'/issues/\(.number)) \(.title) — opened by @\(.author.login) _(closed \(.closedAt[:10]))_"')
          if [ -z "$DOCS" ]; then
            echo "_No documentation defects logged yet._" >> DEFECT_LOG.md
          else
            echo "$DOCS" >> DEFECT_LOG.md
          fi

          echo "" >> DEFECT_LOG.md

          echo "## 🤝 Agreement Defects" >> DEFECT_LOG.md
          echo "" >> DEFECT_LOG.md
          AGREEMENT=$(gh issue list \
            --repo "$REPO" \
            --state closed \
            --label "agreement defect" \
            --limit 100 \
            --json number,title,author,closedAt \
            | jq -r '.[] | "- [#\(.number)](https://github.com/'"$REPO"'/issues/\(.number)) \(.title) — opened by @\(.author.login) _(closed \(.closedAt[:10]))_"')
          if [ -z "$AGREEMENT" ]; then
            echo "_No agreement defects logged yet._" >> DEFECT_LOG.md
          else
            echo "$AGREEMENT" >> DEFECT_LOG.md
          fi

      - name: Generate Conflict Log
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          REPO: ${{ github.repository }}
        run: |
          echo "# ⚠️ Conflict Log" > CONFLICT_LOG.md
          echo "" >> CONFLICT_LOG.md
          CONFLICTS=$(gh issue list \
            --repo "$REPO" \
            --state closed \
            --label "conflict" \
            --limit 100 \
            --json number,title,author,closedAt \
            | jq -r '.[] | "- [#\(.number)](https://github.com/'"$REPO"'/issues/\(.number)) \(.title) — opened by @\(.author.login) _(closed \(.closedAt[:10]))_"')
          if [ -z "$CONFLICTS" ]; then
            echo "_No conflicts logged yet._" >> CONFLICT_LOG.md
          else
            echo "$CONFLICTS" >> CONFLICT_LOG.md
          fi

      - name: Generate Changelog
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          REPO: ${{ github.repository }}
        run: |
          echo "# 📋 Changelog" > CHANGELOG.md
          echo "" >> CHANGELOG.md
          CHANGES=$(gh issue list \
            --repo "$REPO" \
            --state closed \
            --label "changelog" \
            --limit 100 \
            --json number,title,author,closedAt \
            | jq -r '.[] | "- [#\(.number)](https://github.com/'"$REPO"'/issues/\(.number)) \(.title) — opened by @\(.author.login) _(closed \(.closedAt[:10]))_"')
          if [ -z "$CHANGES" ]; then
            echo "_No changes logged yet._" >> CHANGELOG.md
          else
            echo "$CHANGES" >> CHANGELOG.md
          fi

      - name: Warn on Unlabeled Issue
        if: github.event_name == 'issues'
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          REPO: ${{ github.repository }}
          ISSUE_NUMBER: ${{ github.event.issue.number }}
        run: |
          LABELS=$(gh issue view "$ISSUE_NUMBER" \
            --repo "$REPO" \
            --json labels \
            | jq -r '[.labels[].name] | @csv')
          if [[ "$LABELS" != *"content defect"* && "$LABELS" != *"documentation defect"* && "$LABELS" != *"agreement defect"* && "$LABELS" != *"conflict"* && "$LABELS" != *"changelog"* ]]; then
            echo "⚠️ Issue #$ISSUE_NUMBER was closed without a recognized label — it won't appear in any log."
          fi

      - name: Commit and Push Changes
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          git add DEFECT_LOG.md CONFLICT_LOG.md CHANGELOG.md
          git commit -m "docs(changelog): auto-update logs [skip ci]" || echo "No changes to commit"
          git push
```

5. Click **Commit changes** → **Commit changes** to save

---

### Step 2 — Create Labels

Before closing any issues, create the labels in your repository:

1. Go to your repository → **Issues** tab
2. Click **Labels** next to the search bar
3. Click **New label** and create each of the following:

| Label | Color | Description |
|---|---|---|
| `content defect` | `#d73a4a` | Issues related to content errors |
| `documentation defect` | `#e11d48` | Issues related to documentation errors |
| `agreement defect` | `#be123c` | Issues related to agreement or SLA mismatches |
| `conflict` | `#e4e669` | Conflicting changes |
| `changelog` | `#0075ca` | Track changes |

4. Repeat for all labels

> ⚠️ Labels must be created before closing any issues, otherwise the issue won't appear in any log.

---

### Step 3 — Test It

1. Go to your repository → **Actions** tab
2. Find **Generate Changelog** workflow
3. Click **Run workflow** → **Run workflow** to trigger it manually
4. Check that `DEFECT_LOG.md`, `CONFLICT_LOG.md`, and `CHANGELOG.md` were created in your repository

---

### Step 4 — Start Issuing

1. Go to **Issues** → **New issue**
2. Fill in the title and description
3. On the right panel, click **Labels** and pick one of the labels
4. Click **Submit new issue**
5. Once resolved, click **Close issue**
6. The workflow will trigger automatically and update the corresponding log file

---

## Log Output Format

Each log file entry follows this format:

```markdown
- [#42](https://github.com/org/repo/issues/42) Fix login crash — opened by @alice _(closed 2024-06-10)_
```

| Field | Example | Description |
|---|---|---|
| Issue ID | `#42` | Clickable link to the original issue |
| Title | `Fix login crash` | The issue title |
| Opener | `@alice` | GitHub username of who opened the issue |
| Closed date | `2024-06-10` | Date the issue was closed (YYYY-MM-DD) |

---

## Generated Files

| File | Labels | Description |
|---|---|---|
| `DEFECT_LOG.md` | `content defect`, `documentation defect`, `agreement defect` | Tracks all defect types in separate sections |
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
