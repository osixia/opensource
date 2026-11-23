# ⚡ Quickstart

This guide installs the detector and checks connectivity to a database. Add the optional VIP target if you also want to detect which node receives traffic for a virtual IP.

## 1. Get the manifest

You need `kubectl`, access to create CRDs and cluster RBAC, and a cluster that allows `hostNetwork`. The supplied CRDs require Kubernetes 1.30 or newer.

```bash
git clone https://github.com/osixia/kube-network-detector.git
cd kube-network-detector
```

Open `docs/examples/kubernetes/kube-network-detector.yaml` and replace the example Secret's `stringData.hmac-key` with a new key:

```bash
openssl rand -hex 32
```

Use an image built from the same source revision as the manifest, with CRD discovery support. Set the manifest's `image` field to the matching tag or digest before deploying.

All detector pods must use the same key. The default installation starts a VIP responder even if you only declare Peer objects, so it still needs this key and a free TCP port `9876` on each node. For peer-only operation without a responder, see [static targets](configuration.md#static-targets).

## 2. Install

```bash
kubectl apply -f docs/examples/kubernetes/kube-network-detector.yaml
kubectl wait --for=condition=Established --timeout=60s \
  crd/vips.network.cloud.osixia.net crd/peers.network.cloud.osixia.net
kubectl -n kube-network-detector rollout status daemonset/kube-network-detector
```

The manifest installs the CRDs, Secret, RBAC and DaemonSet. It creates no targets yet.

## 3. Add a target

Save this as `my-targets.yaml`. Replace `10.0.0.20` and `5432` with your database's IP and TCP port:

```yaml
apiVersion: network.cloud.osixia.net/v1alpha1
kind: Peer
metadata:
  name: database
  namespace: default
spec:
  address: 10.0.0.20
  port: 5432
```

```bash
kubectl apply -f my-targets.yaml
kubectl get nodes -L kube-network-detector/peer-database.default
```

After two successful probes, nodes that can connect show `true`. The default probe interval is five seconds; allow a few cycles for discovery and confirmation.

To check a VIP too, save this as `my-vip.yaml`, using a VIP that your network already routes to a node:

```yaml
apiVersion: network.cloud.osixia.net/v1alpha1
kind: VIP
metadata:
  name: gateway
  namespace: default
spec:
  address: 203.0.113.10
```

```bash
kubectl apply -f my-vip.yaml
kubectl get nodes -L kube-network-detector/203.0.113.10
```

An observer whose interfaces do not have the VIP must be able to reach it on TCP port `9876`. A node with the VIP on a local interface responds to probes but cannot observe that VIP itself. Do not put a Service or ingress in front of the probe port: the probe must reach the actual node through the VIP.

## 4. Use the label

For example, add this to a pod's `spec` to schedule it on a node that can reach the database:

```yaml
nodeSelector:
  kube-network-detector/peer-database.default: "true"
```

The scheduler uses the label when placing the pod. Removing it later does not evict a pod that is already running.

Read [Configuration](configuration.md) for dry-run and cleanup, or [Troubleshooting](troubleshooting.md) if no labels appear.
