# ⚡ Quickstart

This image requires the `ip_vs` kernel module to be loaded on the host (`modprobe ip_vs`).  
It must be run with the following Docker options:

``` bash
docker run --cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=NET_BROADCAST --network=host osixia/keepalived
```

## Using your own Keepalived configuration

This image ships with a Keepalived configuration template that can be customized using environment variables for quick bootstrapping.  

You can also provide your own `keepalived.conf` by mounting it at the path defined by `KEEPALIVED_CONF` (default: `/etc/keepalived/keepalived.conf`):

``` bash
docker run --volume /data/my-keepalived.conf:/etc/keepalived/keepalived.conf osixia/keepalived
```

## Passing command-line arguments to Keepalived

The `osixia/keepalived` container allows you to pass additional command-line arguments directly to the keepalived binary.

Arguments specified after `--` are forwarded to the keepalived process inside the container:

``` bash
docker run osixia/keepalived -- --dont-release-ipvs
```

## Debugging

To debug the container manually, you can start it with an interactive shell.

The `--debug` option from `osixia/baseimage` enables debug logging, installs debugging tools, and launches an interactive shell.

If Keepalived keeps crashing, you can add `--skip-process` to start the container without launching service processes.

``` bash
docker run -it osixia/keepalived --debug
docker run -it osixia/keepalived --skip-process --debug 
```

You can also enable Keepalived debugging options:
``` bash
docker run osixia/keepalived -- --log-detail --dump-conf
```

To see all available command-line options:
``` bash
docker run --rm osixia/keepalived --help # osixia/baseimage options
docker run --rm osixia/keepalived -x keepalived -- --help # keepalived command-line options
```
 
