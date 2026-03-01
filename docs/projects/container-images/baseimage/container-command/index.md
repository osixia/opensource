# 🤖 Container Command

```
 / _ \ ___(_)_  _(_) __ _   / / __ )  __ _ ___  ___(_)_ __ ___   __ _  __ _  ___ 
| | | / __| \ \/ / |/ _` | / /|  _ \ / _` / __|/ _ \ | '_ ` _ \ / _` |/ _` |/ _ 
| |_| \__ \ |>  <| | (_| |/ / | |_) | (_| \__ \  __/ | | | | | | (_| | (_| |  __/
 \___/|___/_/_/\_\_|\__,_/_/  |____/ \__,_|___/\___|_|_| |_| |_|\__,_|\__, |\___|
                                                                      |___/      
Built with osixia/baseimage (develop) 🐳✨🌴
https://github.com/osixia/container-baseimage

Container runtime with lifecycle management and system utilities

Runs linked services by default, falls back to Bash if none are defined.
You can also pass a custom command after --

Usage:
  container [flags]
  container [command]

Main Commands:
  install     Install the container binary and prepare the runtime
  generate    Generate sample templates
  run         Run a command instead of services

Runtime Commands:
  packages    Manage packages
  processes   Manage processes
  services    Manage services

Environment Commands:
  environment Show environment variables and environment files
  envsubst    Render templates using environment variables

Filesystem Commands:
  groups      Manage groups
  users       Manage users
  watch       Watch files

Logging Commands:
  log         Logging utilities

Additional Commands:
  help        Help about any command
  completion  Generate the autocompletion script for the specified shell

Flags:
  -e, --skip-env-files                skip loading environment variables from environment files

  -s, --skip-startup                  skip running pre-startup-cmd and startup scripts
  -p, --skip-process                  skip running pre-process-cmd and process scripts
  -f, --skip-finish                   skip running pre-finish-cmd and finish scripts
      --step string                   run a single lifecycle step, choices: startup, process, finish

      --pre-startup-cmd stringArray   run command before startup scripts
      --pre-process-cmd stringArray   run command before process scripts
      --pre-finish-cmd stringArray    run command before finish scripts
      --pre-exit-cmd stringArray      run command before container exits

  -x, --exec stringArray              execute only specified services

  -X, --skip stringArray              skip listed services
      --skip-all                      skip all services

  -b, --bash                          run Bash alongside other services or commands

  -k, --kill-all                      kill all remaining container processes on exit (SIGTERM) (default true)
  -t, --kill-all-timeout duration     force kill remaining processes after timeout (SIGKILL after delay) (default 15s)
  -r, --restart                       restart failed processes (default single-process false, multi-process true)
  -a, --keep-alive                    keep container alive after all processes have exited

  -w, --fast-write                    faster writes by disabling fsync (useful for CI, risk of data loss)

  -v, --version                       print container image version

  -l, --log-level string              set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet                         show errors only
  -o, --log-format string             set log format, choices: console, json (default "console")
  -h, --help                          help for container

Use "container [command] --help" for more information about a command.
```
