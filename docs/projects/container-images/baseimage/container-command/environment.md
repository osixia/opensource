# Environment Commands

## environment-files

```
List environment file(s)

Usage:
  container environment-files [flags]

Aliases:
  environment-files, env

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for environment-files
```

## envsubst

```
Envsubst

Usage:
  container envsubst input [output=input] [flags]

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for envsubst
```

## envsubst-templates

```
Envsubst templates

Usage:
  container envsubst-templates templates_dir [output_dir=templates_dir] [templates_files_suffix=.template] [flags]

Flags:
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for envsubst-templates
```
