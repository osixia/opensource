# osixia/baseimage

[docker hub]: https://hub.docker.com/r/osixia/baseimage
[github]: https://github.com/osixia/container-baseimage

[![Docker Pulls](https://img.shields.io/docker/pulls/osixia/baseimage.svg?style=flat-square)][docker hub]
[![Docker Stars](https://img.shields.io/docker/stars/osixia/baseimage.svg?style=flat-square)][docker hub]
[![GitHub Stars](https://img.shields.io/github/stars/osixia/container-baseimage?label=github%20stars&style=flat-square)][github]
[![Contributors](https://img.shields.io/github/contributors/osixia/container-baseimage?style=flat-square)](https://github.com/osixia/container-baseimage/graphs/contributors)

Debian, Alpine, and Ubuntu container base images designed to build reliable containers quickly.

**This image provides a simple, opinionated solution for building single- or multi-process container images with a minimal number of layers and an optimized build process.**

It accelerates image development and CI/CD pipelines by offering:

 - Advanced build tools to reduce image layers and maximize Docker layer caching efficiency.
 - A lightweight init system as the container entrypoint, providing enhanced process management, debugging capabilities, and runtime flexibility.
 - Built-in support for multi-process containers. Run all processes within a single container or split them across multiple containers as needed.

Includes non-root container images and supports read-only container environments.

![Container Baseimage](../../../assets/images/osixia-container-baseimage.jpg)
