# ⚡ Quickstart

## Install a server in 2 minutes

The only values you **must** change to have a working server:

``` yaml
openldap:
  bootstrap:
    organization: "My Org"
    suffix: "dc=my-org,dc=com"
    database:
      rootPasswordHashed: "{ARGON2}..."   # see Passwords below
    config:
      rootPasswordHashed: "{ARGON2}..."
```

Install:

``` bash
helm install my-ldap ./openldap -f my-values.yaml
```

Connect from inside the cluster:

```
ldap://my-ldap.<namespace>.svc.cluster.local:389
```

Root DN: `cn=admin,dc=my-org,dc=com` — password: whatever you hashed above.

!!! note
    On first start, if a `rootPasswordHashed` is empty, the image generates a random
    password and prints it in the pod logs. That password is **not persisted** across
    restarts unless persistence is enabled.

## DN conventions

All DN fields in this chart use a **prefix** pattern. You only set the RDN part
(everything before the suffix); the chart appends the suffix automatically.

| Value | Assembled DN |
|---|---|
| `bootstrap.suffix: dc=my-org,dc=com` | *(the base suffix)* |
| `database.rootDnPrefix: cn=admin` | `cn=admin,dc=my-org,dc=com` |
| `database.readonly.dnPrefix: cn=readonly` | `cn=readonly,dc=my-org,dc=com` |
| `config.rootDnPrefix: cn=admin` | `cn=admin,cn=config` |
| `ppolicy.groupDnPrefix: ou=Policies` | `ou=Policies,dc=my-org,dc=com` |
| `ppolicy.default.policyDnPrefix: cn=default` | `cn=default,ou=Policies,dc=my-org,dc=com` |
| `replication.dataReadonlyDnPrefix: cn=replicator` | `cn=replicator,dc=my-org,dc=com` |

## Passwords

Passwords are stored as **hashed values**. Generate them with the image:

``` bash
# hash a specific password
docker run --rm osixia/openldap run -- openldap-ctl password hash passw0rd

# generate a random password + hash
docker run --rm osixia/openldap run -- openldap-ctl password generate
```

Use the resulting `{ARGON2}...` string in `rootPasswordHashed` fields.

If a `rootPasswordHashed` is left empty, the image generates a random password at
bootstrap time and logs it. **With persistence enabled** the hash is written to the
database and survives restarts; without persistence it is lost on every restart.

## Next steps

Head to [Configuration](configuration.md) for TLS, replication, overlays, custom
volumes, extra environment variables and the backup CronJob.
