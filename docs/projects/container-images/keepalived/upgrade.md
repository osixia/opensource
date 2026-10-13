# ⏫ Upgrade

## From 2.1.5 and Older

Version 2.3.4 moved the image to the new [osixia/baseimage](../baseimage/index.md). Keepalived itself behaves the same, but the way the container is configured changed. Review the points below before upgrading.

| Before                                                                  | Now                                                         |
| ----------------------------------------------------------------------- | ----------------------------------------------------------- |
| `KEEPALIVED_UNICAST_PEERS="#PYTHON2BASH:['192.168.1.10', '192.168.1.11']"` | `KEEPALIVED_UNICAST_PEERS="192.168.1.10 192.168.1.11"`      |
| `KEEPALIVED_VIRTUAL_IPS="#PYTHON2BASH:['192.168.1.231']"`               | `KEEPALIVED_VIRTUAL_IPS="192.168.1.231"`                    |
| `KEEPALIVED_NOTIFY`                                                     | `KEEPALIVED_NOTIFY_SCRIPT`                                  |
| `KEEPALIVED_COMMAND_LINE_ARGUMENTS="--log-detail"`                      | `docker run ... osixia/keepalived -- --log-detail`          |
| Config mounted at `/container/service/keepalived/assets/keepalived.conf` | Config mounted at `/etc/keepalived/keepalived.conf`         |
| Template placeholders `{{ KEEPALIVED_PRIORITY }}`                       | Template placeholders `${KEEPALIVED_PRIORITY}`              |
| YAML files in `/container/environment/01-custom/`                       | Plain environment variables (`--env`, `--env-file`)         |

Other changes:

- `--log-detail --dump-conf` are no longer passed to Keepalived by default. Add them after `--` if you rely on them.
- A mounted `keepalived.conf` is used as is: placeholders are only replaced in a template, see [Your Own Template](configuration.md#your-own-template).
- The default notify script moved to `/container/services/keepalived/assets/scripts/notify.sh`.
- The image name is unchanged (`osixia/keepalived`). The GitHub repository is now [osixia/container-keepalived](https://github.com/osixia/container-keepalived).

### Example

=== "Before"

    ``` bash
    docker run --detach \
      --network=host --cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=NET_BROADCAST \
      --env KEEPALIVED_INTERFACE="eth0" \
      --env KEEPALIVED_UNICAST_PEERS="#PYTHON2BASH:['192.168.1.10', '192.168.1.11']" \
      --env KEEPALIVED_VIRTUAL_IPS="#PYTHON2BASH:['192.168.1.231']" \
      --env KEEPALIVED_COMMAND_LINE_ARGUMENTS="--log-detail" \
      osixia/keepalived:2.1.5
    ```

=== "Now"

    ``` bash
    docker run --detach \
      --network=host --cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=NET_BROADCAST \
      --env KEEPALIVED_INTERFACE="eth0" \
      --env KEEPALIVED_UNICAST_PEERS="192.168.1.11" \
      --env KEEPALIVED_VIRTUAL_IPS="192.168.1.231" \
      osixia/keepalived:2.4.3 -- --log-detail
    ```

Upgrade the backup node first, check its logs, then upgrade the master.

## Between 2.x Versions

Change the image tag and recreate the container, one node at a time. Image changes are listed in the [changelog](https://github.com/osixia/container-keepalived/blob/master/CHANGELOG.md), Keepalived changes in the upstream [release notes](https://www.keepalived.org/release-notes/).
