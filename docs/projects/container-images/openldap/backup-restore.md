# 🛟 Backup / Restore

The image includes the `openldap-ctl` helper with `backup` and `restore` commands.
Backups are gzipped LDIF exports made with `slapcat`, stored in `OPENLDAP_BACKUP_DIR` (default `/var/lib/openldap/openldap-backups`).

!!! tip
    Mount `OPENLDAP_BACKUP_DIR` on a volume so backups survive the container, and copy them off-host regularly.

## Backup

``` bash
docker exec openldap container run -- openldap-ctl backup my-backup
```

This creates two files in `OPENLDAP_BACKUP_DIR` (permissions `600`):

- `my-backup-config.gz`: the `cn=config` database,
- `my-backup-data.gz`: the data database.

Usage:

```
openldap-ctl backup file-prefix [--config] [--data] [--clean days]
```

| Option         | Description                                                             |
| -------------- | ----------------------------------------------------------------------- |
| `--config`     | Back up only the config database                                        |
| `--data`       | Back up only the data database                                          |
| `--clean days` | After the backup, delete files in `OPENLDAP_BACKUP_DIR` older than `days` days |

Without `--config` or `--data`, both databases are backed up.

Create only a data backup and remove files older than 15 days:

``` bash
docker exec openldap container run -- openldap-ctl backup "$(date -I)" --data --clean 15
```

Backups can be taken while OpenLDAP is running.

## Restore

``` bash
docker exec openldap container run -- openldap-ctl restore my-backup --force
```

Usage:

```
openldap-ctl restore file-prefix [--config] [--data] [--force]
```

| Option     | Description                                                                              |
| ---------- | ---------------------------------------------------------------------------------------- |
| `--config` | Restore only the config database                                                         |
| `--data`   | Restore only the data database                                                           |
| `--force`  | Stop OpenLDAP if it is running and delete existing data in the target directories        |

Without `--config` or `--data`, both databases are restored. Data is re-indexed after import, and OpenLDAP is started again if it was stopped.

!!! danger
    `--force` **deletes** the current content of `OPENLDAP_CONF_DIR` and/or `OPENLDAP_DATA_DIR` before restoring. Make a fresh backup first.

### Restore into a new container

To restore on a new, empty deployment, start the container without launching `slapd`, restore, then start it normally:

``` bash
docker run --rm \
  -v openldap-conf:/etc/openldap/slapd.d \
  -v openldap-data:/var/lib/openldap/openldap-data \
  -v openldap-backups:/var/lib/openldap/openldap-backups \
  osixia/openldap run -- openldap-ctl restore my-backup
```

Restoring the config database also restores the root DN passwords, modules, schemas, overlays and ACLs of the backed-up server.

## Scheduled backups

The optional `openldap-cron` service runs `crond` with the job defined by `OPENLDAP_CRON_JOB`. Run it as a sidecar container sharing the OpenLDAP volumes:

``` bash
docker run --name openldap-cron \
  -u root \
  -v openldap-conf:/etc/openldap/slapd.d \
  -v openldap-data:/var/lib/openldap/openldap-data \
  -v openldap-backups:/var/lib/openldap/openldap-backups \
  osixia/openldap -x openldap-cron
```

!!! note
    `crond` requires root: the `-u root` option is mandatory. The job itself runs as the `ldap` user.

The default job runs every day at 02:15, creates a backup named `<date>-<openldap version>-openldap-cron` and deletes backups older than 15 days:

```
15 2 * * * PATH=${PATH} container run -- openldap-ctl backup "$(date -I)-${OPENLDAP_VERSION}-openldap-cron" --clean 15
```

Override `OPENLDAP_CRON_JOB` to change the schedule or options, for example every day at 03:00, data only, 7 days retention:

``` bash
docker run -u root \
  -e OPENLDAP_CRON_JOB='0 3 * * * PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin container run -- openldap-ctl backup "$(date -I)-openldap-cron" --data --clean 7' \
  [...] osixia/openldap -x openldap-cron
```

!!! note
    `OPENLDAP_CRON_JOB` is a crontab line: `%` characters must be escaped as `\%`.

=== "docker compose"

    ``` yaml
    services:
      openldap:
        image: osixia/openldap
        # [...]
        volumes:
          - openldap-conf:/etc/openldap/slapd.d
          - openldap-data:/var/lib/openldap/openldap-data
          - openldap-backups:/var/lib/openldap/openldap-backups

      openldap-cron:
        image: osixia/openldap
        user: root
        command: ["-x", "openldap-cron"]
        volumes:
          - openldap-conf:/etc/openldap/slapd.d
          - openldap-data:/var/lib/openldap/openldap-data
          - openldap-backups:/var/lib/openldap/openldap-backups
    ```
