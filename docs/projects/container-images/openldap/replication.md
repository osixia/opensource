# 🔁 Replication

Set `OPENLDAP_BOOTSTRAP_REPLICATION=true` to bootstrap multi-provider replication: every server accepts writes and replicates the data database to the others with syncrepl.

## Requirements

- `OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS` lists **two to nine** servers, in the **same order** on every server.
- The container hostname (FQDN) must match one of the hosts in that list.
- `OPENLDAP_URLS` must use the same host name as the list, so that each server recognizes its own URL.
- The replication password must be the **same on every server**. Set `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD` explicitly: if it is empty, each server generates a different one and replication fails.
- All servers must be bootstrapped with the same suffix and the same schemas.

## Two-Node Example

=== "Docker"

    Put both containers on the same user-defined network so each host can resolve the other one:

    ``` bash
    docker network create openldap-net
    ```

    First server, `ldap1.example.org`:

    ``` bash
    docker run --name ldap1 --network openldap-net --hostname ldap1.example.org \
      -e OPENLDAP_URLS="ldap://ldap1.example.org:3890 ldaps://ldap1.example.org:6360 ldapi:///" \
      -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
      -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
      -e OPENLDAP_BOOTSTRAP_REPLICATION=true \
      -e OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS="ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890" \
      -e OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD="passw0rd" \
      osixia/openldap:2.6.15-rc
    ```

    Second server, `ldap2.example.org`:

    ``` bash
    docker run --name ldap2 --network openldap-net --hostname ldap2.example.org \
      -e OPENLDAP_URLS="ldap://ldap2.example.org:3890 ldaps://ldap2.example.org:6360 ldapi:///" \
      -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
      -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
      -e OPENLDAP_BOOTSTRAP_REPLICATION=true \
      -e OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS="ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890" \
      -e OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD="passw0rd" \
      osixia/openldap:2.6.15-rc
    ```

=== "Docker Compose"

    ``` yaml
    services:
      ldap1:
        image: osixia/openldap:2.6.15-rc
        container_name: ldap1
        hostname: ldap1.example.org
        networks:
          - openldap-net
        ports:
          - "389:3890"
          - "636:6360"
        environment:
          OPENLDAP_URLS: "ldap://ldap1.example.org:3890 ldaps://ldap1.example.org:6360 ldapi:///"
          OPENLDAP_BOOTSTRAP_ORGANIZATION: "Keycloakers Org"
          OPENLDAP_BOOTSTRAP_SUFFIX: "dc=keycloakers,dc=org"
          OPENLDAP_BOOTSTRAP_REPLICATION: "true"
          OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS: "ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890"
          OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD: "passw0rd"
        volumes:
          - ldap1-conf:/etc/openldap/slapd.d
          - ldap1-data:/var/lib/openldap/openldap-data
          - ldap1-backups:/var/lib/openldap/openldap-backups

      ldap2:
        image: osixia/openldap:2.6.15-rc
        container_name: ldap2
        hostname: ldap2.example.org
        networks:
          - openldap-net
        environment:
          OPENLDAP_URLS: "ldap://ldap2.example.org:3890 ldaps://ldap2.example.org:6360 ldapi:///"
          OPENLDAP_BOOTSTRAP_ORGANIZATION: "Keycloakers Org"
          OPENLDAP_BOOTSTRAP_SUFFIX: "dc=keycloakers,dc=org"
          OPENLDAP_BOOTSTRAP_REPLICATION: "true"
          OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS: "ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890"
          OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD: "passw0rd"
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

An entry added on one server then appears on the other.

!!! warning "Passwords"

    Only the replication password is shared through this example. Root passwords are generated separately on each server
    unless you set `OPENLDAP_BOOTSTRAP_DATA_ROOT_PASSWORD_HASHED` and `OPENLDAP_BOOTSTRAP_CONFIG_ROOT_PASSWORD_HASHED`
    to the same values on all of them. Once replication is running, the data database content is the same everywhere.

## What Bootstrap Configures

- A server ID for each host, in list order (`olcServerID`).
- A replication account, `cn=replicator,<suffix>`, that can read the whole data database, with no size or time limit.
- One `olcSyncRepl` consumer per other server, in `refreshAndPersist` mode.
- Multi-provider mode on the data database (`olcMultiProvider: TRUE`).
- The `syncprov` overlay, loaded last.

Only the data database is replicated. The configuration database (`cn=config`) is not: apply configuration changes on each server.

## TLS

Replication uses StartTLS when `OPENLDAP_BOOTSTRAP_REPLICATION_TLS=true`. It defaults to the value of `OPENLDAP_BOOTSTRAP_TLS_REQUIRED`, so replication is encrypted as soon as [TLS is required](tls.md#require-tls).

Each server then verifies the certificate of the others (`starttls=critical tls_reqcert=demand`): certificates must match the host names in `OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS` and be signed by the configured CA.

## Settings

| Variable                                                 | Description                                              | Default                                      |
| -------------------------------------------------------- | -------------------------------------------------------- | -------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_REPLICATION_HOSTS`                   | Servers list; two to nine, same order on every server    | `ldap://ldap1.example.org:3890 ldap://ldap2.example.org:3890` |
| `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_DN`        | Replication account DN                                   | `cn=replicator,${OPENLDAP_BOOTSTRAP_SUFFIX}` |
| `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD`  | Plain replication password; generated if empty           | empty                                        |
| `OPENLDAP_BOOTSTRAP_REPLICATION_SYNCPROV_CHECKPOINT`     | `syncprov` checkpoint                                    | `100 10`                                     |
| `OPENLDAP_BOOTSTRAP_REPLICATION_TLS`                     | Use StartTLS for replication                             | `${OPENLDAP_BOOTSTRAP_TLS_REQUIRED}`         |
| `OPENLDAP_BOOTSTRAP_REPLICATION_MEMBEROF`                | Exclude `memberOf` from replication                      | `${OPENLDAP_BOOTSTRAP_MEMBEROF}`             |

The generated `olcSyncRepl` value can be changed with `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_SYNC_REPL_TEMPLATE`. See [Environment Variables](environment-variables.md#replication).

!!! note

    The replication password is stored in plain text in the `olcSyncRepl` configuration, as OpenLDAP requires.
    The configuration database is only readable by its root DN.

## Changing the Topology

Replication is configured at bootstrap only. Adding or removing a server later means updating `olcServerID` and `olcSyncRepl` on every existing server with `ldapmodify`, then bootstrapping the new server with the full host list.
