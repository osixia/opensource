# processes

```
Manage processes

Usage:
  container processes [command]

Aliases:
  processes, prcs

Available Commands:
  start       Start processes
  stop        Stop processes
  status      Print processes status
  check       Check processes are up

Flags:
  -h, --help   help for processes

Use "container processes [command] --help" for more information about a command.
```

## Runtime Control

```bash
docker exec app container processes stop nginx
docker exec app container processes start nginx
docker exec app container processes status
```

Any container user can run these commands. `stop` and `start` wait up to 30 seconds for the new state
(`--timeout`, `0` for no limit).

## start

```
Start processes
With no argument: start all processes

Usage:
  container processes start [process|tag:name]... [flags]

Flags:
      --timeout int         timeout in seconds to wait for process to be up (0 = no timeout)
                             (default 30)
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for start
```

## stop

```
Stop processes
With no argument: stop all processes

Usage:
  container processes stop [process|tag:name]... [flags]

Flags:
      --timeout int         timeout in seconds to wait for process to be down (0 = no timeout)
                             (default 30)
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for stop
```

## status

```
Print processes status
With no argument: print all processes status

Usage:
  container processes status [process|tag:name]... [flags]

Flags:
      --json                print status as JSON

  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for status
```

`--json` prints a JSON array with the `name`, `up`, `starting`, `wantedDown` and `tags` of each
process, for scripts and tools:

```bash
container processes status --json --quiet
```

## check

```
Check processes are up, for container health checks
Exit with code 1 if a process is down, intentionally stopped processes excepted
With no argument: check processes of services linked to the entrypoint

Usage:
  container processes check [process|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for check
```

See [Health Checks](../../production-and-integration.md#health-checks) for Docker, Compose and Kubernetes examples.
