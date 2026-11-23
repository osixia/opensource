# 🔧 Configuration

There are three ways to configure Keepalived, from the simplest to the most flexible. Pick the first one that covers your needs.

| Need                                             | Use                                             |
| ------------------------------------------------ | ----------------------------------------------- |
| One VRRP instance, a few VIPs                    | [Environment variables](#environment-variables) |
| Health checks, extra options, still per-node env | [Your own template](#your-own-template)         |
| Full control                                     | [Your own keepalived.conf](#your-own-keepalivedconf) |

## How It Works

The image runs two services:

- `keepalived-conf` generates the configuration file at startup, then watches it and reloads Keepalived when it changes.
- `keepalived` runs the Keepalived process in the foreground.

The configuration file (`KEEPALIVED_CONF`, default `/etc/keepalived/keepalived.conf`) is generated **only if it does not exist yet**. If you mount your own file, it is used as is.

!!! warning "Environment variables and persisted configuration"

    Once the file exists, environment variables are no longer applied to it. If `/etc/keepalived` is on a volume, delete the generated file (or the volume) to regenerate it after changing a variable.

## Environment Variables

The default template creates one VRRP instance using unicast:

``` text
vrrp_instance VI_1 {
  interface ${KEEPALIVED_INTERFACE}

  state ${KEEPALIVED_STATE}
  virtual_router_id ${KEEPALIVED_ROUTER_ID}
  priority ${KEEPALIVED_PRIORITY}
  nopreempt

  unicast_peer {
    ${KEEPALIVED_UNICAST_PEERS}
  }

  virtual_ipaddress {
    ${KEEPALIVED_VIRTUAL_IPS}
  }

  authentication {
    auth_type PASS
    auth_pass ${KEEPALIVED_PASSWORD}
  }

  notify "${KEEPALIVED_NOTIFY_SCRIPT}"
}
```

See the [Quickstart](quickstart.md) for a complete example and [Environment Variables](environment-variables.md) for the list.

### Failback

The default template sets `nopreempt`: a node that comes back does **not** take the VIP from the current master, even with a higher priority. This avoids a second, useless switch.

`nopreempt` requires `KEEPALIVED_STATE=BACKUP` (the default) on every node. The first master is elected by priority. With `KEEPALIVED_STATE=MASTER`, Keepalived ignores `nopreempt` and the node takes the VIP back as soon as it returns.

## Your Own Template

Mount a template and point `KEEPALIVED_CONF_TEMPLATE` to it. Every `${VARIABLE}` is replaced by the matching environment variable, including variables of your own.

This keeps one template for all nodes, with per-node values in the environment.

``` text title="keepalived.conf.template"
vrrp_script check_haproxy {
  script "${CHECK_SCRIPT}"
  interval 2
}

vrrp_instance VI_1 {
  interface ${KEEPALIVED_INTERFACE}
  state BACKUP
  virtual_router_id ${KEEPALIVED_ROUTER_ID}
  priority ${KEEPALIVED_PRIORITY}

  unicast_peer {
    ${KEEPALIVED_UNICAST_PEERS}
  }

  virtual_ipaddress {
    ${KEEPALIVED_VIRTUAL_IPS}
  }

  track_script {
    check_haproxy
  }
}
```

``` bash
docker run --detach --name keepalived \
  --network=host --cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=NET_BROADCAST \
  --volume ./keepalived.conf.template:/etc/keepalived/keepalived.conf.template \
  --env KEEPALIVED_CONF_TEMPLATE="/etc/keepalived/keepalived.conf.template" \
  --env CHECK_SCRIPT="/usr/bin/nc -z 127.0.0.1 443" \
  --env KEEPALIVED_INTERFACE="eth0" \
  --env KEEPALIVED_VIRTUAL_IPS="192.168.1.231" \
  --env KEEPALIVED_UNICAST_PEERS="192.168.1.11" \
  osixia/keepalived
```

Here the node gives up the VIP when nothing listens on port 443 of the host.

## Your Own keepalived.conf

Mount your file at `KEEPALIVED_CONF`. No template is rendered and the `KEEPALIVED_*` settings of the default template are ignored.

``` bash
docker run --detach --name keepalived \
  --network=host --cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=NET_BROADCAST \
  --volume ./keepalived.conf:/etc/keepalived/keepalived.conf \
  osixia/keepalived
```

## Hot Reload

When the configuration file changes, Keepalived is reloaded (`SIGHUP`). The container is not restarted and a master keeps its VIPs if the new configuration still declares them.

``` bash
echo "# any change" >> ./keepalived.conf

docker logs keepalived
# ... INFO    Change detected on /etc/keepalived/keepalived.conf
# ... INFO    Reloading Keepalived ...
# ... Reload complete
```

!!! note "Single-file bind mounts"

    When you bind-mount a single file, write it **in place** (`cat > keepalived.conf`, `>>`). If an editor replaces it with a new file, Docker keeps the original file mounted inside the container. A manual reload then still reads the old contents.

    For editors that save by replacement, mount the containing directory instead. For example, place `keepalived.conf` in `./keepalived/` and use `--volume ./keepalived:/etc/keepalived:ro`. The watcher then sees file replacements too.

To run something else on change, set `KEEPALIVED_CONF_RELOAD_SCRIPT` to your own script.

## Notify Script

Keepalived calls a notify script on every state change. The default one writes the new state to the container logs:

``` text
... INFO    VRRP INSTANCE VI_1 entered MASTER state
```

To run your own actions (call a webhook, reload a service...), mount a script and set `KEEPALIVED_NOTIFY_SCRIPT`:

``` bash title="notify.sh"
#!/bin/bash

# send the output to the container logs
exec 1>/proc/1/fd/1 2>/proc/1/fd/2

TYPE=$1   # INSTANCE or GROUP
NAME=$2   # VI_1
STATE=$3  # MASTER, BACKUP, FAULT, STOP or DELETED

container log info "VRRP ${TYPE} ${NAME} entered ${STATE} state"

if [ "${STATE}" = "MASTER" ]; then
  wget -q -T 5 -O /dev/null "https://example.org/hook?node=$(hostname)"
fi
```

``` bash
chmod +x notify.sh

docker run ... \
  --volume ./notify.sh:/notify.sh \
  --env KEEPALIVED_NOTIFY_SCRIPT="/notify.sh" \
  osixia/keepalived
```

Set `KEEPALIVED_NOTIFY_SCRIPT=""` to disable it. With your own `keepalived.conf`, declare `notify` yourself.

## Keepalived Command-Line Arguments

Arguments after `--` are passed to the `keepalived` binary:

``` bash
docker run ... osixia/keepalived -- --log-detail --dump-conf
```

List them with:

``` bash
docker run --rm osixia/keepalived -x keepalived -- --help
```

## JSON Status

Keepalived is built with JSON support. Ask the running process to dump its VRRP state:

``` bash
docker exec keepalived sh -c \
  'kill -s $(keepalived --signum=JSON) $(cat /run/container/processes/keepalived.pid) && sleep 1 && cat /tmp/keepalived.json'
```

The output contains the state, priority and VIPs of each instance, which makes it easy to feed a monitoring check.
