# packages

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

## install

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

## update

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

## clean

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

## remove

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
