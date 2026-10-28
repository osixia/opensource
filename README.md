# Osixia Open Source Projects Documentation

Osixia specializes in cloud-native infrastructure, containerization and secure deployments.

Through our GitHub organization, we publish and maintain several open source projects used both internally and by the broader community.

This repository aims to:

- Provide structured documentation for all Osixia open source projects
- Offer consistent usage guides and configuration references
- Make adoption and contribution easier for the community
- Centralize best practices and examples

![osixia/opensource illustration.](./docs/assets/images/osixia-opensource-landing.png)

- [Osixia Open Source Projects Documentation](#osixia-open-source-projects-documentation)
  - [📦 Projects Covered](#-projects-covered)
  - [Preview locally](#preview-locally)
  - [🗂️ Documentation Structure](#️-documentation-structure)
  - [🔀 Contributing](#--contributing)
  - [💬 Community \& Support](#-community--support)
  - [🔓 License](#-license)

## 📦 Projects Covered

This documentation includes (but is not limited to) the following projects:

| Project                  | Description                                                                          |
| ------------------------ | ------------------------------------------------------------------------------------ |
| **[container-baseimage](https://github.com/osixia/container-baseimage)**  | A container base image to build reliable single- or multi-process images quickly 🐳✨🌴 |
| **[container-keepalived](https://github.com/osixia/container-keepalived/)** | Keepalived container image 🐳🛟🌴 |
| **[container-openldap](https://github.com/osixia/container-openldap/)** | OpenLDAP container image 🐳🪪🌴 |
| **[kube-network-detector](https://github.com/osixia/kube-network-detector)** | Detect VIP routing and TCP connectivity, and label Kubernetes nodes |
| **[helm-charts](https://github.com/osixia/helm-charts)** | Deploy OpenLDAP and phpLDAPadmin on Kubernetes |
| **[dagger-build](https://github.com/osixia/dagger-build)** | Shared Go tooling to build and test container images with Dagger |

Read the guides at **[opensource.osixia.net](https://opensource.osixia.net)**.

## Preview locally

Requires Python 3 with `venv` support:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Open <http://127.0.0.1:8000>. Before submitting a change, run `mkdocs build --strict` to check navigation and links.

Project READMEs, manifests and chart values are the source for the guides. When updating a guide, check the corresponding repository and keep examples aligned with it.


## 🗂️ Documentation Structure

This repository is organized to provide structured documentation.

Typical structure:

```
.
├── docs
│   ├── assets
│   │   ├── images
│   │   │   ├── [...]
│   │   └── stylesheets
│   │       └── [...]
│   └── projects
│       ├── category
│       │   ├── project
│       │   ├── [...]
│       │   └── index.md
│       └── index.md
└──  mkdocs.yml
```

## 🔀 Contributing

Contributions are welcome and appreciated ✨

You can contribute by:

- Improving documentation clarity
- Fixing inaccuracies or outdated information
- Adding examples or deployment guides
- Reporting issues or inconsistencies

To contribute:

1. Fork the repository
2. Create a feature branch
3. Submit a Pull Request
4. Open an Issue for larger changes or discussions

If you are contributing documentation updates, please ensure:

- Clear technical explanations
- Practical examples
- Consistent formatting
- Accurate configuration references

Please keep documentation technical, precise, and production-oriented.

## 💬 Community & Support

For questions or support:

- Open an issue in the relevant project repository
- Check existing documentation before submitting a new issue
- Participate in discussions via GitHub

## 🔓 License

This project is licensed under the terms of the MIT license. See [LICENSE.md](LICENSE.md) file for more information.

---
Thank you for supporting and using Osixia open source projects 🚀
