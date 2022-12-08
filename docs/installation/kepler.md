# Kepler Installation
## Requirements
- Kernel 4.18+
- Access to a Kubernetes cluster
- `kubectl` v1.21.0+

## Deploy the Kepler
Follow the steps below to deploy the Kepler exporter as a Daemonset to run on all Nodes. The following deployment will also create a service listening on port `9102`.

First, fork the [kepler](https://github.com/sustainable-computing-io/kepler) repository and clone it.

Then, build the manifests file that suit your environment and deploy it with the following steps:

### Build manifests
```bash
make build-manifest OPTS="<deployment options>"
# minimum deployment: 
# > make build-manifest
# deployment with sidecar on openshift: 
# > make build-manifest OPTS="ESTIMATOR_SIDECAR_DEPLOY OPENSHIFT_DEPLOY"
```
Manifests will be generated in  `_output/manifests/kubernetes/generated/` by default.

Deployment Option|Description|Dependency
---|---|---
BM_DEPLOY|baremetal deployment patched with node selector feature.node.kubernetes.io/cpu-cpuid.HYPERVISOR to not exist|-
OPENSHIFT_DEPLOY|patch openshift-specific attribute to kepler daemonset and deploy SecurityContextConstraints|-
PROMETHEUS_DEPLOY|patch prometheus-related resource (ServiceMonitor, RBAC role, rolebinding) |require prometheus deployment which can be OpenShift integrated or [custom deploy](https://github.com/sustainable-computing-io/kepler#deploy-the-prometheus-operator-and-the-whole-monitoring-stack)
CLUSTER_PREREQ_DEPLOY|deploy prerequisites for kepler on openshift cluster| OPENSHIFT_DEPLOY option set
CI_DEPLOY|update proc path for kind cluster using in CI|-
ESTIMATOR_SIDECAR_DEPLOY|patch estimator sidecar and corresponding configmap to kepler daemonset|-
MODEL_SERVER_DEPLOY|deploy model server and corresponding configmap to kepler daemonset|-
TRAIN_DEPLOY|patch online-trainer sidecar to model server| MODEL_SERVER_DEPLOY option set
 -  build-manifest requirements:
    -  kubectl v1.21+
    -  make
    -  go
 -  manifest sources and outputs will be in  `_output/generated-manifests` by default


### Deploy

```
# kubectl apply -f _output/generated-manifests/deployment.yaml
```

## Deploy the Prometheus operator and the whole monitoring stack
If Prometheus is already installed in the cluster, skip this step. Otherwise, follow these steps to install it.

1. Clone the [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus) project to your local folder.
```
# git clone https://github.com/prometheus-operator/kube-prometheus
```

1. Deploy the whole monitoring stack using the config in the `manifests` directory.
Create the namespace and CRDs, and then wait for them to be available before creating the remaining resources
```
# cd kube-prometheus
# kubectl apply --server-side -f manifests/setup
# until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
# kubectl apply -f manifests/
```


