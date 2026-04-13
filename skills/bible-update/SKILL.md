---
name: update-bible
description: Update the project Bible with all code changes since its last version date. Three-phase workflow — triage (shows what changed), update (edits affected sections), CLAUDE.md sync. Use when saying "update the bible", "bible is stale", or after merging features.
---

## When to Use

- After completing a feature branch and merging to main
- After significant code changes (new endpoints, DB migrations, new UI pages)
- Periodically to keep documentation current
- When the user says "update the bible", "sync the bible", "bible is stale"
- After `/bible-status` reports staleness

## Phase 1: Triage (Always Runs)

### Step 1: Load Config

Read `.bible.yaml` from project root. If not found, tell the user to run `/init-bible` first and **stop**.

### Step 2: Parse Bible Version Date

Read the first 20 lines of the Bible file (path from `bible_path` in config) and extract the version header:

```
> **Version:** X.Y · **Date:** Month DD, YYYY
```

Parse both the version number and the date. The date determines the commit cutoff.

### Step 3: Find New Commits

```bash
git log --oneline --no-merges --since="{bible_date}" --format="%H %s" -- ':!{bible_path}'
```

- Exclude the Bible file itself to avoid self-referential loops.
- Use `--no-merges` to avoid double-counting changes from merge commits.
- If no commits found, report "Bible is up to date" and **stop**.

### Step 4: Categorize Changes by Section

For each commit, run `git show {hash} --stat` and match changed file paths against the `sections[].paths` patterns from `.bible.yaml`.

Use the algorithm documented in `references/triage-algorithm.md`.

Key rules:
- First match wins (sections are ordered by specificity in the config)
- Paths listed in `ignore_paths` are skipped entirely
- Unmatched paths go to an "uncategorized" bucket

### Step 5: Score Impact

For each affected section, assess impact:

| Level  | Criteria |
|--------|----------|
| HIGH   | New files in core directories, new migrations, new auth flows, new service components |
| MEDIUM | Modified endpoints, schema changes, config changes, dependency updates |
| LOW    | Test-only changes, doc-only changes, minor bug fixes, formatting |
| NONE   | Gitignore updates, log files, lock files only |

Rules:
- Count new files (git status `A`) vs modified files (status `M`) per section
- If any file in a section scores HIGH, the entire section is HIGH
- Check against `impact_rules` in config if defined (overrides defaults)
- Overall impact = highest impact across all affected sections

### Step 6: Present Triage Summary

Display to the user:

```
Bible Update Triage
-------------------
Bible version: X.Y (Month DD, YYYY)
Commits since last update: N

Changes by section:
  Section 5 (Database & Storage): N commits [HIGH]
    - new: migrations/20260413_add_users.sql
    - modified: prisma/schema.prisma
  Section 8 (API & Services): N commits [MEDIUM]
    - modified: api/auth/login.ts
  [...]

Uncategorized changes: N files
  - .env.example (consider adding to a section's paths in .bible.yaml)

Overall impact: HIGH
Recommended version bump: minor (X.Y -> X.Y+1)

Proceed with update? [Y/n]
```

**Ask the user if they want to proceed.** If they decline, stop. If they want to exclude certain sections, note the exclusions and proceed with the rest.

## Phase 2: Update (On User Approval)

### Step 1: Deep Analysis

Launch sub-agents (up to 3 in parallel) to analyze each affected section's changes in detail:
- What specifically changed in each file (read diffs, not just stats)
- New functions, endpoints, tables, views, components added
- Config changes, dependency updates, security-relevant modifications
- Anything that makes the Bible's current text inaccurate or incomplete

Each agent should return a structured summary: section number, what to add, what to modify, what to remove.

### Step 2: Read Affected Bible Sections

Read ONLY the Bible sections identified in the triage. Find heading line numbers first, then use offset/limit to read specific sections. Do NOT read the entire Bible file -- it may be thousands of lines.

### Step 3: Apply Targeted Edits

Use the Edit tool to make surgical changes to the Bible file. Rules:

- **Never rewrite the entire file** -- only edit affected sections
- **Match existing style** -- technical, direct, comprehensive
- **Update counts** where they appear (e.g., "48 Edge Functions" becomes "50 Edge Functions")
- **Add new entries** to existing tables rather than restructuring them
- **Only add new sections** if a genuinely new system component was introduced (e.g., an entirely new service layer, not just a new endpoint in an existing service)
- **Preserve ASCII diagrams** unless they are provably wrong
- **Preserve revision history** at the bottom of the file

### Step 4: Bump Version

Determine the version bump:
- **Minor bump** (X.Y -> X.Y+1): Existing sections updated, counts changed, bug fixes documented
- **Major bump** (X.Y -> X+1.0): New Bible sections added, architectural changes, new system components

Update the header:
```
> **Version:** X.Y · **Date:** Month DD, YYYY
```

Append to the revision history at the bottom of the file:
```
*Version X.Y updated Month DD, YYYY -- [1-line summary of changes].*
```

### Step 5: Update `.bible.yaml`

Update the `version` and `last_updated` fields in `.bible.yaml` to match the new Bible version and today's date.

### Step 6: Commit

```bash
git add {bible_path} .bible.yaml
git commit -m "docs: update Bible to vX.Y -- [1-line summary]

Co-Authored-By: Claude <noreply@anthropic.com>"
```

Do NOT push. The user decides when to push.

## Phase 3: CLAUDE.md Sync (Automatic)

After the Bible commit:

1. Check `claude_md.enabled` in `.bible.yaml`. If false, skip this phase entirely.
2. Tell the user: "Bible updated to vX.Y. Now syncing CLAUDE.md..."
3. Invoke the `claude-md-management:claude-md-improver` skill.
4. Provide context to the improver: which Bible sections changed, which sections are listed in `claude_md.operational_sections`, and what the specific changes were.
5. The improver handles the actual CLAUDE.md edits and commits separately.

If the `claude-md-management` skill is not installed, warn the user and skip. The Bible update is still valid without the CLAUDE.md sync.

## What This Skill Does NOT Do

- Does not rewrite the entire Bible from scratch (use `/init-bible` for that)
- Does not update documentation files other than the Bible and CLAUDE.md
- Does not deploy code or run tests
- Does not push to remote (the user decides when to push)
- Does not create the initial Bible or `.bible.yaml` (use `/init-bible`)
- Does not modify source code

## Error Handling

- If `.bible.yaml` is missing, stop and direct to `/init-bible`
- If the Bible file is missing or the version header cannot be parsed, stop and report the issue
- If `git log` fails (not a git repo, detached HEAD), stop and report
- If a section referenced in `.bible.yaml` no longer exists in the Bible, flag it in the triage summary as a config mismatch
- If the Bible file has uncommitted changes, warn the user before proceeding (edits would conflict)
