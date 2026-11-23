# ⏫ Upgrade

OpenLDAP database files are not guaranteed to be readable by a newer OpenLDAP release. The safe way to upgrade is to export the databases with the **old** version and import them with the **new** one.

The image automates this with two services:

- `openldap-pre-upgrade` exports the databases. It runs with the **previous** image version, as an init container.
- `openldap-upgrade` imports them. It runs with the **new** image version, in the main container, before `slapd` starts.

## The Principle

Always run two containers on the same volumes, one version apart:

| Container      | Image version | Command                   |
| -------------- | ------------- | ------------------------- |
| Init container | `n-1`         | `-x openldap -x openldap-pre-upgrade --skip-process` |
| Main container | `n`           | default                   |

To upgrade, increment both versions: the init container moves to `n`, the main container to `n+1`.

On each start:

1. The init container compares its own version with the version recorded in the configuration directory. If they match, the directory has not been migrated yet: it backs up both databases, then exits.
2. The main container compares its version with the recorded one. If they differ, it restores the backup made by the init container, then records its own version.
3. `slapd` starts.

On following starts, the recorded version already matches the main container: the init container does nothing, and neither does the upgrade service.

Because the backup is taken by the init container right before the main container starts, with `slapd` stopped, nothing is lost between the export and the import.

## Examples

Replace `<previous-version>` and `<new-version>` with image tags. The three volumes must be shared by both containers.

=== "Docker Compose"

    ``` yaml
    services:
      openldap-pre-upgrade:
        image: osixia/openldap:<previous-version>
        command: ["-x", "openldap", "-x", "openldap-pre-upgrade", "--skip-process"]
        volumes:
          - openldap-conf:/etc/openldap/slapd.d
          - openldap-data:/var/lib/openldap/openldap-data
          - openldap-backups:/var/lib/openldap/openldap-backups

      openldap:
        image: osixia/openldap:<new-version>
        depends_on:
          openldap-pre-upgrade:
            condition: service_completed_successfully
        ports:
          - "389:3890"
          - "636:6360"
        volumes:
          - openldap-conf:/etc/openldap/slapd.d
          - openldap-data:/var/lib/openldap/openldap-data
          - openldap-backups:/var/lib/openldap/openldap-backups

    volumes:
      openldap-conf:
      openldap-data:
      openldap-backups:
    ```

=== "Kubernetes"

    ``` yaml
    spec:
      securityContext:
        fsGroup: 911
      initContainers:
        - name: openldap-pre-upgrade
          image: osixia/openldap:<previous-version>
          args: ["-x", "openldap", "-x", "openldap-pre-upgrade", "--skip-process"]
          volumeMounts:
            - { name: conf, mountPath: /etc/openldap/slapd.d }
            - { name: data, mountPath: /var/lib/openldap/openldap-data }
            - { name: backups, mountPath: /var/lib/openldap/openldap-backups }
      containers:
        - name: openldap
          image: osixia/openldap:<new-version>
          ports:
            - { name: ldap, containerPort: 3890 }
            - { name: ldaps, containerPort: 6360 }
          volumeMounts:
            - { name: conf, mountPath: /etc/openldap/slapd.d }
            - { name: data, mountPath: /var/lib/openldap/openldap-data }
            - { name: backups, mountPath: /var/lib/openldap/openldap-backups }
    ```

=== "Docker"

    ``` bash
    docker stop openldap && docker rm openldap

    docker run --rm \
      -v openldap-conf:/etc/openldap/slapd.d \
      -v openldap-data:/var/lib/openldap/openldap-data \
      -v openldap-backups:/var/lib/openldap/openldap-backups \
      osixia/openldap:<previous-version> -x openldap -x openldap-pre-upgrade --skip-process

    docker run --name openldap -p 389:3890 -p 636:6360 \
      -v openldap-conf:/etc/openldap/slapd.d \
      -v openldap-data:/var/lib/openldap/openldap-data \
      -v openldap-backups:/var/lib/openldap/openldap-backups \
      osixia/openldap:<new-version>
    ```

If the backup fails, the init container exits with an error and the main container does not start.

Selecting `openldap` with `--skip-process` prepares custom modules and schemas before the backup, without starting slapd. Mount the same custom resources in both containers when you use them.

## Migration Level

`OPENLDAP_UPGRADE_MIGRATION_LEVEL` defines which version change triggers a migration:

| Level             | A migration runs when…             | Example                   |
| ----------------- | ---------------------------------- | ------------------------- |
| `major`           | the major version changes          | `2.x` → `3.x`             |
| `minor` (default) | the major or minor version changes | `2.6.x` → `2.7.x`         |
| `patch`           | any version number changes         | `2.6.10` → `2.6.15`       |

With the default level, moving from `2.6.10` to `2.6.15` does not migrate anything: the new image starts directly on the existing files.

Use the same level in the init container and in the main container.

!!! note

    While both containers are within the same level, for example `2.6.10` and `2.6.15` at the `minor` level,
    the init container takes a backup on every start. It is overwritten each time.

## What You See in the Logs

A migration from `2.6` to `2.7`:

``` text
INFO    Migration level set to 'minor'
INFO    OpenLDAP version: 2.7.0-r0, conf version: 2.6.10-r0
WARNING Migration required: 2.6 → 2.7
WARNING Deleting data in /etc/openldap/slapd.d ...
INFO    Restoring OpenLDAP config ...
WARNING Deleting data in /var/lib/openldap/openldap-data ...
INFO    Restoring OpenLDAP data ...
INFO    Writing OpenLDAP version 2.7.0-r0 to /etc/openldap/slapd.d/.version ...
```

A normal start:

``` text
INFO    OpenLDAP version: 2.7.0-r0, conf version: 2.7.0-r0
INFO    No migration needed at minor level
```

## Safeguards

The upgrade service refuses to continue in two cases. In both, the container stops before `slapd` starts and nothing is changed.

**The backup is missing.** The main container needs a migration but the init container did not run:

``` text
ERROR   Missing /var/lib/openldap/openldap-backups/openldap-upgrade-2.7-config.gz or
        /var/lib/openldap/openldap-backups/openldap-upgrade-2.7-data.gz.
        Set OPENLDAP_UPGRADE_FORCE=true to bypass.
```

Run the init container with the previous image version, then start the main container again.

**The new version is older than the recorded one.** Downgrades are refused:

``` text
ERROR   Downgrade forbidden: 2.7 → 2.6. Set OPENLDAP_UPGRADE_FORCE=true to bypass.
```

### Forcing

`OPENLDAP_UPGRADE_FORCE=true` bypasses both safeguards:

- a downgrade is no longer refused;
- when the backup is missing, the migration is skipped and `slapd` starts on the existing files. The recorded version is **not** updated, so the warning is logged on every start until a migration actually runs.

!!! danger

    Forcing means running a version of OpenLDAP on database files written by another one. Take a
    [backup](backup-restore.md) first and keep it outside the volumes.

## Files

| File                                                           | Content                                     |
| -------------------------------------------------------------- | ------------------------------------------- |
| `/etc/openldap/slapd.d/.version`                               | OpenLDAP version of the last migration      |
| `/var/lib/openldap/openldap-backups/openldap-upgrade-<version>-config.gz` | Configuration backup taken by the init container |
| `/var/lib/openldap/openldap-backups/openldap-upgrade-<version>-data.gz`   | Data backup taken by the init container     |

`<version>` is the recorded version truncated to the migration level, for example `2.6` at the `minor` level. Upgrade backups are kept after the migration; remove them when you no longer need them.

## Settings

| Variable                               | Description                                              | Default                         |
| -------------------------------------- | -------------------------------------------------------- | ------------------------------- |
| `OPENLDAP_UPGRADE_MIGRATION_LEVEL`     | Version change that triggers a migration                 | `minor`                         |
| `OPENLDAP_UPGRADE_FORCE`               | Bypass the missing-backup and downgrade safeguards       | `false`                         |
| `OPENLDAP_UPGRADE_BACKUP_FILES_PREFIX` | Prefix of upgrade backup files                           | `openldap-upgrade`              |
| `OPENLDAP_UPGRADE_CONF_VERSION_FILE`   | File storing the recorded version                        | `${OPENLDAP_CONF_DIR}/.version` |

## Replicated Servers

Upgrade one server at a time, each with its own init container, and wait for it to be back and in sync before upgrading the next one.

## Migrating From v1

v2 is a complete rewrite and v1 deployments are **not** migrated automatically. The procedure above only applies between v2 versions.

To move from v1 to v2, export the data from the v1 server as LDIF, start a new v2 server with the same suffix, then import the entries. Configuration (`cn=config`) is not portable as is: recreate it with [bootstrap](bootstrap/index.md) settings and [custom LDIF](bootstrap/custom-ldif.md).
