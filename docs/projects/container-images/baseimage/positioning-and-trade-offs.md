# ⚖️ Positioning and Trade-offs

`osixia/baseimage` is an opinionated base image for teams who want more runtime structure than a minimal
init, without adopting a full supervision framework. It saves you from rebuilding the same entrypoint
logic, service conventions and debugging workflow in every image.

The trade-off is deliberate: it is heavier and more opinionated than a minimal PID 1, but gives a faster
path to reliable application containers.

## When It Is a Good Fit

- one or a few tightly related processes in the same image, such as Nginx + PHP-FPM
- repeatable startup and shutdown behavior, with lifecycle hooks
- shared conventions and runtime tooling across many images
- an image built as multi-process for development, then split into single-service containers in
  production
- non-root, read-only root filesystem and Kubernetes deployments

## When It Is Probably Not the Right Fit

- the smallest possible image, such as distroless or scratch images
- a single binary that handles signals itself: run it directly, or behind a minimal init
- a general-purpose supervision framework with service dependencies and per-service logging pipelines

## Comparison with Alternatives

| Tool | Choose it when you want | `osixia/baseimage` adds |
| --- | --- | --- |
| [tini](https://github.com/krallin/tini) | a tiny PID 1 that forwards signals and reaps zombies, and nothing else | lifecycle scripts, multi-process supervision, service selection, restarts and runtime commands |
| [runit](http://smarden.org/runit/) | a small, low-level supervision toolkit with your own conventions | a Docker-oriented service layout and lifecycle steps ready to use |
| [s6-overlay](https://github.com/just-containers/s6-overlay) | a powerful, flexible supervision framework, with service dependencies and the s6 logging model | a simpler model: three scripts per service and a handful of runtime flags |
| [supervisord](http://supervisord.org/) | a well-known process manager configured with INI files | a static binary with no Python dependency, lifecycle hooks and non-root / read-only support |

In short: `tini` is smaller, `s6-overlay` is more powerful, and `osixia/baseimage` aims for the simplest
operational model for application containers.
