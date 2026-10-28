# 🖥️ phpLDAPadmin

Add a web interface to an existing LDAP directory. This chart uses phpLDAPadmin 2; it does not install an LDAP server.

## Connect to your directory

Save this as `phpldapadmin-values.yaml`, adjusting the host and base DN:

```yaml
app:
  ldap:
    host: my-ldap-openldap.directory.svc.cluster.local
    port: 389
    connection: starttls
    tls: true

extraEnv:
  - name: LDAP_BASE_DN
    value: dc=example,dc=org
  - name: LDAP_LOGIN_ATTR
    value: dn
  - name: LDAP_LOGIN_OBJECTCLASS
    value: organizationalRole
  - name: LDAPTLS_CACERT
    value: /ldap-ca/ca.crt
  - name: LDAPTLS_REQCERT
    value: demand

extraEnvFrom:
  - secretRef:
      name: ldap-admin-bind

extraVolumes:
  - name: ldap-ca
    secret:
      secretName: ldap-tls
extraVolumeMounts:
  - name: ldap-ca
    mountPath: /ldap-ca
    readOnly: true
```

This assumes [OpenLDAP has TLS enabled](openldap.md#tls-for-a-new-installation) and the `ldap-tls` Secret contains `ca.crt` in the same namespace. The LDAP hostname must match the server certificate. Here, `tls: true` enables StartTLS on port `389`. For LDAPS, use port `636`, `connection: ldaps`, `ssl: true` and `tls: false`.

phpLDAPadmin 2 needs a real directory entry to log in. The OpenLDAP data root (`cn=admin,<suffix>`) is a virtual identity and cannot log in. Create a [scoped directory administrator](../../container-images/openldap/bootstrap/access-control.md#directory-administrator) before the first OpenLDAP installation by adding these values alongside your existing bootstrap settings:

```yaml
openldap:
  bootstrap:
    database:
      tree:
        enabled: true
      admin:
        enabled: true
        passwordHashed: "{ARGON2}..."
```

Generate a separate password/hash for this account as in the [OpenLDAP guide](openldap.md#1-prepare-passwords). Put its plaintext password in a local file named `admin-password.txt`, without a trailing newline, then create the search-credentials Secret:

```bash
kubectl -n directory create secret generic ldap-admin-bind \
  --from-literal=LDAP_USERNAME='cn=admin,ou=services,dc=example,dc=org' \
  --from-file=LDAP_PASSWORD=admin-password.txt
```

The same administrator is used for search and login here. For a complete setup with generated credentials, use the [application examples](examples.md).

## Install and open

After [adding the chart repository](index.md#add-the-repository):

```bash
helm install ldap-admin osixia/phpldapadmin \
  -n directory -f phpldapadmin-values.yaml --wait
kubectl -n directory port-forward svc/ldap-admin-phpldapadmin 8080:80
```

Open <http://localhost:8080> and log in with `cn=admin,ou=services,dc=example,dc=org` and its password. The service exposes port `80` and forwards to the image's port `8080`.

## Useful settings

| Value | What it controls |
| --- | --- |
| `app.ldap.host`, `port`, `tls`, `ssl` | LDAP endpoint and transport |
| `extraEnv`, `extraEnvFrom` | Additional application settings and credentials from Secrets |
| `app.existingSecret` | Existing Secret containing `APP_KEY` |
| `ingress.enabled` | Expose through a Kubernetes Ingress |
| `httpRoute.enabled` | Expose through Gateway API; requires its CRDs and a compatible controller |

The chart generates a stable `APP_KEY` across upgrades if you do not supply one. Expose the web interface over HTTPS when accessing it remotely.

[Chart values and source](https://github.com/osixia/helm-charts/tree/main/phpldapadmin)
