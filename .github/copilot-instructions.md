# platform-scheduler Helm Chart — Copilot Instructions

This repo contains a single Helm chart (`platform-scheduler/`) published to GitHub Pages via `helm/chart-releaser-action`.

## Repo structure

```
platform-scheduler/       # Helm chart root
  Chart.yaml              # name=platform-scheduler, version must match git tag
  values.yaml
  templates/
    resources.yaml        # ServiceAccount, ClusterRole, ClusterRoleBinding, ConfigMap (always rendered)
    scaledown.yaml        # CronJob: platform-scaledown
    scaleup.yaml          # CronJob: platform-scaleup
.github/
  workflows/release.yml   # Triggers on v* tags → packages chart → publishes to gh-pages
  prompts/git.prompt.md   # Git agent: commit / push / tag / release
  log/CHANGELOG.md        # Track every chart version change here
README.md                 # User-facing docs: values table, install, ArgoCD section
```

## Key conventions

- **`Chart.yaml` version must match the git tag** pushed to trigger a release. If they differ, chart-releaser skips the release because it compares chart version against previous GitHub releases, not git tags.
- **Always bump `Chart.yaml` version before tagging.** Use the `/git` prompt to do this correctly.
- **State ConfigMap name** is `<release-name>-state`, derived from `.Release.Name`. Do not add a configurable value for it.
- **Values top-level key is `scheduler`**, not `global` — avoids Helm subchart `global` propagation collisions.
- **Image default is `alpine/k8s`** — `bitnami/kubectl` is no longer published to Docker Hub for recent tags.

## Release checklist

See [CHANGELOG.md](log/CHANGELOG.md) for recent changes.

1. Make code changes.
2. Bump `version:` in `platform-scheduler/Chart.yaml`.
3. Update `CHANGELOG.md` with what changed.
4. `git add -A && git commit -m "..."`
5. `git tag v<version> && git push origin main v<version>`
6. Confirm the workflow at `https://github.com/kor4ik/platform-scheduler-helm-chart/actions` completes successfully.

Use the `/git` prompt to handle steps 2–5 automatically.
