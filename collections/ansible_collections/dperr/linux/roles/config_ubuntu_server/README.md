# Configure Ubuntu Mirror Server

Configure an Ubuntu server as an APT mirror using `apt-mirror` and nginx. This role sets up a local mirror of Ubuntu or Docker CE repositories for serving packages to internal networks. This role is extensible and is only set up for Docker and Ubuntu OS repositories from demostration purposes.

## Description

This role transforms an Ubuntu server into a package mirror server that can host and serve:

- **Ubuntu OS repositories** - Full or partial Ubuntu archive mirror (main, restricted, universe, multiverse)
- **Docker CE repositories** - Docker Community Edition package mirror (smaller, faster for POC)

The role installs and configures:
- `apt-mirror` - Syncs packages from upstream repositories
- `nginx` - Serves mirrored packages via HTTP
- Cron jobs for automated synchronization
- Optional tarball export for offline/portable distribution

## Description

This is the **server-side** companion to the `config_ubuntu_mirrors` role (which configures clients). Use this role to create the mirror infrastructure that clients will point to.

## Requirements

- Ubuntu operating system (Jammy 22.04 or Noble 24.04+)
- Sufficient disk space for mirrored repositories:
  - **Docker CE mirror**: ~5-10 GB (single codename, single arch)
  - **Ubuntu partial mirror**: 50-200 GB (single codename, all components, single arch)
  - **Ubuntu full mirror**: Multi-TB (all codenames, all components, all architectures)
- Internet connectivity to upstream repositories (for initial sync)
- Recommended: Dedicated disk/partition for mirror storage at `/var/spool/apt-mirror`

## Role Variables

### AAP Survey Variables

These variables are **recommended for AAP surveys** to provide user-friendly job configuration:

| Variable | Type | Required | Default | AAP Survey Type | Choices/Validation | Description |
|----------|------|----------|---------|-----------------|-------------------|-------------|
| `apt_mirror_server_profile` | string | **Yes** | `docker_ce` | Multiple Choice | `ubuntu`, `docker_ce` | Mirror profile to configure |
| `apt_mirror_server_upstream` | string | **Yes** | `https://download.docker.com/linux/ubuntu` | Text | Valid URL | Upstream repository URL to mirror |
| `apt_mirror_server_codenames` | list | **Yes** | `[noble]` | Multiple Choice (multi-select) | `focal`, `jammy`, `noble` | Ubuntu release codenames to mirror |
| `apt_mirror_server_architectures` | list | **Yes** | `[amd64]` | Multiple Choice (multi-select) | `amd64`, `arm64`, `i386` | CPU architectures to mirror |
| `apt_mirror_server_run_initial_sync` | boolean | No | `false` | Multiple Choice | `true`, `false` | Run full sync immediately (can take hours/days) |
| `apt_mirror_server_create_export_tarball` | boolean | No | `false` | Multiple Choice | `true`, `false` | Create portable tarball of mirror |

### Core Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `apt_mirror_server_profile` | `docker_ce` | Mirror profile: `ubuntu` (full OS) or `docker_ce` (Docker packages only) |
| `apt_mirror_server_upstream` | `https://download.docker.com/linux/ubuntu` | Upstream base URL for apt-mirror |
| `apt_mirror_server_upstream_hostname` | `""` | Override hostname extracted from URL (for non-standard ports or custom paths) |
| `apt_mirror_server_base_path` | `/var/spool/apt-mirror` | Local storage path for mirror data |

### Ubuntu Profile Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `apt_mirror_server_codenames` | `[noble]` | Ubuntu release codenames to mirror (e.g., focal, jammy, noble) |
| `apt_mirror_server_components` | `main restricted universe multiverse` | Repository components to mirror |
| `apt_mirror_server_architectures` | `[amd64]` | CPU architectures to mirror (each adds significant disk usage) |

### Docker CE Profile Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `apt_mirror_server_docker_channel` | `stable` | Docker release channel (stable, test, nightly) |

### Advanced Settings

| Variable | Default | Description |
|----------|---------|-------------|
| `apt_mirror_server_run_postmirror` | `0` | Enable apt-mirror postmirror script (see `/usr/share/doc/apt-mirror/examples`) |
| `apt_mirror_server_nginx_listen` | `80 default_server` | Nginx listen directive |
| `apt_mirror_server_nginx_server_name` | `_` | Nginx server_name directive |
| `apt_mirror_server_cron_hour` | `2` | Hour for cron sync (24-hour format) |
| `apt_mirror_server_cron_minute` | `30` | Minute for cron sync |
| `apt_mirror_server_run_initial_sync` | `false` | Run full apt-mirror sync during playbook execution (WARNING: can take hours to days) |

### Export Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `apt_mirror_server_create_export_tarball` | `false` | Create portable tarball of mirrored packages |
| `apt_mirror_server_export_path` | `/var/spool/apt-mirror/export` | Directory for export tarballs |
| `apt_mirror_server_export_format` | `gz` | Tarball compression format (gz, bz2, xz) |

## Dependencies

None

## AAP Survey Configuration Examples

### Survey Question 1: Mirror Profile

```yaml
Variable: apt_mirror_server_profile
Question: Which repository profile should this mirror server host?
Answer Type: Multiple Choice
Required: Yes
Default: docker_ce
Choices:
  - label: "Docker CE (5-10 GB, fast POC)"
    value: docker_ce
  - label: "Ubuntu OS (50+ GB per codename)"
    value: ubuntu
```

### Survey Question 2: Upstream Repository URL

```yaml
Variable: apt_mirror_server_upstream
Question: What is the upstream repository URL to mirror?
Answer Type: Text
Required: Yes
Default: https://download.docker.com/linux/ubuntu
Examples:
  - https://download.docker.com/linux/ubuntu
  - http://archive.ubuntu.com/ubuntu
  - http://us.archive.ubuntu.com/ubuntu
```

### Survey Question 3: Release Codenames

```yaml
Variable: apt_mirror_server_codenames
Question: Which Ubuntu release codenames should be mirrored?
Answer Type: Multiple Choice (multiple select)
Required: Yes
Default: [noble]
Choices:
  - focal   # Ubuntu 20.04 LTS
  - jammy   # Ubuntu 22.04 LTS
  - noble   # Ubuntu 24.04 LTS
```

### Survey Question 4: CPU Architectures

```yaml
Variable: apt_mirror_server_architectures
Question: Which CPU architectures should be mirrored?
Answer Type: Multiple Choice (multiple select)
Required: Yes
Default: [amd64]
Choices:
  - amd64   # x86_64 (most common)
  - arm64   # ARM 64-bit
  - i386    # 32-bit (legacy)
```

### Survey Question 5: Initial Sync

```yaml
Variable: apt_mirror_server_run_initial_sync
Question: Run initial mirror sync now? (WARNING: Can take hours/days and block playbook)
Answer Type: Multiple Choice
Required: No
Default: false
Choices:
  - label: "No - Schedule cron only (recommended)"
    value: false
  - label: "Yes - Sync now (blocks until complete)"
    value: true
```

### Survey Question 6: Create Export Tarball

```yaml
Variable: apt_mirror_server_create_export_tarball
Question: Create a portable tarball of the mirror for offline distribution?
Answer Type: Multiple Choice
Required: No
Default: false
Choices:
  - label: "No - Mirror only"
    value: false
  - label: "Yes - Create tarball"
    value: true
```

## Example Playbooks

### Docker CE Mirror (Quick POC)

```yaml
---
- name: Configure Docker CE mirror server
  hosts: mirror_servers
  become: true
  
  roles:
    - role: dperr.linux.config_ubuntu_server
      vars:
        apt_mirror_server_profile: docker_ce
        apt_mirror_server_upstream: https://download.docker.com/linux/ubuntu
        apt_mirror_server_codenames:
          - noble
        apt_mirror_server_architectures:
          - amd64
        apt_mirror_server_run_initial_sync: true  # Small mirror, quick sync
```

### Ubuntu Full Mirror (Production)

```yaml
---
- name: Configure Ubuntu OS mirror server
  hosts: mirror_servers
  become: true
  
  roles:
    - role: dperr.linux.config_ubuntu_server
      vars:
        apt_mirror_server_profile: ubuntu
        apt_mirror_server_upstream: http://us.archive.ubuntu.com/ubuntu
        apt_mirror_server_codenames:
          - jammy
          - noble
        apt_mirror_server_components: main restricted universe multiverse
        apt_mirror_server_architectures:
          - amd64
        apt_mirror_server_cron_hour: "2"
        apt_mirror_server_cron_minute: "0"
        apt_mirror_server_run_initial_sync: false  # Large mirror, let cron handle it
```

### Offline/Air-Gap Export

```yaml
---
- name: Mirror and export for offline transport
  hosts: internet_connected_server
  become: true
  
  roles:
    - role: dperr.linux.config_ubuntu_server
      vars:
        apt_mirror_server_profile: docker_ce
        apt_mirror_server_upstream: https://download.docker.com/linux/ubuntu
        apt_mirror_server_codenames:
          - noble
        apt_mirror_server_architectures:
          - amd64
        apt_mirror_server_run_initial_sync: true
        apt_mirror_server_create_export_tarball: true
        apt_mirror_server_export_format: xz  # Better compression for transport
```

## How It Works

1. **Validation**: 
   - Verifies target is Ubuntu distribution
   - Validates mirror profile is `ubuntu` or `docker_ce`
   - Validates upstream URL includes repository path
   
2. **Package Installation**: Installs `apt-mirror` and `nginx`

3. **Mirror Configuration**:
   - Generates `/etc/apt/mirror.list` from template
   - Configures mirror to sync from upstream URL
   - Sets up storage paths under `/var/spool/apt-mirror`

4. **Nginx Configuration**:
   - Creates nginx site configuration for serving mirror content
   - Maps URL paths to match apt-mirror storage layout
   - Enables site and disables default nginx site
   - Reloads nginx to apply changes

5. **Scheduled Sync**: 
   - Creates cron job for automated syncing (default: 2:30 AM daily)
   - Logs sync output to `/var/log/apt-mirror/cron.log`

6. **Optional Initial Sync**: 
   - Runs `apt-mirror` immediately if `apt_mirror_server_run_initial_sync: true`
   - **WARNING**: Can take hours (Docker CE) to days (full Ubuntu)

7. **Optional Export**:
   - Creates compressed tarball of mirror contents
   - Useful for offline/air-gapped deployments

## Post-Installation

### Verify Mirror is Running

```bash
# Check nginx is serving mirror
curl -I http://localhost/linux/ubuntu/dists/noble/Release  # Docker CE
curl -I http://localhost/ubuntu/dists/noble/Release        # Ubuntu

# Check mirror storage
ls -lh /var/spool/apt-mirror/mirror/

# Check cron job
sudo crontab -l | grep apt-mirror

# Check nginx site
sudo nginx -t
sudo systemctl status nginx
```

### Monitor Sync Progress

```bash
# Watch cron log during scheduled sync
sudo tail -f /var/log/apt-mirror/cron.log

# Check disk usage (mirrors grow over time)
df -h /var/spool/apt-mirror
```

### Manual Sync

```bash
# Run sync manually
sudo /usr/bin/apt-mirror

# Sync and log output
sudo /usr/bin/apt-mirror | tee /tmp/mirror-sync.log
```

## Handlers

- `Reload NGINX server` - Tests nginx configuration and reloads service when mirror config changes

## Disk Space Recommendations

| Mirror Type | Codenames | Architectures | Estimated Size |
|-------------|-----------|---------------|----------------|
| Docker CE | 1 (noble) | amd64 | 5-10 GB |
| Docker CE | 3 (focal, jammy, noble) | amd64 | 15-25 GB |
| Ubuntu OS | 1 (noble) | amd64 | 50-100 GB |
| Ubuntu OS | 1 (noble) | amd64 + arm64 | 100-200 GB |
| Ubuntu OS | 3 (focal, jammy, noble) | amd64 | 150-300 GB |
| Ubuntu Full | All supported | All architectures | Multi-TB |

**Note**: Sizes are estimates and grow over time as new packages are published.

## Important Notes

- **Initial Sync Time**: Docker CE mirror (~30-60 min), Ubuntu OS mirror (hours to days)
- **Bandwidth**: Initial sync downloads large amounts of data; schedule during off-hours
- **Disk Growth**: Mirrors grow over time as updates are published
- **Cron Logging**: Sync logs append to `/var/log/apt-mirror/cron.log` (consider log rotation)
- **Nginx Port**: Default listens on port 80; adjust `apt_mirror_server_nginx_listen` for custom ports
- **Client Configuration**: After mirror is synced, use `config_ubuntu_mirrors` role to configure clients

## Troubleshooting

### Mirror sync fails

Check network connectivity to upstream:
```bash
curl -I https://download.docker.com/linux/ubuntu/dists/noble/Release
```

### Nginx serving 404 errors

Verify mirror storage layout matches nginx alias:
```bash
ls -la /var/spool/apt-mirror/mirror/download.docker.com/linux/ubuntu/
```

### Disk full during sync

Monitor disk usage and expand storage:
```bash
df -h /var/spool/apt-mirror
```

### Export tarball creation fails

Ensure `community.general` collection is installed:
```bash
ansible-galaxy collection install community.general
```

## Integration with Client Role

After configuring the mirror server, point clients to it using the `config_ubuntu_mirrors` role:

```yaml
# On mirror server
- hosts: mirror_server
  roles:
    - role: dperr.linux.config_ubuntu_server
      vars:
        apt_mirror_server_profile: docker_ce
        apt_mirror_server_run_initial_sync: true

# On client systems
- hosts: ubuntu_clients
  roles:
    - role: dperr.linux.config_ubuntu_mirrors
      vars:
        mirror_server_hostname: "{{ groups['mirror_server'][0] }}"
        config_client_mirrors_profiles:
          - docker_ce
```

## License

MIT

## Author

Dennis Perrone
