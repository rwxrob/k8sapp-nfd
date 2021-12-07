# K8SAPP: Node Feature Discovery with Limited Labels and SecurityContext

This Kubernetes system application provides the standard Kubernetes Node
Feature Discovery annotations/labels but limits the ones included and
add minimal SecurityContext values. NFD fulfills a critical dependency
for NVIDIA GPU Feature Discovery.

## Application Repo Organization

This repo follows the K8SAPP conventions:

* `README.md` - description, notes, and references
* `k8sapp` - simple to fetch, config, install, update, uninstall, and check
* `k8sapp.yaml` - metadata about this application

This repo also contains the original project source for the Node
Feature Discovery project (from the Kubernetes SIG). 

## Application Management Procedures

Avoid putting any of this into a separate script (except `check`) since
things might change upon update and every change needs to be reviewed
again and *all* the management procedures could potentially need to be
updated.

### Fetching the Code

The Helm chart and Kustomize versions are inconsistent. For example, the
Helm chart grants additional ClusterRole permissions. For this reason,
this k8sapp uses Kustomize as a base.

```
kubectl kustomize \
  "https://github.com/kubernetes-sigs/node-feature-discovery/deployment/overlays/default?ref=v0.9.0" > "nfd.yaml"
```

### Reviewing Code and Container Images

Observations after review of the default NFD source code and
configurations:

* `node-feature-discovery` default namespace created
* No resource constraints of any kind

Only a single image is used:

  `k8s.gcr.io/nfd/node-feature-discovery:v0.9.0`

TODO: report on security vetting of the image

### Customizing and Configuring

* Change the namespace values to 'nfd'

```bash
while IFS= read -r line; do 
  line=${line//namespace: node-feature-discovery/namespace: nfd}
  line=${line//name: node-feature-discovery/name: nfd}
  echo "$line"
done < nfd.yaml > nfd.yaml~
mv nfd.yaml~ nfd.yaml
```

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

No updates currently available.

### Uninstalling

```yaml
kubectl delete ns nfd
kubectl delete clusterrole nfd-master
kubectl delete clusterrolebinding nfd-master
:q!
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

## Checklist

- [ ] Download, review, scan, and mirror container images
- [ ] Add an entry to the main `k8sapps` manifest meta repo 
