# 🔧 Configuration

## Targets in Kubernetes

The detector watches `VIP` and `Peer` objects across all namespaces by default. Adding, changing or deleting an object takes effect without restarting the DaemonSet. The object's namespace must already exist.

| Target | Required `spec` fields | Default node label |
| --- | --- | --- |
| `VIP` | `address` | `kube-network-detector/<IP>=true` |
| `Peer` | `address`, `port` | `kube-network-detector/peer-<name>.<namespace>=true` |

For IPv6 VIPs, the suffix is `ipv6-` followed by the address's 32 hexadecimal digits. For example, `2001:db8::1` becomes `ipv6-20010db8000000000000000000000001`. Long Peer identities use a hash; check the logs for the resulting label.

Deleted or changed CRD targets have their obsolete labels cleaned up by each running instance on its own node. API errors retain the last known configuration. Instances that were offline clean up when they return in CRD mode.

## Useful settings

Set these in the DaemonSet container's `args`:

| Flag | Default | Use it to… |
| --- | --- | --- |
| `--interval` | `5s` | Change the time between probes |
| `--timeout` | `2s` | Allow more or less time for a connection |
| `--success-threshold` | `2` | Require more successes before adding a label |
| `--failure-threshold` | `3` | Require more failures before removing a label |
| `--port` | `9876` | Change the VIP responder port on every node |
| `--vip-label-prefix` | `kube-network-detector/` | Change the start of VIP label keys |
| `--peer-label-prefix` | `kube-network-detector/peer-` | Change the start of Peer label keys |
| `--dry-run` | `false` | Log proposed changes without writing labels or Leases |

For example, use `--timeout=5s` for a slower network. Keep keys, ports, prefixes and CRD mode consistent across pods.

Flags also accept environment variables: `--timeout` becomes `KUBE_NETWORK_DETECTOR_TIMEOUT`. Explicit flags win over environment variables.

Logging uses `--log-level` and `--log-format`, or `CONTAINER_LOG_LEVEL` and `CONTAINER_LOG_FORMAT`. Use `--log-level=debug` when investigating a problem.

## Static targets

To configure targets directly in the DaemonSet instead of using CRDs:

```yaml
args:
  - --disable-crds
  - --peers=database=10.0.0.20:5432,api=[2001:db8::1]:443
```

This peer-only configuration needs the node name and Node `get/patch` permissions. It needs no HMAC key, responder or Lease permissions. If adapting the supplied manifest, remove the responder's readiness/liveness probes and port configuration too: there is no listening server in this mode.

A static peer named `database` uses `kube-network-detector/peer-database`, without a namespace suffix.

For VIPs, also add `--vips=203.0.113.10,2001:db8::1` and keep the shared key, responder and Lease permissions. VIPs need Node `get/list/patch` and Lease `get/create/update` permissions. The supplied manifest includes these.

Keep the Downward API identity variables from the supplied manifest:

| Variable | Source | Needed for |
| --- | --- | --- |
| `KUBE_NETWORK_DETECTOR_NODE_NAME` | `spec.nodeName` | All modes |
| `KUBE_NETWORK_DETECTOR_POD_NAMESPACE` | `metadata.namespace` | VIPs or CRD discovery |
| `KUBE_NETWORK_DETECTOR_POD_UID` | `metadata.uid` | VIPs or CRD discovery |

Without `--disable-crds`, static targets and CRD targets are combined.

## Try changes without writing labels

Set `--dry-run=true` on every pod. Probes still run and proposed changes appear in the logs, but the detector writes no labels, inventory annotations or Leases. VIP observers probe without leader election in this mode.

During a rollout, pods still using `--dry-run=false` can continue to write changes.

## Updates and cleanup

Changing DaemonSet arguments triggers a rollout. If you change the HMAC Secret, restart the pods so they load it again:

```bash
kubectl -n kube-network-detector rollout restart daemonset/kube-network-detector
kubectl -n kube-network-detector rollout status daemonset/kube-network-detector
```

Only one HMAC key is supported at a time. Mixed keys during rotation can cause failed probes and label removal.

Static targets leave their labels behind when removed. After every pod uses the new configuration, remove each obsolete label explicitly, for example:

```bash
kubectl label nodes --all kube-network-detector/peer-database-
```

Shutdown also leaves labels in place. Keep CRD mode running until deleted targets have been cleaned up; switching it off does not clean old CRD labels. Avoid multiple deployments managing the same labels or sharing the CRD inventory annotation on the same nodes.

[Full CLI and Go library reference](https://github.com/osixia/kube-network-detector#configuration)
