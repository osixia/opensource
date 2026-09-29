# osixia/openldap

[docker hub]: https://hub.docker.com/r/osixia/openldap
[github]: https://github.com/osixia/container-openldap

[![Docker Pulls](https://img.shields.io/docker/pulls/osixia/openldap.svg?style=flat-square)][docker hub]
[![Docker Stars](https://img.shields.io/docker/stars/osixia/openldap.svg?style=flat-square)][docker hub]
[![GitHub Stars](https://img.shields.io/github/stars/osixia/container-openldap?label=github%20stars&style=flat-square)][github]
[![Contributors](https://img.shields.io/github/contributors/osixia/container-openldap?style=flat-square)](https://github.com/osixia/container-openldap/graphs/contributors)

[OpenLDAP](https://www.openldap.org/) container image with built-in bootstrap, backup helpers, TLS, replication and upgrade support.

110M+ pulls 🎉 Thanks to everyone using, testing, reporting issues, and contributing to this image. 🙏

!!! warning "v2 is a breaking release"
    Existing v1 deployments are **not** migrated automatically and require a **manual migration** to move to v2.
    Automatic upgrade paths are intended only between v2 versions (for example, v2.x to v2.y).

![Container OpenLDAP](../../../assets/images/osixia-container-openldap.png)

## Features

- **Zero-config bootstrap**: a working directory (`cn=config` + MDB database) is created on first start from environment variables, with random admin passwords when none are provided.
- **Secure defaults**: Argon2 password hashing, closed ACLs (`deny all` on unmatched access), non-root `ldap` user, unprivileged ports `3890` / `6360`.
- **Optional overlays**: password policy (`ppolicy`), referential integrity (`refint`), attribute uniqueness (`unique`) and `memberof`, each enabled with a single variable.
- **TLS**: LDAPS and StartTLS with your own certificates, and an option to reject non-TLS clients.
- **Multi-provider replication**: syncrepl replication between two or more nodes.
- **Backup / restore**: `openldap-ctl backup` and `openldap-ctl restore`, plus an optional cron service for scheduled backups.
- **Upgrade support**: automatic migration between OpenLDAP versions based on pre-upgrade backups.
- **LDAP toolkit**: all OpenLDAP server and client utilities, password helpers and a schema-to-LDIF converter.
- **Extensible**: add your own LDIF (config and data), schemas and modules at bootstrap.

## Start Here

- New to `osixia/openldap`? Start with the [Quickstart](quickstart.md).
- Want to customize the initial directory? Read [Bootstrap](bootstrap.md).
- Going to production? See [TLS](tls.md), [Replication](replication.md), [Backup / Restore](backup-restore.md) and [Upgrade](upgrade.md).
- Looking for a setting? Check the [Environment Variables](environment-variables.md) reference.

This image is based on [osixia/baseimage](../baseimage/index.md): all `osixia/baseimage` runtime options (`--log-level`, `--bash`, `--exec`, `--skip-process`, ...) are available.
