# Microsoft SQL Server Automation Playbooks

Installs Microsoft SQL Server via `microsoft.sql.server` and creates a database with `community.general.mssql_db`.

## Entry points

| How | What runs |
|-----|------------|
| **`install_app.yml`** with `app_role: mssql` | Full stack (install SQL Server + create DB), same as standalone playbook |
| **`playbooks/linux/mssql.yml`** | Same full stack without going through `install_app` |
| **`tasks_from: install.yml`** or **`create_db.yml`** | Partial runs only (install-only or DB-only) |

The role **`tasks/main.yml`** gathers facts (needed when the play uses `gather_facts: false`), then runs **`install.yml`** and **`create_db.yml`** unless toggled off:

| Variable | Default | Purpose |
|----------|---------|---------|
| `mssql_install_sql_server` | `true` | Run SQL Server installation |
| `mssql_create_database` | `true` | Run database creation after install |

Set either to `false` in extra-vars when you need only one phase while still using `main.yml`.

## Secrets

Do **not** commit plaintext passwords. Supply **`mssql_sa_password`** (or legacy **`mssql_pass`**) via vault, inventory, AAP survey/credential, or uncomment **`vars_files`** in **`playbooks/linux/mssql.yml`** / **`install_app.yml`** pointing at `vars/mssql.vault.yml`. See **`vars/mssql.vault.yml.example`**.

## Variables (see `defaults/main.yml`)

- **Install:** `mssql_version`, `mssql_edition`, EULA toggles  
- **Database:** `mssql_host`, `mssql_user`, `mssql_db_name` — plus aliases **`database_name`** / **`db_name`** for the DB name  

## Execution environment

Collections: **`microsoft.sql`**, **`community.general`** (see **`collections/requirements.yml`**).
