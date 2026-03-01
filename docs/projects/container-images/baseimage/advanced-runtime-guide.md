# 🎯 Advanced Runtime Guide

## Passing Arguments to `process.sh`

Any argument passed after the image name at runtime is captured by the baseimage and then handled in one of two ways:

- If at least one service `process.sh` is run, the same argument list is appended to every running `process.sh`.
- If no service `process.sh` is run, the argument list is treated as a standalone command to execute instead.
- If no service is run and no command is provided, Bash is started automatically.

Examples:

```bash
docker run --rm my-image -- --config /etc/my-app.yml
```

If `my-image` has one linked service, its `process.sh` is executed as:

```bash
/container/services/service-1/process.sh --config /etc/my-app.yml
```

In a multi-process image, the same arguments are passed to every running service process:

```bash
docker run --rm my-image -- --verbose
```

This means:

```bash
/container/services/service-a/process.sh --verbose
/container/services/service-b/process.sh --verbose
```

Use this only when the same runtime arguments make sense for all running services.

## Controlling Container Lifecycle Steps

The entrypoint lifecycle has three steps:

1. `startup`
2. `process`
3. `finish`

Behavior by step:

- `startup`: runs `--pre-startup-cmd` commands, then every running service `startup.sh` in service priority order.
- `process`: runs `--pre-process-cmd` commands, then the main command and/or every running service `process.sh`.
- `finish`: runs `--pre-finish-cmd` commands, then every running service `finish.sh` in service priority order.
- `exit`: always runs `--pre-exit-cmd` before the container exits.

Useful flags:

- `--skip-startup`, `--skip-process`, `--skip-finish`
- `--step startup|process|finish`
- `--exec service-name`
- `--skip service-name`
- `--skip-all`

Important details:

- `.priority` affects `install.sh`, `startup.sh`, and `finish.sh`.
- `.priority` does not affect `process.sh`; all running service processes are started concurrently.
- `--step process` runs only the process step and skips startup and finish.
- If `--skip-process` is used together with `--bash`, process services are skipped but Bash still runs.
- Environment files are loaded before lifecycle pre-commands and service scripts, unless `--skip-env-files` is used.

## Running Extra Commands

There are two distinct ways to run extra commands.

### 1. Pre-step commands

Use:

- `--pre-startup-cmd`
- `--pre-process-cmd`
- `--pre-finish-cmd`
- `--pre-exit-cmd`

Each flag can be repeated. Commands are split with shell-like parsing, but they are not executed through a shell unless you explicitly call one yourself.

Example:

```bash
docker run --rm my-image \
  --pre-startup-cmd 'mkdir -p /run/container/cache' \
  --pre-process-cmd 'sh -c "echo ready > /tmp/health"'
```

If you need pipes, redirects, shell expansion, or compound shell syntax, wrap the command with `sh -c` or `bash -c`.

### 2. Run a command alongside services

Use:

```bash
docker run --rm my-image --bash
```

This starts Bash in parallel with running service processes.

If no service process is run:

- arguments after the flags are run as a command
- `--bash` can still be used to run Bash alongside that command

This is mainly useful for debugging and container inspection.

## Running Multi-Process Images in Separate Containers

The same image can be used either as:

- one multi-process container
- several single-purpose containers

Use `container services link` during build to define the default linked services. At runtime, narrow the selection with `--exec` or remove services with `--skip`.

Examples:

```bash
docker run --rm my-image
```

Runs all linked services.

```bash
docker run --rm my-image --exec nginx
```

Runs only the `nginx` service lifecycle.

```bash
docker run --rm my-image --exec php-fpm
```

Runs only the `php-fpm` service lifecycle.

This pattern is useful when:

- you want one build artifact but different deployment topologies
- you want to split a multi-process development image into separate production containers
- you want to debug one service without starting the others

Keep in mind:

- startup and finish scripts still run for the services being run only
- process arguments are shared across all running services
- restart behavior changes with the number of running process scripts

By default:

- single running process: restart is disabled
- multiple running processes: restart is enabled

Override this with `--restart=true` or `--restart=false`.
