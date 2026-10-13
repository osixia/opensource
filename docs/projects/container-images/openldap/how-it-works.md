# 🧩 How It Works

`osixia/openldap` is built on [osixia/baseimage](../baseimage/index.md). The image is split into small services, each with its own startup script. Understanding which service does what makes the rest of this documentation easier to follow.

## Services

| Service                | Enabled by default | Role                                                                                   |
| ---------------------- | ------------------ | -------------------------------------------------------------------------------------- |
| `openldap`             | yes                | Checks directories, copies custom modules and schemas, then runs `slapd`               |
| `openldap-bootstrap`   | yes                | Creates the configuration and data on first start                                      |
| `openldap-upgrade`     | yes                | Records the OpenLDAP version and migrates the databases when the image version changes |
| `openldap-pre-upgrade` | no                 | Backs up the databases before an upgrade; meant to run as an init container            |
| `openldap-cron`        | no                 | Runs scheduled backups; meant to run in a separate container                           |

The two optional services are started explicitly with the baseimage `-x` (`--exec`) flag, for example `osixia/openldap -x openldap-cron`. See [Backup / Restore](backup-restore.md) and [Upgrade](upgrade.md).

## Startup Sequence

Startup scripts run in priority order before `slapd` is started:

1. **`openldap`** checks that the configuration and data directories exist, then copies custom modules and schemas into the OpenLDAP directories.
2. **`openldap-bootstrap`** creates a new directory if both the configuration and data directories are empty. Otherwise it does nothing.
3. **`openldap-upgrade`** compares the image version with the version recorded in the configuration directory and migrates the databases if needed.
4. **`slapd`** starts in the foreground.

## Bootstrap Runs Once

Bootstrap only runs when **both** `/etc/openldap/slapd.d` and `/var/lib/openldap/openldap-data` are empty. Hidden files and `lost+found` are ignored.

This has two consequences:

- `OPENLDAP_BOOTSTRAP_*` variables are only read on first start. Changing them later has no effect on an existing directory.
- After bootstrap, the directory is managed like any OpenLDAP server: change the configuration through `cn=config` with `ldapmodify`.

See [Bootstrap](bootstrap/index.md) for what is created and how to customize it.

## Directories

| Path                                             | Variable                      | Content                                    |
| ------------------------------------------------ | ----------------------------- | ------------------------------------------ |
| `/etc/openldap/slapd.d`                          | `OPENLDAP_CONF_DIR`           | Configuration database (`cn=config`)       |
| `/var/lib/openldap/openldap-data`                | `OPENLDAP_DATA_DIR`           | Data database (MDB)                        |
| `/var/lib/openldap/openldap-backups`             | `OPENLDAP_BACKUP_DIR`         | Backups                                    |
| `/etc/openldap/schema`                           | `OPENLDAP_SCHEMAS_DIR`        | Schemas available to bootstrap             |
| `/usr/lib/openldap`                              | `OPENLDAP_MODULES_DIR`        | OpenLDAP modules                           |
| `/container/services/openldap/assets/schema`     | `OPENLDAP_CUSTOM_SCHEMAS_DIR` | Custom schemas copied at startup           |
| `/container/services/openldap/assets/module`     | `OPENLDAP_CUSTOM_MODULES_DIR` | Custom modules copied at startup           |
| `/container/services/openldap/assets/certs`      |                               | Default location of TLS certificates       |

The first three should be persisted with volumes.

## User and Ports

- The container runs as the `ldap` user (uid `911`, gid `911`), not as root.
- `slapd` listens on `ldap://:3890`, `ldaps://:6360` and the local `ldapi:///` socket, as defined by `OPENLDAP_URLS`. Unprivileged ports are used because the process is not root; publish them on `389` and `636` if needed.
- The local `ldapi:///` socket gives the `ldap` user full access to the data and monitor databases. This is what `docker exec openldap container run -- ldapsearch -Q -Y EXTERNAL -H ldapi:/// ...` uses.

## Environment Variables

Defaults are stored in `.env` files inside the image and loaded at startup. Variables passed to the container take precedence, and defaults that reference other variables follow them: setting `OPENLDAP_BOOTSTRAP_SUFFIX` also changes `OPENLDAP_BOOTSTRAP_DATA_ROOT_DN`, which defaults to `cn=admin,${OPENLDAP_BOOTSTRAP_SUFFIX}`.

See [Environment Variables](environment-variables.md) for the full list.
