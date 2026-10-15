# 🧩 How It Works

An image based on `osixia/baseimage` is built from three parts:

```
osixia-baseimage-example
├── Dockerfile
├── environment
│   └── .env
└── services
    └── service-1
        ├── finish.sh
        ├── install.sh
        ├── process.sh
        └── startup.sh
```

- the `Dockerfile` installs packages and prepares the services at build time
- `services/` holds one directory per service, with its lifecycle scripts
- `environment/` holds the default environment variables

## Dockerfile

A typical `Dockerfile` extending `osixia/baseimage` looks like this:

```Dockerfile
FROM osixia/baseimage

ARG IMAGE="osixia/baseimage-example:latest"
ENV CONTAINER_IMAGE=${IMAGE}

# Download services required packages or resources
# RUN container packages install --update --clean \
#        [...]
#     && curl -o resources.tar.gz https://[...].tar.gz \
#     && tar -xzf resources.tar.gz \
#     && rm resources.tar.gz

COPY services /container/services

# Install and link services to the entrypoint
RUN container services install \
    && container services link

COPY environment /container/environment
```

Line by line:

- `ARG IMAGE` / `ENV CONTAINER_IMAGE`: the image name is shown in the container logs, which helps
  debugging. Set it at build time:
  `docker build -t example/my-image:develop --build-arg IMAGE=example/my-image:develop .`
- The commented `RUN` installs packages and downloads resources. Keep slow downloads here: this layer
  stays cached while you iterate on service scripts. `container packages install --update --clean cowsay`
  updates the package index, installs the package and cleans the cache, the same way on Debian, Ubuntu
  and Alpine.
- `COPY services`: copies the services into `/container/services`. Changing a script only re-runs the
  following layers.
- `container services install && container services link`: runs the `install.sh` script of every
  non-optional service, then links these services to the entrypoint so they run when the container
  starts.
- `COPY environment`: copied last, so changing a default value does not invalidate the installation
  layers.

## Service Files

Every file is optional. Other files can be added to a service directory without changing its behavior.

| File | Role |
| --- | --- |
| `download.sh` | build time: downloads the service resources |
| `install.sh` | build time: installs the service |
| `startup.sh` | container start: prepares the service from the runtime environment |
| `process.sh` | process step: the long-running command |
| `finish.sh` | finish step, after the process step has ended |
| `.priority` | order of the download, install, startup and finish scripts |
| `.stop-signal` | signal sent to stop `process.sh` |
| `.optional` | service not prepared nor linked by default |
| `.tags/` | tags used to select services and processes |

### download.sh
Downloads the resources a service needs. It runs at build time with `container services download`,
which downloads all non-optional services when called with no argument.

Prefer installing packages and downloading large resources directly in the `Dockerfile`, where Docker
caches them. `download.sh` is mostly useful for [optional services](#optional) shipped by a base image.

### install.sh
Contains the initial setup of the service. It runs during the build, so runtime environment variables
cannot customize it: use `startup.sh` for that.

Keep package installations and downloads out of `install.sh` when possible: changing it then does not
download the dependencies again.

### startup.sh
Prepares `process.sh` and adapts the service setup to runtime environment variables.

### process.sh
Starts the long-running command of the service.

In a multi-process image, all `process.sh` scripts start at the same time: `.priority` does not
order them.

Start the command with `exec`: signals such as `SIGHUP` are forwarded to the `process.sh` main process
only, and a shell that does not `exec` its command would not pass them on.

### finish.sh
Runs during the finish step, after the process step has ended. In a multi-process container, it does not
run each time an individual process exits or restarts.

### .priority
Sets the order of the `download.sh`, `install.sh`, `startup.sh` and `finish.sh` scripts, which run one
after the other. The smaller the number, the earlier the script runs; services with the same priority
are sorted by name. The default is `500`.

### .stop-signal
Sets the signal sent to stop `process.sh`, for example `SIGQUIT` for a graceful nginx or php-fpm
shutdown. The default is `SIGTERM`.

The stop signal is sent to the whole process group; remaining processes are killed with `SIGKILL` after
`--kill-all-timeout`. With `--kill-all-timeout 0`, shutdown uses `SIGKILL` immediately.

Signal names are case-insensitive and the `SIG` prefix is optional: `SIGQUIT`, `QUIT` and `quit` are the
same. An unknown name produces a warning and falls back to `SIGTERM`.

`container generate bootstrap --stop-signal SIGQUIT` and
`container generate services --stop-signal SIGQUIT` generate this file.

### .optional
Marks an optional service: by default, `container services download` and `link` skip it, and
`container services install` only installs it once downloaded. A derived image enables it with
`container services require`:

```Dockerfile
FROM my-base-image

# Require and download optional services
RUN container services require service-name \
    && container services download

# Install and link services to the entrypoint
RUN container services install \
    && container services link
```

### .tags
A service directory may contain a `.tags` directory: each file in it defines a tag named after the file.
Services and processes commands accept `tag:name` to select every service with this tag:

```bash
container services status tag:web
container processes stop tag:web
```

### Markers Written by the Container Command

- `.downloaded` records a completed `download.sh`
- `.installed` records a completed `install.sh`
- `.entrypoint` selects the service for the entrypoint: `container services link` creates it and
  `container services unlink` removes it (it is not a symbolic link)

Prepare downloads and installations during the image build. At startup, the entrypoint still runs the
missing downloads and installations of the selected services, before loading environment files. These
operations need the Unix permissions of their scripts and marker files.

## Environment Files

Files whose names start with `.env` in `/container/environment` and its sub-directories are loaded
before the lifecycle pre-commands and the service scripts. Their variables are set in the container
environment.

- Files are read in alphabetical order of their paths; a variable defined again in a later file
  overrides the previous value.
- **Variables already present in the container environment, including those provided with
  `docker run -e`, always win.** They are kept exactly as received, including empty values, quotes,
  newlines and `$` characters.
- `--skip-env-files` disables loading.

### Variable Interpolation

`$VAR` and `${VAR}` references in environment files are expanded by the
[gotenv](https://github.com/subosito/gotenv) parser, and can use variables already present in the
container environment. To keep a reference literal, use single quotes or escape the dollar sign:

```dotenv
DOMAIN=example.org
URL="https://${DOMAIN}"           # https://example.org
LITERAL='${DOMAIN}'              # ${DOMAIN}
TEMPLATE="https://\${DOMAIN}"     # https://${DOMAIN}
```

Values already present in the environment are not expanded again. To render a template explicitly,
use [`container envsubst`](container-command/environment/envsubst.md).

Your shell or Docker Compose may interpolate a value before passing it to the container: the rules
above apply to the value the container receives.
