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

```
Print processes status
With no argument: print all processes status

Usage:
  container processes status [process|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for status
```
