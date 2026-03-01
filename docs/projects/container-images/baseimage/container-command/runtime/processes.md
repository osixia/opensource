# processes

```
Processes subcommands

Usage:
  container processes [command]

Aliases:
  processes, prcs

Available Commands:
  start       Start process(es)
  stop        Stop process(es)
  status      Print process(es) status

Flags:
  -h, --help   help for processes

Use "container processes [command] --help" for more information about a command.
```

## start

```
Start process(es)

Usage:
  container processes start process|tag:name [process|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for star
```

## stop

```
Stop process(es)

Usage:
  container processes stop [service|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for stop
```

## status

```
Print process(es) status

Usage:
  container processes status [process|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for status
```
