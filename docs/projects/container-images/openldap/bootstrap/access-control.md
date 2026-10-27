# Access Control

Bootstrap creates a closed-by-default set of access rules (ACLs). Anything that is not explicitly allowed is denied.

## Who Can Do What

On the data database, with default settings:

| Identity                              | Access                                                                                   |
| ------------------------------------- | ---------------------------------------------------------------------------------------- |
| Data root DN (`cn=admin,<suffix>`)    | Full access                                                                              |
| `ldap` user through `ldapi:///`       | Full access (`manage`)                                                                   |
| Authenticated user                    | Reads a short list of attributes on every entry; changes their own password              |
| Anonymous                             | Can only authenticate (bind); cannot read anything                                       |
| Read-only account (optional)          | Reads everything except passwords                                                        |
| Replication account (optional)        | Reads everything, including passwords                                                    |

### Authenticated Users

Any authenticated user can read these attributes, on their own entry and on others:

`objectClass`, `cn`, `sn`, `givenName`, `mail`, `uid`, `ou`, `o`, `dc`

Other attributes are hidden, including on the user's own entry. A user can change their own `userPassword`.

!!! note

    Applications that need more than this list, such as group membership or POSIX attributes,
    should bind with the [read-only account](#read-only-data-account) instead.

### Local Administration

Inside the container, the `ldap` user has full access to the data and monitor databases through the local socket, without a password:

``` bash
docker exec openldap container run -- \
  ldapsearch -Q -Y EXTERNAL -H ldapi:/// -b "dc=example,dc=org"
```

The configuration database (`cn=config`) is only accessible with its root DN, `cn=admin,cn=config`, and its password.

## Read-Only Data Account

Set `OPENLDAP_BOOTSTRAP_DATA_READONLY=true` to create an account that can read the whole data database except `userPassword` and `shadowLastChange`. It is intended for applications that look up users and groups.

``` bash
docker run --name openldap \
  -e OPENLDAP_BOOTSTRAP_DATA_READONLY=true \
  osixia/openldap:2.6.15-rc
```

| Variable                                           | Default                                    |
| -------------------------------------------------- | ------------------------------------------ |
| `OPENLDAP_BOOTSTRAP_DATA_READONLY_DN`              | `cn=readonly,${OPENLDAP_BOOTSTRAP_SUFFIX}` |
| `OPENLDAP_BOOTSTRAP_DATA_READONLY_PASSWORD_HASHED` | empty: a password is generated and logged  |

## Monitor Database

The monitor database (`cn=Monitor`) is always created. `OPENLDAP_BOOTSTRAP_MONITOR` sets its `olcMonitoring` value.

Set `OPENLDAP_BOOTSTRAP_MONITOR_READONLY=true` to create an account that can read it, for example for a metrics exporter:

| Variable                                              | Default                                            |
| ----------------------------------------------------- | -------------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_MONITOR_READONLY_DN`              | `cn=readonly-monitor,${OPENLDAP_BOOTSTRAP_SUFFIX}` |
| `OPENLDAP_BOOTSTRAP_MONITOR_READONLY_PASSWORD_HASHED` | empty: a password is generated and logged          |

## Default Rules

These are the rules generated for the data database, in order. Rules for optional accounts are only present when the feature is enabled.

``` ldif
olcAccess: to *
  by dn.exact="gidNumber=911+uidNumber=911,cn=peercred,cn=external,cn=auth" manage
  by * break
olcAccess: to attrs=userPassword,shadowLastChange
  by self write
  by anonymous auth
  by * break
olcAccess: to attrs=entry,objectClass,cn,sn,givenName,mail,uid,ou,o,dc
  by self read
  by users read
  by * break
# read-only account, when enabled
olcAccess: to attrs=userPassword,shadowLastChange
  by dn.exact="cn=readonly,dc=example,dc=org" none
  by * break
olcAccess: to *
  by dn.exact="cn=readonly,dc=example,dc=org" read
  by * break
# replication account, when enabled
olcAccess: to *
  by dn.exact="cn=replicator,dc=example,dc=org" read
  by * break
# final rule
olcAccess: to *
  by * none
```

!!! info "Schema dependencies"

    The default rules reference `mail` (from `cosine.ldif`) and `shadowLastChange` (from `rfc2307bis.ldif`).
    If you remove one of these schemas from `OPENLDAP_BOOTSTRAP_SCHEMAS`, replace the default rules as described below.

## Changing the Rules

Access rules are built from separate LDIF fragments, so one part can be changed without rewriting the others:

| Fragment                                     | Content                                         |
| -------------------------------------------- | ----------------------------------------------- |
| `65-database-acl.ldif.template`              | Local administration, passwords, basic attributes |
| `66-database-readonly-acl.ldif.template`     | Read-only account                               |
| `67-database-replication-acl.ldif.template`  | Replication account                             |
| `69-database-acl-close.ldif`                 | Final deny-all rule                             |

To change the default rules before first start, replace `65-database-acl.ldif.template` with your own file. See [Custom LDIF](custom-ldif.md#replace-a-built-in-fragment).

On an existing directory, change `olcAccess` on `olcDatabase={1}mdb,cn=config` with `ldapmodify`, bound as the configuration root DN.
