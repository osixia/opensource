# 🔐 TLS

Set `OPENLDAP_BOOTSTRAP_TLS=true` to configure TLS during [bootstrap](bootstrap.md). This enables both LDAPS (port `6360`) and StartTLS on the LDAP port (`3890`).

The default certificate paths are:

| File                                                  | Variable                          |
| ----------------------------------------------------- | --------------------------------- |
| `/container/services/openldap/assets/certs/tls.crt`   | `OPENLDAP_BOOTSTRAP_TLS_CERT`     |
| `/container/services/openldap/assets/certs/tls.key`   | `OPENLDAP_BOOTSTRAP_TLS_CERT_KEY` |
| `/container/services/openldap/assets/certs/ca.crt`    | `OPENLDAP_BOOTSTRAP_TLS_CA_CERT`  |

The simplest way is to mount a directory that contains files with exactly those names:

``` bash
docker run --name openldap --hostname ldap.example.org -p 389:3890 -p 636:6360 \
  -e OPENLDAP_BOOTSTRAP_TLS=true \
  -v $PWD/certs:/container/services/openldap/assets/certs \
  osixia/openldap
```

!!! note
    Files must be readable by the `ldap` user (uid `911`), including the private key.
    The certificate should match the name clients use to connect (here `ldap.example.org`).

Test the connection:

``` bash
# LDAPS
ldapsearch -H ldaps://ldap.example.org -x -b "dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -W
# StartTLS
ldapsearch -H ldap://ldap.example.org -ZZ -x -b "dc=example,dc=org" -D "cn=admin,dc=example,dc=org" -W
```

## TLS settings

| Variable                               | Description                    | Default |
| -------------------------------------- | ------------------------------ | ------- |
| `OPENLDAP_BOOTSTRAP_TLS_VERIFY_CLIENT` | Value for `olcTLSVerifyClient` (`never`, `allow`, `try`, `demand`) | `allow` |
| `OPENLDAP_BOOTSTRAP_TLS_PROTOCOL_MIN`  | Value for `olcTLSProtocolMin` (`3.3` = TLS 1.2, `3.4` = TLS 1.3) | `3.4`   |

## Require TLS

Set `OPENLDAP_BOOTSTRAP_TLS_REQUIRED=true` to reject non-encrypted connections. Bootstrap sets `olcSecurity: ssf=128`: clients must use LDAPS or StartTLS for every operation, including binds.
Local connections through `ldapi:///` are still accepted (`olcLocalSSF: 128`).

``` bash
docker run \
  -e OPENLDAP_BOOTSTRAP_TLS=true \
  -e OPENLDAP_BOOTSTRAP_TLS_REQUIRED=true \
  -v $PWD/certs:/container/services/openldap/assets/certs \
  osixia/openldap
```

!!! warning
    `OPENLDAP_BOOTSTRAP_TLS_REQUIRED` does not configure certificates: always use it together with `OPENLDAP_BOOTSTRAP_TLS=true`.
    With replication, it also enables StartTLS between nodes by default (see [Replication](replication.md#replication-over-tls)).

## Renewing certificates

The certificate files are read by `slapd` at startup. To renew them, replace the files in the mounted directory (keeping the same names) and restart the container.
