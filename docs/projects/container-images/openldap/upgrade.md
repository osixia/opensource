# ⬆️ Upgrade

OpenLDAP stores its database in a format tied to its version. Upgrading from one OpenLDAP version to another is done by exporting the directory with the old version and importing it with the new one.

`osixia/openldap` automates this with two services:

- `openldap-pre-upgrade` (optional): creates an upgrade backup with the **current** image,
- `openldap-upgrade` (runs at every start): detects a version change and restores the upgrade backup with the **new** image.

!!! warning
    Automatic upgrade works only between v2 images. Moving from v1 to v2 requires a manual migration (export your data with `slapcat` from v1, then import it into a fresh v2 deployment).

## Version tracking

On the first start, `openldap-upgrade` tests the configuration (`slapd -T test`) and writes the OpenLDAP version to `OPENLDAP_UPGRADE_CONF_VERSION_FILE` (default `/etc/openldap/slapd.d/.version`).

At each start, the image version (`OPENLDAP_VERSION`) is compared with this file, at the level defined by `OPENLDAP_UPGRADE_MIGRATION_LEVEL`:

| Level             | Versions compared | Example: `2.6.10` → `2.6.12` | Example: `2.6.10` → `2.7.0` |
| ----------------- | ----------------- | ---------------------------- | --------------------------- |
| `major`           | `2`               | no migration                 | no migration                |
| `minor` (default) | `2.6`             | no migration                 | **migration**               |
| `patch`           | `2.6.10`          | **migration**                | **migration**               |

## Upgrade procedure

1. **Create the upgrade backup with the current image.** Run the `openldap-pre-upgrade` service with the image currently in use, on the same volumes:

    ``` bash
    docker run --rm \
      -v openldap-conf:/etc/openldap/slapd.d \
      -v openldap-data:/var/lib/openldap/openldap-data \
      -v openldap-backups:/var/lib/openldap/openldap-backups \
      osixia/openldap:<current-version> -x openldap-pre-upgrade
    ```

    This creates `openldap-upgrade-<version>-config.gz` and `openldap-upgrade-<version>-data.gz` in `OPENLDAP_BACKUP_DIR` (for example `openldap-upgrade-2.6-config.gz` with the `minor` level).
    It does nothing if the image version differs from the version file.

    !!! tip
        Stop writes to the directory (or stop the OpenLDAP container) before this step: changes made after the backup are lost.

2. **Stop the running container and start the new image** on the same volumes:

    ``` bash
    docker stop openldap && docker rm openldap
    docker run --name openldap [...] osixia/openldap:<new-version>
    ```

    `openldap-upgrade` detects the version change, restores the upgrade backup (`openldap-ctl restore --force`) with the new OpenLDAP version, then updates the version file.

3. **Check the logs** and your data, then keep the upgrade backup files until you are sure no rollback is needed.

## Safety checks

| Situation                                        | Behavior                                                                     |
| ------------------------------------------------ | ---------------------------------------------------------------------------- |
| New version lower than the version file (downgrade) | Startup fails, unless `OPENLDAP_UPGRADE_FORCE=true`                       |
| Upgrade backup files are missing                 | Startup fails, unless `OPENLDAP_UPGRADE_FORCE=true` (then the migration is skipped and the existing files are used as is) |

!!! danger
    `OPENLDAP_UPGRADE_FORCE=true` bypasses these checks. Use it only if you know the database files are compatible, and always keep a backup.

## Rollback

To roll back, start the previous image on volumes restored from a regular backup made before the upgrade (see [Backup / Restore](backup-restore.md)).
