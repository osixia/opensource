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

Use `container processes status --json` for structured output.

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

## check

Use this in a container health check:

```bash
container processes check openldap
```

The command exits with code `1` if a selected process is down. Intentionally stopped processes are excluded. Without names, it checks processes of services linked to the entrypoint; specify names when using `--exec` to run a different selection.

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
