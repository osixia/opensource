# 🛟 Backup / Restore

The image includes the `openldap-ctl` helper with `backup` and `restore` commands. Backups are gzip-compressed LDIF exports of the configuration and data databases.

## Create a Backup

``` bash
docker exec openldap container run -- openldap-ctl backup my-backup
```

This creates two files in `/var/lib/openldap/openldap-backups` (`OPENLDAP_BACKUP_DIR`):

- `my-backup-config.gz`
- `my-backup-data.gz`

Backups can be taken while OpenLDAP is running.

Usage:

``` bash
openldap-ctl backup file-prefix [--config] [--data] [--clean days] [--clean-prefix prefix]
```

| Option         | Description                                                            |
| -------------- | ---------------------------------------------------------------------- |
| `--config`     | Back up only the configuration database                                |
| `--data`       | Back up only the data database                                         |
| `--clean days` | After the backup, delete backup files older than `days` days |
| `--clean-prefix prefix` | Limit cleanup to backup filenames starting with this prefix; requires `--clean` |

Without `--config` or `--data`, both databases are backed up.

Create a data backup and remove only `daily-` backups older than 15 days:

``` bash
docker exec openldap container run -- \
  openldap-ctl backup "daily-$(date -I)" --data --clean 15 --clean-prefix daily-
```

!!! warning

    Without `--clean-prefix`, cleanup applies to all old files ending in `-config.gz` or `-data.gz` in the backup directory, including manual and upgrade backups. A data-only backup can also clean configuration backups. Always set a prefix when you want to preserve other backup sets.

## Restore a Backup

``` bash
docker exec openldap container run -- openldap-ctl restore my-backup --force
```

Usage:

``` bash
openldap-ctl restore file-prefix [--config] [--data] [--force]
```

| Option     | Description                                                              |
| ---------- | ------------------------------------------------------------------------ |
| `--config` | Restore only the configuration database                                  |
| `--data`   | Restore only the data database                                           |
| `--force`  | Stop OpenLDAP if running and replace the selected databases |

A restore goes through these steps:

1. The backup files are checked. If one is missing or is not a valid gzip archive, the command stops and **nothing is changed**.
2. If OpenLDAP is running, it is stopped. This requires `--force`.
3. The selected databases are moved into `.restore-previous` directories. Replacing existing data requires `--force`.
4. The selected databases are restored, and the data database is re-indexed.
5. On success, the previous data is removed. On failure, the command attempts to put it back. OpenLDAP is started again if it was running.

If a restore is interrupted and leaves a `.restore-previous` directory, the next restore refuses to proceed. Inspect the files and recover the previous data before trying again.

Without `--force`, the command refuses to run when OpenLDAP is running or when the target directories are not empty.

!!! danger

    `--force` deletes the current configuration and data. Take a backup first if you may need them.

## Scheduled Backups

The `openldap-cron` service runs backups on a schedule. It is disabled by default and is meant to run in a **separate container** that shares the configuration, data and backup volumes:

``` bash
docker run --name openldap-cron \
  -u root \
  -v openldap-conf:/etc/openldap/slapd.d \
  -v openldap-data:/var/lib/openldap/openldap-data \
  -v openldap-backups:/var/lib/openldap/openldap-backups \
  osixia/openldap:2.6.15-rc -x openldap-cron
```

The container must run as `root` for `crond`; the backup itself runs as the `ldap` user.

The schedule and command are defined by `OPENLDAP_CRON_JOB`. By default, a full backup runs every day at 02:15 and only scheduled backups older than 15 days are deleted:

``` bash
15 2 * * * PATH=${PATH} container run -- openldap-ctl backup "openldap-cron-$(date -I)-${OPENLDAP_VERSION}" --clean 15 --clean-prefix openldap-cron-
```

This produces files such as `openldap-cron-2026-10-13-2.6.15-r0-config.gz` and preserves manual and upgrade backups.

## Keeping Backups Safe

Backups contain password hashes, and the configuration backup also contains the replication password when replication is enabled. Files are created readable by the `ldap` user only.

Copy backups out of the backup volume to another machine or storage: a backup stored next to the data it protects does not survive the loss of the host.
