# Helm charts

Deploy OpenLDAP and phpLDAPadmin on Kubernetes with the Osixia charts.

![A Shiba Inu captain assembles a deployment kit beside orange containers.](../../../assets/images/osixia-helm-charts.png)

| Chart | What it provides |
| --- | --- |
| [openldap](openldap.md) | An LDAP directory with persistent storage, TLS, replication and optional scheduled backups |
| [phpldapadmin](phpldapadmin.md) | A web interface for managing LDAP users and groups |

## Add the repository

```bash
helm repo add osixia https://charts.opensource.osixia.net
helm repo update osixia
helm search repo osixia
```

Follow the [OpenLDAP guide](openldap.md) to prepare values and install a directory, then add [phpLDAPadmin](phpldapadmin.md) if you want a web interface.

The guides follow the source charts: OpenLDAP chart `0.3.0` targets `osixia/openldap:2.6.15-rc`; phpLDAPadmin chart `0.2.1` targets `phpldapadmin/phpldapadmin:2.3.11`. Check `helm search repo osixia --versions` for published versions.

## Use a local checkout

To use source changes before they are published:

```bash
git clone https://github.com/osixia/helm-charts.git
cd helm-charts
```

Use `./openldap` or `./phpldapadmin` as the chart path in place of `osixia/openldap` or `osixia/phpldapadmin`.

For a working stack with generated credentials and verified StartTLS, see the [Authelia and Keycloak examples](examples.md).

[Source code and complete values](https://github.com/osixia/helm-charts)
