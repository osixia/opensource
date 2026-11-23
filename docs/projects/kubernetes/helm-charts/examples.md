# 🎮 LDAP with Authelia or Keycloak

The chart repository contains two ready-to-try stacks. Each creates its own namespace, credentials and local CA, and uses verified StartTLS between the applications and OpenLDAP.

| Example | What you get |
| --- | --- |
| [Authelia](https://github.com/osixia/helm-charts/tree/main/examples/ldap-stack-authelia) | OpenLDAP, phpLDAPadmin and the official Authelia chart |
| [Keycloak](https://github.com/osixia/helm-charts/tree/main/examples/ldap-stack-keycloak) | OpenLDAP, phpLDAPadmin, the community keycloakx chart with the official Keycloak image, and PostgreSQL |

## Try one

You need Helm 3, `kubectl`, Python 3, OpenSSL and a cluster with dynamic volume provisioning. The OpenLDAP image used by the example must be available to your cluster.

```bash
git clone https://github.com/osixia/helm-charts.git
cd helm-charts/examples/ldap-stack-authelia
python3 prepare.py
```

Then follow that example's README to create the namespace, apply the generated resources and install the releases. For Keycloak, use `examples/ldap-stack-keycloak` instead.

The generator creates a private `generated/` directory containing credentials, Secrets and certificates. Read `generated/credentials.json` for the passwords. It refuses to overwrite an existing directory. These examples are for a fresh installation: regenerating a Secret does not rotate passwords in an existing directory or database.

## Understand the accounts

| Account | Purpose |
| --- | --- |
| Data and config roots | LDAP maintenance and recovery |
| Directory admin | Log in to phpLDAPadmin and manage users/groups |
| Application bind account | Read users/groups and change user passwords for Authelia or Keycloak |
| Alice | Sample user for trying login flows |

Application bind accounts cannot read password hashes or edit profiles/groups. Their privileged password resets bypass LDAP quality and history checks, so configure a matching password policy in the application.

## Before adapting it for real users

Replace the local CA and sample users, configure your public HTTPS hostnames, and set up backups. Authelia needs a real notifier for emails; Keycloak needs its own SMTP settings and database backup. Neither example enables MFA for you.

Each README also includes a reproducible integration test using a disposable kind cluster, covering login flows, password changes, LDAP access rules and StartTLS verification.
