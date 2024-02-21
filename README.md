# Ansible role nginx

![GitHub](https://img.shields.io/github/license/nginx/ansible-role-nginx) ![GitHub last commit](https://img.shields.io/github/last-commit/nginx/ansible-role-nginx) ![GitHub issues](https://img.shields.io/github/issues-raw/nginx/ansible-role-nginx)

**Ansible role for setting up the nginx sever.**

## Description

This role installs the Nginx package, sets up the configuration files,
modules, virtual hosts or reverse-proxy definitons and manages the service.

## Table of Contents

- [Description](#description)
- [Table of Contents](#table-of-contents)
- [Prerequisites](#prerequisites)
  - [System packages (Fedora)](#system-packages-fedora)
  - [Python (requirements.txt)](#python-requirementstxt)
- [Dependencies (requirements.yml)](#dependencies-requirementsyml)
- [Supported Platforms](#supported-platforms)
- [Role Variables](#role-variables)
- [Example Playbook](#example-playbook)
- [The default example playbook](#the-default-example-playbook)
- [License](#license)
- [Author Information](#author-information)
- [Contributing](#contributing)
- [References](#references)

## Prerequisites

### System packages (Fedora)

- `python3` (Python 3.6 or later)

### Python (requirements.txt)

- ansible >= 2.15

## Dependencies (requirements.yml)

This role has no dependencies.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
|-----------|--------------|---------|-----------------|
| Debian | Debian | 11 | [jam82/molecule-debian:11]( https://hub.docker.com/r/jam82/molecule-debian ) |
| | | 12 | [jam82/molecule-debian:12]( https://hub.docker.com/r/jam82/molecule-debian ) |
| | | latest | [jam82/molecule-debian:latest]( https://hub.docker.com/r/jam82/molecule-debian ) |
| | | testing | [jam82/molecule-debian:testing]( https://hub.docker.com/r/jam82/molecule-debian ) |
| Debian | Ubuntu | 20.04 | [jam82/molecule-ubuntu:20.04]( https://hub.docker.com/r/jam82/molecule-ubuntu ) |
| | | 22.04 | [jam82/molecule-ubuntu:22.04]( https://hub.docker.com/r/jam82/molecule-ubuntu ) |
| | | latest | [jam82/molecule-ubuntu:latest]( https://hub.docker.com/r/jam82/molecule-ubuntu ) |
| RedHat | AlmaLinux | 8 | [jam82/molecule-almalinux:8]( https://hub.docker.com/r/jam82/molecule-almalinux ) |
| | | 9 | [jam82/molecule-almalinux:9]( https://hub.docker.com/r/jam82/molecule-almalinux ) |
| | | latest | [jam82/molecule-almalinux:latest]( https://hub.docker.com/r/jam82/molecule-almalinux ) |
| RedHat | Fedora | 39 | [jam82/molecule-fedora:39]( https://hub.docker.com/r/jam82/molecule-fedora ) |
| | | latest | [jam82/molecule-fedora:latest]( https://hub.docker.com/r/jam82/molecule-fedora ) |
| | | rawhide | [jam82/molecule-fedora:rawhide]( https://hub.docker.com/r/jam82/molecule-fedora ) |

## Role Variables

No role default variables specified, see [defaults/main.yml](defaults/main.yml).

## Example Playbook

Example(s) of how to use this role:

## The default example playbook

The default example playbook for the role.


```yaml
---
# play: "nginx"
# file: "nginx.yml"
- hosts: all
  gather_facts: true
  roles:
    - role: "jam82.nginx"

```

## License

This role is published under the [MIT License](LICENSE).

## Author Information

This role was created in 2024 by Jonas Mauer (@jam82).

## Contributing

Contributions to this role are welcome.
Please follow the steps described in [CONTRIBUTING.md](CONTRIBUTING.md).

## References

No references.

---
