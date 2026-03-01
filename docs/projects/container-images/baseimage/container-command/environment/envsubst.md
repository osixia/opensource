# envsubst

```
Render templates using environment variables

Usage:
  container envsubst string [string]... [flags]
  container envsubst [command]

Available Commands:
  file        Render a single file using environment variables
  templates   Render all .template files recursively in a directory

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for envsubst

Use "container envsubst [command] --help" for more information about a command.

```

## file

```
Render a single file using environment variables

Usage:
  container envsubst file input [output=input] [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for file

```

## templates

```
Render all .template files recursively in a directory

Usage:
  container envsubst templates templates_dir [output_dir=templates_dir] [templates_files_suffix=.template] [flags]

Flags:
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for templates
```
