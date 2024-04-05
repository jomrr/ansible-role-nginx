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
| Debian | Debian | 11 | [jomrr/molecule-debian:11]( https://hub.docker.com/r/jomrr/molecule-debian ) |
| | | 12 | [jomrr/molecule-debian:12]( https://hub.docker.com/r/jomrr/molecule-debian ) |
| | | latest | [jomrr/molecule-debian:latest]( https://hub.docker.com/r/jomrr/molecule-debian ) |
| | | testing | [jomrr/molecule-debian:testing]( https://hub.docker.com/r/jomrr/molecule-debian ) |
| Debian | Ubuntu | 20.04 | [jomrr/molecule-ubuntu:20.04]( https://hub.docker.com/r/jomrr/molecule-ubuntu ) |
| | | 22.04 | [jomrr/molecule-ubuntu:22.04]( https://hub.docker.com/r/jomrr/molecule-ubuntu ) |
| | | latest | [jomrr/molecule-ubuntu:latest]( https://hub.docker.com/r/jomrr/molecule-ubuntu ) |
| RedHat | AlmaLinux | 8 | [jomrr/molecule-almalinux:8]( https://hub.docker.com/r/jomrr/molecule-almalinux ) |
| | | 9 | [jomrr/molecule-almalinux:9]( https://hub.docker.com/r/jomrr/molecule-almalinux ) |
| | | latest | [jomrr/molecule-almalinux:latest]( https://hub.docker.com/r/jomrr/molecule-almalinux ) |
| RedHat | Fedora | 39 | [jomrr/molecule-fedora:39]( https://hub.docker.com/r/jomrr/molecule-fedora ) |
| | | latest | [jomrr/molecule-fedora:latest]( https://hub.docker.com/r/jomrr/molecule-fedora ) |
| | | rawhide | [jomrr/molecule-fedora:rawhide]( https://hub.docker.com/r/jomrr/molecule-fedora ) |

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
    - role: "jomrr.nginx"

```

## License

This role is published under the [MIT License](LICENSE).

## Author Information

This role was created in 2024 by Jonas Mauer (@jomrr).

## Contributing

Contributions to this role are welcome.
Please follow the steps described in [CONTRIBUTING.md](CONTRIBUTING.md).

## References

No references.

---
