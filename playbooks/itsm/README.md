# ServiceNow (ITSM) Playbooks

Two sets of examples: standalone plays that create, update and close ServiceNow
records, and a six-node **provision and patch** workflow that wraps an EC2 build in a
single incident.

Most of these are meant to run as workflow nodes in Ansible Automation Platform rather
than on their own. Where a playbook can be run standalone, it says so in its assertion
failure message.


## Standalone ITSM examples

| Playbook | What it does |
|---|---|
| [snow_create_change_request.yml](snow_create_change_request.yml) | Create a change request |
| [snow_create_incident.yml](snow_create_incident.yml) | Create an incident |
| [snow_create_problem.yml](snow_create_problem.yml) | Create a problem record |
| [snow_update_incident.yml](snow_update_incident.yml) | Update an existing incident |
| [snow_close_incident.yml](snow_close_incident.yml) | Close an incident |

## Provision and patch workflow

Build a RHEL 9 EC2 instance, patch it, reboot it and close the ticket — with every step
written back to the same incident as a work note.

| # | Playbook | What it does |
|---|---|---|
| 1 | [snow_pp_open_ticket.yml](snow_pp_open_ticket.yml) | Opens (or adopts) the incident for the request |
| 2 | [snow_pp_provision_ec2.yml](snow_pp_provision_ec2.yml) | Resolves the subnet and AMI, creates the instance, tags it with the incident number |
| 3 | [snow_pp_patch_ec2.yml](snow_pp_patch_ec2.yml) | Adds the instance to an in-memory inventory, applies all `dnf` updates, checks whether a reboot is needed |
| 4 | [snow_pp_reboot_and_verify.yml](snow_pp_reboot_and_verify.yml) | Reboots if required, then confirms systemd is running and records the kernel |
| 5 | [snow_pp_close_ticket.yml](snow_pp_close_ticket.yml) | Resolves the incident with a summary |
| 6 | [snow_pp_teardown.yml](snow_pp_teardown.yml) | Terminates the demo instance |

Teardown is deliberately outside the chain — run it as its own job template when you're
done with the demo, not as a workflow node.

### How the nodes talk to each other

Each node publishes `set_stats` artifacts that the next one reads. Nothing is passed
through inventory, so the workflow works with an empty inventory.

| Published by | Artifacts |
|---|---|
| Open ticket | `snow_sys_id`, `snow_number` |
| Provision | `ec2_instance_id`, `ec2_private_ip`, `ec2_public_ip`, `ec2_az`, `ec2_ami`, `ci_payload` |
| Patch | `patch_count`, `reboot_required`, `patch_seconds` |
| Reboot and verify | `verify_kernel`, `verify_systemd`, `verify_rebooted` |

Every node asserts on its inputs up front, so a missing artifact fails with a message
naming the variable instead of a Jinja undefined error halfway through.

The patch and reboot plays prefer `ec2_public_ip`, on the assumption that the AAP
execution node sits outside the demo VPC. Set `ec2_connect_ip` to override.

### Variables

Defaults live in [vars/snow_ec2.yml](../../vars/snow_ec2.yml) and are loaded by every
`snow_pp_*` playbook. The names match the APD **Cloud | AWS | Create VM** survey, so
that survey drives these templates unchanged. They're play-level vars, so survey
answers and `-e` both override them.

Required for node 1: `create_vm_vm_name`, `create_vm_aws_region`, `vm_blueprint`.

Two optional shortcuts skip a lookup: `create_vm_aws_instance_ami` pins the image, and
`create_vm_aws_subnet_id` skips the subnet-by-name search.


### Demo limitations

- The blueprint check rejects anything containing `windows`; the patch play is `dnf`-only.
- Tickets are created low impact / low urgency so they don't trip notification rules on
  a shared instance.
- `ci_payload` is published but nothing consumes it yet — it's there for a future CMDB
  update node.
