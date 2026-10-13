# 🔤 Environment Variables

## Configuration File

| Variable                        | Description                                                           | Default                                                                     |
| ------------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `KEEPALIVED_CONF`               | Path to the Keepalived configuration file                             | `/etc/keepalived/keepalived.conf`                                           |
| `KEEPALIVED_CONF_TEMPLATE`      | Path to the configuration template used to generate `keepalived.conf` | `/container/services/keepalived-conf/assets/confs/keepalived.conf.template` |
| `KEEPALIVED_CONF_RELOAD_SCRIPT` | Script executed when configuration changes are detected               | `/container/services/keepalived-conf/assets/scripts/reload.sh`              |

See [Configuration](configuration.md).

## Default Template

These variables are used by the default template. They are ignored when you mount your own `keepalived.conf`.

| Variable                   | Description                                                                          | Default                                                   |
| -------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------- |
| `KEEPALIVED_INTERFACE`     | Network interface used by VRRP, as named on the host                                 | `eth0`                                                    |
| `KEEPALIVED_STATE`         | Initial VRRP state (`MASTER` or `BACKUP`), see [Failback](configuration.md#failback) | `BACKUP`                                                  |
| `KEEPALIVED_ROUTER_ID`     | VRRP router ID (1-255), the same on every node sharing the VIPs                      | `51`                                                      |
| `KEEPALIVED_PRIORITY`      | VRRP priority of the node (1-254), the highest wins the election                     | `150`                                                     |
| `KEEPALIVED_UNICAST_PEERS` | Space-separated addresses of the other nodes                                         | `192.168.1.10 192.168.1.11`                               |
| `KEEPALIVED_VIRTUAL_IPS`   | Space-separated virtual IP addresses managed by Keepalived                           | `192.168.1.231 192.168.1.232`                             |
| `KEEPALIVED_PASSWORD`      | VRRP authentication password, the same on every node (8 characters used)             | `d0cker`                                                  |
| `KEEPALIVED_NOTIFY_SCRIPT` | Script executed when the VRRP state changes, empty to disable                        | `/container/services/keepalived/assets/scripts/notify.sh` |

!!! warning

    Always set `KEEPALIVED_UNICAST_PEERS`, `KEEPALIVED_VIRTUAL_IPS` and `KEEPALIVED_PASSWORD`: the defaults are examples.
