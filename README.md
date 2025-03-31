# git_packages_downloader

Ansible role to download and mirror the latest release artifacts from Git-based repositories (e.g. GitHub) into a local RPM repository.

## Features

- Downloads `.rpm` (or any other format) releases dynamically from GitHub
- Filters by architecture and extension (e.g. `amd64`, `rpm`)
- Idempotent and Foreman-compatible
- Supports local repo creation and SELinux contexts

## Example Variables

```yaml
git_packages:
  - name: mercure
    source:
      provider: github
      owner: dunglas
      repo: mercure
      arch: amd64
      ext: rpm
    repository:
      name: mercure
      base_path: /var/www/html/repos
      mode: "0755"
      owner: apache
      group: apache
      selinux_enable: true
```

## Foreman Integration

Use Ansible Parameters in Foreman to override `git_packages`. Assign this role to a host or host group.

## Design Philosophy

This role is portable, declarative, and avoids hardcoding. It is designed to integrate easily with Foreman but works without it.

## License

Apache License 2.0 — © Mipsou  
See [LICENSE](LICENSE) for details.
