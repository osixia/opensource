# 🩺 Troubleshooting

Start with the pods and logs:

```bash
kubectl -n kube-network-detector get pods -o wide
kubectl -n kube-network-detector logs \
  -l app.kubernetes.io/name=kube-network-detector --prefix --tail=100
```

## The pod does not start

Check its events with `kubectl -n kube-network-detector describe pod <pod-name>`.

| Message or symptom | Check |
| --- | --- |
| CRD missing or access forbidden | Apply both CRDs and the supplied RBAC, or use `--disable-crds` for static targets |
| Invalid HMAC key | Use 64 hexadecimal characters, identical on every pod |
| Address already in use | Free port `9876`, or change the responder port consistently across pods, container ports and health probes |
| Admission rejection | Check that the namespace allows `hostNetwork` |
| No pod on a particular node | Check the Linux node selector and tolerations; custom taints need matching tolerations |

## A VIP has no label

- Check the address in the `VIP` object and allow a few probe cycles.
- Make sure at least one detector node does **not** have the VIP on a local interface. Only those nodes can observe it.
- Check routes and firewalls between that observer and the VIP on the responder port.
- Check that all responders use the same key and that node clocks are synchronized.
- Check Node patch and Lease permissions. API errors do not count as failed network probes.

A running pod and a passing TCP readiness probe only show that the responder accepts connections. They do not confirm correct VIP routing or successful label updates.

## A Peer has no label

Check the IP and TCP port from the affected node's network. The target does not need a detector, but it must accept a TCP connection.

Check the label name too: a CRD Peer named `database` in namespace `default` uses `peer-database.default`; a static peer named `database` uses `peer-database`.

## The wrong node is detected

Inspect your routing, NAT, WireGuard setup and proxies. An address absent from local interfaces can still take a different path from public inbound traffic. A Service, NodePort or load balancer in front of the probe port can change which node answers.

## Old labels remain

Labels stay on shutdown. Static target removal and prefix changes need explicit cleanup; see [Updates and cleanup](configuration.md#updates-and-cleanup).

For CRD targets, a running instance on the affected node must reconcile the deletion. A node that stays offline retains its labels until it returns.
