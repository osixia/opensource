# Filesystem Commands

## groups

```
Manage groups

Usage:
  container groups [command]

Available Commands:
  add         Add group

Flags:
  -h, --help   help for groups

Use "container groups [command] --help" for more information about a command.
```

### add

```
Add group

Usage:
  container groups add id name [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for add
```

## users

```
Manage users

Usage:
  container users [command]

Available Commands:
  add         Add user

Flags:
  -h, --help   help for users

Use "container users [command] --help" for more information about a command.
```

### add

```
Add user

Usage:
  container users add id name --group-id group_id --group-name group_name [flags]

Flags:
  -i, --group-id string     group id
  -n, --group-name string   group name

  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for add
```

## watch

`container watch` runs scripts when files change, for example to reload a service when its
configuration is updated. It ignores `SIGHUP`, so a reload signal forwarded by the entrypoint does not
stop it.

Run a command when a configuration file changes:

```bash
container watch /etc/my-app/config.yaml --exec /usr/local/bin/reload-my-app
```

The file must exist when the watcher starts. It detects writes, file replacements and Kubernetes projected-volume symlink updates. Mount a ConfigMap or Secret as a directory to receive those updates.

Callbacks run one after another. Use `--once` to exit after the first change, or repeat `--exec` to run several commands. For shell operators such as `&&`, put the logic in a script and pass its path to `--exec`.

```
Watch files

Usage:
  container watch file [file]... [flags]

Flags:
  -x, --exec stringArray    execute the given script on file change
  -1, --once                exit after the first detected change
                            
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for watch
```
