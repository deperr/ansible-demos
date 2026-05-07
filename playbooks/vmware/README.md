# VMware Automation Playbooks

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation & Setup](#installation--setup)
- [Playbook Documentation](#playbook-documentation)
- [Usage Examples](#usage-examples)
- [Security Considerations](#security-considerations)


## Overview

This repository contains production-ready Ansible playbooks for VMware vSphere automation, including:

- **VM Provisioning**: Automated Linux VM deployment with customizable hardware specifications
- **Snapshot Management**: Intelligent snapshot cleanup with age-based filtering
- **CD-ROM Management**: Automated CD-ROM device cleanup and ejection
- **Infrastructure Maintenance**: Comprehensive VM lifecycle management

## Prerequisites

### System Requirements

- **Ansible**: Version 2.9+ (tested with 2.15+)
- **Python**: Version 3.8+
- **VMware Collections**: 
  - `vmware.vmware_ops`
  - `vmware.vmware_rest`
  - `community.vmware`

### VMware Environment

- **vCenter Server**: 6.7+ (tested with 7.0+)
- **ESXi Hosts**: 6.7+ (tested with 7.0+)
- **Network Access**: Ansible control node must reach vCenter and ESXi hosts
- **Credentials**: vCenter administrator account with appropriate permissions

### Required Permissions

| Permission | Purpose |
|------------|---------|
| VirtualMachine.Config.AddRemoveDevice | VM provisioning and device management |
| VirtualMachine.State.CreateSnapshot | Snapshot operations |
| VirtualMachine.State.RemoveSnapshot | Snapshot cleanup |
| VirtualMachine.Inventory.Create | VM creation |
| VirtualMachine.Inventory.Delete | VM deletion |
| Datastore.Browse | Datastore access for VM provisioning |
| Network.Assign | Network configuration |

## AAP 2.5 Setup & Configuration

### 1. Execution Environment Setup

**In AAP 2.5 UI:**
1. Navigate to **Administration** → **Execution Environments**
2. Click **Add** to create new execution environment
3. Configure with the following settings:
   - **Name**: `VMWare EE`
   - **Image**: `quay.io/dperrone/vmware_ee:latest`
   - **Pull**: Always pull container before running
   - **Organization**: Select your organization

### 2. Credential Configuration

**Create VMware vCenter Credential:**
1. Navigate to **Resources** → **Credentials**
2. Click **Add** → **Machine Credential**
3. Configure credential with:
   - **Name**: `vSphere Machine`
   - **Credential Type**: Machine
   - **Username**: `administrator@vsphere.local`
   - **Password**: Your vCenter password
   - **Organization**: Select your organization

**Create Ansible Vault Credential:**
1. Navigate to **Resources** → **Credentials**
2. Click **Add** → **Vault Credential**
3. Configure with:
   - **Name**: `vSphere Vault`
   - **Vault Password**: Your Ansible Vault password
   - **Organization**: Select your organization

### 3. Project Setup

**Create Project:**
1. Navigate to **Resources** → **Projects**
2. Click **Add** to create new project
3. Configure project settings:
   - **Name**: `VMWare Automation Project`
   - **Organization**: Select your organization
   - **Source Control Type**: Git
   - **Source Control URL**: Your repository URL
   - **Branch/Tag/Commit**: `main`
   - **Execution Environment**: `VMWare EE`

## Playbook Documentation

### 1. VM Provisioning (`vmware_provision_vm.yml`)

**Purpose**: Automates the creation and configuration of Linux VMs in VMware vSphere.

**Key Features**:
- Customizable CPU and memory allocation
- Hot-add capability for CPU and memory
- Thin-provisioned disk configuration
- Automatic power-on after provisioning

**AAP Job Template Configuration**:
1. Navigate to **Resources** → **Templates**
2. Click **Add** → **Add job template**
3. Configure template:
   - **Name**: `VMware VM Provisioning`
   - **Job Type**: Run
   - **Inventory**: Select your inventory
   - **Project**: `VMWare Automation Project`
   - **Playbook**: `vmware_provision_vm.yml`
   - **Execution Environment**: `VMWare EE`
   - **Credentials**: 
     - `vSphere Machine`
     - `vSphere Vault`
   - **Variables**:
     ```yaml
     vm_name: "dp_new-vm"
     vm_cpus: 2
     memsize_MiB: 4096
     ```

**Survey Configuration**:
1. In job template, go to **Survey** tab
2. Add survey questions:
    - **Question 1**
        - **Question**: `VM Name`
        - **Answer Variable Name**: `vm_name`
        - **Answer Type**: Text
        - **Default**: `dp_rhel9-test`
        - **Required**: Yes
    - **Question 2**
        - **Question**: `CPU Count`
        - **Answer Variable Name**: `vm_cpus`
        - **Answer Type**: Integer
        - **Default**: `2`
        - **Min Value**: `1`
        - **Max Value**: `32`
        - **Required**: Yes
   - **Question 3**
        - **Question**: `Memory (MiB)`
        - **Answer Variable Name**: `memsize_MiB`
        - **Answer Type**: Integer
        - **Default**: `4096`
        - **Min Value**: `1024`
        - **Max Value**: `131072`
        - **Required**: Yes


### 2. Snapshot Cleanup (`vmware_remove_snapshots.yml`)

**Purpose**: Removes old VM snapshots based on configurable age thresholds.

**Key Features**:
- Age-based snapshot filtering (1-365 days)
- Comprehensive validation and error handling
- Detailed logging of cleanup operations
- Safe operation with extensive checks

**AAP Job Template Configuration**:
1. Navigate to **Resources** → **Templates**
2. Click **Add** → **Add job template**
3. Configure template:
   - **Name**: `VMware Snapshot Cleanup`
   - **Job Type**: Run
   - **Inventory**: Select your inventory
   - **Project**: `VMWare Automation Project`
   - **Playbook**: `vmware_remove_snapshots.yml`
   - **Execution Environment**: `VMWare EE`
   - **Credentials**: 
     - `vSphere Machine`
     - `vSphere Vault`
   - **Variables**:
     ```yaml
     snapshot_age: 7
     datacenter: "SDDC-Datacenter"
     ```

**Survey Configuration** (Optional):
1. In job template, go to **Survey** tab
2. Add survey question:
   - **Question**: `Snapshot Age Threshold (days)`
   - **Answer Variable Name**: `snapshot_age`
   - **Answer Type**: Integer
   - **Default**: `7`
   - **Min Value**: `1`
   - **Max Value**: `365`

### 3. Advanced Snapshot Cleanup (`vmware_remove_all_snapshots.yml`)

**Purpose**: Enhanced snapshot cleanup with improved error handling and validation.

**Key Features**:
- Input validation for snapshot age parameters
- Comprehensive error reporting
- Detailed execution logging
- Safe operation with extensive validation

**AAP Job Template Configuration**:
1. Navigate to **Resources** → **Templates**
2. Click **Add** → **Add job template**
3. Configure template:
   - **Name**: `VMware Advanced Snapshot Cleanup`
   - **Job Type**: Run
   - **Inventory**: Select your inventory
   - **Project**: `VMWare Automation Project`
   - **Playbook**: `vmware_remove_all_snapshots.yml`
   - **Execution Environment**: `VMWare EE`
   - **Credentials**: 
     - `vSphere Machine`
     - `vSphere Vault`
   - **Variables**:
     ```yaml
     datacenter: "SDDC-Datacenter"
     ```

### 4. CD-ROM Management (`vmware_remove_cd_drive.yml`)

**Purpose**: Comprehensive CD-ROM device cleanup and ejection for VMware guests.

**Key Features**:
- Automatic process detection and termination
- Safe unmounting with fallback options
- CD-ROM media ejection
- Comprehensive status reporting
- Multi-VM support

**AAP Job Template Configuration**:
1. Navigate to **Resources** → **Templates**
2. Click **Add** → **Add job template**
3. Configure template:
   - **Name**: `VMware CD-ROM Cleanup`
   - **Job Type**: Run
   - **Inventory**: Select your inventory (with target VMs)
   - **Project**: `VMWare Automation Project`
   - **Playbook**: `vmware_remove_cd_drive.yml`
   - **Execution Environment**: `VMWare EE`
   - **Credentials**: 
     - `vSphere Machine`
     - `vSphere Vault`
   - **Limit**: Leave empty to target all hosts in inventory

**Survey Configuration**:
1. In job template, go to **Survey** tab
2. Add survey question:
   - **Question**: `Target Hosts (comma-separated, leave empty for all)`
   - **Answer Variable Name**: `_hosts`
   - **Answer Type**: Text
   - **Default**: (empty)
   - **Required**: No

## AAP 2.5 Usage Examples

### 1. Running Individual Job Templates

**VM Provisioning:**
1. Navigate to **Resources** → **Templates**
2. Find `VMware VM Provisioning` template
3. Click **Launch** button
4. Fill in survey parameters:
   - VM Name: `production-web-01`
   - CPU Count: `4`
   - Memory (MiB): `8192`
5. Click **Next** → **Launch**

**Snapshot Cleanup:**
1. Navigate to **Resources** → **Templates**
2. Find `VMware Snapshot Cleanup` template
3. Click **Launch** button
4. Set snapshot age threshold (default: 7 days)
5. Click **Next** → **Launch**

### 2. Creating Workflow Templates

**VMware Maintenance Workflow:**
1. Navigate to **Resources** → **Templates**
2. Click **Add** → **Add workflow template**
3. Configure workflow:
   - **Name**: `VMware Maintenance Workflow`
   - **Organization**: Select your organization
4. Design workflow nodes:
   - **Node 1**: `VMware Snapshot Cleanup` (Job Template)
   - **Node 2**: `VMware CD-ROM Cleanup` (Job Template)
   - **Node 3**: `VMware Advanced Snapshot Cleanup` (Job Template)
5. Configure node connections and success/failure paths

### 3. Scheduling Automated Jobs

**Create Schedule for Snapshot Cleanup:**
1. Navigate to **Resources** → **Templates**
2. Select `VMware Snapshot Cleanup` template
3. Go to **Schedules** tab
4. Click **Add** to create new schedule:
   - **Name**: `Weekly Snapshot Cleanup`
   - **Schedule**: `0 2 * * 0` (Every Sunday at 2 AM)
   - **Enabled**: Yes
5. Save schedule

### 4. Inventory Management

**Create VMware Inventory:**
1. Navigate to **Resources** → **Inventories**
2. Click **Add** → **Add inventory**
3. Configure inventory:
   - **Name**: `VMware VMs`
   - **Organization**: Select your organization
4. Add hosts manually or use dynamic inventory:
   - **Host Name**: `web-server-01`
   - **Variables**:
     ```yaml
     ansible_host: 192.168.1.100
     ansible_user: root
     vm_name: web-server-01
     ```

## Security Considerations

### AAP 2.5 Credential Management

**Credential Types:**
- **vSphere Credentials**: For vCenter authentication
- **vSphere Machine Credential**: For SSH access to VM's
- **Vault Credentials**: For Ansible Vault password management

**Best Practices:**
1. **Dedicated Service Accounts**: Use dedicated VMware service accounts
   - Username: `ansible-automation@vsphere.local`
   - Minimal required permissions only
2. **Credential Rotation**: Regularly rotate service account passwords
3. **Organization Isolation**: Use organization-specific credentials
4. **Access Control**: Limit credential access to authorized users/teams


**⚠️ Important**: Always test playbooks in a development environment before running in production. Ensure you have proper backups and rollback procedures in place.