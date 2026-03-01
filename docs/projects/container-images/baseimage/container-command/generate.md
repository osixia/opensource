# generate

```
Generate sample templates

Usage:
  container generate [command]

Aliases:
  generate, gen

Available Commands:
  bootstrap   Generate bootstrap
  dockerfile  Generate Dockerfile
  environment Generate environment
  services    Generate services

Flags:
  -h, --help   help for generate

Use "container generate [command] --help" for more information about a command.
```

## bootstrap
```
Generate bootstrap

Usage:
  container generate bootstrap [service name]... [flags]

Aliases:
  bootstrap, boot

Flags:
  -m, --multi-process       generate multi-process example
                            
  -i, --image string        image name (default "osixia/baseimage-example:latest")
  -f, --from-image string   overwrite from image
                            
  -p, --priority int        services priority (default 500)
  -t, --tags strings        services tags
      --optional            optional service
                            
      --print               print generated files content
                            
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for bootstrap
```

## dockerfile

```
Generate Dockerfile

Usage:
  container generate dockerfile [flags]

Aliases:
  dockerfile, df

Flags:
  -i, --image string        image name (default "osixia/baseimage-example:latest")
  -f, --from-image string   overwrite from image
                            
      --print               print generated files content
                            
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for dockerfile
```

## environment

```
Generate environment

Usage:
  container generate environment [flags]

Aliases:
  environment, env

Flags:
      --print               print generated files content
                            
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for environment
```

## services

```
Generate services

Usage:
  container generate services [name]... [flags]

Aliases:
  services, svcs

Flags:
  -p, --priority int        services priority (default 500)
  -t, --tags strings        services tags
      --optional            optional service
                            
      --print               print generated files content
                            
  -l, --log-level string    set log level, choices: error, warning, info, debug, trace (default "info")
      --quiet               show errors only
  -o, --log-format string   set log format, choices: console, json (default "console")
  -h, --help                help for services
```
