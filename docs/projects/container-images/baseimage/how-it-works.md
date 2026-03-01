# 🧩 How It Works

Quick overview of project files:

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

## Dockerfile

A typical `Dockerfile` extending `osixia/baseimage` usually looks like this:

```
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

Below is a line-by-line explanation

```
FROM osixia/baseimage
```
Create a new build stage from osixia base image.

```
ARG IMAGE="osixia/multi-process-example"
ENV CONTAINER_IMAGE=${IMAGE}
```
Define a build-time variable `IMAGE` and assign its value to the `CONTAINER_IMAGE` environment variable.
This allows the `container` command to determine the image name, which can be useful for debugging purposes.

To leverage this feature, build the image with the `--build-arg` option: `docker build -t example/my-image:develop --build-arg IMAGE=example/my-image:develop .`

```
# Download services required packages or resources
# RUN container packages install --update --clean \
#        [...]
#     && curl -o resources.tar.gz https://[...].tar.gz \
#     && tar -xzf resources.tar.gz \
#     && rm resources.tar.gz
```
Uncomment and adapt this stage to download the image service resources required at build time.
This can include anything that takes time to download and can be cached to speed up image rebuilds.

The `container` command provided by `osixia/baseimage` offers a simple, distribution-agnostic command-line interface to install distribution packages.
For example, to install cowsay: `container packages install --update --clean cowsay`

This command updates the package index, installs cowsay, and cleans the index cache afterward.
It works transparently across Debian, Ubuntu, and Alpine.

```
COPY services /container/services
```
Copy the `services` directory into the image at `/container/services`.

If changes occur in this directory — for example in the `install.sh` script — only this stage will be re-executed.
The resources downloaded in the previous stage will remain cached.

This makes it easier to iterate quickly on service script development.

```
RUN container services install \
    && container services link
```
This will execute all non-optional service `install.sh` scripts and link those services to the entrypoint.

As a result, these services will run by default when the container starts.

```
COPY environment /container/environment
```
Finally, copy the `environment` directory, which defines the default environment variables, into the image at `/container/environment`.

## Service Files

This section describes all files used by the container command within a service directory. Additional files may also be included without impacting the service behavior.

The files outlined below are not mandatory.

### install.sh
This script should exclusively contain instructions for the initial setup of the service.

For improved image construction, all package installations or file downloads should occur within the `Dockerfile`.

By separating time-intensive download operations from the setup, the docker build cache is utilized effectively.
Changes to the `install.sh` file will not necessitate re-downloading dependencies,
as the `Dockerfile` builder will only execute the service installation script.

Note: The `install.sh` script executes during the docker build, thus runtime environment variables cannot be used for setup customization.
Such customizations are handled in the `startup.sh` file.

### startup.sh
This script prepares `process.sh` for execution and tailors the service setup to runtime environment variables.

### process.sh
This script specifies the command to be executed.

For multi-process images, all `process.sh` scripts are launched simultaneously.
The order defined in the service `.priority` file is irrelevant.

### finish.sh
This script is executed once `process.sh` has concluded.

### .priority
The `.priority` file establishes the sequence in which services `install.sh`, `startup.sh` or `finish.sh` scripts are invoked.
The smaller the number, the greater the priority. The default is `500`.

### Optional Service Files

#### .optional
This file indicates that the service is optional and is not installed by `container services install` by default.
It can be incorporated later via the `container services require service-name` command.

#### download.sh
This script is called during container build to download optional service resources.

Example of a `Dockerfile` using an optional service from a base image:

```
FROM [...]

# Require and download optional services
RUN container services require service-name \
    && container services download

# Install and link services to the entrypoint
RUN container services install \
    && container services link
```

### Service Tags

A service directory may contain a `.tags` sub-directory.
Each file in this directory defines a tag, where the filename is used as the tag name.

Services and their related processes can then be filtered by tag through the services and processes subcommands.

To learn more about tag usage, run `container services status --help`

## Environment Files

`.env` files in the `environment` directory and any sub-directories are loaded before executing services lifecycle scripts (`startup.sh`, `process.sh`, and `finish.sh`) or entrypoint lifecycle pre-commands.
The variables they contain are defined as environment variables in the container.

Files are loaded in alphabetical order, and variables are overwritten if they were already defined in a previous file.

**Container environment variables set at runtime override values defined in `.env` files.**
