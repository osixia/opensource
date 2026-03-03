# ⚡ Quick Start

## Create Your First Image in 2 Minutes

Generate image templates in the **osixia-baseimage-example** directory

```
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
```
tree -a osixia-baseimage-example
```

```
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
```
FROM osixia/baseimage

ARG IMAGE="osixia/baseimage-example:latest"
ENV CONTAINER_IMAGE=${IMAGE}

# Download service(s) required packages or resources
# RUN container packages install --update --clean \
#        [...]
#     && curl -o resources.tar.gz https://[...].tar.gz \
#     && tar -xzf resources.tar.gz \
#     && rm resources.tar.gz

COPY services /container/services

# Install and link service(s) to the entrypoint
RUN container services install \
    && container services link

COPY environment /container/environment
```

Build the image **example/my-image:develop** using files in the **osixia-baseimage-example** directory
```
docker build --tag example/my-image:develop --build-arg IMAGE example/my-image:develop  ./osixia-baseimage-example
```

Note: `--build-arg IMAGE example/my-image:develop` is used to set the image name inside the container

Run **example/my-image:develop** image
```
docker run example/my-image:develop
```

```
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

Detailed documentation regarding the structure, purpose, and usage of the files within the `environment` and `service` directories can be found in their respective `README.md` files.

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
Container image built with osixia/baseimage (develop) 🐳✨🌴
https://github.com/osixia/container-baseimage

Usage:
  container entrypoint [flags]
  container entrypoint [command]

Aliases:
  entrypoint, ep

Available Commands:
  generate    Generate sample templates
  thanks      List contributors

Flags:
  -e, --skip-env-files                      skip getting environment variables values from environment file(s)

  -s, --skip-startup                        skip running pre-startup-cmd and service(s) startup.sh script(s)
  -p, --skip-process                        skip running pre-process-cmd and service(s) process.sh script(s)
  -f, --skip-finish                         skip running pre-finish-cmd and service(s) finish.sh script(s)
  -c, --run-only-lifecycle-step string      run only one lifecycle step pre-command and script(s) file(s), choices: startup, process, finish

  -1, --pre-startup-cmd stringArray         run command passed as argument before service(s) startup.sh script(s)
  -3, --pre-process-cmd stringArray         run command passed as argument before service(s) process.sh script(s)
  -5, --pre-finish-cmd stringArray          run command passed as argument before service(s) finish.sh script(s)
  -7, --pre-exit-cmd stringArray            run command passed as argument before container exits

  -x, --exec stringArray                    execute only listed service(s) (default run service(s) linked to the entrypoint)

  -b, --bash                                run Bash along with other service(s) or command

  -k, --kill-all-on-exit                    kill all processes on the system upon exiting (send sigterm to all processes) (default true)
  -t, --kill-all-on-exit-timeout duration   kill all processes timeout (send sigkill to all processes after sigterm timeout has been reached) (default 15s)
  -r, --restart                             automatically restart failed services process.sh scripts (single-process: default false, multi-process: default true)
  -a, --keep-alive                          keep alive container after all processes have exited

  -w, --unsecure-fast-write                 disable fsync and friends with eatmydata LD_PRELOAD library

  -d, --debug                               set log level to debug and install debug packages
  -i, --install-packages strings            install packages

  -v, --version                             print container image version

  -l, --log-level string                    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string                   set log format, choices: console, json (default "console")
  -h, --help                                help for entrypoint

Use "container entrypoint [command] --help" for more information about a command.
```
