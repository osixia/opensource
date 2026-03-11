# 🔤 Environment Variables

| Variable                        | Description                                                           | Default                                                                     |
| ------------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `KEEPALIVED_CONF`               | Path to the Keepalived configuration file                             | `/etc/keepalived/keepalived.conf`                                           |
| `KEEPALIVED_CONF_TEMPLATE`      | Path to the configuration template used to generate `keepalived.conf` | `/container/services/keepalived-conf/assets/confs/keepalived.conf.template` |
| `KEEPALIVED_CONF_RELOAD_SCRIPT` | Script executed when configuration changes are detected               | `/container/services/keepalived-conf/assets/scripts/reload.sh`              |
| `KEEPALIVED_INTERFACE`          | Network interface used by VRRP                                        | `eth0`                                                                      |
| `KEEPALIVED_STATE`              | Initial VRRP state (`MASTER` or `BACKUP`)                             | `BACKUP`                                                                    |
| `KEEPALIVED_ROUTER_ID`          | VRRP router ID                                                        | `51`                                                                        |
| `KEEPALIVED_PRIORITY`           | VRRP priority of the node                                             | `150`                                                                       |
| `KEEPALIVED_UNICAST_PEERS`      | List of peer nodes used for VRRP unicast communication                | `192.168.1.10` `192.168.1.11`                                               |
| `KEEPALIVED_VIRTUAL_IPS`        | Virtual IP addresses managed by Keepalived                            | `192.168.1.231` `192.168.1.232`                                             |
| `KEEPALIVED_PASSWORD`           | VRRP authentication password                                          | `d0cker`                                                                    |
| `KEEPALIVED_NOTIFY_SCRIPT`      | Script executed when Keepalived state changes                         | `/container/services/keepalived/assets/scripts/notify.sh`                   |
