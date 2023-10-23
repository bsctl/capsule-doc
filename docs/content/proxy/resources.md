
# Cluster resources selection

Starting from [Capsule v0.2.0](https://github.com/projectcapsule/capsule/releases/tag/v0.2.0), selection of cluster-scoped resources based on labels has been introduced.

Due to the limitations of Kubernetes API Server which not support `OR` label selector, the Capsule core team decided to give precedence to the label selector over the exact and regex match.

Capsule is going to deprecate in the upcoming feature the selection based on exact names and regex in order to approach entirely to the matching labels approach of Kubernetes itself.

## Namespaces

As tenant owner `alice`, you can use `kubectl` to create some namespaces:

```
$ kubectl --context alice-oidc@mycluster create namespace oil-production
$ kubectl --context alice-oidc@mycluster create namespace oil-development
$ kubectl --context alice-oidc@mycluster create namespace gas-marketing
```

and list only those namespaces:

```
$ kubectl --context alice-oidc@mycluster get namespaces
NAME                STATUS   AGE
gas-marketing       Active   2m
oil-development     Active   2m
oil-production      Active   2m
```

Capsule Proxy supports applying a Namespace configuration using the `apply` command, as follows.

```
$: cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: solar-development
EOF

namespace/solar-development unchanged
# or, in case of non-existing Namespace:
namespace/solar-development created
```

## Nodes

The Capsule Proxy gives the owners the ability to access the nodes matching the `.spec.nodeSelector` in the Tenant manifest:

```yaml
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata:
  name: oil
spec:
  owners:
    - kind: User
      name: alice
      proxySettings:
        - kind: Nodes
          operations:
            - List
  nodeSelector:
    kubernetes.io/hostname: capsule-gold-qwerty
```

```bash
$ kubectl --context alice-oidc@mycluster get nodes
NAME                    STATUS   ROLES    AGE   VERSION
capsule-gold-qwerty     Ready    <none>   43h   v1.19.1
```

> Warning: when no `nodeSelector` is specified, the tenant owners has access to all the nodes, according to the permissions listed in the `proxySettings` specs.

### Special routes for kubectl describe

When issuing a `kubectl describe node`, some other endpoints are put in place:

* `api/v1/pods?fieldSelector=spec.nodeName%3D{name}`
* `/apis/coordination.k8s.io/v1/namespaces/kube-node-lease/leases/{name}`

These are mandatory to retrieve the list of the running Pods on the required node and provide info about its lease status.

## Storage Classes

A Tenant may be limited to use a set of allowed Storage Class resources, as follows.

```yaml
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - kind: User
    name: alice
    proxySettings:
    - kind: StorageClasses
      operations:
      - List
  storageClasses:
    allowed:
      - custom
    allowedRegex: "\\w+fs"
```

In the Kubernetes cluster we could have more Storage Class resources, some of them forbidden and non-usable by the Tenant owner.

```bash
$ kubectl --context admin@mycluster get storageclasses
NAME                 PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
cephfs               rook.io/cephfs           Delete          WaitForFirstConsumer   false                  21h
custom               custom.tls/provisioner   Delete          WaitForFirstConsumer   false                  43h
default(standard)    rancher.io/local-path    Delete          WaitForFirstConsumer   false                  43h
glusterfs            rook.io/glusterfs        Delete          WaitForFirstConsumer   false                  54m
zol                  zfs-on-linux/zfs         Delete          WaitForFirstConsumer   false                  54m
```

The expected output using `capsule-proxy` is the retrieval of the `custom` Storage Class as well as the other ones matching the regex `\w+fs`.

```bash
$ kubectl --context alice-oidc@mycluster get storageclasses
NAME                 PROVISIONER              RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
cephfs               rook.io/cephfs           Delete          WaitForFirstConsumer   false                  21h
custom               custom.tls/provisioner   Delete          WaitForFirstConsumer   false                  43h
glusterfs            rook.io/glusterfs        Delete          WaitForFirstConsumer   false                  54m
```

> The `name` label reflecting the resource name is mandatory, otherwise filtering of resources cannot be put in place

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  labels:
    name: cephfs
  name: cephfs
provisioner: cephfs
```

## Ingress Classes

As for Storage Class, also Ingress Class can be enforced.

```yaml
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - kind: User
    name: alice
    proxySettings:
    - kind: IngressClasses
      operations:
      - List
  ingressOptions:
    allowedClasses:
        allowed:
          - custom
        allowedRegex: "\\w+-lb"
```

In the Kubernetes cluster, we could have more Ingress Class resources, some of them forbidden and non-usable by the Tenant owner.

```bash
$ kubectl --context admin@mycluster get ingressclasses
NAME              CONTROLLER                 PARAMETERS                                      AGE
custom            example.com/custom         IngressParameters.k8s.example.com/custom        24h
external-lb       example.com/external       IngressParameters.k8s.example.com/external-lb   2s
haproxy-ingress   haproxy.tech/ingress                                                       4d
internal-lb       example.com/internal       IngressParameters.k8s.example.com/external-lb   15m
nginx             nginx.plus/ingress                                                         5d
```

The expected output using `capsule-proxy` is the retrieval of the `custom` Ingress Class as well the other ones matching the regex `\w+-lb`.

```bash
$ kubectl --context alice-oidc@mycluster get ingressclasses
NAME              CONTROLLER                 PARAMETERS                                      AGE
custom            example.com/custom         IngressParameters.k8s.example.com/custom        24h
external-lb       example.com/external       IngressParameters.k8s.example.com/external-lb   2s
internal-lb       example.com/internal       IngressParameters.k8s.example.com/internal-lb   15m
```

> The `name` label reflecting the resource name is mandatory, otherwise filtering of resources cannot be put in place

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  labels:
    name: external-lb
  name: external-lb
spec:
  controller: example.com/ingress-controller
  parameters:
    apiGroup: k8s.example.com
    kind: IngressParameters
    name: external-lb
```

## Priority Classes

Allowed PriorityClasses assigned to a Tenant Owner can be enforced as follows:

```yaml
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - kind: User
    name: alice
    proxySettings:
    - kind: PriorityClasses
      operations:
      - List
  priorityClasses:
    allowed:
      - custom
    allowedRegex: "\\w+priority"
```

In the Kubernetes cluster we could have more PriorityClasses resources, some of them forbidden and non-usable by the Tenant owner.

```bash
$ kubectl --context admin@mycluster get priorityclasses.scheduling.k8s.io
NAME                      VALUE        GLOBAL-DEFAULT   AGE
custom                    1000         false            18s
maxpriority               1000         false            18s
minpriority               1000         false            18s
nonallowed                1000         false            8m54s
system-cluster-critical   2000000000   false            3h40m
system-node-critical      2000001000   false            3h40m
```

The expected output using `capsule-proxy` is the retrieval of the `custom` PriorityClass as well the other ones matching the regex `\w+priority`.

```bash
$ kubectl --context alice-oidc@mycluster get ingressclasses
NAME                      VALUE        GLOBAL-DEFAULT   AGE
custom                    1000         false            18s
maxpriority               1000         false            18s
minpriority               1000         false            18s
```

> The `name` label reflecting the resource name is mandatory, otherwise filtering of resources cannot be put in place

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  labels:
    name: custom
  name: custom
value: 1000
globalDefault: false
description: "Priority class for Tenants"
```

## Runtime Classes

Allowed RuntimeClasses assigned to a Tenant Owner can be enforced as follows:

```yaml
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata:
  name: oil
spec:
  owners:
  - kind: User
    name: alice
    proxySettings:
    - kind: PriorityClasses
      operations:
      - List
  runtimeClasses:
    matchExpressions:
      - key: capsule.clastix.io/qos
        operator: Exists
        values:
          - bronze
          - silver 
```

In the Kubernetes cluster we could have more RuntimeClasses resources, some of them forbidden and non-usable by the Tenant owner.

```bash
$ kubectl --context admin@mycluster get runtimeclasses.node.k8s.io --show-labels
NAME      HANDLER           AGE   LABELS
bronze    bronze            21h   capsule.clastix.io/qos=bronze
default   myconfiguration   21h   <none>
gold      gold              21h   capsule.clastix.io/qos=gold
silver    silver            21h   capsule.clastix.io/qos=silver
```

The expected output using `capsule-proxy` is the retrieval of the `bronze` and `silver` ones.

```bash
$ kubectl --context alice-oidc@mycluster get runtimeclasses.node.k8s.io
NAME     HANDLER   AGE
bronze   bronze    21h
silver   silver    21h
```

> `RuntimeClass` is one of the latest implementations in Capsule Proxy and is adhering to the new selection strategy based on labels selector, rather than exact match and regex ones.
>
> The latter ones are going to be deprecated in the upcoming releases of Capsule.

## Persistent Volumes

A Tenant can request persistent volumes through the `PersistentVolumeClaim` API, and get a volume from it.

Starting from release v0.2.0, all the `PersistentVolumes` are labelled with the Capsule label that is used by the Capsule Proxy to allow the retrieval.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
  finalizers:
    - kubernetes.io/pv-protection
  labels:
    capsule.clastix.io/tenant: oil
  name: data-01
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /mnt/data
    type: ""
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  volumeMode: Filesystem
```

> Please, notice the label `capsule.clastix.io/tenant` matching the Tenant name.

With that said, a multi-tenant cluster can be made of several volumes, each one for different tenants.

```bash
$ kubectl --context admin@mycluster get persistentvolumes --show-labels
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE   LABELS
data-01   10Gi       RWO            Retain           Available           manual                  17h   capsule.clastix.io/tenant=oil
data-02   10Gi       RWO            Retain           Available           manual                  17h   capsule.clastix.io/tenant=gas

```

For the `oil` Tenant, Alice has the required permission to list Volumes.

```yaml
apiVersion: capsule.clastix.io/v1beta2
kind: Tenant
metadata:
  name: oil
spec:
  owners:
    - kind: User
      name: alice
      proxySettings:
        - kind: PersistentVolumes
          operations:
            - List
```

The expected output using `capsule-proxy` is the retrieval of the PVs used currently, or in the past, by the PVCs in their Tenants.

```bash
$ kubectl --context alice-oidc@mycluster get persistentvolumes
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
data-01   10Gi       RWO            Retain           Available           manual                  17h
```

## ProxySetting Use Case
Consider a scenario, where a cluster admin creates a tenant and assigns ownership of the tenant to a user, the so-called tenant owner. Afterwards, tenant owner would in turn like to provide access to their cluster-scoped resources to a set of users (e.g. non-owners or tenant users), groups and service accounts, who doesn't require tenant-owner-level permissions.

Tenant Owner can provide access to the following cluster-scoped resources to their tenant users, groups and service account by creating `ProxySetting` resource
- `Nodes`
- `StorageClasses`
- `IngressClasses`
- `PriorityClasses`
- `RuntimeClasses`
- `PersistentVolumes`

Each Resource kind can be granted with the following verbs, such as:
- `List`
- `Update`
- `Delete`

These tenant users, groups and services accounts have less privileged access than tenant owners.

As a Tenant Owner `alice`, you can create a `ProxySetting` resource to allow `bob` to list nodes, storage classes, ingress classes and priority classes
```yaml
apiVersion: capsule.clastix.io/v1beta2
kind: ProxySetting
metadata:
  name: sre-readers
  namespace: solar-production
spec:
  subjects:
  - name: bob
    kind: User
    proxySettings:
    - kind: Nodes
      operations:
      - List
    - kind: StorageClasses
      operations:
      - List
    - kind: IngressClasses
      operations:
      - List
    - kind: PriorityClasses
      operations:
      - List
```
As a Tenant User `bob`, you can list nodes, storage classes, ingress classes and priority classes

```bash
$ kubectl auth can-i --context bob-oidc@mycluster get nodes
yes
$ kubectl auth can-i --context bob-oidc@mycluster get storageclasses
yes
$ kubectl auth can-i --context bob-oidc@mycluster get ingressclasses
yes
$ kubectl auth can-i --context bob-oidc@mycluster get priorityclasses
yes
```
