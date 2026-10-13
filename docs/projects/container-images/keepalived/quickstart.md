# ⚡ Quickstart

This guide shares one virtual IP (VIP) between two hosts. The VIP is held by one node at a time and moves to the other one when that node goes down.

| Host   | Address         |
| ------ | --------------- |
| Node 1 | `192.168.1.10`  |
| Node 2 | `192.168.1.11`  |
| VIP    | `192.168.1.231` |

## Requirements

- The `ip_vs` kernel module loaded on each host (`modprobe ip_vs`).
- Host networking (`--network=host`), so Keepalived manages the real interface of the host.
- The `NET_ADMIN`, `NET_RAW` and `NET_BROADCAST` capabilities.

## Run Two Nodes

Run one container per host. Only the peer address and the priority change between the two.

=== "Node 1"

    ``` bash
    docker run --detach --name keepalived \
      --network=host --cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=NET_BROADCAST \
      --env KEEPALIVED_INTERFACE="eth0" \
      --env KEEPALIVED_VIRTUAL_IPS="192.168.1.231" \
      --env KEEPALIVED_UNICAST_PEERS="192.168.1.11" \
      --env KEEPALIVED_PRIORITY="150" \
      --env KEEPALIVED_PASSWORD="s3cret" \
      osixia/keepalived
    ```

=== "Node 2"

    ``` bash
    docker run --detach --name keepalived \
      --network=host --cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=NET_BROADCAST \
      --env KEEPALIVED_INTERFACE="eth0" \
      --env KEEPALIVED_VIRTUAL_IPS="192.168.1.231" \
      --env KEEPALIVED_UNICAST_PEERS="192.168.1.10" \
      --env KEEPALIVED_PRIORITY="100" \
      --env KEEPALIVED_PASSWORD="s3cret" \
      osixia/keepalived
    ```

=== "Docker Compose (node 1)"

    ``` yaml
    services:
      keepalived:
        image: osixia/keepalived
        container_name: keepalived
        network_mode: host
        restart: unless-stopped
        cap_add:
          - NET_ADMIN
          - NET_RAW
          - NET_BROADCAST
        environment:
          KEEPALIVED_INTERFACE: "eth0"
          KEEPALIVED_VIRTUAL_IPS: "192.168.1.231"
          KEEPALIVED_UNICAST_PEERS: "192.168.1.11"
          KEEPALIVED_PRIORITY: "150"
          KEEPALIVED_PASSWORD: "s3cret"
    ```

Three things to get right:

- `KEEPALIVED_INTERFACE` is the interface name **on the host** (`eth0`, `ens18`, `enp2s0`...). Check it with `ip -br addr`.
- `KEEPALIVED_UNICAST_PEERS` lists the **other** nodes only, never the node itself.
- `KEEPALIVED_PASSWORD` must be the same on every node. VRRP only uses its first 8 characters.

With several values, separate them with spaces: `KEEPALIVED_VIRTUAL_IPS="192.168.1.231 192.168.1.232"`.

## Check That It Works

The node with the highest priority takes the VIP:

``` bash
docker logs keepalived
# ... INFO    VRRP INSTANCE VI_1 entered MASTER state

ip addr show eth0
# inet 192.168.1.10/24 ...
# inet 192.168.1.231/32 scope global eth0
```

Stop the container on node 1: within a few seconds node 2 logs `entered MASTER state` and holds the VIP.

Start node 1 again: it comes back as `BACKUP` and the VIP **stays on node 2**. This is intended, see [Failback](configuration.md#failback).

## Next Steps

- [Configuration](configuration.md): use your own template or your own `keepalived.conf`, hot reload, notify script.
- [Troubleshooting](troubleshooting.md): debugging options and common issues.
- [Environment Variables](environment-variables.md): all settings.
