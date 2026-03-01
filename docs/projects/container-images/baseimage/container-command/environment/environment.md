# environment

```
Show environment variables and environment files

Usage:
  container environment [command]

Aliases:
  environment, env

Available Commands:
  print       Print environment
  files       List environment files

Flags:
  -h, --help   help for environment

Use "container environment [command] --help" for more information about a command.

```

## print

```
Print environment

Usage:
  container environment print [variable]... [flags]

Aliases:
  print, p

Flags:
  -s, --shell                 print environment as shell variables
                              
  -e, --exclude stringArray   exclude environment variable
  -c, --clean                 exclude standard environment variables: HOSTNAME, PWD, HOME, LS_COLORS, TERM, SHLVL, PATH, _
                              
  -l, --log-level string      set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet                 show errors only
  -o, --log-format string     set log format, choices: console, json (default "console")
  -h, --help                  help for print
```

## files

```
List environment files

Usage:
  container environment files [flags]

Aliases:
  files, f

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for files
```
