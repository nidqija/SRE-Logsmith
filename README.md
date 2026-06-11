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

### Step 1 — Create Labels

Before using SRE-Logsmith, create the labels in your repository:

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

### Step 2 — Add the Workflow

1. In your repository, create the following folder path if it doesn't exist:
