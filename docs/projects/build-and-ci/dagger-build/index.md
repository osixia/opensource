# dagger-build

Shared Go tooling for building and testing Osixia container images with Dagger. Each image project keeps its own build executable, image matrix and integration tests; `dagger-build` provides the common jobs and CLI.

It is a Go library using the standalone Dagger SDK, rather than a standalone application or a native Dagger module. It requires Go 1.26.1 or newer and uses Dagger SDK/engine 0.21.10.

## ⚡ Try it in an existing project

You need Go and a running Docker engine for the default local Dagger setup. From a checkout of `container-baseimage`:

```bash
cd build
go run . --help
go run . job test ../Dockerfile --target base --distributions debian --arches amd64
```

Use `--arches arm64` on an ARM64 runner. Limiting the architecture and distribution keeps the first run small. The SDK downloads the matching Dagger CLI when it connects.

`job test` builds the images and runs the tests; it does not publish images. The default test checks that the image's `--version` output matches its generated image name and tag. The consuming project can add its own checks.

## 🔧 Use it in your own project

Create a Go build program in your project's `build/` directory. Add a dependency pinned to an existing tag or commit:

```bash
go get github.com/osixia/dagger-build@<existing-tag-or-commit>
```

The program configures an image, then runs the shared CLI:

```go
package main

import (
    "context"
    "os"

    "github.com/osixia/dagger-build/cmd"
    "github.com/osixia/dagger-build/config"
)

func main() {
    image := &config.Image{
        BaseImage:    "debian:trixie-slim",
        Distribution: config.Debian,
        Name:         "example/my-image",
        TagPrefixes:  []string{"debian"},
    }
    config.Images = []*config.Image{image}
    config.DefaultImage = image
    config.ProjectGithubRepo = &config.GithubRepo{
        Organization: "example",
        Project:      "my-image",
    }
    if err := cmd.Run(context.Background()); err != nil {
        os.Exit(1)
    }
}
```

Your Dockerfile must accept the `BASE_IMAGE` and `IMAGE` build arguments. For the default test, its entrypoint must print `<image-name>:<generated-tag>` when called with `--version`.

## 🧪 Common commands

Run these from the consuming project's build directory:

| Command | What it does |
| --- | --- |
| `go run . job build ../Dockerfile` | Build the configured images |
| `go run . job test ../Dockerfile` | Build and test, using the project's test hook |
| `go run . job test ../Dockerfile --export /usr/sbin/container --export-to ../bin` | Also export a binary from the default image |
| `go run . job test ../Dockerfile --oci-export-to ../oci` | Export images as OCI archives for later use |

Use `--target`, `--distributions` and `--arches` to select what to build. Use `--help` on any command for its options.

## 🏗️ Project tests and CI

To add integration tests, assign `job.Test` before calling `cmd.Run`. Call `job.DefaultTest` first, then run your checks against the returned containers. Keep the fixtures and tests in the consuming project.

For multiple architectures, build and test on native runners, export with `--oci-export-to`, then collect the archives for the final job with `--oci-import-from`. Imported images are not tested again before deployment.

The `provider github` command selects jobs from the GitHub ref. Tag pipelines, `job deploy` and `job publish` can push images and change GitHub releases or assets. Use `job test` for local validation.

[Source and full setup examples](https://github.com/osixia/dagger-build)
