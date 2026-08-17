# GLOSSARY

Terms used throughout the vedicon project.

---

### CODENAME

A short name that identifies a Proxmox infrastructure (a physical server or cluster).
Used in directory names, SSH config, key names.

Examples: `hv-lab-01`, `lab-proxmox-01`, `proxmox-home`

One CODENAME = one Proxmox server. You can have multiple CODENAMEs if you manage
multiple Proxmox servers.

### SCENARIO

A lab environment deployed on a CODENAME infrastructure.
Each scenario defines which VMs to create, how to configure them, and what
services to run.

Examples: `demo_lab`, `ctf_game_01`, `misp_lab`

One CODENAME can host multiple scenarios. Each combination CODENAME-SCENARIO
is an independent workspace with its own credentials, config, and secrets.

### CODENAME-SCENARIO

The fundamental unit of vedicon. Everything is organized by this pair:

```
~/vedicon.config/hv-lab-01-demo_lab/          — workspace
~/.ssh/vedicon/hv-lab-01-demo_lab/            — SSH keys
~/.ssh/config_vedicon-hv-lab-01-demo_lab      — SSH config
```

### Workspace

A directory at `~/vedicon.config/CODENAME-SCENARIO/` on the deployer-cli.
Contains everything specific to one deployment:

```
~/vedicon.config/CODENAME-SCENARIO/
├── inventory/          — Ansible inventory for this lab
├── ssh_keys/           — backup of all SSH keys
├── secrets/            — encrypted vault + vault password
├── bin/                — utility scripts
├── sourced_vedicon.sh  — environment variables
└── scenario            — symlink to the scenario vedicon_playbook
```

### Deployer-CLI

The machine where you run Ansible vedicon_playbook and manage the lab.
Can be your laptop, a dedicated VM, or any Linux machine with SSH access
to the Proxmox server.

The deployer-cli holds:
- `~/vedicon/` — git repos (shared across all workspaces)
- `~/vedicon.config/` — workspaces (one per CODENAME-SCENARIO)
- `~/.ssh/vedicon/` — SSH keys (one set per CODENAME-SCENARIO)

### Jump host

The lab VMs run on a private network (e.g. 192.168.42.x). They are not
directly reachable from outside. To reach them, SSH bounces through a
"jump host" — usually the Proxmox server itself:

```
[your machine] --SSH--> [Proxmox / jump_user] --SSH--> [VM: 192.168.42.x]
```

This happens automatically via `ProxyJump` in the SSH config.
You just type `ssh r42.admin-wazuh` and SSH handles the bounce.

### Vault

An Ansible vault file (`default_vault.yml`) that stores all secrets for a
CODENAME-SCENARIO: API tokens, passwords, SSH public keys, Tailscale keys.

The file is encrypted with `ansible-vault`. The password to decrypt it is in
`vault_pass.txt` in the same directory (intentional by design — the deployer-cli
is a trusted machine).

### Context

The currently active workspace on the deployer-cli. Only one context is active
at a time. Switching context changes:
- which SSH config is active (which VMs you can reach)
- which environment variables are loaded (which inventory, which paths)
- which vault password file is used
- the zsh prompt shows the active context in green

Switch with: `vedicon-context use <codename> <scenario>`

### vedicon-context

A zsh function available on the deployer-cli. Central tool for managing workspaces.

**Workspace:**

| Command | What it does |
|---------|-------------|
| `vedicon-context list` | list all available workspaces |
| `vedicon-context current` | show the active workspace |
| `vedicon-context use <codename> <scenario>` | switch to a workspace |
| `vedicon-context status` | health check (vault, SSH keys, inventory, symlinks) |
| `vedicon-context init` | launch the setup wizard from anywhere |

**Navigation:**

| Command | What it does |
|---------|-------------|
| `vedicon-context cd config` | go to workspace config directory |
| `vedicon-context cd scenario` | go to scenario vedicon_playbook directory |
| `vedicon-context cd secrets` | go to vault/secrets directory |

**Operations:**

| Command | What it does |
|---------|-------------|
| `vedicon-context deploy` | run full scenario setup (templates + VMs) |
| `vedicon-context deploy-vms` | deploy VMs only (skip template download, faster) |
| `vedicon-context delete` | delete all scenario VMs + templates |
| `vedicon-context delete-vms` | delete VMs only (keep templates for fast redeploy) |
| `vedicon-context reset` | delete + recreate all VMs |
| `vedicon-context ssh-reload` | reload SSH keys for the active workspace |

**Info:**

| Command | What it does |
|---------|-------------|
| `vedicon-context show-vault` | show ansible vault contents (decrypted on the fly) |
| `vedicon-context show-config` | show workspace orientation (paths + SSH hosts) |
| `vedicon-context show-inventory` | show the Ansible inventory tree |
| `vedicon-context ssh <pattern>` | quick SSH to a VM by partial name (e.g. `ssh wazuh`) |
| `vedicon-context help` | show all commands |

**Catalog testing:**

Fast-iteration mode for deploying a single catalog element on a disposable
VM (the `catalog_try` scenario) without rebuilding a full lab. See
[vedicon-catalog](https://github.com/vedicon/vedicon_cyber_range_platform-catalog) for the
available elements.

| Command | What it does |
|---------|-------------|
| `vedicon-context catalog-try <path>` | deploy + smoke-check a single catalog element on the `catalog_try` VM (e.g. `catalog-try docker/_ctf/hello`) |
| `vedicon-context catalog-try-list` | list catalog elements deployable via `catalog-try` (excludes `docker/admin/*` by default) |
| `vedicon-context catalog-try-list-admin` | list only the admin catalog elements (`docker/admin/*`) |

### vedicon-workspace

A zsh function for exporting/importing workspaces between machines.

| Command | What it does |
|---------|-------------|
| `vedicon-workspace export` | create a portable archive of the active workspace |
| `vedicon-workspace import <file>` | import a workspace from an archive |

### vedicon-init.py

Interactive setup wizard (Python/Textual TUI). Creates an inventory with
your Proxmox settings and optionally runs the full deployment automatically.
Requires: `pip install --user textual` (not `apt install python3-textual`, version too old)

First time: `python3 vedicon-init.py`
After first deployment: `vedicon-context init` (shortcut, available once tools are deployed)

Run with: `python3 vedicon-init.py` or `vedicon-context init`

### Host groups

VMs in a scenario are organized in groups:

| Group | Purpose | Example hosts |
|-------|---------|--------------|
| **admin** | Infrastructure services (monitoring, API, registry) | r42.admin-wazuh, r42.admin-web-api-kong |
| **student** | Workstations for learners | r42.student-box-01 |
| **vuln** | Vulnerable targets for attack/defense exercises | r42.vuln-box-00 to r42.vuln-box-04 |

### Inventory

Two types of inventory in vedicon:

1. **Bootstrapping inventory** (`inventories/<your-infra>/hosts.yml`) — defines the
   Proxmox server and deployer-cli. Used by vedicon_playbook 01-03 and the wizard.

2. **Scenario inventory** (`~/vedicon.config/CODENAME-SCENARIO/inventory/inventory_default.yml`)
   — defines the lab VMs (admin, student, vuln groups). Used by the scenario vedicon_playbook
   on the deployer-cli.
