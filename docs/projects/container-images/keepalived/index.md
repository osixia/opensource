# Keepalived

[docker hub]: https://hub.docker.com/r/osixia/keepalived
[github]: https://github.com/osixia/container-keepalived

[![Docker Pulls](https://img.shields.io/docker/pulls/osixia/keepalived.svg?style=flat-square)][docker hub]
[![Docker Stars](https://img.shields.io/docker/stars/osixia/keepalived.svg?style=flat-square)][docker hub]
[![GitHub Stars](https://img.shields.io/github/stars/osixia/container-keepalived?label=github%20stars&style=flat-square)][github]
[![Contributors](https://img.shields.io/github/contributors/osixia/container-keepalived?style=flat-square)](https://github.com/osixia/container-keepalived/graphs/contributors)

[Keepalived](https://keepalived.org/) container image for VRRP high availability and VIP failover with dynamic configuration and hot reload.

![Container Keepalived](../../../assets/images/osixia-container-keepalived.png)

## Features

- **Ready in one command**: a virtual IP shared between your nodes, configured with a few environment variables.
- **Three levels of configuration**: environment variables, your own template, or your own `keepalived.conf`.
- **Hot reload**: Keepalived reloads when its configuration file changes, without restarting the container.
- **State changes in the container logs**: `MASTER`, `BACKUP` and `FAULT` transitions are logged, and you can plug in your own notify script.
- **Up-to-date Keepalived**: built from the upstream sources on Alpine, with JSON status support.

## Start Here

- New to `osixia/keepalived`? Start with the [Quickstart](quickstart.md) to share a virtual IP between two nodes.
- Need more than the default setup? See [Configuration](configuration.md).
- Something is not working? Read [Troubleshooting](troubleshooting.md).
- Coming from `osixia/keepalived` 2.1.5 or older? Read [Upgrade](upgrade.md).
- Looking for a setting? All of them are listed in [Environment Variables](environment-variables.md).

This image is based on [osixia/baseimage](../baseimage/index.md).

On Kubernetes, [kube-network-detector](../../kubernetes/kube-network-detector/index.md) can detect which node receives traffic for a VIP and label that node.
