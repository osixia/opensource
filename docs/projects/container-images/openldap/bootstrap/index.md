# 🚀 Bootstrap

On first start, the `openldap-bootstrap` service creates a complete directory from environment variables. This page describes what is created and which features can be enabled.

!!! note "Bootstrap runs once"

    Bootstrap only runs when both the configuration and data directories are empty.
    `OPENLDAP_BOOTSTRAP_*` variables are ignored on an existing directory. See [How It Works](../how-it-works.md#bootstrap-runs-once).

## What Is Created

With default settings, bootstrap creates:

- the global configuration, with size, time and idle limits;
- the modules listed in `OPENLDAP_BOOTSTRAP_MODULES`;
- the schemas listed in `OPENLDAP_BOOTSTRAP_SCHEMAS` (`core`, `cosine`, `inetorgperson` and `rfc2307bis` by default);
- the configuration database, with its root DN `cn=admin,cn=config`;
- one MDB data database for `OPENLDAP_BOOTSTRAP_SUFFIX`, with its root DN `cn=admin,<suffix>`, indexes and [access rules](access-control.md);
- the monitor database;
- the root entry of the directory, named after `OPENLDAP_BOOTSTRAP_ORGANIZATION`.

Passwords are hashed with Argon2 (`OPENLDAP_BOOTSTRAP_FRONTEND_PASSWORD_HASH`).

## Suffix and Organization

``` bash
docker run --name openldap \
  -e OPENLDAP_BOOTSTRAP_ORGANIZATION="Keycloakers Org" \
  -e OPENLDAP_BOOTSTRAP_SUFFIX="dc=keycloakers,dc=org" \
  osixia/openldap
```

`OPENLDAP_BOOTSTRAP_SUFFIX` must start with `dc=`, in lowercase. Bootstrap stops with an error for any other suffix, such as `o=acme`.

Variables derived from the suffix follow it automatically: the data root DN becomes `cn=admin,dc=keycloakers,dc=org`.

## Passwords

Each account password is taken from a `*_PASSWORD_HASHED` variable. When that variable is empty, a random password is generated, hashed, and printed once in the container logs:

``` bash
docker logs openldap 2>&1 | grep -i password
```

To set a password yourself, hash it first and pass the hash:

``` bash
docker run --rm osixia/openldap run -- openldap-ctl password hash 'my-secret-password'
```

``` bash
docker run --name openldap \
  -e OPENLDAP_BOOTSTRAP_DATA_ROOT_PASSWORD_HASHED='{ARGON2}$argon2id$v=19$m=7168,t=5,p=1$...' \
  -e OPENLDAP_BOOTSTRAP_CONFIG_ROOT_PASSWORD_HASHED='{ARGON2}$argon2id$v=19$m=7168,t=5,p=1$...' \
  osixia/openldap
```

!!! warning

    Hashes contain `$` characters. Use single quotes in a shell, and double every `$` (`$$`) in a Docker Compose file.

## Optional Features

All optional features are disabled by default and enabled with one variable.

| Feature                    | Variable                                   | Documentation                              |
| -------------------------- | ------------------------------------------ | ------------------------------------------ |
| Read-only data account     | `OPENLDAP_BOOTSTRAP_DATA_READONLY=true`    | [Access Control](access-control.md)        |
| Read-only monitor account  | `OPENLDAP_BOOTSTRAP_MONITOR_READONLY=true` | [Access Control](access-control.md)        |
| Password policy            | `OPENLDAP_BOOTSTRAP_PPOLICY=true`          | [Overlays](overlays.md#password-policy)    |
| Referential integrity      | `OPENLDAP_BOOTSTRAP_REFINT=true`           | [Overlays](overlays.md#referential-integrity) |
| Unique attributes          | `OPENLDAP_BOOTSTRAP_UNIQUE=true`           | [Overlays](overlays.md#unique-attributes)  |
| Reverse group membership   | `OPENLDAP_BOOTSTRAP_MEMBEROF=true`         | [Overlays](overlays.md#memberof)           |
| TLS                        | `OPENLDAP_BOOTSTRAP_TLS=true`              | [TLS](../tls.md)                           |
| TLS required               | `OPENLDAP_BOOTSTRAP_TLS_REQUIRED=true`     | [TLS](../tls.md#require-tls)               |
| Multi-provider replication | `OPENLDAP_BOOTSTRAP_REPLICATION=true`      | [Replication](../replication.md)           |

## Indexes

The data database is created with these indexes:

| Attributes                         | Index      |
| ---------------------------------- | ---------- |
| `objectClass`                      | `eq`       |
| `uid`, `mail`, `cn`, `sn`, `givenName` | `eq,sub` |
| `member`, `uniqueMember`           | `eq`       |
| `entryCSN`, `entryUUID`            | `eq`       |

The maximum database size is set by `OPENLDAP_BOOTSTRAP_DATA_DATABASE_MAX_SIZE` (10 GB by default).

To change indexes or any other part of the generated configuration, see [Custom LDIF](custom-ldif.md).

## Going Further

- [Access Control](access-control.md): who can read and write what.
- [Overlays](overlays.md): password policy, referential integrity, uniqueness, memberOf.
- [Schemas](schemas.md): schemas shipped with the image and how to add yours.
- [Custom LDIF](custom-ldif.md): add, replace or remove any part of the bootstrap.
