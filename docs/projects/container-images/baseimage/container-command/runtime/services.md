
# services

```
Manage services

Usage:
  container services [command]

Aliases:
  services, svcs

Available Commands:
  require     Require optional service
  download    Download services
  install     Install services
  link        Link services to entrypoint
  unlink      Unlink entrypoint's services
  status      Print services status

Flags:
  -h, --help   help for services

Use "container services [command] --help" for more information about a command.
```

## require

```
Require optional service

Usage:
  container services require service|tag:name [service|tag:name]... [flags]

Aliases:
  require, req

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for require
```

## download

```
Download services
With no argument: download all not optional services

Usage:
  container services download [service|tag:name]... [flags]

Aliases:
  download, dl

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for download
```

## install

```
Install services
With no argument: install all not optional services and only downloaded optional services

Usage:
  container services install [service|tag:name]... [flags]

Aliases:
  install, inst

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for install
```

## link

```
Link services to entrypoint
With no argument: link all not optional services

Usage:
  container services link [service|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for link
```

## unlink

```
Unlink entrypoint's services
With no argument: unlink all linked services

Usage:
  container services unlink [service|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for unlink
```

## status

```
Print services status

Usage:
  container services status [service|tag:name]... [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for status
```
