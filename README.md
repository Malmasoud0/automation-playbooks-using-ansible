# System Provisioning and Monitoring Stack with Ansible

**Last Updated:** July 20, 2026

This repository contains a modular and highly automated Ansible provisioning setup designed to deploy, configure, and monitor a robust open-source server stack on **Ubuntu 24.04 LTS** (fully compatible with WSL 2 and standalone environments).

---

## 🏗️ Architecture Stack

The setup configures a self-contained local loop monitoring and database stack:
1.  **PostgreSQL 16**: High-performance relational database with secure peer-to-peer authentication, pre-configured with query statement-level metrics tracing.
2.  **Elasticsearch**: High-performance log and metrics search engine with basic authentication enabled.
3.  **Kibana**: Beautiful administration interface for querying, visualizing, and managing system and database metrics.
4.  **Metricbeat**: Lightweight system monitor collecting server metrics (CPU, RAM, storage, network) and PostgreSQL transaction/query analytics, piping them directly to Elasticsearch and loading dashboards into Kibana.

---

## 📂 Repository Directory Structure

```text
/opt/malmasoud/
├── ansible.cfg            # Global Ansible settings (fixes privilege escalation tmp permissions)
├── setup.yml              # Main orchestrator provisioning playbook
├── uninstall.yml          # Main orchestrator uninstallation playbook
├── secrets.vault          # Ansible Vault-encrypted credentials file
├── .vault_pass            # Plaintext password file to decrypt secrets.vault on-the-fly
└── roles/
    ├── install_ansible/        # Configures / Uninstalls Ansible PPA and dependencies
    ├── install_elasticsearch/  # Provisions / Purges Elasticsearch engine and heap configs
    ├── install_kibana/         # Deploys / Purges the Kibana web dashboard
    ├── install_metricbeat/     # Deploys / Purges system & PostgreSQL monitoring agents
    └── install_postgresql_16/  # Installs / Purges PostgreSQL server, users, and stats extension
```

---

## 🔐 Secrets and Vault Management

All sensitive details (passwords and API keys) are secured using **Ansible Vault** inside [secrets.vault](file:///opt/malmasoud/secrets.vault). 

### ⚙️ How Vault Decryption Works
1.  The main playbook [setup.yml](file:///opt/malmasoud/setup.yml) imports [secrets.vault](file:///opt/malmasoud/secrets.vault) automatically:
    ```yaml
    vars_files:
      - secrets.vault
    ```
2.  Role defaults dynamically reference the vault variables instead of storing plaintext keys. For example:
    *   **Elasticsearch password**: `{{ vault_elasticsearch_admin_password }}`
    *   **PostgreSQL password**: `{{ vault_postgresql_database_password }}`
3.  Ansible reads the decryption key (`myLean133`) automatically from [`.vault_pass`](file:///opt/malmasoud/.vault_pass) during playbook runs.

### 📝 Editing or Viewing Secrets
To edit or view the secrets in plaintext, run:
```bash
ansible-vault edit secrets.vault --vault-password-file .vault_pass
```

---

## 🚀 How to Execute the Playbook

Ensure you are in the directory `/opt/malmasoud/` before running any commands.

### 1️⃣ Run the Complete Stack Provisioning
To run all roles (Elasticsearch, Kibana, Metricbeat, PostgreSQL, and base setups) in a single command:
```bash
ansible-playbook setup.yml --vault-password-file .vault_pass
```

### 2️⃣ Run Specific Stack Components (Using Tags)
You can target individual components using their respective Ansible tags:

*   **Only Provision PostgreSQL Database**:
    ```bash
    ansible-playbook setup.yml --vault-password-file .vault_pass --tags "postgresql"
    ```
*   **Only Update Metricbeat Configurations & Dashboards**:
    ```bash
    ansible-playbook setup.yml --vault-password-file .vault_pass --tags "metricbeat"
    ```
*   **Only Manage Elasticsearch**:
    ```bash
    ansible-playbook setup.yml --vault-password-file .vault_pass --tags "elasticsearch"
    ```
*   **Only Manage Kibana**:
    ```bash
    ansible-playbook setup.yml --vault-password-file .vault_pass --tags "kibana"
    ```

### 3️⃣ Run the Complete Stack Uninstallation
To completely remove all installed packages, GPG keys, system repositories, configuration files, logs, and database directories, run the uninstallation orchestrator:
```bash
ansible-playbook uninstall.yml --vault-password-file .vault_pass
```

You can also target specific components for teardown using tags (e.g., just uninstalling Metricbeat):
```bash
ansible-playbook uninstall.yml --vault-password-file .vault_pass --tags "metricbeat"
```

---

## 📊 PostgreSQL Query Tracing & Analytics

To enable Metricbeat's `statement` analyzer for monitoring individual query durations, execution counts, and background writer metrics:
1.  The database provisioning task templates out and applies a `CREATE EXTENSION IF NOT EXISTS pg_stat_statements;` instruction.
2.  The server's `postgresql.conf` file is automatically configured with:
    ```ini
    shared_preload_libraries = 'pg_stat_statements'
    ```
3.  **Monitoring Security**: Metricbeat is configured to connect to PostgreSQL as the user `ahmed` (rather than the root `postgres` superuser). To allow this, the playbook automatically runs a SQL command granting him monitoring reader access:
    ```sql
    GRANT pg_read_all_stats TO ahmed;
    ```

---

## 🛠️ Troubleshooting & Local System Configuration

This workspace contains automated optimizations to handle local WSL permission limits:
*   **Unprivileged Privileges (`ansible.cfg`)**: Setting `allow_world_readable_tmpfiles = True` prevents Ansible from failing during unprivileged systemd/postgres operations when temporary files are generated.
*   **Python interpreter**: The main playbook targets `/usr/bin/python3` directly to ensure non-root users (like the `postgres` system account) can access and execute library modules safely outside local virtual environments.
