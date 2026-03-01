# 🏗️ Production and Integration

## CI/CD Pipeline Optimization

The project is designed to keep Docker builds cache-friendly.

Recommended pattern:

1. Download packages and large external resources in Dockerfile `RUN` layers.
2. Copy services into `/container/services`.
3. Run `container services install && container services link`.
4. Copy runtime environment files last.

That is why generated service documentation recommends:

- keep `install.sh` for service installation logic only
- keep heavy downloads outside `install.sh` when possible
- use `startup.sh` for runtime customization based on environment variables

Why this helps:

- Docker can reuse download layers even if a service script changes.
- Changing `install.sh` does not force external dependencies to be downloaded again if those downloads live in earlier Dockerfile layers.
- Environment files are copied after service installation, so runtime config changes do not invalidate installation layers.

For CI jobs where write durability is not important, `--fast-write` can also help reduce package install and file operation overhead by disabling `fsync` and related sync operations through `eatmydata`.
Use it for short-lived build or test containers, not for cases where you need conservative disk-write guarantees.

## Adding `container` to an Existing Image

You can also add the `container` binary to an existing image instead of starting directly from `osixia/baseimage`.

This is useful when:

- you already have a working image and want to adopt the lifecycle tooling incrementally
- you want to keep a custom base image but still use `container` as entrypoint
- you want access to commands such as `container install`, `container services`, `container log`, and the managed runtime lifecycle

Example:

```Dockerfile
FROM debian:trixie-slim

ARG OSIXIA_BASEIMAGE_VERSION="2.0.0-beta"
ARG OSIXIA_BASEIMAGE_ARCH="amd64"

RUN apt-get update && apt-get install -y nginx xz-utils
RUN echo "daemon off;" >> /etc/nginx/nginx.conf
CMD ["/usr/sbin/nginx"]

ADD https://github.com/osixia/container-baseimage/releases/download/${OSIXIA_BASEIMAGE_VERSION}/container_linux_${OSIXIA_BASEIMAGE_ARCH}.tar.gz /tmp
RUN tar -xf /tmp/container_linux_${OSIXIA_BASEIMAGE_ARCH}.tar.gz \
    && rm -rf /tmp/* \
    && mv container_linux_${OSIXIA_BASEIMAGE_ARCH} /usr/sbin/container \
    && /usr/sbin/container install

ENTRYPOINT ["/usr/sbin/container", "--"]
```

What this does:

- installs your application packages as usual
- downloads a released `container` binary matching the target architecture
- installs the baseimage helper files with `container install`
- switches the image entrypoint to the osixia runtime wrapper

In practice, once the binary is installed, the image can use the same runtime patterns as an image built directly from `osixia/baseimage`.

For full entrypoint support, it is better to define a real service with a `process.sh` script instead of relying only on a plain `CMD`.
That way the image can benefit from the full lifecycle and runtime controls, including:

- `--exec` and `--skip`
- `--restart`
- `--step startup|process|finish`
- `--pre-startup-cmd`, `--pre-process-cmd`, `--pre-finish-cmd`, `--pre-exit-cmd`
- `--bash`

Recommended pattern:

```Dockerfile
FROM debian:trixie-slim

ARG OSIXIA_BASEIMAGE_VERSION="2.0.0-beta"
ARG OSIXIA_BASEIMAGE_ARCH="amd64"

RUN apt-get update && apt-get install -y nginx xz-utils

ADD https://github.com/osixia/container-baseimage/releases/download/${OSIXIA_BASEIMAGE_VERSION}/container_linux_${OSIXIA_BASEIMAGE_ARCH}.tar.gz /tmp
RUN tar -xf /tmp/container_linux_${OSIXIA_BASEIMAGE_ARCH}.tar.gz \
    && rm -rf /tmp/* \
    && mv container_linux_${OSIXIA_BASEIMAGE_ARCH} /usr/sbin/container \
    && /usr/sbin/container install

RUN mkdir -p /container/services/nginx

COPY process.sh /container/services/nginx/process.sh

RUN chmod +x /container/services/nginx/process.sh \
    && container services link nginx

ENTRYPOINT ["/usr/sbin/container"]
```

Example `process.sh`:

```bash
#!/bin/bash -e

container log level eq trace && set -x

exec /usr/sbin/nginx -g "daemon off;" "$@"
```

Keep in mind:

- you are responsible for downloading the right binary for your target architecture
- `container install` prepares the runtime environment, but your image still needs service files or commands that make sense for your application
- if you keep a `CMD`, it becomes the default command executed by `container` entrypoint
- if you want the richest integration with the baseimage runtime, prefer service scripts over a bare `CMD`

## Container Debugging

Useful runtime options:

- `--bash`: starts Bash alongside services or a standalone command
- `--log-level debug|trace`
- `--log-format console|json`
- `--fast-write`: disables `fsync` via `eatmydata` for faster ephemeral writes

Useful package-management command:

- `container packages install --debug-tools --update`: installs common debugging tools for the current distribution

Useful commands inside a running container:

- `container services status`
- `container processes status`
- `container processes stop <name>`
- `container processes start <name>`
- `container environment print`

How debugging works internally:

- process PID files are stored under `/run/container/processes`
- a `.down` file is used to request a managed process stop
- stale PID files are cleaned up on startup
- when the entrypoint is PID 1, it reaps zombie processes and can terminate remaining child processes on exit

Practical debugging examples:

```bash
docker run --rm -it my-image --bash --log-level debug
```

```bash
docker run --rm -it my-image --skip-process --bash
```

```bash
docker run --rm my-image --exec nginx --log-level trace
```

## Running Read-Only & Rootless Containers

The image supports non-root execution and can work with read-only container filesystems. If you want baseimage runtime process management to work normally, `/run/container` must be writable.

Why it is needed for process management:

- runtime state is stored in `/run/container`
- process PID files and stop markers are created there, including the state used by `container processes`
- the generator also writes there when used inside a container
- the entrypoint ensures `/run/container`, `/run/container/processes`, and `/run/container/generator` exist and tries to set them to `0777`

If your image or runtime flow does not rely on those runtime management features, you may not need `/run/container` to be writable. But process management and many baseimage features assume it is available.

Typical read-only runtime pattern:

```bash
docker run --rm \
  --read-only \
  --tmpfs /run/container \
  my-image
```

Typical rootless runtime pattern with the provided non-root image:

```bash
docker run --rm \
  --user 65532:65532 \
  my-image
```

Notes:

- the repository publishes a dedicated `nonroot` image target that switches to user `nonroot` with uid/gid `65532`
- for most non-root use cases, prefer using the provided `nonroot` image variant directly
- you can also create your own application user in your Dockerfile if you want explicit uid/gid values or service-specific ownership
- example:

```Dockerfile
# Add application group and user
ARG APP_GROUP_GID=10001
ARG APP_USER_UID=10001

RUN container groups add "${APP_GROUP_GID}" app \
    && container users add "${APP_USER_UID}" app --group-id "${APP_GROUP_GID}" --group-name app
```

- runtime package installation commands such as `container packages install --debug-tools --update` generally require enough privileges to use the distribution package manager
- if the container filesystem is read-only, any service that writes outside writable mounts will still fail

## Container Logging

Use `container log` in service scripts when you want messages that follow the container log level and format.

Common commands:

```bash
container log info "Starting application"
container log warning "Configuration file is missing"
container log error "Startup failed"
container log debug "Resolved config: ${CONFIG_PATH}"
container log trace "Shell tracing enabled"
```

`container log level` can also be used in scripts to inspect the current log level or compare against it.

Show current level:

```bash
container log level
```

Comparison commands:

```bash
container log level eq trace
container log level ne info
container log level gt debug
container log level ge debug
container log level lt warning
container log level le info
```

Those commands exit with status `0` when the comparison is true and `1` otherwise, which makes them convenient in shell scripts.

Typical example:

```bash
container log level eq trace && set -x
```

This enables shell tracing only when the container log level is exactly `trace`.

## Limitations

Current behavior has a few important constraints:

- `process.sh` arguments are shared across all running services; there is no per-service argument mapping
- multi-process restart is coarse-grained: when restart is enabled, any running process is restarted after it exits, not only on specific exit codes
- runtime process management expects `/run/container` to be writable, including `container processes` state tracking
- pre-step commands are shell-split, not shell-evaluated; shell features require `sh -c` or `bash -c`
- `startup.sh` and `finish.sh` are sequential; only `process.sh` runs concurrently
- runtime package installation and debug tooling depend on the underlying distribution package manager and privileges
- environment files are loaded from `/container/environment`, but existing process environment variables are applied again afterward, so runtime-provided environment variables win over `.env` values
