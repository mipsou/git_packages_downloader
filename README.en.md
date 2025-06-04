[Version Française](README.md)
# Ansible Role: git_packages_downloader 📥

[![Ansible Galaxy](https://img.shields.io/badge/Ansible%20Galaxy-mipsou.git__packages__downloader-blue?style=flat-square)](https://galaxy.ansible.com/mipsou/git_packages_downloader)
[![GitHub tag](https://img.shields.io/github/v/tag/mipsou/git_packages_downloader?sort=semver&style=flat-square)](https://github.com/mipsou/git_packages_downloader/tags)
[![Ansible Lint](https://github.com/mipsou/git_packages_downloader/actions/workflows/lint.yml/badge.svg)](https://github.com/mipsou/git_packages_downloader/actions/workflows/lint.yml)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg?style=flat-square)](LICENSE)

This Ansible role downloads and mirrors the latest release artifacts from Git-based repositories (e.g., GitHub) into a local RPM repository.

## ✨ Features

- Dynamically downloads `.rpm` (or any other format) releases from GitHub 📦
- Filters by architecture and extension (e.g., `amd64`, `aarch64`) 🔩
- Idempotent and Foreman-compatible ✅
- Supports local repo creation and SELinux contexts 🛡️

## 📋 Requirements

- The `python3-jmespath` package must be installed on the target host where this role is executed. The role includes a task to attempt its installation.
- If using SELinux, the necessary tools (`restorecon`, `semanage`) should be available on the target.

## ⚙️ Configuration

This role is configured primarily through the `git_packages` variable, or alternatively via `corp_mirror_git` for centralized setups.

### Defining Packages (`git_packages`)

The `git_packages` variable is a list of dictionaries, where each dictionary defines a package to download and manage. By default, this list is empty (`git_packages: []`), so you must define it to use the role.

**Structure for each package item:**

*   `name` (string, required): A descriptive name for the package (e.g., "mercure", "my-app"). Used for logging and labels.
*   `source` (dictionary, required): Defines where to download the package from.
    *   `provider` (string, required): The type of Git provider. Currently, `"github"` is supported.
    *   `owner` (string, required for GitHub): The owner or organization of the GitHub repository (e.g., `dunglas`).
    *   `repo` (string, required for GitHub): The name of the GitHub repository (e.g., `mercure`).
    *   `arch` (string, required for GitHub): The architecture to filter for in the release assets (e.g., `amd64`, `x86_64`, `aarch64`, `noarch`).
    *   `ext` (string, required for GitHub): The file extension to filter for (e.g., `rpm`, `deb`, `tar.gz`).
*   `repository` (dictionary, required): Defines the local repository details.
    *   `name` (string, required): The name of the local repository directory to be created (e.g., `mercure`, `my-custom-repo`). This will be a subdirectory under `base_path`.
    *   `base_path` (string, required): The base path on the target host where the repository directory will be created (e.g., `/var/www/html/public/repos`, `/opt/mirrors`).
    *   `mode` (string, optional, default: `"0755"`): Permissions for the repository directory.
    *   `owner` (string, optional, default: `apache`): Owner for the repository directory and downloaded files. Common alternatives include `nginx` or `www-data` depending on your web server.
    *   `group` (string, optional, default: `apache`): Group for the repository directory and downloaded files. Common alternatives include `nginx` or `www-data`.
    *   `selinux_enable` (boolean, optional, default: `false`): Whether to manage SELinux contexts for the repository path (sets `httpd_sys_content_t`).

**Example `git_packages` definition:**

```yaml
git_packages:
  - name: mercure_rpm # Descriptive package name
    source:
      provider: github
      owner: dunglas
      repo: mercure
      arch: amd64 # Can also be x86_64, etc.
      ext: rpm
    repository:
      name: mercure # This will create /var/www/html/public/repos/mercure
      base_path: /var/www/html/public/repos # Common web-accessible path
      mode: "0755"
      owner: apache # Or nginx, www-data, etc., depending on your web server user
      group: apache # Or nginx, www-data, etc.
      selinux_enable: true
  - name: another_tool_tarball
    source:
      provider: github
      owner: someuser
      repo: sometool
      arch: x86_64 # Ensure this matches an artifact name part
      ext: tar.gz
    repository:
      name: sometool_archive
      base_path: /opt/pkg_archives # Example for non-web accessible archives
      owner: root # Different owner/group for non-web content
      group: root
      # selinux_enable defaults to false if not specified
```

## 🏢 Centralized Configuration & The Foreman Integration

For managing package lists centrally, especially in environments like The Foreman, this role supports an alternative configuration method using the `corp_mirror_git` variable.

### Using `corp_mirror_git`

If you define a variable named `corp_mirror_git`, and this variable contains a key `git_packages`, the role will use the list provided under `corp_mirror_git.git_packages` as its source for packages to download. This will override any direct definition of the `git_packages` variable.

This is useful for maintaining a single source of truth for your package definitions across multiple hosts or environments.

**Structure for `corp_mirror_git`:**

The `corp_mirror_git` variable should be a dictionary containing a single key, `git_packages`. The value of this `git_packages` key must be a list structured exactly as described in the "Defining Packages (`git_packages`)" section above.

**Example `corp_mirror_git` definition:**

```yaml
corp_mirror_git:
  git_packages:
    - name: mercure_rpm_from_corp_mirror # Descriptive name
      source:
        provider: github
        owner: dunglas
        repo: mercure
        arch: amd64
        ext: rpm
      repository: # Ensure this key is present and correctly indented
        name: mercure_corp
        base_path: /var/www/html/public/repos/corporate # Web-accessible path
        owner: apache # Or nginx, www-data, etc.
        group: apache # Or nginx, www-data, etc.
        selinux_enable: true
    - name: utility_scripts_from_corp_mirror
      source:
        provider: github
        owner: my-org
        repo: internal-utils
        arch: noarch # Example for no specific architecture
        ext: tar.gz
      repository:
        name: internal_utils
        base_path: /srv/internal_repo/utils
        owner: ansible_svc_user # Example with a specific service user
        group: ansible_svc_user
```

### Configuring in The Foreman 🛠️

To use this role with The Foreman, the recommended approach is to define the `corp_mirror_git` variable as an Ansible Variable within Foreman. This allows you to manage your package list centrally.

**Steps:**

1.  In Foreman, navigate to **Configure > Ansible > Variables**.
2.  Click **Create Variable** (or select an existing one to override at the desired context like Host Group, OS, or specific Host).
3.  **Variable Name:** `corp_mirror_git`
4.  **Value:** Paste the complete YAML content for your `corp_mirror_git` structure (as shown in the "Using `corp_mirror_git`" example above) into the value field.
    *   *Ensure the YAML is correctly formatted in the value field.*
    *   *Note: The `git_packages` variable itself is not typically managed as an editable Smart Parameter due to its complex structure.*

This method ensures Foreman passes the entire package configuration to the role.

## 💡 Design Philosophy

This role is portable, declarative, and avoids hardcoding. It is designed to integrate easily with Foreman but works without it.

## 📜 License

Apache License 2.0 — © Mipsou  
See [LICENSE](LICENSE) for details.