# 🎯 Advanced Runtime Guide

## PID 1 and Container Init

`container` is the init and supervisor of the container: run it directly as the entrypoint, so it is
PID 1. Do not place another init in front of it: `docker run --init`, Compose `init: true`, tini or
docker-init are not supported.

Orphan reaping, the final `--kill-all` cleanup and the cleanup of runtime state at startup only run when
`container` is PID 1. Its subcommands work with any PID, including in `docker exec`.

## Lifecycle Steps

The entrypoint runs three steps, then exits:

| Step | Runs |
| --- | --- |
| `startup` | `--pre-startup-cmd` commands, then the `startup.sh` scripts, one after the other in priority order |
| `process` | `--pre-process-cmd` commands, then the `process.sh` scripts and/or the custom command, concurrently |
| `finish` | `--pre-finish-cmd` commands, then the `finish.sh` scripts, one after the other in priority order |
| exit | `--pre-exit-cmd` commands, then `--kill-all` cleanup |

- Environment files are loaded before the first step, unless `--skip-env-files` is used.
- `--skip-startup`, `--skip-process` and `--skip-finish` skip a step; `--step startup|process|finish`
  runs a single one.
- A failing startup script stops the container: the process and finish steps do not run.
- Pre-exit commands always run. A shutdown signal interrupts them, and the exit code of a failed step is
  kept.

## Selecting Services

The entrypoint runs the services linked with `container services link`. Narrow the selection at runtime:

- `--exec name`: run only these services (repeatable)
- `--skip name`: skip these services (repeatable)
- `--skip-all`: run no service

Startup and finish scripts only run for the selected services.

The same image can then run as one multi-process container, or as several single-purpose containers:

```bash
docker run --rm my-image                 # all linked services
docker run --rm my-image --exec nginx    # only nginx
docker run --rm my-image --exec php-fpm  # only php-fpm
```

This gives one build artifact for different deployment topologies, and lets you debug one service
without starting the others. See the [Nginx + PHP-FPM tutorial](tutorials/nginx-php-fpm.md) for a
Compose example.

## Passing Arguments

Arguments after the image name (after `--` if they start with a dash) are handled as follows:

- if at least one `process.sh` runs, they are appended to **every** running `process.sh`
- otherwise, they are run as a command instead of the services
- with no service and no command, Bash starts

```bash
docker run --rm my-image -- --config /etc/my-app.yml
# runs: /container/services/service-1/process.sh --config /etc/my-app.yml
```

In a multi-process image, use this only when the same arguments make sense for all running services.

## Running Extra Commands

### Pre-step commands

`--pre-startup-cmd`, `--pre-process-cmd`, `--pre-finish-cmd` and `--pre-exit-cmd` can be repeated.
Commands are split like a shell would, but not run through a shell: wrap pipes, redirects or
expansions in `sh -c`.

```bash
docker run --rm my-image \
  --pre-startup-cmd 'mkdir -p /run/container/cache' \
  --pre-process-cmd 'sh -c "echo ready > /tmp/health"'
```

### Bash or a command alongside services

`--bash` starts an interactive Bash session alongside the services:

```bash
docker run --rm -it my-image --bash
```

When Bash exits (including with Ctrl+D), or when the custom command exits whatever its exit status, the
process step ends and the services are stopped. Finish scripts then run, unless skipped.

With `--skip-process --bash` or `--step ... --bash`, Bash replaces the `process.sh` scripts only:
startup and finish scripts still run, and a failing startup script exits before Bash starts. See
[Container Debugging](production-and-integration.md#container-debugging) for shell recipes.

## Restarting Processes

| Running `process.sh` scripts | Default |
| --- | --- |
| one | not restarted: the container exits with the process |
| several | restarted |

Override this with `--restart=true` or `--restart=false`.

When restart is enabled, a process is restarted after any exit, clean or failed, with an exponential
backoff: from 100 milliseconds up to 5 seconds, reset after a run of at least 30 seconds.
`--no-restart-delay` restarts immediately; it does not enable restarting on its own, and a process that
keeps failing then uses a lot of CPU.

Before a process is restarted:

- the reason it ended is logged: a warning with its exit status or signal, or an info line after a clean exit
- the children left in its process group are stopped with its stop signal, then killed with `SIGKILL`
  after `--kill-all-timeout`, so old workers cannot keep the ports or files used by the new process

A service stopped with [`container processes stop`](container-command/runtime/processes.md) stays
stopped until `container processes start`, even with `--restart=false`, as long as its supervisor runs.

## Signals

- `SIGINT`, `SIGTERM` and `SIGQUIT` stop the container: during the process step, services are stopped
  and finish scripts still run.
- Stopping a service sends its stop signal (`SIGTERM`, or its [`.stop-signal`](how-it-works.md#stop-signal))
  to its whole process group, then `SIGKILL` after `--kill-all-timeout`. With `--kill-all-timeout 0`,
  `SIGKILL` is sent immediately.
- Other signals (`SIGHUP`, `SIGUSR1`, `SIGUSR2`, `SIGWINCH`...) are forwarded to the main process of each
  service and of the custom command, which handles its children: `docker kill -s HUP` reloads nginx,
  php-fpm or keepalived.
- `SIGHUP` is not forwarded during the startup and finish steps.
- `container watch` ignores `SIGHUP`, so a forwarded reload signal does not stop a watching service.

`docker stop` kills the container after 10 seconds by default, while `--kill-all-timeout` is `15s`. For
a graceful shutdown that may take longer, raise `docker stop --time`, Compose `stop_grace_period` or
Kubernetes `terminationGracePeriodSeconds`.
