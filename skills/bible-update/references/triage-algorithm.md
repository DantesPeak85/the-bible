# Triage Categorization Algorithm

Reference for the path-matching and impact-scoring logic used in Phase 1 of bible-update.

## Path Matching

```
for each commit in git log:
    files = parse_stat_output(git show {hash} --stat)
    for each file_path in files:
        if matches_any(file_path, config.ignore_paths):
            skip
            continue

        matched = false
        for section in config.sections:          # ordered by specificity
            for pattern in section.paths:
                if glob_match(file_path, pattern):
                    section.changes.append(file_path)
                    matched = true
                    break                        # first section match wins
            if matched:
                break

        if not matched:
            uncategorized.append(file_path)
```

Key invariant: each file is assigned to at most one section. Section order in `.bible.yaml` determines priority -- more specific sections should come first.

## Glob Matching Rules

Patterns are matched against the full relative path from the project root.

| Pattern | Matches | Does Not Match |
|---------|---------|----------------|
| `src/api/` | `src/api/auth.ts`, `src/api/v2/users.ts` | `src/api-docs/readme.md` |
| `*.sql` | `migrations/001.sql`, `seeds/data.sql` | `docs/sql-guide.md` |
| `docker-compose*` | `docker-compose.yml`, `docker-compose.prod.yaml` | `docs/docker-compose-guide.md` |
| `src/*.config.*` | `src/jest.config.ts`, `src/babel.config.js` | `src/utils/config.ts` |

Trailing slash on directory patterns means "anything under this directory." Patterns without slashes match against the filename component at any depth.

## Determining File Status

Extract the change type from `git show --stat` or use `git diff-tree --no-commit-id -r {hash}`:

| Git Status | Meaning | Impact Implication |
|------------|---------|-------------------|
| `A` | Added (new file) | Likely HIGH or MEDIUM |
| `M` | Modified | MEDIUM or LOW depending on scope |
| `D` | Deleted | Flag section for review -- something was removed |
| `R` | Renamed | Count as removal from old section + addition to new |

For `R` (renamed) files, match both the old and new paths. The old path's section may need a note about the removal, and the new path's section gets the addition.

## Impact Scoring

Per-section scoring -- evaluate each section independently, then take the max.

### HIGH
- Any new file (`A` status) in a core directory (as defined by section paths)
- New database migrations
- New authentication or authorization flows
- New service or infrastructure components
- Changes to security-critical files (auth, encryption, permissions)

### MEDIUM
- Modified API endpoints (signature changes, new parameters)
- Schema or model changes
- Configuration changes (environment, build, deploy)
- Dependency additions or major version bumps
- Modified infrastructure files (Dockerfile, CI/CD)

### LOW
- Test file changes only (new tests, test fixes)
- Documentation-only changes (README, comments, JSDoc)
- Minor bug fixes (small diff, single file)
- Code formatting or linting fixes
- Dependency patch version bumps

### NONE
- Changes only to gitignored file patterns (lock files if in ignore list)
- Log file changes
- IDE configuration files (`.vscode/`, `.idea/`)

Precedence: if any file within a section scores HIGH, the section is HIGH. Overall project impact = max(all section impacts).

If `.bible.yaml` defines `impact_rules` with custom overrides, those take precedence over the defaults above.

## Uncategorized File Handling

Files that match no section pattern are collected into an "uncategorized" bucket.

In the triage summary: list each path, suggest which section it might belong to, and suggest updating `.bible.yaml`. Uncategorized files do NOT contribute to impact scoring -- they indicate gaps in section coverage, not necessarily changes that need documenting.

## Edge Cases

- **Merge commits**: Always use `--no-merges` in git log. Merges duplicate parent changes, inflating counts.
- **Binary files**: Skip during matching. Note in summary but do not score. Detect via `Bin 0 -> 1234 bytes` in stat output.
- **Empty commits**: Ignore commits with no file changes (`--allow-empty` used for CI triggers).
- **Submodule updates**: Treat as MEDIUM in whatever section the submodule path falls under. Do not recurse.
- **Large changesets** (50+ files): Flag in triage and ask the user whether to include or skip. Bulk renames and formatter runs inflate triage without meaningful Bible changes.
