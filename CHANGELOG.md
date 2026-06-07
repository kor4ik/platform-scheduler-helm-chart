# Changelog

## [1.0.7] - 2026-06-07
### Changed
- Default image switched from `bitnami/kubectl` to `alpine/k8s` — Docker Hub no longer hosts new Bitnami kubectl tags.
- Image is fully overridable via `scheduler.image.repository` / `scheduler.image.tag`.

## [1.0.6] - 2026-06-07
### Changed
- State ConfigMap name is now derived from the Helm release name (`<release-name>-state`). `scheduler.stateConfigMapName` value removed — no user configuration needed.
- ArgoCD `ignoreDifferences` snippet updated accordingly in README.

## [1.0.5] - 2026-06-07
### Changed
- Renamed `global` values key to `scheduler` to avoid collision with Helm subchart `global` propagation.
- Added ArgoCD section to README explaining `ignoreDifferences` + `Replace=false` annotation.

## [1.0.4] - 2026-06-07
### Fixed
- Added `argocd.argoproj.io/sync-options: Replace=false` annotation to state ConfigMap to prevent ArgoCD from deleting and recreating it on sync.

## [1.0.3] - 2026-06-07
### Fixed
- Added `fetch-depth: 0` to checkout step so chart-releaser can detect chart changes across tags.
- Added `Configure Git` step (user.name / user.email) required for `cr index` commit.
- Created empty orphan `gh-pages` branch required before chart-releaser can push index.

## [1.0.2] - 2026-06-07
### Changed
- Moved chart files into `platform-scheduler/` subdirectory as required by `helm/chart-releaser-action`.

## [1.0.1] - 2026-06-07
### Added
- Initial release: `platform-scaledown` and `platform-scaleup` CronJobs.
- RBAC (ServiceAccount, ClusterRole, ClusterRoleBinding) always created.
- State ConfigMap for persisting replica counts across scale cycles.
- KEDA ScaledJob support.
- Multi-namespace `applications.namespaces` list.
- GitHub Pages Helm repository via `helm/chart-releaser-action`.
