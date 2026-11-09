# Changelog

## [1.0.12] - 2026-11-09
### Added
- `platformSchedule.karpenterSweep` (off by default): a last step of `platform-scaledown` that removes Karpenter nodes whose only non-DaemonSet pods are AKS add-ons (`addonNamespaces`, default `kube-system` and `gatekeeper-system`). Karpenter itself keeps such nodes: on sreming the second Azure Policy gatekeeper replica prefers a node apart from the first one, so consolidation reported "Can't replace with a cheaper node" and a platform node ran all night. Other nodes are deleted through their NodeClaim (Karpenter drains them, PDBs apply); the node running the job is cordoned and drained of its add-on pods with `kubectl drain` (eviction API) and Karpenter removes it once the job has finished. Nodes or pods annotated `karpenter.sh/do-not-disrupt: "true"` are skipped. Nodes still holding application pods are re-checked every 15s until `timeoutSeconds` (default 900). The ClusterRole gains nodes get/list/patch, pods get/list, pods/eviction create, daemonsets get and `karpenter.sh/nodeclaims` get/list/delete only when the sweep is enabled.

## [1.0.11] - 2026-11-09
### Fixed
- `platform-scaledown` now pauses every KEDA ScaledObject in the application namespaces with `autoscaling.keda.sh/paused-replicas: "0"`, waits up to 60s for KEDA to report it `Paused`, and then scales back to 0 any Deployment that an autoscaler raised again. Before, only ScaledJobs were paused: applications were scaled to 0 while the KEDA operator was still running, so a ScaledObject with `minReplicaCount: 0` and an active trigger (a Service Bus backlog) activated its Deployment from 0 to 1 again before KEDA itself was scaled down, and the Deployment stayed at 1 all night (seen with `ask-mia-indexer` on misquad01). ScaledObjects that were already paused are left alone and carry their saved state forward.
- `platform-scaleup` removes the annotation only from ScaledObjects that scaledown paused (`<ns>_sop_<name>` in the state ConfigMap), after the Deployments are restored.
- ClusterRole: `get`/`list`/`patch` on `keda.sh/scaledobjects`.

## [1.0.10] - 2026-11-02
### Fixed
- `platform-scaledown` now replaces the `data` of the state ConfigMap with what the run observed instead of merge-patching keys into it. The merge patch only added or overwrote keys, so a ScaledJob whose `minReplicaCount` was lowered to 0 in git kept the non-zero value saved on an earlier night and every scaleup restored it (seen with `homelander-runner` on five dev clusters; ArgoCD cannot correct it because `/spec/minReplicaCount` is ignored there). Keys for objects that no longer exist are dropped the same way. Objects the run finds already shut down (replicas 0, suspended, paused with `minReplicaCount` 0) carry their saved value forward, so a scaledown on an environment that is already down, which happens when `platformSchedule.start` is empty and nobody scaled up that day, no longer risks losing state. `platform-scaleup` is unchanged: a missing `sjmr_` key still means nothing to restore.
- An unset `minReplicaCount` is no longer saved as an empty value.

## [1.0.9] - 2026-10-01
### Added
- `scheduler.namespace.create` / `scheduler.namespace.labels`: optionally render the release Namespace with labels. Default labels carry `config.linkerd.io/admission-webhooks: disabled` so the scheduler pods are never intercepted by the Linkerd proxy-injector webhook. Without this, scaling down Linkerd or cert-manager (which injects the webhook caBundle) makes every later scaleup Job fail with `failed calling webhook linkerd-proxy-injector` and the environment cannot recover on its own.
- `platformSchedule.dependencies.apps[].waitForReady`: when true, scaleup waits for that dependency's rollout before scaling the next one. Use it on cert-manager so cainjector is back before Linkerd and the rest start.

## [1.0.8] - 2026-06-08
### Changed
- Remove unused platformSchedule.enabled value and docs references

## [1.0.7] - 2026-06-07
### Changed
- test 001

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
