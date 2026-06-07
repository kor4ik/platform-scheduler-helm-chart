---
description: "Git operations for platform-scheduler: commit, push, tag, or full release. Use when you want to commit changes, push to GitHub, create a version tag, or do a full release (bump version + update changelog + commit + push + tag)."
name: "Git"
argument-hint: "commit | push | tag <version> | release <version> <summary>"
agent: "agent"
tools: ["run_in_terminal", "read_file", "replace_string_in_file"]
---

You are helping manage git operations for the `platform-scheduler` Helm chart repo.

## Rules

- **Chart.yaml version must always match the git tag** for chart-releaser to publish correctly.
- **Always update CHANGELOG.md** when doing a release.
- Never force-push or amend published commits.
- Confirm before any destructive operation (tag delete, branch delete).

## Commands

### `commit`
Commit all staged+unstaged changes.
1. Run `git diff --stat` to show what will be committed.
2. Ask for a commit message if not provided.
3. Run `git add -A && git commit -m "<message>"`.

### `push`
Push current branch and any local tags.
1. Run `git push origin main`.

### `tag <version>`
Create and push a tag (e.g. `tag 1.0.8`).
1. Verify `platform-scheduler/Chart.yaml` version matches `<version>`. If not, stop and warn.
2. Run `git tag v<version> && git push origin v<version>`.

### `release <version> <summary>`
Full release flow — use this when you have uncommitted changes ready to ship.
1. Read current version from `platform-scheduler/Chart.yaml`.
2. Update `version:` in `platform-scheduler/Chart.yaml` to `<version>`.
3. Prepend a new entry to `CHANGELOG.md`:
   ```
   ## [<version>] - <today's date YYYY-MM-DD>
   ### Changed
   - <summary>
   ```
4. Run `git diff --stat` and confirm files look correct.
5. Run `git add -A && git commit -m "chore: release <version> — <summary>"`.
6. Run `git tag v<version> && git push origin main v<version>`.
7. Confirm: "Release v<version> pushed. The GitHub Actions workflow will publish it to the Helm repo."

If there are no uncommitted changes (clean working tree), skip steps 2–5 and only create and push the tag — but first verify Chart.yaml version matches `<version>`.
