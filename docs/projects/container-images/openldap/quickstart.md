# ⚡ Quickstart

## Run a First Server

Run a basic OpenLDAP server:

``` bash
docker run --name openldap --hostname ldap.example.org -p 389:3890 -p 636:6360 \
  -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
  -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
  osixia/openldap:2.6.15-rc
```

On first start the container bootstraps a new directory: it creates the configuration, the database for the suffix and the root entry, then starts `slapd`.

Notes:

- The container listens on ports `3890` (LDAP) and `6360` (LDAPS), as defined by `OPENLDAP_URLS`.
- `OPENLDAP_BOOTSTRAP_SUFFIX` must start with `dc=` (for example `dc=example,dc=org`).
- Passwords are generated when no hashed value is provided and are printed in the logs, once.
- Bootstrap runs only when the configuration and data directories are empty.

## Keep Your Data

For persistent deployments, mount the configuration, data and backup directories:

=== "Docker"

    ``` bash
    docker run --name openldap --hostname ldap.example.org -p 389:3890 -p 636:6360 \
      -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
      -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
      -v openldap-conf:/etc/openldap/slapd.d \
      -v openldap-data:/var/lib/openldap/openldap-data \
      -v openldap-backups:/var/lib/openldap/openldap-backups \
      osixia/openldap:2.6.15-rc
    ```

=== "Docker Compose"

    ``` yaml
    services:
      openldap:
        image: osixia/openldap:2.6.15-rc
        container_name: openldap
        ports:
          - "389:3890"
          - "636:6360"
        environment:
          OPENLDAP_BOOTSTRAP_ORGANIZATION: "Keycloakers Org"
          OPENLDAP_BOOTSTRAP_SUFFIX: "dc=keycloakers,dc=org"
        volumes:
          - openldap-conf:/etc/openldap/slapd.d
          - openldap-data:/var/lib/openldap/openldap-data
          - openldap-backups:/var/lib/openldap/openldap-backups

    volumes:
      openldap-conf:
      openldap-data:
      openldap-backups:
    ```

| Path                                 | Content                                    |
| ------------------------------------ | ------------------------------------------ |
| `/etc/openldap/slapd.d`              | OpenLDAP configuration (`cn=config`)       |
| `/var/lib/openldap/openldap-data`    | Directory data (MDB database)              |
| `/var/lib/openldap/openldap-backups` | Backups created by `openldap-ctl backup`   |

The container runs as the `ldap` user (uid `911`, gid `911`). Bind-mounted host directories must be writable by that user.

## Verify the Server Is Up

Retrieve the generated passwords from the container logs:

``` bash
docker logs openldap 2>&1 | grep -i password
```

Then search the base DN to confirm the server is responding, replacing `<database root password>` with the password from the logs:

``` bash
docker exec openldap container run -- \
  ldapsearch -H ldap://localhost:3890 \
  -D "cn=admin,dc=keycloakers,dc=org" -w '<database root password>' \
  -b "dc=keycloakers,dc=org" "(objectClass=*)"
```

A successful response returns the organization entry.

!!! tip "Set your own passwords"

    To choose the passwords instead of reading generated ones from the logs, hash them with
    `openldap-ctl password hash` and pass the result in `OPENLDAP_BOOTSTRAP_DATA_ROOT_PASSWORD_HASHED`
    and `OPENLDAP_BOOTSTRAP_CONFIG_ROOT_PASSWORD_HASHED`. See [LDAP Toolkit](ldap-toolkit.md#password-helpers).

## Passing Command-Line Arguments to OpenLDAP

Arguments specified after `--` are forwarded to the `slapd` process inside the container:

``` bash
docker run osixia/openldap:2.6.15-rc -- -d -1
```

## Debugging

To debug the container manually, you can start it with an interactive shell.

The `--log-level debug --bash` options from `osixia/baseimage` enable debug logging and launch an interactive shell.

If OpenLDAP keeps crashing, add `--skip-process` to start the container without launching service processes.

``` bash
docker run -it osixia/openldap:2.6.15-rc --log-level debug --bash
docker run -it osixia/openldap:2.6.15-rc --skip-process --log-level debug --bash
```

You can also increase the `slapd` log level with `OPENLDAP_DEBUG_LEVEL`:

``` bash
docker run -e OPENLDAP_DEBUG_LEVEL=-1 osixia/openldap:2.6.15-rc
```

!!! note

    `OPENLDAP_DEBUG_LEVEL` only affects `slapd` logging. To increase container logs, use the baseimage flag
    `--log-level` (for example `--log-level debug`). Bootstrap errors reported by `slapadd` are logged at the `debug` level.

To see all available command-line options:

``` bash
docker run --rm osixia/openldap:2.6.15-rc --help # osixia/baseimage options
docker run --rm osixia/openldap:2.6.15-rc -x openldap -- --help # slapd command-line options
```
