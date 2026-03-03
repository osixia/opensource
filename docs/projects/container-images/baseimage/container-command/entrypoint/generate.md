# generate

```
Generate sample templates

Usage:
  container entrypoint generate [command]

Aliases:
  generate, gen

Available Commands:
  bootstrap   Generate bootstrap
  dockerfile  Generate Dockerfile
  environment Generate environment
  services    Generate services

Flags:
  -h, --help   help for generate

Use "container entrypoint generate [command] --help" for more information about a command.
```

## bootstrap
```
Generate bootstrap

Usage:
  container entrypoint generate bootstrap [service name]... [flags]

Aliases:
  bootstrap, b

Flags:
  -m, --multi-process       generate multi-process example
                            
  -i, --image string        image name (default "osixia/baseimage-example:latest")
  -f, --from-image string   overwrite from image
                            
  -p, --priority int        services priority (default 500)
  -t, --tags strings        services tags
      --optional            optional service
                            
      --print               print generated files content
                            
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for bootstrap
```

## dockerfile

```
Generate Dockerfile

Usage:
  container entrypoint generate dockerfile [flags]

Aliases:
  dockerfile, d

Flags:
  -i, --image string        image name (default "osixia/baseimage-example:latest")
  -f, --from-image string   overwrite from image
                            
      --print               print generated files content
                            
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for dockerfile
```

## environment

```
Generate environment

Usage:
  container entrypoint generate environment [flags]

Aliases:
  environment, e

Flags:
      --print               print generated files content
                            
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for environment
```

## services

```
Generate services

Usage:
  container entrypoint generate services [name]... [flags]

Aliases:
  services, s

Flags:
  -p, --priority int        services priority (default 500)
  -t, --tags strings        services tags
      --optional            optional service
                            
      --print               print generated files content
                            
  -l, --log-level string    set log level, choices: none, error, warning, info, debug, trace (default "info")
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for services
```
