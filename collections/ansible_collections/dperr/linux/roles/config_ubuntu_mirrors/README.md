# Configure Ubuntu Clients

Configure Ubuntu APT clients to use internal mirrors for package management. Supports Ubuntu OS repositories, Docker CE repositories, and optional external source cleanup for air-gapped or security-hardened environments.

## Description

This role configures APT package sources to point to internal mirrors rather than external upstream repositories. It consolidates three related mirror configuration tasks:

- **Ubuntu mirror** — Point APT at your internal Ubuntu package mirror (main, restricted, universe, multiverse)
- **Docker CE mirror** — Configure Docker CE repository to use your internal mirror
- **External sources cleanup** — Remove or disable all APT sources that don't reference your internal mirror (air-gap mode)

The role detects and supports both modern deb822 format (`ubuntu.sources`) and classic `sources.list` configurations.

## Requirements

- Ubuntu operating system (Jammy 22.04 or Noble 24.04+)
- Internal mirror server(s) configured and accessible via HTTP/HTTPS
- Target systems must have network connectivity to the mirror server
- For Docker CE: mirror server must host Docker CE repository structure

## Role Variables

### AAP Survey Variables

These variables are **recommended for AAP surveys** to provide user-friendly job configuration:

| Variable | Type | Required | Default | AAP Survey Type | Choices/Validation | Description |
|----------|------|----------|---------|-----------------|-------------------|-------------|
| `mirror_server_hostname` | string | **Yes** | `ubuntu-svr-01.dperrone.dev` | Text | FQDN or IP | Hostname or IP address of internal mirror server |
| `config_client_mirrors_profiles` | list | **Yes** | `[]` | Multiple Choice (multi-select) | `ubuntu`, `docker_ce` | Mirror profiles to configure |
| `config_client_mirrors_external_sources_enabled` | boolean | No | `false` | Multiple Choice | `true`, `false` | Remove external sources (air-gap mode) |

### Ubuntu Mirror Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `config_client_mirrors_ubuntu_base_url` | `http://{{ mirror_server_hostname }}/ubuntu` | Base URL of Ubuntu mirror (must match archive.ubuntu.com/ubuntu layout) |
| `config_client_mirrors_ubuntu_suites` | `{{ ansible_distribution_release }} ...` | Space-separated suite names (e.g., noble, noble-updates, noble-backports, noble-security) |
| `config_client_mirrors_ubuntu_signed_by` | `/usr/share/keyrings/ubuntu-archive-keyring.gpg` | GPG keyring path for Ubuntu archive verification |

### Docker CE Mirror Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `config_client_mirrors_docker_origin` | `http://{{ mirror_server_hostname }}` | Base URL of Docker mirror host (no trailing slash) |
| `config_client_mirrors_docker_path` | `/linux/ubuntu` | URL path to Docker repository (must match nginx location) |
| `config_client_mirrors_docker_codename` | `noble` | Ubuntu codename for Docker packages |
| `config_client_mirrors_docker_channel` | `stable` | Docker release channel (stable, test, nightly) |
| `config_client_mirrors_docker_arch` | `amd64` | System architecture |
| `config_client_mirrors_docker_key_url` | `https://download.docker.com/linux/ubuntu/gpg` | Docker official GPG key URL |
| `config_client_mirrors_docker_signed_by` | `/etc/apt/keyrings/docker.asc` | Local path for Docker GPG key |

### External Sources Cleanup Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `config_client_mirrors_external_sources_allowed_origin` | `{{ mirror_server_hostname }}` | Only source files containing this origin hostname are kept |
| `config_client_mirrors_external_sources_disable_sources_list` | `true` | Blank `/etc/apt/sources.list` to prevent fallback to archive.ubuntu.com |

## Dependencies

None

## AAP Survey Configuration Examples

### Survey Question 1: Mirror Server Hostname

```yaml
Variable: mirror_server_hostname
Question: What is the hostname or IP of your internal mirror server?
Answer Type: Text
Default: ubuntu-svr-01.dperrone.dev
Required: Yes
```

### Survey Question 2: Mirror Profiles

```yaml
Variable: config_client_mirrors_profiles
Question: Which mirror profiles should be configured?
Answer Type: Multiple Choice (multiple select)
Required: Yes
Choices:
  - ubuntu
  - docker_ce
```

### Survey Question 3: Air-Gap Mode

```yaml
Variable: config_client_mirrors_external_sources_enabled
Question: Enable air-gap mode? (Remove all external APT sources)
Answer Type: Multiple Choice
Required: No
Default: false
Choices:
  - label: "No - Keep external sources available"
    value: false
  - label: "Yes - Remove all external sources (air-gap)"
    value: true
```

## Example Playbooks

### Basic Mirror Configuration

```yaml
---
- name: Configure APT mirrors on Ubuntu clients
  hosts: ubuntu_clients
  become: true
  
  roles:
    - role: dperr.linux.config_ubuntu_mirrors
      vars:
        mirror_server_hostname: mirror.example.com
        config_client_mirrors_profiles:
          - ubuntu
          - docker_ce
```

### Air-Gapped/Secure Environment

```yaml
---
- name: Configure air-gapped Ubuntu clients
  hosts: secure_zone
  become: true
  
  roles:
    - role: dperr.linux.config_ubuntu_mirrors
      vars:
        mirror_server_hostname: 10.100.1.50
        config_client_mirrors_profiles:
          - ubuntu
        config_client_mirrors_external_sources_enabled: true
        config_client_mirrors_external_sources_disable_sources_list: true
```

### Ubuntu Mirror Only (No Docker)

```yaml
---
- name: Configure Ubuntu mirror only
  hosts: workstations
  become: true
  
  roles:
    - role: dperr.linux.config_ubuntu_mirrors
      vars:
        mirror_server_hostname: ubuntu-mirror.corp.local
        config_client_mirrors_profiles:
          - ubuntu
```

## How It Works

1. **Validation**: Verifies target system is Ubuntu distribution
2. **Ubuntu Mirror** (when `ubuntu` in profiles):
   - Detects deb822 format (`ubuntu.sources`) vs classic format (`sources.list`)
   - Configures appropriate sources file pointing to internal mirror
   - Backs up original configuration before modification
3. **Docker CE Mirror** (when `docker_ce` in profiles):
   - Installs CA certificates for HTTPS key download
   - Downloads Docker official GPG signing key
   - Creates Docker CE source configuration pointing to internal mirror
4. **External Source Cleanup** (when enabled):
   - Removes all source files not matching allowed origin hostname
   - Optionally blanks `/etc/apt/sources.list` to prevent fallback
5. **APT Cache Update**: Handler automatically refreshes package cache after changes

## Handlers

- `Update APT package cache` - Executes `apt update` when source configurations change

## Testing After Configuration

Verify mirror configuration:

```bash
# Check deb822 format sources (Ubuntu 24.04+)
sudo cat /etc/apt/sources.list.d/ubuntu.sources

# Check classic format sources
sudo cat /etc/apt/sources.list

# Check Docker CE source (if configured)
sudo cat /etc/apt/sources.list.d/docker-ce-mirror.list

# Update package cache
sudo apt update

# Verify packages resolve from internal mirror
apt-cache policy
apt-cache policy docker-ce  # for docker_ce profile
```

## Important Notes

- **Backup Safety**: Original source files are automatically backed up with `.backup` extension
- **Format Detection**: Ubuntu 24.04+ uses deb822 format by default; role handles both formats automatically
- **Air-Gap Warning**: Enabling external source cleanup is difficult to reverse without manual intervention
- **Docker GPG Key**: Role fetches Docker GPG key from official servers; requires internet access or pre-staged key in air-gapped environments
- **Mirror Synchronization**: Ensure mirror server is actively syncing before pointing clients to it
- **Testing**: Always test on non-production systems first

## Troubleshooting

### APT update fails after configuration

Check mirror server accessibility:
```bash
curl -I http://your-mirror-server/ubuntu/dists/noble/Release
```

### External sources not removed

Verify `config_client_mirrors_external_sources_enabled` is set to `true` and check allowed origin matches mirror hostname exactly.

### Docker packages not found

Ensure Docker mirror profile is enabled and mirror server has Docker CE repository synced.

## License

MIT

## Author

Dennis Perrone
