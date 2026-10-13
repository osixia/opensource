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
openldap-ctl backup file-prefix [--config] [--data] [--clean days]
```

| Option         | Description                                                            |
| -------------- | ---------------------------------------------------------------------- |
| `--config`     | Back up only the configuration database                                |
| `--data`       | Back up only the data database                                         |
| `--clean days` | After the backup, delete files older than `days` days in the backup directory |

Without `--config` or `--data`, both databases are backed up.

Create only a data backup and remove files older than 15 days:

``` bash
docker exec openldap container run -- openldap-ctl backup my-backup --data --clean 15
```

!!! warning

    `--clean` deletes **every** file older than the given number of days in the backup directory, not only files with the same prefix.

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
| `--force`  | Stop OpenLDAP if it is running and delete existing data before restoring |

A restore goes through these steps:

1. The backup files are checked. If one is missing or is not a valid gzip archive, the command stops and **nothing is changed**.
2. If OpenLDAP is running, it is stopped. This requires `--force`.
3. Existing configuration and data are deleted. This requires `--force`.
4. The selected databases are restored, and the data database is re-indexed.
5. OpenLDAP is started again if it was running.

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
  osixia/openldap -x openldap-cron
```

The container must run as `root` for `crond`; the backup itself runs as the `ldap` user.

The schedule and command are defined by `OPENLDAP_CRON_JOB`. By default, a full backup runs every day at 02:15 and backups older than 15 days are deleted:

``` bash
15 2 * * * PATH=${PATH} container run -- openldap-ctl backup "$(date -I)-${OPENLDAP_VERSION}-openldap-cron" --clean 15
```

This produces files such as `2026-10-13-2.6.15-r0-openldap-cron-config.gz`.

## Keeping Backups Safe

Backups contain password hashes, and the configuration backup also contains the replication password when replication is enabled. Files are created readable by the `ldap` user only.

Copy backups out of the backup volume to another machine or storage: a backup stored next to the data it protects does not survive the loss of the host.
