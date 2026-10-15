# run

`container run` runs a command instead of the services, with the environment files loaded unless
`--skip-env-files` is used. It is meant for one-off commands, for example in a `Dockerfile` or with
`docker exec`; use the entrypoint itself as PID 1.

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
