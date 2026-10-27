# Custom LDIF

The whole bootstrap is built from small LDIF fragments. You can add your own, replace a built-in one, or remove it, without rebuilding the image.

## Custom Directories

| Directory inside the container                                     | Content                                                                 |
| ------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| `/container/services/openldap-bootstrap/assets/ldif/config/custom` | Configuration (`cn=config`): ACLs, indexes, overlays and other settings |
| `/container/services/openldap-bootstrap/assets/ldif/data/custom`   | Initial directory entries: users, groups and other LDAP data            |

Create local directories and mount them into the container:

``` bash
mkdir -p custom-config custom-data
```

``` bash
docker run --name openldap \
  -v "$(pwd)/custom-config:/container/services/openldap-bootstrap/assets/ldif/config/custom" \
  -v "$(pwd)/custom-data:/container/services/openldap-bootstrap/assets/ldif/data/custom" \
  osixia/openldap:2.6.15-rc
```

Put `.ldif` files in these directories, or `.ldif.template` files with `${VARIABLE}` placeholders that are replaced by environment variables. Leave the mounts writable so bootstrap can convert templates into `.ldif` files.

!!! note

    Custom files are only used by bootstrap, on first start. They do not change an existing directory.

## Add Content

Use a new file name. For example, `custom-data/70-users.ldif`:

``` ldif
dn: ou=people,dc=example,dc=org
objectClass: organizationalUnit
ou: people

dn: uid=jdoe,ou=people,dc=example,dc=org
objectClass: inetOrgPerson
uid: jdoe
cn: John Doe
sn: Doe
mail: jdoe@example.org
```

Each entry needs a `dn:` and its required attributes, with a blank line between entries. Parent entries must come before their children.

Entries are imported with `slapadd`: write plain entries, without modification instructions such as `changetype: modify`.

## Replace a Built-In Fragment

Copy the built-in file into the matching custom directory, keep the file name, and edit it.

For example, to change the default access rules, copy [`65-database-acl.ldif.template`](https://github.com/osixia/container-openldap/blob/develop/services/openldap-bootstrap/assets/ldif/config/base/65-database-acl.ldif.template) into `custom-config/` and edit it. To change database settings and indexes, do the same with `60-database.ldif.template`.

Replacing one fragment leaves the others in place: replacing `65-database-acl.ldif.template` keeps the final deny-all rule from `69-database-acl-close.ldif` and the rules added by enabled features such as the read-only account or replication.

## Remove a Built-In Fragment

Provide an empty `.ldif` file with the same output file name. For example, an empty `custom-config/65-database-acl.ldif` removes the rules from `65-database-acl.ldif.template`.

## How Fragments Are Assembled

Files are sorted by their numeric prefix, then concatenated into one LDIF file per database. Prefixes are grouped by **tens**:

- Files in the **same ten-group** (for example `60-*` and `65-*`) are concatenated **without a blank line**: they form a single entry.
- When the **ten-group changes** (for example `69-*` to `70-*`), **one blank line** is inserted: a new entry starts.
- Leading and trailing empty lines are removed from every file.

This is how several files build one entry. The data database entry is assembled from:

| File                                 | Adds                                |
| ------------------------------------ | ----------------------------------- |
| `60-database.ldif.template`          | The entry itself, with its indexes  |
| `61-database-replication.ldif.template` | Replication settings, when enabled |
| `65-database-acl.ldif.template`      | Default access rules                |
| `66-…` and `67-…`                    | Access rules of optional accounts   |
| `69-database-acl-close.ldif`         | Final deny-all rule                 |

So a custom file named `62-my-indexes.ldif` containing only `olcDbIndex:` lines adds indexes to the data database, while a file named `200-my-overlay.ldif` starts a new entry.

File names without a numeric prefix, such as `users.ldif`, are accepted. Each one forms its own group.

## Built-In Fragments

Configuration fragments, in `config/base`:

| File                             | Description                                             |
| -------------------------------- | ------------------------------------------------------- |
| `00-global.ldif.template`        | Global limits (size, time, idle timeout)                |
| `10-module.ldif.template`        | Module path and module list                             |
| `20-schema.ldif`                 | Schema container                                        |
| `30-schema-import.ldif.template` | Schemas to load                                         |
| `40-frontend.ldif.template`      | Frontend database (password hash scheme)                |
| `50-config.ldif.template`        | Configuration database, root DN and password            |
| `59-config-acl-close.ldif`       | Deny-all rule on the configuration database             |
| `60-database.ldif.template`      | Data database (MDB) with indexes                        |
| `65-database-acl.ldif.template`  | Data database access rules                              |
| `69-database-acl-close.ldif`     | Deny-all rule on the data database                      |
| `70-monitor.ldif.template`       | Monitor database                                        |
| `75-monitor-acl.ldif.template`   | Monitor database access rules                           |
| `79-monitor-acl-close.ldif`      | Deny-all rule on the monitor database                   |

Optional features add their own fragments: `01-` and `02-` for TLS, `03-`, `61-`, `67-` and `180-` for replication, `66-` and `76-` for read-only accounts, `100-` to `160-` for overlays.

Overlays are created in prefix order, and the order matters. Keep `syncprov` last, as with the built-in `180-` prefix.

The complete list is in the [LDIF directory](https://github.com/osixia/container-openldap/tree/develop/services/openldap-bootstrap/assets/ldif) of the repository.
