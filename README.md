# platform-scheduler Helm Chart

`platform-scheduler` is a Helm chart that creates `platform-scaledown` and `platform-scaleup` CronJobs to automate environment scale-down and scale-up windows for cost optimization.

## What it does

- Scales down and up entire environments on a cron schedule.
- Supports on-demand execution by triggering the CronJob manually (`kubectl create job --from=cronjob/platform-scaledown`).
- Saves state for all resources (Deployments, StatefulSets, CronJobs, KEDA ScaledJobs) before scale-down.
- Restores exact previous replica counts on scale-up, with dependency-first ordering and readiness gates.

## Prerequisites

- Kubernetes cluster with CronJob support (`batch/v1`).
- RBAC resources (ServiceAccount, ClusterRole, ClusterRoleBinding) are created by this chart.
- Optional: KEDA CRDs if you use ScaledJobs.

## Install

```bash
helm repo add platform-scheduler https://kor4ik.github.io/platform-scheduler-helm-chart/
helm repo update
helm install platform-scheduler platform-scheduler/platform-scheduler \
  --namespace platform-scheduler \
  --create-namespace \
  --set platformSchedule.enabled=true
```

## Key values

| Key | Type | Default | Description |
|---|---|---|---|
| `global.serviceAccountName` | string | `platform-scheduler-sa` | Name of the ServiceAccount created and used by the CronJobs. |
| `global.stateConfigMapName` | string | `platform-state-cm` | ConfigMap used to persist replica counts between scale-down and scale-up. |
| `global.image.repository` | string | `bitnami/kubectl` | Image repository for CronJob containers. |
| `global.image.tag` | string | `1.30.13` | Pinned kubectl image tag. |
| `global.image.pullPolicy` | string | `IfNotPresent` | Image pull policy. |
| `global.nodeSelector` | object | `{}` | Optional node selector for CronJob pods. Omit to use default scheduling. |
| `platformSchedule.enabled` | bool | `false` | Enables the scheduler CronJobs. |
| `platformSchedule.start` | string | `""` | Scale-up cron expression. Empty keeps the scaleup CronJob suspended. |
| `platformSchedule.end` | string | `""` | Scale-down cron expression. |
| `platformSchedule.timezone` | string | `""` | Cron timezone; defaults to `Asia/Jerusalem`. |
| `platformSchedule.dependencies.enabled` | bool | `false` | Enables scale-down/up of dependency namespaces. |
| `platformSchedule.dependencies.apps` | list | `[]` | List of `{namespace, labelSelector}` pairs for dependency workloads. |
| `platformSchedule.applications.enabled` | bool | `false` | Enables scale-down/up of application namespaces. |
| `platformSchedule.applications.namespace` | string | `""` | Target application namespace. |

## Example values

```yaml
global:
  serviceAccountName: platform-scheduler-sa
  stateConfigMapName: platform-state-cm
  nodeSelector:
    agentpool: system

platformSchedule:
  enabled: true
  timezone: UTC
  start: "0 7 * * 1-5"
  end: "0 20 * * 1-5"
  dependencies:
    enabled: true
    apps:
      - name: dapr
        namespace: dapr-system
      - name: keda
        namespace: keda
        labelSelector: "app.kubernetes.io/name=keda"
  applications:
    enabled: true
    namespace: 
        - my-namespace
```

## Release workflow

This repository includes `.github/workflows/release.yml` using `helm/chart-releaser-action`.

- Push a semantic version tag like `v1.0.1`.
- The workflow packages the chart and publishes the Helm index to `gh-pages`.
- Consumers install from `https://kor4ik.github.io/platform-scheduler-helm-chart/`.

## Local validation

```bash
helm lint .
helm template test . -f values.yaml
```

## Notes

- `platform-scaleup` is created in suspended state when `platformSchedule.start` is empty.
- Replica counts and KEDA state are saved to the ConfigMap on scale-down and restored on scale-up.
- RBAC is always created regardless of `platformSchedule.enabled` so pods are ready when the schedule fires.
