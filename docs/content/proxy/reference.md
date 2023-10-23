# CLI flags

- `capsule-configuration-name`: name of the `CapsuleConfiguration` resource which is containing the [Capsule configurations](/docs/general/references/#capsule-configuration) (default: `default`)
- `capsule-user-group` (deprecated): the old way to specify the user groups whose request must be intercepted by the proxy
- `ignored-user-group`: names of the groups whose requests must be ignored and proxy-passed to the upstream server
- `listening-port`: HTTP port the proxy listens to (default: `9001`)
- `oidc-username-claim`: the OIDC field name used to identify the user (default: `preferred_username`), the proper value can be extracted from the Kubernetes API Server flags
- `enable-ssl`: enable the bind on HTTPS for secure communication, allowing client-based certificate, also known as mutual TLS (default: `true`)
- `ssl-cert-path`: path to the TLS certificate, then TLS mode is enabled (default: `/opt/capsule-proxy/tls.crt`)
- `ssl-key-path`: path to the TLS certificate key, when TLS mode is enabled (default: `/opt/capsule-proxy/tls.key`)
- `rolebindings-resync-period`: resync period for RoleBinding resources reflector, lower values can help if you're facing [flaky etcd connection](https://github.com/projectcapsule/capsule-proxy/issues/174) (default: `10h`)
