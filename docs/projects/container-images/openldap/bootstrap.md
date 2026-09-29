# 🌱 Bootstrap

The bootstrap creates a complete OpenLDAP directory (`cn=config` + data) the first time the container starts.
It is fully driven by environment variables and LDIF templates, and can be extended with your own LDIF, schemas and modules.

## Startup sequence

The image is made of several [osixia/baseimage services](../baseimage/how-it-works.md). At each start, their `startup.sh` scripts run in this order:

| Order | Service              | What it does                                                                                               |
| ----- | -------------------- | ---------------------------------------------------------------------------------------------------------- |
| 1     | `openldap`           | Checks config/data directories exist, copies custom modules and schemas to the OpenLDAP directories        |
| 2     | `openldap-bootstrap` | Bootstraps config and data **only if both `OPENLDAP_CONF_DIR` and `OPENLDAP_DATA_DIR` are empty**          |
| 3     | `openldap-upgrade`   | Tests the config on first run, or migrates the directory when the OpenLDAP version changed (see [Upgrade](upgrade.md)) |

Then `slapd` is started with `-h "${OPENLDAP_URLS}" -F "${OPENLDAP_CONF_DIR}" -d "${OPENLDAP_DEBUG_LEVEL}"`.

!!! info
    Hidden files and `lost+found` are ignored when checking if the directories are empty.
    Once bootstrapped, changing `OPENLDAP_BOOTSTRAP_*` variables has no effect: modify the running configuration with LDAP tools instead.

## How bootstrap works

1. **Environment**: each script of `OPENLDAP_BOOTSTRAP_SCRIPTS_DIR` is called with `env` to compute extra variables (hashed passwords, module and schema LDIF, replication server IDs, ...).
2. **LDIF rendering**: each script is called with `ldif`. Enabled features copy their LDIF files from `OPENLDAP_BOOTSTRAP_LDIF_CONFIG_DIR` / `OPENLDAP_BOOTSTRAP_LDIF_DATA_DIR`. Files ending with `.template` are rendered with environment variables (`${VARIABLE}` syntax).
3. **Import**: all rendered LDIF files are sorted by name, concatenated and imported with `slapadd`: config LDIF into database `0` (`cn=config`), data LDIF into database `1`.

### Default directory

With default settings, bootstrap creates:

- global limits (`olcSizeLimit`, `olcTimeLimit`, `olcIdleTimeout`),
- modules loaded from `OPENLDAP_BOOTSTRAP_MODULES` and schemas from `OPENLDAP_BOOTSTRAP_SCHEMAS`,
- `{ARGON2}` as default password hash (`olcPasswordHash`),
- the `cn=config` database with root DN `cn=admin,cn=config`,
- an MDB database for `OPENLDAP_BOOTSTRAP_SUFFIX` with root DN `cn=admin,<suffix>` and indexes on `objectClass`, `uid`, `mail`, `cn`, `sn`, `givenName`, `member`, `uniqueMember`, `entryCSN`, `entryUUID`,
- the monitor database (monitoring disabled unless `OPENLDAP_BOOTSTRAP_MONITOR_ENABLED=true`),
- the organization entry (`dcObject` + `organization`).

### Default ACLs

On the data database:

- the local `ldap` system user connected through `ldapi:///` (SASL `EXTERNAL`) has `manage` access,
- users can change their own `userPassword` and `shadowLastChange`; anonymous users can only authenticate,
- authenticated users can read `cn`, `sn`, `givenName`, `mail` and `uid`,
- everything else is denied (`access to * by * none`).

Config and monitor databases are closed the same way: only their root DN (and enabled read-only accounts) can access them.

### Passwords

| Account                                  | Variable                                              | If empty                                     |
| ---------------------------------------- | ----------------------------------------------------- | -------------------------------------------- |
| Config root (`cn=admin,cn=config`)       | `OPENLDAP_BOOTSTRAP_CONFIG_ROOT_PASSWORD_HASHED`      | Random password generated and printed in logs |
| Data root (`cn=admin,<suffix>`)          | `OPENLDAP_BOOTSTRAP_DATA_ROOT_PASSWORD_HASHED`        | Random password generated and printed in logs |
| Read-only data account                   | `OPENLDAP_BOOTSTRAP_DATA_READONLY_PASSWORD_HASHED`    | Random password generated and printed in logs |
| Read-only monitor account                | `OPENLDAP_BOOTSTRAP_MONITOR_READONLY_PASSWORD_HASHED` | Random password generated and printed in logs |
| Replication account                      | `OPENLDAP_BOOTSTRAP_REPLICATION_DATA_READONLY_PASSWORD` (**plain text**) | Random password generated and printed in logs |

Hash a password with:

``` bash
docker run --rm osixia/openldap run -- openldap-ctl password hash 'my-s3cret-passw0rd'
```

!!! warning
    Generated passwords are printed only once, during bootstrap. Save them, then consider purging the container logs.

## Optional features

| Variable                             | Feature                                                                          |
| ------------------------------------ | -------------------------------------------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_TLS`             | TLS certificates, see [TLS](tls.md)                                              |
| `OPENLDAP_BOOTSTRAP_TLS_REQUIRED`    | Reject non-TLS connections, see [TLS](tls.md)                                    |
| `OPENLDAP_BOOTSTRAP_DATA_READONLY`   | Read-only account on the data database, see [Overlays & Accounts](overlays-and-accounts.md) |
| `OPENLDAP_BOOTSTRAP_MONITOR_ENABLED` | Enable the monitor database                                                      |
| `OPENLDAP_BOOTSTRAP_MONITOR_READONLY`| Read-only account on the monitor database                                        |
| `OPENLDAP_BOOTSTRAP_REFINT`          | `refint` overlay                                                                 |
| `OPENLDAP_BOOTSTRAP_PPOLICY`         | `ppolicy` overlay and default password policy                                    |
| `OPENLDAP_BOOTSTRAP_UNIQUE`          | `unique` overlay                                                                 |
| `OPENLDAP_BOOTSTRAP_MEMBEROF`         | `memberof` overlay                                                               |
| `OPENLDAP_BOOTSTRAP_REPLICATION`     | Multi-provider replication, see [Replication](replication.md)                    |

## Adding your own LDIF

Put `.ldif` or `.ldif.template` files in the `custom` directories. They are imported last, after all built-in files:

- `/container/services/openldap-bootstrap/assets/ldif/config/custom/`: added to `cn=config` (database `0`)
- `/container/services/openldap-bootstrap/assets/ldif/data/custom/`: added to the data database (database `1`)

Example `data/custom/10-people.ldif.template`:

``` ldif
dn: ou=people,${OPENLDAP_BOOTSTRAP_SUFFIX}
objectClass: organizationalUnit
ou: people

dn: uid=jdoe,ou=people,${OPENLDAP_BOOTSTRAP_SUFFIX}
objectClass: inetOrgPerson
uid: jdoe
cn: John Doe
sn: Doe
mail: jdoe@keycloakers.org
```

``` bash
docker run --name openldap \
  -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
  -v $PWD/ldif/data:/container/services/openldap-bootstrap/assets/ldif/data/custom \
  osixia/openldap
```

!!! note
    Templates are rendered in place: a directory that contains `.template` files must be writable by the `ldap` user (uid `911`). Do not mount it read-only.

!!! warning "Numbered prefix grouping"
    Because `slapadd` imports **one** LDIF stream, files are concatenated with a specific rule based on their numeric prefix, grouped by tens:

    - files in the same ten-group (for example `40-a.ldif` and `49-b.ldif`) are concatenated **without** a blank line: the second file continues the entry of the first one (useful to append `olcAccess` rules to a database entry),
    - when switching group (for example `49-b.ldif` to `50-c.ldif`), a blank line is inserted: a new entry starts.

    Files without a numeric prefix are always separated by a blank line.
    If your custom file contains full entries, separate them with a blank line inside the file and choose a prefix in its own ten-group.

### Built-in config LDIF order

Use these prefixes to position custom config files (for example an overlay must come after the database it applies to):

| Prefix      | Content                                         |
| ----------- | ----------------------------------------------- |
| `00`–`03`   | Global settings (limits, TLS, server IDs)       |
| `10`        | Modules                                         |
| `20`–`30`   | Schemas                                         |
| `40`        | Frontend database                               |
| `50`–`59`   | Config database and its ACLs                    |
| `60`–`69`   | Data database (`{1}mdb`), syncrepl and its ACLs |
| `70`–`79`   | Monitor database and its ACLs                   |
| `100`       | `ppolicy` overlay                               |
| `120`       | `unique` overlay                                |
| `140`       | `refint` overlay                                |
| `160`       | `memberof` overlay                              |
| `180`       | `syncprov` overlay (keep it last)               |

## Schemas

Schemas listed in `OPENLDAP_BOOTSTRAP_SCHEMAS` are loaded from `OPENLDAP_SCHEMAS_DIR` (LDIF format).
Default: `core.ldif cosine.ldif inetorgperson.ldif rfc2307bis.ldif`.

Besides OpenLDAP stock schemas, the image ships these extra schemas:

| Schema        | Description                                                        |
| ------------- | ------------------------------------------------------------------ |
| `eduperson`   | Academic identity schema used in higher-education federations      |
| `kerberos`    | Kerberos principals for LDAP-backed KDC setups                     |
| `openssh-lpk` | OpenSSH public keys (`sshPublicKey`)                               |
| `rfc2307bis`  | UNIX accounts and groups (improved RFC2307 / NIS)                  |
| `samba`       | Samba domain and account attributes                                |
| `sudo`        | sudo rules (`sudoRole`)                                            |
| `totp`        | TOTP secrets for LDAP-based 2FA                                    |
| `virtualMail` | Virtual mailbox accounts and aliases                               |

``` bash
docker run -e OPENLDAP_BOOTSTRAP_SCHEMAS="core.ldif cosine.ldif inetorgperson.ldif rfc2307bis.ldif openssh-lpk.ldif sudo.ldif" \
  osixia/openldap
```

!!! note
    `rfc2307bis` and `nis` define the same attributes: load only one of them.

### Custom schemas

Files in `OPENLDAP_CUSTOM_SCHEMAS_DIR` (`/container/services/openldap/assets/schema`) are copied to `OPENLDAP_SCHEMAS_DIR` at each start.
Convert a `.schema` file to LDIF with [`openldap-ctl schema2ldif`](ldap-toolkit.md#convert-schema-to-ldif), then:

``` bash
docker run \
  -v $PWD/my-schema.ldif:/container/services/openldap/assets/schema/my-schema.ldif \
  -e OPENLDAP_BOOTSTRAP_SCHEMAS="core.ldif cosine.ldif inetorgperson.ldif rfc2307bis.ldif my-schema.ldif" \
  osixia/openldap
```

## Custom modules

Files in `OPENLDAP_CUSTOM_MODULES_DIR` (`/container/services/openldap/assets/module`) are copied to `OPENLDAP_MODULES_DIR` at each start.
Add them to `OPENLDAP_BOOTSTRAP_MODULES` to load them at bootstrap.

Default modules: `back_mdb.so argon2.so ppolicy.so unique.so refint.so memberof.so syncprov.so`.

## Extending the bootstrap in a derived image

Bootstrap scripts live in `OPENLDAP_BOOTSTRAP_SCRIPTS_DIR` and are executed in numeric order. Each script accepts two operations:

- `env <env-file>`: append `export VAR=value` lines needed by templates,
- `ldif <source-dir> <output-dir>`: copy the rendered LDIF files of the feature to the output directory.

To add a feature in an image built `FROM osixia/openldap`:

1. create a numbered script (for example `55-myfeature.sh`) guarded by your own environment variable,
2. source `helpers/common.sh` (`check_args`, `prepare_env_file`, `prepare_ldif_files`, `get_dn_attr`) and `helpers/password.sh` (`generate_password`, `hash_password`, `ensure_hashed_password`) as needed,
3. add its LDIF templates under `ldif/config/<feature>/` and/or `ldif/data/<feature>/`,
4. add default values for your variables in `/container/environment`.

See the [bootstrap scripts](https://github.com/osixia/container-openldap/tree/develop/services/openldap-bootstrap/assets/scripts) for examples.
