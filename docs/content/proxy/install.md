# Install the proxy

Capsule Proxy is an optional add-on of the main Capsule Operator, so make sure you have a working instance of Capsule before attempting to install it.
Use the `capsule-proxy` only if you want Tenant Owners to list their Cluster-Scope resources.

The `capsule-proxy` can be deployed in standalone mode, e.g. running as a pod bridging any Kubernetes client to the APIs server.
Optionally, it can be deployed as a sidecar container in the backend of a dashboard.

Running outside a Kubernetes cluster is also viable, although a valid `KUBECONFIG` file must be provided, using the environment variable `KUBECONFIG` or the default file in `$HOME/.kube/config`.

A Helm Chart is available [here](https://github.com/projectcapsule/capsule-proxy/blob/master/charts/capsule-proxy/README.md).

Depending on your environment, you can expose the `capsule-proxy` by:

- Ingress
- NodePort Service
- LoadBalance Service
- HostPort
- HostNetwork

Here how it looks like when exposed through an Ingress Controller:

```
                +-----------+          +-----------+         +-----------+
 kubectl ------>|:443       |--------->|:9001      |-------->|:6443      |
                +-----------+          +-----------+         +-----------+
                ingress-controller     capsule-proxy         kube-apiserver
``` 
