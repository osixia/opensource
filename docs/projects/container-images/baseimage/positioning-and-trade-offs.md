# ⚖️ Positioning and Trade-offs

`osixia/baseimage` is an opinionated container base image for teams who want more runtime structure than a minimal init, without adopting a full supervision framework.

It is designed to help you build containers faster when you need:

- predictable lifecycle hooks
- simple multi-process support
- operational runtime commands
- a consistent service layout across images

In practice, it fits well when you do not want to keep rebuilding the same entrypoint logic, service conventions, and debugging workflows in every image.

The trade-off is deliberate: `osixia/baseimage` is heavier and more opinionated than a minimal PID 1 solution, but it gives you a faster path to reliable application containers.

## When It Is a Good Fit

`osixia/baseimage` is usually a good fit if you want to:

- package one or several tightly related processes in the same image
- standardize how services start, run, and stop
- keep image build logic and runtime lifecycle logic clearly separated
- gain practical runtime tooling for debugging, operations, and CI workflows
- prototype as a multi-process image first, then split responsibilities later if needed

Typical examples include:

- application containers with a main process and one or two supporting processes
- images that need repeatable startup and shutdown behavior
- teams that want stronger conventions across multiple container images
- development images that may later be split into more specialized production containers

## When It Is Probably Not the Right Fit

It is probably not the best choice if you mainly want:

- the smallest possible image surface
- a minimal PID 1 with signal forwarding and zombie reaping only
- a fully generic supervision framework with broader orchestration primitives
- strict adherence to a one-process-only model with no added runtime conventions

## Comparison with Alternatives

`osixia/baseimage` sits between a minimal init wrapper and a full supervision framework.

### Compared with `tini`

Choose `tini` if you mainly need a proper PID 1 and want to keep everything else custom.

Choose `osixia/baseimage` if you also want built-in lifecycle hooks, service conventions, and operational tooling without assembling them yourself.

`tini` is excellent when you mainly want:

- a small PID 1
- signal forwarding
- zombie reaping

Compared with `tini`, `osixia/baseimage` adds:

- service lifecycle scripts with `startup.sh`, `process.sh`, and `finish.sh`
- multi-process support
- service selection with `--exec` and `--skip`
- restart behavior
- built-in operational commands such as `container services`, `container processes`, and `container log`

Trade-off:

- `tini` is smaller and simpler
- `osixia/baseimage` is heavier, but gives you much more runtime structure

### Compared with `runit`

Choose `runit` if you want a lightweight supervision toolkit and prefer to define your own conventions.

Choose `osixia/baseimage` if you want a more Docker-oriented workflow with an opinionated service layout already in place.

`runit` is a lightweight supervision suite often used to keep several services running in one container.

Compared with `runit`, `osixia/baseimage` is generally:

- easier to approach for Docker users who want a simple image layout
- more opinionated around service scripts and lifecycle steps
- more focused on runtime ergonomics than raw supervision primitives

Trade-off:

- `runit` is a solid low-level supervisor
- `osixia/baseimage` is usually easier when you want a more Docker-oriented workflow and service model

### Compared with `s6-overlay`

Choose `s6-overlay` if you want a more advanced and flexible supervision framework.

Choose `osixia/baseimage` if you want a simpler operational model with practical conventions for application containers.

[`s6-overlay`](https://github.com/just-containers/s6-overlay) is a stronger fit when you want a more general-purpose supervision layer on top of an existing image.

Compared with `s6-overlay`, `osixia/baseimage` is generally:

- easier to adopt for straightforward application containers
- more opinionated around `startup.sh`, `process.sh`, and `finish.sh`
- more focused on practical Docker operations and runtime flags

Compared with `osixia/baseimage`, `s6-overlay` is generally:

- more flexible as a supervision system
- richer for advanced init, service orchestration, and supervision scenarios
- better suited when you want more of the `s6` ecosystem, including its logging model

Trade-off:

- `s6-overlay` is more powerful
- `osixia/baseimage` is usually simpler to operate

## Choice Summary

| If you mainly want... | Choose |
| --- | --- |
| a proper PID 1 with signal forwarding and zombie reaping, and little else | `tini` |
| a small supervision toolkit and the freedom to define your own conventions | `runit` |
| a more advanced and flexible supervision framework | `s6-overlay` |
| an opinionated, application-oriented runtime with simple service conventions and operational tooling | `osixia/baseimage` |
