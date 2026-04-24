# System Overview

## Purpose

This repository contains a Kubernetes operator for deploying and managing MarkLogic clusters. The operator exposes two custom resources:

- `MarklogicCluster`: the top-level cluster intent.
- `MarklogicGroup`: the workload slice used to realize each group in a cluster.

The manager process is started from `cmd/main.go` and registers both reconcilers with controller-runtime.

## Control Plane Split

The main design boundary is between cluster-level orchestration and group-level workload ownership.

### MarklogicCluster path

`internal/controller/marklogiccluster_controller.go` creates a `ClusterContext` and delegates to `ReconsileMarklogicClusterHandler` in `pkg/k8sutil/handler.go`.

That handler owns cluster-scoped or shared resources, including:

- service account creation
- admin secret reconciliation
- optional network policy
- optional HAProxy resources
- optional ingress

The cluster reconciliation path also fans out the desired `MarklogicCluster.spec.markLogicGroups` entries into child `MarklogicGroup` resources in `pkg/k8sutil/marklogicServer.go`.

### MarklogicGroup path

`internal/controller/marklogicgroup_controller.go` creates an `OperatorContext` and delegates to `ReconsileMarklogicGroupHandler` in `pkg/k8sutil/handler.go`.

That handler owns the concrete workload resources for a single group:

- Services
- ConfigMaps
- optional Fluent Bit ConfigMap for log collection
- StatefulSet and its readiness/status handling

In practice, workload ownership lives with the `MarklogicGroup` controller, while `MarklogicCluster` acts as the fan-out and shared-infrastructure coordinator.

## Runtime Layout

- `api/v1/`: CRD types and generated deepcopy code.
- `cmd/main.go`: operator entrypoint and manager wiring.
- `internal/controller/`: controller-runtime reconcilers.
- `pkg/k8sutil/`: reconciliation handlers and Kubernetes object builders.
- `config/`: kustomize bases for CRDs, RBAC, manager deployment, and samples.
- `charts/marklogic-operator-kubernetes/`: Helm packaging for operator installation.
- `test/e2e/`: end-to-end tests using e2e-framework against local Kubernetes.

## Delivery Surfaces

The repo supports multiple ways to consume the operator:

- local build/run from the Go binary
- container image build and push via the Makefile
- kustomize deployment from `config/default`
- Helm install via the chart in `charts/marklogic-operator-kubernetes`
- operator bundle and catalog image generation for Operator Lifecycle Manager flows

## Good Entry Points

If you are new to the codebase, start here:

1. `README.md` for install and user-facing workflow.
2. `Makefile` for build, lint, test, deploy, and bundle commands.
3. `cmd/main.go` for manager wiring.
4. `internal/controller/marklogiccluster_controller.go` for cluster orchestration.
5. `internal/controller/marklogicgroup_controller.go` for workload ownership.
6. `pkg/k8sutil/marklogicServer.go` for `MarklogicCluster` to `MarklogicGroup` fan-out.