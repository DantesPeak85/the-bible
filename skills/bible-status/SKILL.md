---
name: bible-status
description: Quick staleness check — how current is the Bible? Shows commits since last update, affected sections, and staleness level. Read-only, no edits. Use when asking "is the bible current?", "bible status", or before deciding whether to run /update-bible.
---

# Bible Status Skill

Reports how stale the Bible is relative to recent codebase changes. Entirely read-only — never edits files or creates commits.

## Trigger Phrases

- `/bible-status`
- "is the bible current?"
- "bible status"
- "how stale is the bible?"
- "do I need to update the bible?"

## Workflow

### Step 1: Load Config

Read `.bible.yaml` from the project root.

If not found, stop with:

> No Bible configured for this project. Run `/init-bible` to get started.

Parse the YAML and extract:
- `bible_path` — relative path to the Bible file
- `last_updated` — ISO date of the last Bible update
- `sections` — the section-to-paths mapping array
- `ignore_paths` — globs to exclude from analysis

### Step 2: Parse Bible Version

Read the Bible file at `bible_path`. Extract the version number and date from the header.

Look for a line matching either of these patterns:
```
> **Version:** X.Y · **Date:** Month DD, YYYY
```
or a YAML front-matter block with `version` and `date` fields.

If the file does not exist, stop with:

> Bible file not found at `[bible_path]`. Check `.bible.yaml` configuration.

### Step 3: Count Changes Since Last Update

Run:

```bash
git log --oneline --since="{last_updated}" -- ':!{bible_path}' | wc -l
```

This counts all commits since the Bible was last updated, excluding changes to the Bible file itself. Also capture the date range:

```bash
git log -1 --format="%ai" --since="{last_updated}" -- ':!{bible_path}'
```

Calculate the age in days from `last_updated` to today.

### Step 4: Quick Category Scan

For the **10 most recent commits** (not all commits — keep this fast), run:

```bash
git log -10 --format="%H" --since="{last_updated}" -- ':!{bible_path}'
```

For each commit hash, get the changed files:

```bash
git show --stat --format="" {hash}
```

Match each changed file against the `paths` globs defined in each `sections[]` entry from `.bible.yaml`. Increment a counter per section.

Files matching `ignore_paths` patterns are skipped entirely.

If a file matches multiple sections, count it in each.

### Step 5: Determine Staleness Level

Apply these thresholds using both commit count AND age in days. The **higher** staleness wins:

| Level | Commits | OR | Age |
|-------|---------|----|-----|
| FRESH | 0-5 | AND | < 7 days |
| MODERATE | 6-20 | OR | 7-21 days |
| HIGH | 21-50 | OR | 21-60 days |
| CRITICAL | 50+ | OR | 60+ days |

Logic:
- FRESH requires BOTH conditions (low commits AND recent)
- MODERATE, HIGH, CRITICAL trigger if EITHER condition is met
- Always take the worst (highest staleness) that applies

### Step 6: Output Report

Print:

```
Bible Status
============
File: {bible_path}
Version: {version} ({date})
Age: {N} days

Changes since last update: {total_commits} commits
  {Section Name}: {count} commits
  {Section Name}: {count} commits
  (uncategorized): {count} commits

Staleness: {FRESH | MODERATE | HIGH | CRITICAL}
```

Rules for the report:
- Only list sections with 1+ commits
- Sort sections by commit count descending
- "uncategorized" captures commits that matched no section's paths
- If total commits is 0, skip the per-section breakdown

If staleness is MODERATE or above, append:

> Run `/update-bible` to bring it current.

If staleness is CRITICAL, append:

> The Bible is significantly out of date. Run `/update-bible` with priority.

## What This Skill Does NOT Do

- Does not edit any files
- Does not create commits
- Does not perform deep diff analysis (that is what `/update-bible` triage does)
- Does not modify `.bible.yaml`
- Does not read file contents beyond the Bible header — only git metadata

## Dependencies

- `.bible.yaml` in project root (see `references/bible-yaml-schema.md`)
- Git history available in the working directory
- The Bible file exists at the configured path

## Performance

This skill should complete in under 10 seconds. It inspects at most 10 commits and performs only `git log` and `git show --stat` operations. No file content reads beyond the Bible header line.
