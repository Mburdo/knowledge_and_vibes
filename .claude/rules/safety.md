# Safety Rules

## File Deletion — FORBIDDEN

Never delete any file or directory without explicit user permission in this session.
- This includes files you just created (tests, scripts, temp files)
- You do not get to decide something is "safe" to remove
- If you think something should be deleted, stop and ask

## Destructive Git Commands — FORBIDDEN

These commands require explicit user approval with the exact command in the same message:
- `git reset --hard`
- `git clean -fd`
- `rm -rf`
- Any command that can delete or overwrite code/data

Rules:
1. If uncertain what a command will delete, do not run it — ask first
2. Prefer non-destructive alternatives: `git status`, `git diff`, `git stash`, backups
3. After approval, restate the command verbatim, list what it affects, wait for confirmation

## Code Modification Scripts — FORBIDDEN

Never run scripts that bulk-modify code:
- No codemods
- No one-off regex scripts
- No giant sed/awk refactors

Large mechanical changes: break into smaller explicit edits.
Subtle/complex changes: edit by hand, file-by-file.
