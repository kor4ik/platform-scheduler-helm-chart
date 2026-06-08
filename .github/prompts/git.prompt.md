---
description: "Git operations for platform-scheduler: commit, push, tag, or full release. Use when you want to commit changes, push to GitHub, create a version tag, or do a full release (default patch bump + changelog + commit + push + tag, or explicit version)."
name: "Git"
argument-hint: "commit | push | tag <version> | release <summary> | release <version> <summary>"
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

1. Run `git diff --stat` to see what will be committed.
2. Run `git diff` to read the actual changes.
3. Construct a concise [Conventional Commits](https://www.conventionalcommits.org/) message from the diff (e.g. `fix: ...`, `feat: ...`, `chore: ...`). Do **not** ask the user for a message.
4. Run `git add -A && git commit -m "<generated message>"`.

### `push`

Push current branch and any local tags.

1. Run `git push origin main`.

### `tag <version>`

Create and push a tag (e.g. `tag 1.0.8`).

1. Verify `platform-scheduler/Chart.yaml` version matches `<version>`. If not, stop and warn.
2. Run `git tag v<version> && git push origin v<version>`.

### `release <summary>` or `release <version> <summary>`

Full release flow — use this when you have uncommitted changes ready to ship.

1. Read current version from `platform-scheduler/Chart.yaml`.
2. Determine target version:
   - If a `<version>` argument is provided, use it.
   - Otherwise, compute patch bump from current chart version (`X.Y.Z` -> `X.Y.(Z+1)`), e.g. `1.0.7` -> `1.0.8`.
3. Update `version:` in `platform-scheduler/Chart.yaml` to target version.
4. Prepend a new entry to `.github/log/CHANGELOG.md`:
   ```
   ## [<target-version>] - <today's date YYYY-MM-DD>
   ### Changed
   - <summary>
   ```
5. Run `git diff --stat` and confirm files look correct.
6. Run `git add -A && git commit -m "chore: release <target-version> - <summary>"`.
7. Run `git tag v<target-version> && git push origin main v<target-version>`.
8. Confirm: "Release v<target-version> pushed. The GitHub Actions workflow will publish it to the Helm repo."

If there are no uncommitted changes (clean working tree), skip steps 3–6 and only create and push the tag — but first verify `platform-scheduler/Chart.yaml` already matches the target version.
