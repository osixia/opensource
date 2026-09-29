# 🧩 Overlays & Accounts

All features on this page are configured at [bootstrap](bootstrap.md) time: set the variables on the **first** start, with empty config and data directories.
See [Environment Variables](environment-variables.md) for every available setting.

## Read-only accounts

### Data database

Creates a service account (for example for applications that only search the directory) with read access to the whole data database, except `userPassword` and `shadowLastChange`.

``` bash
docker run --rm osixia/openldap run -- openldap-ctl password hash 'r3adonly-passw0rd'
# {ARGON2}$argon2id$v=19$m=...

docker run \
  -e OPENLDAP_BOOTSTRAP_DATA_READONLY=true \
  -e OPENLDAP_BOOTSTRAP_DATA_READONLY_PASSWORD_HASHED='{ARGON2}$argon2id$v=19$m=...' \
  osixia/openldap
```

The account DN is `OPENLDAP_BOOTSTRAP_DATA_READONLY_DN` (default `cn=readonly,<suffix>`). If no hashed password is set, a random one is generated and printed in the logs.

### Monitor database

Enable the monitor database and a read-only account to scrape `cn=monitor` (for example with an OpenLDAP exporter):

``` bash
docker run \
  -e OPENLDAP_BOOTSTRAP_MONITOR_ENABLED=true \
  -e OPENLDAP_BOOTSTRAP_MONITOR_READONLY=true \
  osixia/openldap
```

The account DN is `OPENLDAP_BOOTSTRAP_MONITOR_READONLY_DN` (default `cn=readonly-monitor,<suffix>`).

## Password policy (`ppolicy`)

``` bash
docker run -e OPENLDAP_BOOTSTRAP_PPOLICY=true osixia/openldap
```

Bootstrap adds the `ppolicy` overlay on the data database, creates the `OPENLDAP_BOOTSTRAP_PPOLICY_GROUP_DN` organizational unit (default `ou=policies,<suffix>`) and a default policy `OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_POLICY_DN` (default `cn=default,ou=policies,<suffix>`) applied to all accounts:

| Setting                   | Default                         |
| ------------------------- | ------------------------------- |
| Minimum length            | `12`                            |
| Password history          | `12`                            |
| Max failures before lock  | `5` within `300` s              |
| Lockout duration          | `900` s (15 min)                |
| Max age                   | `7776000` s (90 days)           |
| Expiration warning        | `1209600` s (14 days)           |
| Grace logins              | `3`                             |

With `OPENLDAP_BOOTSTRAP_PPOLICY_HASH_CLEAR_TEXT=true` (default), clear-text passwords sent with add/modify operations are hashed with the default hash (`{ARGON2}`).

## Referential integrity (`refint`)

``` bash
docker run -e OPENLDAP_BOOTSTRAP_REFINT=true osixia/openldap
```

When an entry is renamed or deleted, references to it are updated in the attributes listed in `OPENLDAP_BOOTSTRAP_REFINT_ATTRIBUTES` (default `member uniqueMember manager owner`).

## Attribute uniqueness (`unique`)

``` bash
docker run -e OPENLDAP_BOOTSTRAP_UNIQUE=true osixia/openldap
```

Rejects writes that would create duplicate values for the URIs listed in `OPENLDAP_BOOTSTRAP_UNIQUE_URIS` (default `ldap:///?uid?sub ldap:///?mail?sub`: `uid` and `mail` must be unique in the whole database).

## Group membership (`memberof`)

``` bash
docker run -e OPENLDAP_BOOTSTRAP_MEMBEROF=true osixia/openldap
```

Maintains the `memberOf` attribute on user entries when they are added to `groupOfNames` groups through the `member` attribute. The group object class and attributes can be changed with `OPENLDAP_BOOTSTRAP_MEMBEROF_*` variables.

!!! tip
    Enable `refint` together with `memberof` so group membership is cleaned up when users are deleted or renamed.

!!! note
    With [replication](replication.md), `memberOf` is excluded from replicated attributes (`exattrs`) and computed locally on each node.
