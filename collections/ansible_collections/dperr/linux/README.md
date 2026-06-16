# Linux Ansible Collection

Enterprise-grade Ansible roles for managing Ubuntu Linux systems, with specialized support for APT mirror infrastructure and client configuration.

## Description

This collection provides production-ready roles for:
- **APT mirror infrastructure** - Host internal Ubuntu and Docker CE package mirrors
- **Client mirror configuration** - Point Ubuntu clients to internal mirrors
- **Air-gap/secure environments** - Lock systems to internal-only package sources

Designed for Ansible Automation Platform (AAP) with survey-friendly variable definitions.

## Roles

### [config_ubuntu_server](roles/config_ubuntu_server/README.md)

Configure an Ubuntu server as an APT mirror using `apt-mirror` and nginx. Host internal mirrors of Ubuntu OS repositories and/or Docker CE packages.

**Use cases:**
- Create internal package mirrors for enterprise networks
- Reduce external bandwidth usage
- Enable offline/air-gapped package installations
- Accelerate package installs across multiple systems

**Key features:**
- Ubuntu OS mirror (main, restricted, universe, multiverse)
- Docker CE mirror (smaller, faster for POC)
- Automated sync via cron
- Nginx serving with proper directory structure
- Optional tarball export for offline transport

**Quick example:**
```yaml
- role: dperr.linux.config_ubuntu_server
  vars:
    apt_mirror_server_profile: docker_ce
    apt_mirror_server_upstream: https://download.docker.com/linux/ubuntu
    apt_mirror_server_codenames: [noble]
```

### [config_ubuntu_mirrors](roles/config_ubuntu_mirrors/README.md)

Configure Ubuntu APT clients to use internal mirrors. Points package sources to your internal mirror server instead of external upstream repositories.

**Use cases:**
- Configure clients to use internal package mirrors
- Air-gap systems from external repositories
- Standardize package sources across fleet
- Security-hardened environments requiring isolated package sources

**Key features:**
- Ubuntu OS repository configuration
- Docker CE repository configuration
- External source cleanup (air-gap mode)
- Supports both deb822 and classic sources.list formats
- Automatic backup of original configurations

**Quick example:**
```yaml
- role: dperr.linux.config_ubuntu_mirrors
  vars:
    mirror_server_hostname: mirror.corp.local
    config_client_mirrors_profiles: [ubuntu, docker_ce]
```

## Installation

### From Ansible Galaxy (when published)

```bash
ansible-galaxy collection install dperr.linux
```

### From Local Source

```bash
# From the repository root
ansible-galaxy collection install collections/ansible_collections/dperr/linux/ -p ~/.ansible/collections

# Or build and install
cd collections/ansible_collections/dperr/linux/
ansible-galaxy collection build
ansible-galaxy collection install dperr-linux-*.tar.gz
```

## Requirements

- Ansible 2.9 or higher
- Ubuntu target systems (Jammy 22.04 or Noble 24.04+)
- For `config_ubuntu_server` role: `community.general` collection (for archive module)

```bash
ansible-galaxy collection install community.general
```

## AAP Integration

All roles in this collection are designed with Ansible Automation Platform (AAP) surveys in mind:

- Variables documented with survey types (Text, Multiple Choice, etc.)
- Sensible defaults for quick setup
- Clear validation requirements
- Examples of survey question configurations in each role README

See individual role READMEs for detailed AAP survey configuration examples.

## Quick Start

### 1. Set up a mirror server

```yaml
---
- name: Configure Docker CE mirror server
  hosts: mirror_servers
  become: true
  
  roles:
    - role: dperr.linux.config_ubuntu_server
      vars:
        apt_mirror_server_profile: docker_ce
        apt_mirror_server_codenames: [noble]
        apt_mirror_server_run_initial_sync: true
```

### 2. Configure clients to use the mirror

```yaml
---
- name: Point clients to internal mirror
  hosts: ubuntu_clients
  become: true
  
  roles:
    - role: dperr.linux.config_ubuntu_mirrors
      vars:
        mirror_server_hostname: "{{ groups['mirror_servers'][0] }}"
        config_client_mirrors_profiles:
          - docker_ce
```

## Common Workflows

### Workflow 1: Air-Gapped Environment Setup

```yaml
# Step 1: Set up mirror on internet-connected staging server
- hosts: staging_mirror
  roles:
    - role: dperr.linux.config_ubuntu_server
      vars:
        apt_mirror_server_profile: ubuntu
        apt_mirror_server_codenames: [noble]
        apt_mirror_server_run_initial_sync: true
        apt_mirror_server_create_export_tarball: true

# Step 2: Transfer tarball to air-gapped network (manual step)

# Step 3: Extract and serve on air-gapped mirror server
# (manual extraction, then configure nginx)

# Step 4: Configure air-gapped clients
- hosts: airgap_clients
  roles:
    - role: dperr.linux.config_ubuntu_mirrors
      vars:
        mirror_server_hostname: 10.100.1.50
        config_client_mirrors_profiles: [ubuntu]
        config_client_mirrors_external_sources_enabled: true
```

### Workflow 2: Multi-Mirror Setup (Ubuntu + Docker)

```yaml
# Configure server to host both mirrors
- hosts: mirror_server
  roles:
    - role: dperr.linux.config_ubuntu_server
      vars:
        apt_mirror_server_profile: ubuntu
        apt_mirror_server_codenames: [jammy, noble]
        
    - role: dperr.linux.config_ubuntu_server
      vars:
        apt_mirror_server_profile: docker_ce
        apt_mirror_server_codenames: [noble]
        apt_mirror_server_run_initial_sync: true

# Configure clients for both
- hosts: docker_hosts
  roles:
    - role: dperr.linux.config_ubuntu_mirrors
      vars:
        mirror_server_hostname: mirror.example.com
        config_client_mirrors_profiles:
          - ubuntu
          - docker_ce
```

## Testing

After configuring mirrors and clients:

```bash
# On client systems
sudo apt update
sudo apt-cache policy
sudo apt-cache policy docker-ce  # if using docker_ce profile

# Verify packages resolve from internal mirror
apt-cache show docker-ce | grep -i origin
```

## Contributing

Contributions welcome! Please:
1. Test changes on Ubuntu 22.04 and 24.04
2. Update role READMEs with new variables
3. Include AAP survey examples for new variables
4. Follow existing variable naming conventions

## License

MIT

## Author

Dennis Perrone

## Support

- Report issues: [GitHub Issues](https://github.com/dperrone/ansible-demos/issues)
- Documentation: See individual role READMEs