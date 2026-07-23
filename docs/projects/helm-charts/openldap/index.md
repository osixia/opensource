# openldap Helm Chart

[github]: https://github.com/osixia/helm-charts

[![GitHub Stars](https://img.shields.io/github/stars/osixia/helm-charts?label=github%20stars&style=flat-square)][github]
[![Contributors](https://img.shields.io/github/contributors/osixia/helm-charts?style=flat-square)](https://github.com/osixia/helm-charts/graphs/contributors)

Helm chart for the [osixia/openldap](https://hub.docker.com/r/osixia/openldap) container image
([source](https://github.com/osixia/container-openldap)).

It deploys OpenLDAP as a StatefulSet, with support for bootstrap, TLS, persistence,
multi-replica replication, common overlays (ppolicy, refint, memberof, unique), custom
volumes and a scheduled backup CronJob.

![Container OpenLDAP](../../../assets/images/osixia-container-openldap.png)

## Start Here

- New to this chart? Start with the [Quickstart](quickstart.md) to install a working server in 2 minutes.
- Need TLS, replication, overlays, custom volumes, or backups? See [Configuration](configuration.md).
- Looking for a specific value? See the [Values reference](values-reference.md).

Chart source: [osixia/helm-charts](https://github.com/osixia/helm-charts/tree/main/openldap).
