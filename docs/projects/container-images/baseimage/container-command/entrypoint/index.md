# Entrypoint command

```
 / _ \ ___(_)_  _(_) __ _   / / __ )  __ _ ___  ___(_)_ __ ___   __ _  __ _  ___ 
| | | / __| \ \/ / |/ _` | / /|  _ \ / _` / __|/ _ \ | '_ ` _ \ / _` |/ _` |/ _ 
| |_| \__ \ |>  <| | (_| |/ / | |_) | (_| \__ \  __/ | | | | | | (_| | (_| |  __/
 \___/|___/_/_/\_\_|\__,_/_/  |____/ \__,_|___/\___|_|_| |_| |_|\__,_|\__, |\___|
                                                                      |___/      
Container image built with osixia/baseimage (develop) 🐳✨🌴
https://github.com/osixia/container-baseimage

Usage:
  container entrypoint [flags]
  container entrypoint [command]

Aliases:
  entrypoint, ep

Available Commands:
  generate    Generate sample templates
  thanks      List contributors

Flags:
  -e, --skip-env-files                      skip getting environment variables values from environment file(s)
                                            
  -s, --skip-startup                        skip running pre-startup-cmd and service(s) startup.sh script(s)
  -p, --skip-process                        skip running pre-process-cmd and service(s) process.sh script(s)
  -f, --skip-finish                         skip running pre-finish-cmd and service(s) finish.sh script(s)
  -c, --run-only-lifecycle-step string      run only one lifecycle step pre-command and script(s) file(s), choices: startup, process, finish
                                            
  -1, --pre-startup-cmd stringArray         run command passed as argument before service(s) startup.sh script(s)
  -3, --pre-process-cmd stringArray         run command passed as argument before service(s) process.sh script(s)
  -5, --pre-finish-cmd stringArray          run command passed as argument before service(s) finish.sh script(s)
  -7, --pre-exit-cmd stringArray            run command passed as argument before container exits
                                            
  -x, --exec stringArray                    execute only listed service(s) (default run service(s) linked to the entrypoint)
                                            
  -b, --bash                                run Bash along with other service(s) or command
                                            
  -k, --kill-all-on-exit                    kill all processes on the system upon exiting (send sigterm to all processes) (default true)
  -t, --kill-all-on-exit-timeout duration   kill all processes timeout (send sigkill to all processes after sigterm timeout has been reached) (default 15s)
  -r, --restart                             automatically restart failed services process.sh scripts (single-process: default false, multi-process: default true)
  -a, --keep-alive                          keep alive container after all processes have exited
                                            
  -w, --unsecure-fast-write                 disable fsync and friends with eatmydata LD_PRELOAD library
                                            
  -d, --debug                               set log level to debug and install debug packages
  -i, --install-packages strings            install packages
                                            
  -v, --version                             print container image version
                                            
  -l, --log-level string                    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string                   set log format, choices: console, json (default "console")
  -h, --help                                help for entrypoint

Use "container entrypoint [command] --help" for more information about a command.
```
