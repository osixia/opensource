# ⚡ Quickstart

## Create Your First Image in 2 Minutes

Generate image templates in the **osixia-baseimage-example** directory

``` bash
mkdir osixia-baseimage-example
```

=== "Debian"

    ``` bash
    docker run --rm --user $UID --volume $(pwd)/osixia-baseimage-example:/run/container/generator \
    osixia/baseimage generate bootstrap
    ```

=== "Alpine"

    ``` bash
    docker run --rm --user $UID --volume $(pwd)/osixia-baseimage-example:/run/container/generator \
    osixia/baseimage:alpine generate bootstrap
    ```

=== "Ubuntu"

    ``` bash
    docker run --rm --user $UID --volume $(pwd)/osixia-baseimage-example:/run/container/generator \
    osixia/baseimage:ubuntu generate bootstrap
    ```

Note: add `--multi-process` to get a multi-process image sample.

List generated directories and files in **osixia-baseimage-example** directory
``` bash
tree -a osixia-baseimage-example
```

``` bash
osixia-baseimage-example
├── Dockerfile
├── environment
│   ├── .env
│   └── README.md
└── services
    └── service-1
        ├── finish.sh
        ├── install.sh
        ├── .priority
        ├── process.sh
        ├── README.md
        └── startup.sh
```

Take a quick look at the Dockerfile
``` Dockerfile
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

Build the image **example/my-image:develop** using files in the **osixia-baseimage-example** directory
``` bash
docker build --tag example/my-image:develop --build-arg IMAGE=example/my-image:develop ./osixia-baseimage-example
```

Note: `--build-arg IMAGE=example/my-image:develop` is used to set the image name inside the container.

Run **example/my-image:develop** image
``` bash
docker run example/my-image:develop
```

``` log
2026-02-26T11:30:59+01:00 INFO    Container image: example/my-image:develop
2026-02-26T11:30:59+01:00 INFO    Loading environment variables from /container/environment/.env ...
2026-02-26T11:30:59+01:00 INFO    Running /container/services/service-1/startup.sh ...
service-1: Doing some container start setup ...
service-1: EXAMPLE_ENV_VAR=Hello :) ...
2026-02-26T11:30:59+01:00 INFO    Running /container/services/service-1/process.sh ...
service-1: Just going to sleep for 10 seconds ...
2026-02-26T11:31:09+01:00 INFO    Running /container/services/service-1/finish.sh ...
service-1: process ended ...
2026-02-26T11:31:09+01:00 INFO    Exiting ...
```

That's it you have a single-process image based on osixia/baseimage.

Detailed documentation regarding the structure, purpose, and usage of the files within the `environment` and `services` directories can be found in their respective `README.md` files.

## Entrypoint Options

=== "Debian"

    ``` bash
    docker run --rm osixia/baseimage --help
    ```

=== "Alpine"

    ``` bash
    docker run --rm osixia/baseimage:alpine --help
    ```

=== "Ubuntu"

    ``` bash
    docker run --rm osixia/baseimage:ubuntu --help
    ```

```
 / _ \ ___(_)_  _(_) __ _   / / __ )  __ _ ___  ___(_)_ __ ___   __ _  __ _  ___ 
| | | / __| \ \/ / |/ _` | / /|  _ \ / _` / __|/ _ \ | '_ ` _ \ / _` |/ _` |/ _ 
| |_| \__ \ |>  <| | (_| |/ / | |_) | (_| \__ \  __/ | | | | | | (_| | (_| |  __/
 \___/|___/_/_/\_\_|\__,_/_/  |____/ \__,_|___/\___|_|_| |_| |_|\__,_|\__, |\___|
                                                                      |___/      
Built with osixia/baseimage (develop) 🐳✨🌴
https://github.com/osixia/container-baseimage

Container runtime with lifecycle management and system utilities

Runs linked services by default, falls back to Bash if none are defined.
You can also pass a custom command after --

Usage:
  container [flags]
  container [command]

Main Commands:
  install     Install the container binary and prepare the runtime
  generate    Generate sample templates
  run         Run a command instead of services

Runtime Commands:
  packages    Manage packages
  processes   Manage processes
  services    Manage services

Environment Commands:
  environment Show environment variables and environment files
  envsubst    Render templates using environment variables

Filesystem Commands:
  groups      Manage groups
  users       Manage users
  watch       Watch files

Logging Commands:
  log         Logging utilities

Additional Commands:
  help        Help about any command
  completion  Generate the autocompletion script for the specified shell

Flags:
  -e, --skip-env-files                skip loading environment variables from environment files

  -s, --skip-startup                  skip running pre-startup-cmd and startup scripts
  -p, --skip-process                  skip running pre-process-cmd and process scripts
  -f, --skip-finish                   skip running pre-finish-cmd and finish scripts
      --step string                   run a single lifecycle step, choices: startup, process, finish

      --pre-startup-cmd stringArray   run command before startup scripts
      --pre-process-cmd stringArray   run command before process scripts
      --pre-finish-cmd stringArray    run command before finish scripts
      --pre-exit-cmd stringArray      run command before container exits

  -x, --exec stringArray              execute only specified services

  -X, --skip stringArray              skip listed services
      --skip-all                      skip all services

  -b, --bash                          run Bash alongside other services or commands

  -k, --kill-all                      kill all remaining container processes on exit (SIGTERM) (default true)
  -t, --kill-all-timeout duration     force kill remaining processes after timeout (SIGKILL after delay) (default 15s)
  -r, --restart                       restart failed processes (default single-process false, multi-process true)
  -a, --keep-alive                    keep container alive after all processes have exited

  -w, --fast-write                    faster writes by disabling fsync (useful for CI, risk of data loss)

  -v, --version                       print container image version

  -l, --log-level string              set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet                         show errors only
  -o, --log-format string             set log format, choices: console, json (default "console")
  -h, --help                          help for container

Use "container [command] --help" for more information about a command.
```
