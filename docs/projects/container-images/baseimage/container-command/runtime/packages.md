# packages

```
Manage packages

Usage:
  container packages [command]

Aliases:
  packages, pkgs

Available Commands:
  install     Install packages
  update      Update list of available packages
  clean       Clean packages cache
  remove      Remove packages

Flags:
  -h, --help   help for packages

Use "container packages [command] --help" for more information about a command.
```

## install

```
Install packages

Usage:
  container packages install package [package]... [flags]

Aliases:
  install, inst

Flags:
  -u, --update              update list of available packages before install
  -c, --clean               clean packages cache after install
                            
  -d, --debug-tools         include common debugging tools
                            
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for install
```

### Debug tools

When `--debug-tools` is used, the following packages are installed by default depending on the base distribution:

- **Alpine**: `curl less procps psmisc strace vim`
- **Debian**: `curl less procps psmisc strace vim-tiny`

Use the `CONTAINER_DEBUG_PACKAGES` environment variable to add extra packages to this default list (space-separated):

```dockerfile
ENV CONTAINER_DEBUG_PACKAGES="tcpdump netcat-openbsd"
```

Then at build time:

```bash
container packages install --debug-tools
```

This installs both the default debug packages and the ones listed in `CONTAINER_DEBUG_PACKAGES`.

## update

```
Update list of available packages

Usage:
  container packages update [flags]

Aliases:
  update, upd

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for update
```

## clean

```
Clean packages cache

Usage:
  container packages clean [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for clean
```

## remove

```
Remove packages

Usage:
  container packages remove package [package]... [flags]

Aliases:
  remove, rm

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for remove
```
