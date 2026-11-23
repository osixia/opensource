# 🩺 Troubleshooting

## Debugging

Show more logs, from the image and from Keepalived:

``` bash
docker run ... osixia/keepalived --log-level debug -- --log-detail --dump-conf
```

`--log-level debug` also prints the generated configuration file at startup.

Open a shell next to Keepalived, or without starting it if it keeps crashing:

``` bash
docker run -it ... osixia/keepalived --log-level debug --bash
docker run -it ... osixia/keepalived --skip-process --log-level debug --bash
```

List the available options:

``` bash
docker run --rm osixia/keepalived --help # osixia/baseimage options
docker run --rm osixia/keepalived -x keepalived -- --help # keepalived options
```

## Common Issues

### Both nodes are MASTER

The nodes do not see each other's VRRP adverts.

- Allow the VRRP protocol (IP protocol `112`) between the nodes in your firewalls.
- Check that `KEEPALIVED_UNICAST_PEERS` contains the addresses of the other nodes, on the network of `KEEPALIVED_INTERFACE`.
- Check that `KEEPALIVED_ROUTER_ID` and `KEEPALIVED_PASSWORD` are the same on every node.

### `equal priority advert received from remote host with our IP address`

The node is listed in its own `KEEPALIVED_UNICAST_PEERS`. Keep only the other nodes.

### `Non-existent interface specified in configuration`

Keepalived stops and restarts in a loop because `KEEPALIVED_INTERFACE` does not exist. With `--network=host` it must be the interface name of the host, rarely `eth0`. Check it with `ip -br addr`.

### Changing an environment variable has no effect

The configuration file already exists, so it is not generated again. See [How It Works](configuration.md#how-it-works).

### The configuration is not reloaded after an edit

If you bind-mounted a single file, an editor may have replaced the host file while Docker still mounts the old one. Mount the containing directory or write the file in place. See [Hot Reload](configuration.md#hot-reload).

### `nopreempt will not work with initial state MASTER`

Expected with `KEEPALIVED_STATE=MASTER`. See [Failback](configuration.md#failback).

### The VIP does not come back to the first node

Expected with the default template. See [Failback](configuration.md#failback).

### A service cannot bind the VIP on the backup node

The VIP only exists on the master. Bind the service to `0.0.0.0`, or allow binding a missing address on the host:

``` bash
sysctl -w net.ipv4.ip_nonlocal_bind=1
```

### The VIP is up but unreachable in a cloud network

Many cloud networks drop traffic for addresses they did not assign. The VIP has to be declared to the provider (floating IP, allowed address pairs...), usually from a [notify script](configuration.md#notify-script).
