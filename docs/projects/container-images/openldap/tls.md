# 🔐 TLS

TLS is configured at bootstrap. Set `OPENLDAP_BOOTSTRAP_TLS=true` and provide a certificate, its private key and the CA certificate.

## Enable TLS

The default certificate paths are:

- `/container/services/openldap/assets/certs/tls.crt`
- `/container/services/openldap/assets/certs/tls.key`
- `/container/services/openldap/assets/certs/ca.crt`

The simplest way is to mount a directory that contains files with exactly those names:

``` bash
docker run --name openldap --hostname ldap.example.org -p 389:3890 -p 636:6360 \
  -e OPENLDAP_BOOTSTRAP_TLS=true \
  -v $PWD/certs:/container/services/openldap/assets/certs \
  osixia/openldap:2.6.15-rc
```

The files must be readable by the `ldap` user (uid `911`).

Once TLS is enabled, clients can connect with:

- **LDAPS** on port `6360` (`ldaps://`);
- **StartTLS** on port `3890` (`ldap://`).

The certificate must match the name clients use to reach the server, here `ldap.example.org`.

## Settings

| Variable                               | Description                    | Default                                             |
| -------------------------------------- | ------------------------------ | --------------------------------------------------- |
| `OPENLDAP_BOOTSTRAP_TLS_CERT`          | Server certificate path        | `/container/services/openldap/assets/certs/tls.crt` |
| `OPENLDAP_BOOTSTRAP_TLS_CERT_KEY`      | Server private key path        | `/container/services/openldap/assets/certs/tls.key` |
| `OPENLDAP_BOOTSTRAP_TLS_CA_CERT`       | CA certificate path            | `/container/services/openldap/assets/certs/ca.crt`  |
| `OPENLDAP_BOOTSTRAP_TLS_VERIFY_CLIENT` | Value for `olcTLSVerifyClient` | `allow`                                             |
| `OPENLDAP_BOOTSTRAP_TLS_PROTOCOL_MIN`  | Value for `olcTLSProtocolMin`  | `3.4`                                               |

- `OPENLDAP_BOOTSTRAP_TLS_PROTOCOL_MIN=3.4` means TLS 1.3 is the minimum version. Use `3.3` to also accept TLS 1.2.
- `OPENLDAP_BOOTSTRAP_TLS_VERIFY_CLIENT=allow` means a client certificate is requested but not required. Use `demand` to require a valid client certificate, or `never` to not request one.

## Require TLS

By default, enabling TLS does not forbid unencrypted connections on port `3890`.

Set `OPENLDAP_BOOTSTRAP_TLS_REQUIRED=true` to reject them:

``` bash
docker run --name openldap --hostname ldap.example.org -p 389:3890 -p 636:6360 \
  -e OPENLDAP_BOOTSTRAP_TLS=true \
  -e OPENLDAP_BOOTSTRAP_TLS_REQUIRED=true \
  -v $PWD/certs:/container/services/openldap/assets/certs \
  osixia/openldap:2.6.15-rc
```

Bootstrap then requires a security strength of at least 128 bits for every operation (`olcSecurity: ssf=128`). The local `ldapi:///` socket is treated as secure, so administration from inside the container keeps working.

When TLS is required, [replication](replication.md#tls) uses StartTLS by default.

!!! warning

    `OPENLDAP_BOOTSTRAP_TLS_REQUIRED=true` without `OPENLDAP_BOOTSTRAP_TLS=true` makes the server unreachable over the network.
    Bootstrap logs a warning in that case.

## Renewing Certificates

Certificate paths are stored in the configuration at bootstrap. To renew a certificate, replace the files at the same paths and restart the container.

To change TLS settings on an existing directory, modify the `olcTLS*` attributes of `cn=config` with `ldapmodify`.
