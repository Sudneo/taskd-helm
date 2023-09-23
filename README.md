# Taskd Helm Chart

The chart in this repository is used to deploy an instance of `taskd` ([Taskserver](https://github.com/GothenburgBitFactory/taskserver)).

The chart uses the image from https://github.com/DJAndries/taskd-docker, but expects the `rootless` version to be available (see https://github.com/DJAndries/taskd-docker/pull/1/).

## Installation

To get started:

### Using Helm

* Clone this repository
* Modify the `values.yaml` to your needs, or create a new file with only the values you want to specify.
* Run `helm install -f values.yaml -n $NAMESPACE $NAME .`. For example, to install the chart in the `k8s-apps` namespace and the release called `taskd`:

```sh
helm install -f values.yaml -n k8s-apps taskd .
```

### Using Flux

In order to use `flux`, the chart needs to be pullable from some OCI repository.
I am using my private `gitea` instance for storing Charts, but it seems Github does not support this functionality (it requires the usage of Github-Pages?), so you will need to clone this repo
and push the chart to your registry, for example using `gitea`:

```
helm package .
helm repo add  --username {username} --password {password} {repo} https://gitea.example.com/api/packages/{owner}/helm
helm cm-push ./taskd-{version}.tgz {repo}
```

Once this step is done, you can use a simple `HelmRelease` to start the reconciliation:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: taskd
  namespace: { namespace }
spec:
  interval: 5m
  chart:
    spec:
      version: "1.1.0"
      chart: taskd
      sourceRef:
        kind: HelmRepository
        name: { helmRepo }
      interval: 60m
  install:
    crds: Create
  upgrade:
    crds: CreateReplace
  values:
    namespace: { namespace }
    image:
      pullPolicy: IfNotPresent
    persistence:
      enabled: true
      name: taskd
      accessMode: ReadWriteOnce
      size: 1Gi
      storageClass: "openebs-hostpath"
      path: /home/taskd/data/
    securityContext:
      capabilities:
        drop:
        - ALL
      readOnlyRootFilesystem: true
      runAsNonRoot: true
      runAsUser: 65532
      capabilities:
        drop:
        - ALL
    resources:
      limits:
        cpu: 100m
        memory: 256Mi
      requests:
        cpu: 10m
        memory: 50Mi
    service:
      type: LoadBalancer
      port: 53589
    taskd:
      hostname: "taskd.example.org"
      org: "homelab"
      country: "EE"
      state: "Harju"
      locality: "Tallinn"
      cert_expiration_days: "365"
```
