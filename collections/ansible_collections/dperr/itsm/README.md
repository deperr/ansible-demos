# Role: itsm

The role intentionally has **no default action**. Always name one with `tasks_from:`, so that grepping
for `tasks_from:` finds every place a ticket is touched.

## Actions

### `open`: Create or adopt the ticket

```yaml
- name: Open the ticket
  ansible.builtin.include_role:
    name: snow_ticket
    tasks_from: open
  vars:
    snow_ticket_short_description: "Provision & patch EC2 instance: web-demo-01"
    snow_ticket_description: "…"
    snow_ticket_opening_note: "Request received."   # optional
```

Sets the `snow_ticket_sys_id` and `snow_ticket_number` as facts.

If `snow_sys_id` is supplied and non-empty, the role **adopts** that ticket instead of
creating one, looks up its number, and asserts it exists. Nothing supplies `snow_sys_id`
during the workflow.

### `note`: Append a work note

```yaml
- name: Say what we're doing
  ansible.builtin.include_role:
    name: snow_ticket
    tasks_from: note
  vars:
    snow_ticket_note: "Instance reachable. Applying all available updates…"
```

Takes the ticket from `snow_ticket_sys_id` (same play) or `snow_sys_id` (workflow
artifact), in that order.

### `resolve`: Close the ticket

```yaml
- ansible.builtin.include_role:
    name: snow_ticket
    tasks_from: resolve
  vars:
    snow_ticket_close_notes: "…"
```

ServiceNow rejects a move to resolved without **both** `close_code` and `close_notes`,
returning a generic HTTP 400. `snow_ticket_close_code` defaults to
`Solved (Permanently)`; adopt this to the list on the specific instance.

`snow_ticket_incident_mapping` supplies an identity map of modern close codes, because
the collection ships an older list and otherwise logs
`Encountered unknown value ... while mapping field close_code` on every resolve.

### `escalate`: Hand the ticket to a human

```yaml
- ansible.builtin.include_role:
    name: snow_ticket
    tasks_from: escalate
  vars:
    snow_ticket_note: "…"
```

Raises urgency and impact and posts the note. It deliberately **does not change state**. The ticket must stay open during the escalation because the work did not finish.

`snow_ticket_assignment_group` is only sent when non-empty: an empty value would clear
whatever the instance had assigned.

## Variables

| Variable | Default | Purpose |
|---|---|---|
| `snow_ticket_caller` | `admin` | Ticket caller. Override with a real PDI user. |
| `snow_ticket_impact` | `low` | Creation impact. |
| `snow_ticket_urgency` | `low` | Creation urgency. |
| `snow_ticket_delegate_host` | `localhost` | Where API calls execute. |
| `snow_sys_id` | `""` | Adopt this ticket instead of creating one. |

## Why every task is `delegate_to: localhost`

Once plays start running against
the EC2 instance, an undelegated `servicenow.itsm` task executes *on that instance*, so
the API call egresses from AWS and needs the collection in the instance's Python
environment. It fails in a confusing way. Delegating inside the role means no caller can
forget.

`run_once: true` is paired with it so a play against N hosts still writes one note, not N.

`become: false` is set for the same reason. localhost here is the execution environment
container, which has no `sudo`, and a REST call needs no privilege. A caller that sets
`become: true` at play level, as a patching play naturally does, would otherwise fail
these tasks with `sudo: command not found`.

## Authentication

The modules read `SN_HOST`, `SN_USERNAME`, `SN_PASSWORD` from the environment. In AAP these come from the ServiceNow credential. If you're running this locally, export them before running. No `instance:` block is needed anywhere.
