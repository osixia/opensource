# 🏗️ Production and Integration

## CI/CD Pipeline Optimization

Keep Docker builds cache-friendly by ordering the `Dockerfile` from the slowest to the most frequently
changed layers:

1. Install packages and download large resources in `RUN` layers.
2. Copy services into `/container/services`.
3. Run `container services install && container services link`.
4. Copy environment files last.

Changing a service script then re-runs only the install layer, without downloading anything again, and
changing a default environment value does not invalidate the installation.

For CI jobs where write durability does not matter, `--fast-write` disables `fsync` with `eatmydata` to
speed up package installs and file operations. Use it for short-lived build or test containers only.

## Adding `container` to an Existing Image

The `container` binary can be added to any Debian, Ubuntu or Alpine based image, to adopt the lifecycle
and runtime commands without starting from `osixia/baseimage`:

```Dockerfile
FROM debian:trixie-slim

ARG TARGETARCH

RUN apt-get update && apt-get install -y nginx

ADD https://github.com/osixia/container-baseimage/releases/latest/download/container_linux_${TARGETARCH}.tar.gz /tmp
RUN tar -xf /tmp/container_linux_${TARGETARCH}.tar.gz -C /tmp \
    && mv /tmp/container_linux_${TARGETARCH} /usr/sbin/container \
    && rm -rf /tmp/* \
    && container install

COPY --chmod=755 process.sh /container/services/nginx/process.sh
RUN container services link nginx

ENTRYPOINT ["/usr/sbin/container"]
```

With `process.sh`:

```bash
#!/bin/bash
set -euo pipefail

container log level eq trace && set -x

exec /usr/sbin/nginx -g "daemon off;" "$@"
```

- Release binaries are published for `amd64` and `arm64`; `TARGETARCH` is set by BuildKit for the
  target platform.
- `container install` prepares the runtime: the image still needs services that make sense for your
  application.
- A plain `CMD` also works: with no linked service, it runs as the container command. A service with a
  `process.sh` gives access to the full lifecycle: `--exec`, `--skip`, `--restart`, `--step`, pre-step
  commands and `--bash`.

## Container Debugging

Useful options and commands:

- `--log-level debug|trace` and `--log-format console|json`
- `container packages install --debug-tools --update`: installs common debugging tools
- inside a running container: `container services status`, `container processes status`,
  `container processes stop|start <name>`, `container environment print`

Shell recipes:

```bash
# Bash in the base image (no service, no command)
docker run --rm -it osixia/baseimage

# Bash without running any service script
docker run --rm -it my-image --skip-all

# Bash alongside the linked services, with debug logs
docker run --rm -it my-image --bash --log-level debug

# startup scripts, then Bash instead of the service processes, then finish scripts
docker run --rm -it my-image --skip-process --bash

# a single service with trace logs
docker run --rm my-image --exec nginx --log-level trace
```

Process state lives in `/run/container/processes`: a `.pid` file per running process, and a `.down` file
to request a stop. When the entrypoint starts as PID 1, it removes the stop requests, starting markers
and PID files left by a previous run, only for the services selected in that container.

## Read-Only and Non-Root Containers

The runtime does not need root: services and `container processes` commands work with any UID,
even one with no user account.

Run without root in one of two ways:

- build on a `-nonroot` tag, such as `osixia/baseimage:latest-nonroot` or
  `osixia/baseimage:alpine-nonroot`: the image runs as `nonroot` (uid/gid `65532`) by default.
  Switch to `root` for the build steps that need it, then back to `nonroot`:

    ```Dockerfile
    FROM osixia/baseimage:latest-nonroot

    USER root
    RUN container packages install --update --clean nginx

    COPY services /container/services
    RUN container services install \
        && container services link

    USER nonroot
    ```

- or set the user at runtime: `docker run --user 10001:10001 my-image`, or `runAsUser` in Kubernetes.

For a read-only root filesystem, keep `/run/container` writable:

```bash
docker run --rm --read-only --tmpfs /run/container:rw,mode=0777 my-image
```

Prepare downloads and installations at build time: a non-root or read-only runtime cannot install
packages.

### Mounted Volumes

A volume mounted on `/run/container` (`tmpfs`, Kubernetes `emptyDir`, bind mount...) is used as is: the
runtime never changes its owner or permissions. Make it writable by the users that run and control the
services. Directories the runtime creates itself are writable by everyone.

## Sharing Runtime State Between Containers

Containers can share `/run/container` while supervising different services, for example with
`--exec nginx` in one and `--exec php-fpm` in another. Each service must have a single supervisor:
use distinct service names.

Each PID 1 supervisor only cleans the files of its own services at startup, and `container processes`
controls the services of every container through the shared markers. Status is read from these files:
a PID only makes sense in the PID namespace of the supervisor that wrote it.

This directory is shared control state, not a security boundary: a user allowed to modify it can stop
services, change the reported state or disrupt supervision. Share it only between trusted participants.

## Health Checks

`container processes check` reports whether processes are up according to the supervisor runtime
files. Use it as a Docker health check or a Kubernetes probe:

- it exits with status `0` and prints nothing when every checked process is up
- it exits with status `1` and logs `<name>: process is down` for each process that is down
- a process stopped on purpose with `container processes stop` is not reported as down
- any container user can run it

With no argument, it checks the processes of the services linked to the entrypoint. A process waiting to
be restarted is reported as down until it runs again.

Pass process names or `tag:name` to check a selection. This is required when the container runs with
`--exec` or `--skip`: the default check would report the services that are not run as down. A selection
matching no process makes the check fail, so a typo does not silently pass.

```bash
container processes check nginx php-fpm
container processes check tag:web
```

=== "Docker"

    ```Dockerfile
    HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
      CMD container processes check
    ```

=== "Docker Compose"

    ```yaml
    services:
      app:
        image: my-image
        healthcheck:
          test: ["CMD", "container", "processes", "check"]
          interval: 30s
          timeout: 5s
          start_period: 30s
          retries: 3
    ```

=== "Kubernetes"

    ```yaml
    livenessProbe:
      exec:
        command: ["container", "processes", "check"]
      initialDelaySeconds: 30
      periodSeconds: 30
      timeoutSeconds: 5
      failureThreshold: 3
    ```

- Cover the startup step with the start period (`--start-period`, a `startupProbe` or
  `initialDelaySeconds`): services do not run yet while `startup.sh` scripts run.
- With restart enabled (the multi-process default), the supervisor already restarts failed processes:
  set the probe `failureThreshold` above the time a process needs to restart.
- The check tells whether processes run, not whether they work, and stopped processes pass it: use
  application checks (HTTP, TCP) for readiness.

## Container Logging

Use `container log` in service scripts so messages follow the container log level and format:

```bash
container log info "Starting application"
container log warning "Configuration file is missing"
container log error "Startup failed"
container log debug "Resolved config: ${CONFIG_PATH}"
```

`container log level` prints the current level, and compares it with `eq`, `ne`, `gt`, `ge`, `lt` or
`le`: the command exits with status `0` when the comparison is true. For example, to trace a script
only at `trace` level:

```bash
container log level eq trace && set -x
```

Secrets and log levels:

- commands run by `container` are logged with their executable path only at `info` level and in
  errors; their arguments, which may hold passwords, are shown at `debug` level
- package, user and group commands keep their full command line: their arguments are not secrets
- environment variable values are only logged at `debug` level: avoid `debug` and `trace` in production
  when environment variables hold secrets

## Limitations

- `process.sh` arguments are shared by all running services: there is no per-service mapping.
- When restart is enabled, a process is restarted after any exit, whatever its exit code.
- Process management needs a writable `/run/container`.
- Pre-step commands are split like a shell would, not evaluated by a shell.
- Run the supervisor directly as PID 1: another init in front of it is not supported (see
  [PID 1 and container init](advanced-runtime-guide.md#pid-1-and-container-init)).
