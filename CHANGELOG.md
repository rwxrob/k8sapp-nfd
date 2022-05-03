# Change Log

These changes are the ones noticed between versions from the source
material and any changes to how those changes are mixed with custom
configuration (such as resource constraints).

## v0.11.0

### Added

* Controller (CRD) resource nodefeaturerules.nfd.k8s-sigs.io

* ClusterRole nfd-master now requires topology.node.k8s.io and
  nfd.k8s-sigs.io apiGroups

* DaemonSet mountPath /host-usr/src and hostPath /usr/src

### Changed

* Namespace nfd -> node-feature-discovery

* New image version k8s.gcr.io/nfd/node-feature-discovery:v0.11.0

* Leaving commented out verbose ConfigMap for reference

## v0.9.0

This was the first installed version.
