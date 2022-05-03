# K8SAPP: Node Feature Discovery with Reduced Labels and Resource Limits

This Kubernetes system application provides the standard Kubernetes Node
Feature Discovery annotations/labels but limits the ones included and
add minimal resource constraints. 

> ⚡ NFD fulfills a critical dependency for NVIDIA GPU Feature
Discovery and should not be upgraded unless GFD is also included in the
upgrade if you use it.

## Application Repo Organization

* `README.md` - description, notes, and references
* `CHANGELOG.md` - changes between vendor versions and customizations
* `check` - check for latest update
* `get` - overwrite `nfd.yaml` with latest

## Application Management Procedures

Avoid putting any of this into a separate script (except `check`) since
things might change upon update and every change needs to be reviewed
again and *all* the management procedures could potentially need to be
updated.

### Fetching the Code

The Helm chart and Kustomize versions are inconsistent. For example, the
Helm chart grants additional ClusterRole permissions. For this reason,
this k8sapp uses Kustomize as a base. Remember to change the `ref` to
the latest version (output of `check`).

The `get` script will overwrite the `nfd.yaml` file with the
latest so that a `git diff nfd.yaml` file can show all the precise
changes.

### Reviewing Code and Container Images

Observations after review of the default NFD source code and
configurations:

* `node-feature-discovery` default namespace created
* Creates a CRD with reasonable permissions
* No resource constraints of any kind

Only a single image is used:

  `k8s.gcr.io/nfd/node-feature-discovery:v0.11.0`

### Customizing and Configuring

* Add resource constraints to both worker and master DaemonSets:

```yaml
spec:
  template:
    spec:
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi
```

* Limit the labels that are created as configured in
  the following worker configuration (ConfigMap):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nfd-worker-conf
  namespace: nfd
data:
  nfd-worker.conf: |
    core:
      labelWhiteList: '^(custom|system|pci|cuda|gpu|gfd)'
```

> ⚠️ 
> Note that the regular expression for the `labelWhiteList` is
> evaluated from the `/` that separates the prefix (for example,
> `feature.node.kubernetes.io/`) and therefore needs the caret (`^`) to
> lock it down. This information is useful when adding GPU Feature
> discovery from NVIDIA which has a different prefix.

### Installing

***⚠️  Be sure to change into the proper context (dev, prod, etc.)!***

```
kubectl apply -f nfd.yaml
```

Note that the `--namespace` is not needed since it has been explicitly added to all the resources in the `nfd.yaml` file.

### Updating

* Download the latest `nfd.yaml` with `get` script
* Run `git diff nfd.yaml` to see what has changed
* Summarize changes in CHANGELOG.md
* Add the whitelist change (see above)
* Add the resource constraints (see above)
* Pull the latest image and push to home image registry
* Uninstall previous (see below)
* Install new (see above)

### Uninstalling

```yaml
kubectl delete ns node-feature-discovery
kubectl delete crd nodefeaturerules.nfd.k8s-sigs.io
kubectl delete clusterrole nfd-master
kubectl delete clusterrolebinding nfd-master
```

### Check for Updates

Check that the latest GitHub release has changed (which is what the
`check` script does).

```bash
curl -sSL https://api.github.com/repos/kubernetes-sigs/node-feature-discovery/releases/latest | jq -r .name
```

## Related References

* Get started · Node Feature Discovery  
  <https://kubernetes-sigs.github.io/node-feature-discovery/stable/get-started/index.html>
* GitHub - kubernetes-sigs/node-feature-discovery: Node feature discovery for Kubernetes  
  <https://github.com/kubernetes-sigs/node-feature-discovery>
