# run

```
Run a command instead of services

Usage:
  container run [args]... [flags]

Flags:
  -e, --skip-env-files              skip loading environment variables from environment files

  -t, --kill-all-timeout duration   force kill command after timeout (SIGKILL after delay)
                                     (default 15s)
  -l, --log-level string            set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet                       show errors only
  -o, --log-format string           set log format, choices: console, json (default "console")
  -h, --help                        help for run
```
