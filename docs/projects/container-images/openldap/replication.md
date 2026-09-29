# 🔁 Replication

Set `OPENLDAP_BOOTSTRAP_REPLICATION=true` to bootstrap **multi-provider** (multi-master) syncrepl replication of the data database: every node accepts writes and replicates them to all other nodes.

## How it works

At bootstrap, each node:

- gets an `olcServerID` for every host of `OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS` (1, 2, 3, ... in list order),
- adds the `syncprov` overlay to the data database,
- adds one `olcSyncRepl` consumer for every **other** host,
- creates a replication account `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_DN` (default `cn=replicator,<suffix>`) with read access and no size/time limits.

## Requirements

- `OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS` must contain **at least two hosts**, in the **same order on every node**.
- The FQDN of each node (`hostname -f`, set with `--hostname`) must match one of these hosts.
- `OPENLDAP_URLS` should use the node hostname so advertised endpoints match replication hosts, for example `ldap://ldap1.example.org:3890 ldaps://ldap1.example.org:6360 ldapi:///`.
- `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD` (plain text) must be **the same on every node**. If empty, each node generates a different one and replication fails.
- The suffix and bootstrap settings must be identical on every node.

## Example

For a local Docker setup, put the containers on the same user-defined network so each host can resolve the other one:

=== "docker run"

    ``` bash
    docker network create openldap-net

    docker run --name ldap1 --network openldap-net --hostname ldap1.example.org \
      -e OPENLDAP_URLS="ldap://ldap1.example.org:3890 ldaps://ldap1.example.org:6360 ldapi:///" \
      -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
      -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
      -e OPENLDAP_BOOTSTRAP_REPLICATION=true \
      -e OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS="ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890" \
      -e OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD="passw0rd" \
      --detach osixia/openldap

    docker run --name ldap2 --network openldap-net --hostname ldap2.example.org \
      -e OPENLDAP_URLS="ldap://ldap2.example.org:3890 ldaps://ldap2.example.org:6360 ldapi:///" \
      -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
      -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
      -e OPENLDAP_BOOTSTRAP_REPLICATION=true \
      -e OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS="ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890" \
      -e OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD="passw0rd" \
      --detach osixia/openldap
    ```

=== "docker compose"

    ``` yaml
    x-openldap-env: &openldap-env
      OPENLDAP_BOOTSTRAP_ORGANIZATION: "Keycloakers Org"
      OPENLDAP_BOOTSTRAP_SUFFIX: "dc=keycloakers,dc=org"
      OPENLDAP_BOOTSTRAP_REPLICATION: "true"
      OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS: "ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890"
      OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD: "passw0rd"

    services:
      ldap1:
        image: osixia/openldap
        hostname: ldap1.example.org
        networks: [openldap-net]
        environment:
          <<: *openldap-env
          OPENLDAP_URLS: "ldap://ldap1.example.org:3890 ldaps://ldap1.example.org:6360 ldapi:///"
        volumes:
          - ldap1-conf:/etc/openldap/slapd.d
          - ldap1-data:/var/lib/openldap/openldap-data
          - ldap1-backups:/var/lib/openldap/openldap-backups

      ldap2:
        image: osixia/openldap
        hostname: ldap2.example.org
        networks: [openldap-net]
        environment:
          <<: *openldap-env
          OPENLDAP_URLS: "ldap://ldap2.example.org:3890 ldaps://ldap2.example.org:6360 ldapi:///"
        volumes:
          - ldap2-conf:/etc/openldap/slapd.d
          - ldap2-data:/var/lib/openldap/openldap-data
          - ldap2-backups:/var/lib/openldap/openldap-backups

    networks:
      openldap-net:

    volumes:
      ldap1-conf:
      ldap1-data:
      ldap1-backups:
      ldap2-conf:
      ldap2-data:
      ldap2-backups:
    ```

!!! warning
    Each node generates its own root passwords unless you set `OPENLDAP_BOOTSTRAP_DATA_ROOT_PASSWORD_HASHED` and `OPENLDAP_BOOTSTRAP_CONFIG_ROOT_PASSWORD_HASHED`.
    Root passwords are stored in `cn=config`, which is not replicated: set them explicitly to get the same credentials on every node.

Check replication by adding an entry on `ldap1` and searching it on `ldap2`, or by comparing the `contextCSN` of the suffix on both nodes:

``` bash
for node in ldap1 ldap2; do
  docker exec $node container run -- ldapsearch -Y EXTERNAL -H ldapi:/// -LLL -s base \
    -b "dc=keycloakers,dc=org" contextCSN 2>/dev/null
done
```

## Replication over TLS

Set `OPENLDAP_BOOTSTRAP_REPLICATION_TLS=true` to add `OPENLDAP_BOOTSTRAP_REPLICATION_TLS_SYNC_REPL` (default `starttls=critical tls_reqcert=demand`) to every syncrepl consumer.
It defaults to the value of `OPENLDAP_BOOTSTRAP_TLS_REQUIRED`.

Configure [TLS](tls.md) on every node with certificates valid for the node hostnames and signed by the CA in `OPENLDAP_BOOTSTRAP_TLS_CA_CERT`.

## Customizing syncrepl

The consumer configuration is generated from `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_SYNC_REPL_TEMPLATE`:

```
rid=${OPENLDAP_BOOTSTRAP_REPLICATION_DATA_SYNC_REPL_RID}
provider=${OPENLDAP_BOOTSTRAP_REPLICATION_SYNC_REPL_PROVIDER}
bindmethod=simple
binddn="${OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_DN}"
credentials=${OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD}
searchbase="${OPENLDAP_BOOTSTRAP_SUFFIX}"
schemachecking=on
type=refreshAndPersist
retry="5 5 60 +"
timeout=3
```

`OPENLDAP_BOOTSTRAP_REPLICATION_DATA_SYNC_REPL_RID` and `OPENLDAP_BOOTSTRAP_REPLICATION_SYNC_REPL_PROVIDER` are computed for each remote host. Escape `$` (`\$`) when overriding this template so variables are rendered per host.

When `memberof` is enabled, `OPENLDAP_BOOTSTRAP_REPLICATION_MEMBEROF_SYNC_REPL` (default `exattrs=memberOf`) is appended: `memberOf` is not replicated but computed locally by each node's overlay.
