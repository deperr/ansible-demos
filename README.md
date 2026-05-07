# Ansible Playbooks

Ansible playbooks and supporting collection content for Linux, Windows, VMware vSphere, ServiceNow ITSM, and compliance-style automation. The examples are structured for reuse from the CLI or from **Ansible Automation Platform (AAP)** job templates and surveys.

## Contents at a glance

| Area | Location | Examples |
|------|----------|----------|
| **Linux** | [`playbooks/linux/`](playbooks/linux/) | Package and app installs, patching, troubleshooting, MSSQL-related tasks, RHEL STIG-oriented plays, vault usage |
| **Windows** | [`playbooks/windows/`](playbooks/windows/) | Connection tests, PowerShell and DSC, patching, user management, Windows STIG-oriented playbook |
| **VMware** | [`playbooks/vmware/`](playbooks/vmware/) | VM provisioning, snapshot cleanup, external device cleanup |
| **ITSM (ServiceNow)** | [`playbooks/itsm/`](playbooks/itsm/) | Create/update/close incidents, problems, change requests |
| **Collections (local)** | [`collections/ansible_collections/dperr/`](collections/ansible_collections/dperr/) | Roles and plugins grouped under `apps`, `common`, `compliance`, and `windows` |

Detailed VMware setup (execution environments, credentials, surveys, and job templates) lives in [`playbooks/vmware/README.md`](playbooks/vmware/README.md).

## Prerequisites

- **ansible-core** — use a supported release for your target modules (many playbooks align with **2.14+** patterns).
- **Collections** — install whatever your chosen playbook requires (for example `servicenow.itsm`, `vmware.vmware_rest`, `ansible.windows`, `community.general`). Declare them in `collections/requirements.yml` and install with:

  ```bash
  ansible-galaxy collection install -r collections/requirements.yml
  ```

  The repo’s [`collections/requirements.yml`](collections/requirements.yml) is currently empty; extend it as you pin dependencies for your environment.

- **Credentials** — provide vault passwords, cloud/API credentials, or AAP credentials via extra vars, inventory, or Tower/AAP credential types; do not commit secrets.

## Quick start

From the repository root:

```bash
# Optional: install declared Galaxy collections
ansible-galaxy collection install -r collections/requirements.yml

# Example: run a playbook (adjust inventory and limits)
ansible-playbook -i inventory playbooks/linux/install_package.yml \
  -e 'selected_packages=["curl","vim"]'
```

Many playbooks accept survey-oriented variables (for example `_hosts`, `selected_packages`, ServiceNow fields). Inspect the playbook header and tasks for the variables each example expects.

## Local collections (`dperr.*`)

Under [`collections/ansible_collections/dperr/`](collections/ansible_collections/dperr/) you’ll find in-repo collection layouts used by these demos—for example application roles (Kafka, NGINX, Apache, OPA, MSSQL), a shared `run` role, compliance-oriented STIG-related roles, and Windows helpers including custom plugins.

To use a local collection path without publishing to Galaxy, configure **collection paths** (for example `collections_paths` in `ansible.cfg`) or install from the filesystem per [Ansible’s collection documentation](https://docs.ansible.com/ansible/latest/user_guide/collections_using.html).

## Continuous integration

Pull requests targeting `main` run **ansible-lint** via the reusable workflow in [`.github/workflows/tests.yml`](.github/workflows/tests.yml) ([ansible/ansible-content-actions](https://github.com/ansible/ansible-content-actions)).

Validate locally before pushing:

```bash
ansible-playbook --syntax-check playbooks/linux/install_package.yml
ansible-lint
```

## Security notes

- Treat this repository as **sample automation**: review modules, privilege escalation (`become`), and network scope before running against production.
- Use **Ansible Vault** or your platform’s secret store for passwords and API keys.
- Follow least-privilege accounts for VMware, ServiceNow, and OS access.
