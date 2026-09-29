# ⚡ Quickstart

## Run a basic OpenLDAP server

``` bash
docker run --name openldap --hostname ldap.example.org -p 389:3890 -p 636:6360 \
  -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
  -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
  osixia/openldap
```

On first start, the container:

1. creates the `cn=config` configuration and an MDB database for `dc=keycloakers,dc=org`,
2. creates the organization entry,
3. generates random passwords for the config root (`cn=admin,cn=config`) and the database root (`cn=admin,dc=keycloakers,dc=org`) and prints them in the logs.

!!! note
    The container runs as the non-root `ldap` user (uid/gid `911`) and listens on unprivileged ports:
    `3890` (LDAP) and `6360` (LDAPS), as defined by `OPENLDAP_URLS`. Map them to `389` / `636` on the host with `-p`.

## Persist config, data and backups

For any real deployment, mount the config, data and backup directories:

=== "docker run"

    ``` bash
    docker run --name openldap --hostname ldap.example.org -p 389:3890 -p 636:6360 \
      -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
      -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
      -v openldap-conf:/etc/openldap/slapd.d \
      -v openldap-data:/var/lib/openldap/openldap-data \
      -v openldap-backups:/var/lib/openldap/openldap-backups \
      osixia/openldap
    ```

=== "docker compose"

    ``` yaml
    services:
      openldap:
        image: osixia/openldap
        container_name: openldap
        hostname: ldap.example.org
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

Bootstrap runs **only when both the config and data directories are empty**. On the next starts, the existing directory is reused and `OPENLDAP_BOOTSTRAP_*` variables are ignored.

!!! tip "Bind mounts"
    When using host directories instead of named volumes, make sure they are writable by uid/gid `911`:
    `chown -R 911:911 /data/openldap`.

## Verify the server is up

Retrieve the generated passwords from the container logs:

``` bash
docker logs openldap 2>&1 | grep -i password
```

Then search the base DN:

``` bash
docker exec openldap container run -- \
  ldapsearch -H ldap://localhost:3890 \
  -D "cn=admin,dc=keycloakers,dc=org" -w <database root password> \
  -b "dc=keycloakers,dc=org" "(objectClass=*)"
```

A successful response returns the organization entry.

!!! tip "Set your own passwords"
    To avoid random passwords, provide hashed values at first start with
    `OPENLDAP_BOOTSTRAP_DATA_ROOT_PASSWORD_HASHED` and `OPENLDAP_BOOTSTRAP_CONFIG_ROOT_PASSWORD_HASHED`.
    Hash a password with:

    ``` bash
    docker run --rm osixia/openldap run -- openldap-ctl password hash 'my-s3cret-passw0rd'
    ```

## Passing command-line arguments to slapd

Arguments specified after `--` are forwarded to the `slapd` process inside the container:

``` bash
docker run osixia/openldap -- -d -1
```

## Debugging

The `--log-level debug --bash` options from `osixia/baseimage` enable debug logging and launch an interactive shell.

If OpenLDAP keeps crashing, add `--skip-process` to start the container without launching `slapd`:

``` bash
docker run -it osixia/openldap --log-level debug --bash
docker run -it osixia/openldap --skip-process --log-level debug --bash
```

Increase the `slapd` log level with `OPENLDAP_DEBUG_LEVEL`:

``` bash
docker run -e OPENLDAP_DEBUG_LEVEL=-1 osixia/openldap
```

!!! note
    `OPENLDAP_DEBUG_LEVEL` only affects `slapd` logging. To increase container/runtime logs, use the baseimage flag `--log-level` (for example `--log-level debug`, or `--log-level trace` to print every command run by the startup scripts).

To see all available command-line options:

``` bash
docker run --rm osixia/openldap --help                # osixia/baseimage options
docker run --rm osixia/openldap -x openldap -- --help # slapd command-line options
```
