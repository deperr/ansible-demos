# config_client_mirrors

Configure Ubuntu apt to use internal mirrors for Ubuntu packages and Docker CE, and optionally lock out external sources.

This role consolidates three related mirror configuration tasks:
- **Ubuntu mirror** — Point apt at your internal Ubuntu package mirror
- **Docker CE mirror** — Configure Docker CE repository to use your internal mirror
- **External sources cleanup** — Remove or disable all apt sources that don't reference your internal mirror

## Requirements

- Ubuntu (Jammy or Noble)
- Internal mirror server hosting Ubuntu and/or Docker CE repositories

## Role Variables

### Core Configuration

```yaml
# Profiles that determine which mirrors to configure.
# Valid values: 'ubuntu', 'docker_ce'
config_client_mirrors_profiles: []

# Enable removal of external sources (locks to internal mirrors only)
config_client_mirrors_external_sources_enabled: false
```

### Ubuntu Mirror Settings

```yaml
# Base URL of your mirror (paths must match archive.ubuntu.com/ubuntu layout)
config_client_mirrors_ubuntu_base_url: "http://{{ mirror_server_hostname }}/ubuntu"

# Space-separated suite names
config_client_mirrors_ubuntu_suites: >-
  {{ ansible_distribution_release }}
  {{ ansible_distribution_release }}-updates
  {{ ansible_distribution_release }}-backports
  {{ ansible_distribution_release }}-security

# Signing key location (Ubuntu keyring package)
config_client_mirrors_ubuntu_signed_by: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

### Docker Mirror Settings

```yaml
# Base URL of your mirror host
config_client_mirrors_docker_origin: "http://{{ mirror_server_hostname }}"

# Path nginx serves for the Docker mirror
config_client_mirrors_docker_path: "/linux/ubuntu"

config_client_mirrors_docker_codename: noble
config_client_mirrors_docker_channel: stable
config_client_mirrors_docker_arch: amd64

# Official Docker archive signing key URL
config_client_mirrors_docker_key_url: https://download.docker.com/linux/ubuntu/gpg
config_client_mirrors_docker_signed_by: /etc/apt/keyrings/docker.asc
```

### External Sources Cleanup Settings

```yaml
# Only source files containing this origin are kept
config_client_mirrors_external_sources_allowed_origin: "{{ mirror_server_hostname }}"

# Also blank /etc/apt/sources.list (Ubuntu default OS sources)
config_client_mirrors_external_sources_disable_sources_list: true
```

## Dependencies

None.

## Example Playbook

```yaml
- hosts: ubuntu_clients
  become: true
  roles:
    - role: config_client_mirrors
      vars:
        mirror_server_hostname: ubuntu-svr-01.example.com
        config_client_mirrors_profiles:
          - ubuntu
          - docker_ce
        config_client_mirrors_external_sources_enabled: true
```

## License

MIT

## Author Information

aap-ubuntu
