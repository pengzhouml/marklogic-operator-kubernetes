# Build And Test

## Core Prerequisites

- Go toolchain
- Docker or another compatible container tool
- kubectl
- Helm for chart-based install flows
- minikube for local end-to-end testing

The Makefile bootstraps several local tools into `bin/`, including `kustomize`, `controller-gen`, `setup-envtest`, `istioctl`, `operator-sdk`, `golangci-lint`, and `opm` when their targets are invoked.

## Common Development Commands

### Build and run

```sh
make build
make run
```

`make build` generates manifests, runs formatting and vetting, and writes the manager binary to `bin/manager`.

### Unit and controller tests

```sh
make test
```

This runs non-e2e Go tests with envtest assets and writes coverage to `cover.out`.

### Lint

```sh
make lint
make lint-fix
```

### Image build and push

```sh
make docker-build IMG=<registry>/<image>:<tag>
make docker-push IMG=<registry>/<image>:<tag>
```

For multi-architecture images:

```sh
make docker-buildx IMG=<registry>/<image>:<tag>
```

## Local Kubernetes Deploy Flow

Install CRDs only:

```sh
make install
```

Deploy the operator from `config/default`:

```sh
make deploy IMG=<registry>/<image>:<tag>
```

Remove the deployment:

```sh
make undeploy
make uninstall
```

## End-To-End Tests

Standard minikube-backed flow:

```sh
make e2e-setup-minikube
make e2e-test
make e2e-cleanup-minikube
```

Istio ambient mode flow:

```sh
make e2e-setup-minikube-istio
make e2e-test-istio
make e2e-cleanup-minikube
```

Target a labeled e2e subset directly:

```sh
go test -v ./test/e2e -count=1 -args --labels="type=tls-self-signed"
go test -v ./test/e2e -count=1 -args --labels="type=tls-named-cert"
```

Notes:

- `test/e2e/main_test.go` performs setup, tool installation, operator deployment, and teardown.
- `VERIFY_HUGE_PAGES=true make e2e-test` enables the hugepages branch in the Makefile.
- The default e2e image variables are exported from the Makefile and can be overridden through the environment.

## Packaging

Build operator bundle artifacts:

```sh
make bundle
make bundle-build
make bundle-push
```

Build a catalog image:

```sh
make catalog-build
```

## Practical Doc Trail

- `README.md`: user-facing install and sample usage.
- `test/README.md`: short e2e usage notes.
- `config/samples/`: sample custom resources.
- `config/manager/` and `config/default/`: deployed manager manifests.