# Runtime commands

## packages

```
Packages subcommands

Usage:
  container packages [command]

Aliases:
  packages, pkgs

Available Commands:
  install     Install package(s)
  update      Update list of available packages
  clean       Clean packages cache
  remove      Remove package(s)

Flags:
  -h, --help   help for packages

Use "container packages [command] --help" for more information about a command.
```

### install

```
Install package(s)

Usage:
  container packages install package [package]... [flags]

Aliases:
  install, i

Flags:
  -u, --update              update list of available packages before install
  -c, --clean               clean packages cache after install

  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for install
```

### update

```
Update list of available packages

Usage:
  container packages update [flags]

Aliases:
  update, u

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for update
```

### clean

```
Clean packages cache

Usage:
  container packages clean [flags]

Aliases:
  clean, c

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for clean
```

### remove

```
Remove package(s)

Usage:
  container packages remove package [package]... [flags]

Aliases:
  remove, r

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for remove
```

## processes

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

### start

```
Start process(es)

Usage:
  container processes start process|tag:name [process|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for star
```

### stop

```
Stop process(es)

Usage:
  container processes stop [service|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for stop
```

### status

```
Print process(es) status

Usage:
  container processes status [process|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for status
```

## services

```
Services subcommands

Usage:
  container services [command]

Aliases:
  services, svcs

Available Commands:
  require     Require optional service
  download    Download service(s)
  install     Install service(s)
  link        Link service(s) to entrypoint
  unlink      Unlink entrypoint's service(s)
  status      Print service(s) status

Flags:
  -h, --help   help for services

Use "container services [command] --help" for more information about a command.
```

### require

```
Require optional service

Usage:
  container services require service|tag:name [service|tag:name]... [flags]

Aliases:
  require, r

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for require
```

### download

```
Download service(s)
With no argument: download all not optional services

Usage:
  container services download [service|tag:name]... [flags]

Aliases:
  download, d

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for download
```

### install

```
Install service(s)
With no argument: install all not optional services and only downloaded optional services

Usage:
  container services install [service|tag:name]... [flags]

Aliases:
  install, i

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for install
```

### link

```
Link service(s) to entrypoint
With no argument: link all installed and not optional services

Usage:
  container services link [service|tag:name]... [flags]

Aliases:
  link, l

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for link
```

### unlink

```
Unlink entrypoint's service(s)
With no argument: unlink all linked services

Usage:
  container services unlink [service|tag:name]... [flags]

Aliases:
  unlink, u

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for unlink
```

### status

```
Print service(s) status

Usage:
  container services status [service|tag:name]... [flags]

Aliases:
  status, s

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for status
```
