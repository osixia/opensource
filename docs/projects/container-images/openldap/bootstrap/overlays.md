# Overlays

Four common OpenLDAP overlays can be enabled at bootstrap. Each one is disabled by default and is added to the data database when its variable is set to `true`.

The required modules are already loaded by the default value of `OPENLDAP_BOOTSTRAP_MODULES`.

``` bash
docker run --name openldap \
  -e OPENLDAP_BOOTSTRAP_PPOLICY=true \
  -e OPENLDAP_BOOTSTRAP_REFINT=true \
  -e OPENLDAP_BOOTSTRAP_UNIQUE=true \
  -e OPENLDAP_BOOTSTRAP_MEMBEROF=true \
  osixia/openldap
```

## Password Policy

`OPENLDAP_BOOTSTRAP_PPOLICY=true` enables the `ppolicy` overlay and creates a default policy applied to every account.

Bootstrap creates two entries in the data database:

- a container, `ou=policies,<suffix>` (`OPENLDAP_BOOTSTRAP_PPOLICY_GROUP_DN`);
- the default policy, `cn=default,ou=policies,<suffix>` (`OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_POLICY_DN`).

The default policy is:

| Setting                         | Default            | Variable suffix (`OPENLDAP_BOOTSTRAP_PPOLICY_DEFAULT_…`) |
| ------------------------------- | ------------------ | -------------------------------------------------------- |
| Minimum length                  | 12 characters      | `MIN_LENGTH`                                             |
| Password history                | 12 passwords       | `IN_HISTORY`                                             |
| Maximum age                     | 90 days            | `MAX_AGE`                                                |
| Expiration warning              | 14 days before     | `EXPIRE_WARNING`                                         |
| Grace logins after expiration   | 3                  | `GRACE_AUTH_NLIMIT`                                      |
| Lockout                         | after 5 failures   | `LOCKOUT`, `MAX_FAILURE`                                 |
| Failure counter reset           | 5 minutes          | `FAILURE_COUNT_INTERVAL`                                 |
| Lockout duration                | 15 minutes         | `LOCKOUT_DURATION`                                       |
| Users can change their password | yes                | `ALLOW_USER_CHANGE`                                      |
| Old password required to change | yes                | `SAFE_MODIFY`                                            |
| Must change after admin reset   | no                 | `MUST_CHANGE`                                            |

Durations are set in seconds. Overlay-level settings are `OPENLDAP_BOOTSTRAP_PPOLICY_HASH_CLEAR_TEXT`, `OPENLDAP_BOOTSTRAP_PPOLICY_USE_LOCKOUT` and `OPENLDAP_BOOTSTRAP_PPOLICY_CHECK_MODULE`.

See [Environment Variables](../environment-variables.md#ppolicy) for every setting.

## Referential Integrity

`OPENLDAP_BOOTSTRAP_REFINT=true` enables the `refint` overlay. When an entry is renamed or deleted, references to it are updated or removed in other entries, so groups do not keep members that no longer exist.

| Variable                               | Default                             |
| -------------------------------------- | ----------------------------------- |
| `OPENLDAP_BOOTSTRAP_REFINT_ATTRIBUTES` | `member uniqueMember manager owner` |

## Unique Attributes

`OPENLDAP_BOOTSTRAP_UNIQUE=true` enables the `unique` overlay, which rejects an entry when another entry already has the same value for a given attribute.

| Variable                         | Default                              |
| -------------------------------- | ------------------------------------ |
| `OPENLDAP_BOOTSTRAP_UNIQUE_URIS` | `ldap:///?uid?sub ldap:///?mail?sub` |

With the default value, `uid` and `mail` are unique across the whole directory.

## MemberOf

`OPENLDAP_BOOTSTRAP_MEMBEROF=true` enables the `memberof` overlay, which maintains a `memberOf` attribute on each member of a group.

| Variable                                  | Default        | Description                                   |
| ----------------------------------------- | -------------- | --------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_GROUP_OC`    | `groupOfNames` | Object class treated as a group               |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_MEMBER_AD`   | `member`       | Membership attribute watched on groups        |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_MEMBEROF_AD` | `memberOf`     | Reverse attribute maintained on members       |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_ADD_CHECK`   | `true`         | Check new entries against existing groups     |
| `OPENLDAP_BOOTSTRAP_MEMBEROF_DANGLING`    | `ignore`       | Handling of members that do not exist         |

!!! note

    `memberOf` is not in the list of attributes readable by authenticated users. Applications should read it with the
    [read-only account](access-control.md#read-only-data-account).

When [replication](../replication.md) is also enabled, `memberOf` is excluded from replication and computed locally on each server.

## Overlay Order

Overlays are created in this order: `ppolicy`, `unique`, `refint`, `memberof`, then `syncprov` when replication is enabled.

To add another overlay or change the order, see [Custom LDIF](custom-ldif.md). Keep `syncprov` last.
