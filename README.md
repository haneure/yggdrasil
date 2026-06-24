# Yggdrasil

Yggdrasil is a production-oriented Hermes Agent system for customer support automation.

It is designed to:

* monitor customer service complaints from Telegram
* classify complaints and extract important information
* create and update support tickets
* notify internal team groups
* generate daily and weekly reports
* maintain the codebase and infrastructure through a dedicated engineering agent
* be reproducible through Ansible

## Project Codename

```text
yggdrasil
```

## Agent Profiles

The system uses Norse mythology themed Hermes profiles.

| Profile    | Role                       | Purpose                                                                              |
| ---------- | -------------------------- | ------------------------------------------------------------------------------------ |
| `heimdall` | Complaint watcher          | Watches customer chats, detects complaints, extracts facts, and prepares ticket data |
| `tyr`      | Ticket judge               | Creates tickets, updates status, assigns owners, handles severity and SLA            |
| `mimir`    | Report and knowledge agent | Generates reports, summaries, trends, and SOP/FAQ answers                            |
| `eitri`    | Creator / maintainer       | Codes, debugs, tests, deploys, and maintains the Yggdrasil codebase                  |

Core rule:

```text
Heimdall watches.
Tyr decides.
Mimir explains.
Eitri builds.
```

## Repository Structure

```text
yggdrasil/
├── README.md
├── ansible/
│   ├── ansible.cfg
│   ├── requirements.yml
│   ├── inventories/
│   │   └── prod/
│   │       ├── hosts.yml
│   │       └── group_vars/
│   │           └── all/
│   │               ├── main.yml
│   │               ├── vault.example.yml
│   │               └── vault.yml      # local only, encrypted, not committed
│   ├── playbooks/
│   │   ├── site.yml
│   │   └── bootstrap.yml
│   └── roles/
│       ├── base/
│       ├── docker/
│       └── hermes/
└── .gitignore
```

## Current Deployment Target

Initial deployment target:

```text
VPS server
Hermes runtime user: hermes
Deployment method: Ansible
Runtime backend: Docker
```

Hermes must run as a non-root user.

Expected runtime layout on the VPS:

```text
/home/hermes/
├── .hermes/
│   └── profiles/
│       ├── heimdall/
│       ├── tyr/
│       ├── mimir/
│       └── eitri/
└── workspaces/
    └── yggdrasil/
        └── AGENTS.md
```

## Local Requirements

Install Ansible locally.

### macOS

```bash
brew install ansible
```

### Ubuntu / WSL

```bash
sudo apt update
sudo apt install -y python3-pip python3-venv pipx
pipx ensurepath
pipx install --include-deps ansible
```

Verify:

```bash
ansible --version
ansible-galaxy --version
```

Install Ansible collections:

```bash
cd ansible
ansible-galaxy collection install -r requirements.yml
```

## Inventory Setup

Edit:

```bash
ansible/inventories/prod/hosts.yml
```

Example:

```yaml
all:
  children:
    yggdrasil:
      hosts:
        yggdrasil-prod-1:
          ansible_host: YOUR_VPS_IP
          ansible_user: ubuntu
          ansible_ssh_private_key_file: ~/.ssh/id_ed25519
```

If the VPS initially only allows root login:

```yaml
ansible_user: root
```

After bootstrap, prefer using a normal sudo-capable deploy user.

Test connection:

```bash
cd ansible
ansible yggdrasil -m ping
```

Expected:

```text
yggdrasil-prod-1 | SUCCESS
```

## Secret Setup

Real secrets are not committed to Git.

Create local vault file:

```bash
cd ansible
cp inventories/prod/group_vars/all/vault.example.yml inventories/prod/group_vars/all/vault.yml
ansible-vault encrypt inventories/prod/group_vars/all/vault.yml
ansible-vault edit inventories/prod/group_vars/all/vault.yml
```

The real file:

```text
ansible/inventories/prod/group_vars/all/vault.yml
```

is ignored by Git.

The example file:

```text
ansible/inventories/prod/group_vars/all/vault.example.yml
```

is committed to show required variables.

Required initial variables:

```yaml
telegram_heimdall_bot_token: "CHANGE_ME"
telegram_tyr_bot_token: "CHANGE_ME"
telegram_mimir_bot_token: "CHANGE_ME"
telegram_eitri_bot_token: "CHANGE_ME"

telegram_admin_user_ids:
  - "CHANGE_ME"

telegram_customer_group_id: "CHANGE_ME"
telegram_internal_group_id: "CHANGE_ME"
telegram_engineering_group_id: "CHANGE_ME"

opencode_go_api_key: "CHANGE_ME"
deepseek_api_key: "CHANGE_ME"
openrouter_api_key: "CHANGE_ME"
```

Optional future variables:

```yaml
ticket_api_secret: "CHANGE_ME"
postgres_password: "CHANGE_ME"
redis_password: "CHANGE_ME"
hindsight_llm_api_key: "CHANGE_ME"
n8n_encryption_key: "CHANGE_ME"
```

## Vault Password

This project can use a local vault password file:

```text
ansible/.vault_pass
```

This file is ignored by Git.

Create one:

```bash
cd ansible
openssl rand -base64 32 > .vault_pass
chmod 600 .vault_pass
```

If not using `.vault_pass`, run playbooks with:

```bash
--ask-vault-pass
```

## Bootstrap VPS

Run check mode first:

```bash
cd ansible
ansible-playbook playbooks/bootstrap.yml --check --diff --ask-become-pass
```

Run real bootstrap:

```bash
ansible-playbook playbooks/bootstrap.yml --diff --ask-become-pass
```

If passwordless sudo is configured for the deploy user, `--ask-become-pass` can be omitted.

## Bootstrap Responsibilities

The bootstrap playbook currently handles:

* apt package installation
* timezone setup
* `hermes` Linux user creation
* workspace directory creation
* systemd linger for `hermes`
* UFW firewall baseline
* fail2ban
* Docker Engine installation
* Docker Compose plugin installation
* Docker group access for `hermes`
* Hermes installation under non-root `hermes` user
* Hermes profile directory creation
* base `SOUL.md` files
* base profile `config.yaml`
* base profile `.env`
* shared `AGENTS.md`

## Post-Bootstrap Validation

SSH into the VPS:

```bash
ssh ubuntu@YOUR_VPS_IP
```

Check the Hermes user:

```bash
id hermes
```

Check systemd linger:

```bash
loginctl show-user hermes | grep Linger
```

Expected:

```text
Linger=yes
```

Check Docker as `hermes`:

```bash
sudo su - hermes -c 'docker ps'
```

Check Hermes:

```bash
sudo su - hermes -c 'export PATH="$HOME/.local/bin:$HOME/.hermes/bin:$PATH"; hermes --version'
```

Check profiles:

```bash
sudo su - hermes -c 'ls -la ~/.hermes/profiles'
```

Expected profiles:

```text
heimdall
tyr
mimir
eitri
```

Check workspace:

```bash
sudo su - hermes -c 'ls -la ~/workspaces/yggdrasil'
```

## Git Safety

The following must not be committed:

```text
ansible/.vault_pass
ansible/inventories/*/group_vars/**/vault.yml
```

Check before committing:

```bash
git status --ignored
```

Commit safe setup files:

```bash
git add README.md .gitignore ansible/
git commit -m "bootstrap yggdrasil infrastructure"
```

## Progress Tracker

### Phase 0 — Project Planning

* [x] Define project codename: `yggdrasil`
* [x] Define core agent profiles
* [x] Define Norse mythology profile naming
* [x] Decide Ansible-first deployment
* [x] Decide Hermes must run as non-root user

### Phase 1 — Ansible Bootstrap

* [x] Create Ansible repo structure
* [x] Configure inventory
* [x] Configure group vars
* [x] Configure Ansible Vault pattern
* [x] Add `vault.example.yml`
* [x] Add base role
* [x] Add Docker role
* [x] Add Hermes role
* [x] Fix Ansible callback config
* [x] Fix Docker apt repository setup
* [x] Fix fail2ban check-mode behavior
* [x] Run bootstrap on VPS
* [x] Install Hermes under non-root `hermes` user

### Phase 2 — Hermes Profile Baseline

* [x] Create profile directories
* [x] Create initial `SOUL.md` files
* [x] Create initial profile `config.yaml`
* [x] Create initial profile `.env`
* [x] Create shared `AGENTS.md`
* [ ] Confirm Hermes profile commands work correctly
* [ ] Confirm each profile is recognized by Hermes CLI
* [ ] Replace direct profile directory creation with Hermes CLI profile creation if needed

### Phase 3 — Telegram Gateway Setup

* [ ] Create Telegram bots with BotFather
* [ ] Configure BotFather privacy mode for `heimdall`
* [ ] Fill Telegram tokens in Ansible Vault
* [ ] Fill Telegram allowed user IDs
* [ ] Fill Telegram group chat IDs
* [ ] Generate profile-specific Telegram config
* [ ] Install Hermes gateway service for `eitri`
* [ ] Start `eitri` gateway first
* [ ] Test private admin DM with `eitri`
* [ ] Install and test `mimir`
* [ ] Install and test `tyr`
* [ ] Install and test `heimdall`
* [ ] Verify no profile responds in unauthorized chats

### Phase 4 — Ticket Backend

* [ ] Create Go ticket API project
* [ ] Create PostgreSQL schema
* [ ] Create ticket tables
* [ ] Create ticket event tables
* [ ] Create ticket comment tables
* [ ] Create customer table
* [ ] Implement ticket create endpoint
* [ ] Implement ticket search endpoint
* [ ] Implement ticket update endpoint
* [ ] Implement report endpoints
* [ ] Add API authentication
* [ ] Deploy API with Ansible
* [ ] Deploy PostgreSQL with Ansible
* [ ] Add database backup

### Phase 5 — Agent Skills

* [ ] Create `complaint_intake` skill
* [ ] Create `ticket_creation` skill
* [ ] Create `ticket_deduplication` skill
* [ ] Create `sla_classification` skill
* [ ] Create `support_report` skill
* [ ] Create `knowledge_lookup` skill
* [ ] Create `codebase_maintenance` skill
* [ ] Create `fine_tuning_dataset` skill
* [ ] Deploy skills through Ansible

### Phase 6 — Reporting

* [ ] Configure Mimir daily report cron
* [ ] Configure Mimir weekly report cron
* [ ] Test daily report manually
* [ ] Test weekly report manually
* [ ] Add internal team notification format
* [ ] Add SLA breach report

### Phase 7 — Hindsight Memory

* [ ] Decide Hindsight deployment mode
* [ ] Add Hindsight Docker Compose
* [ ] Deploy Hindsight bound to localhost only
* [ ] Add Hindsight secrets to vault
* [ ] Add memory event schema
* [ ] Make ticket API write memory events
* [ ] Let Eitri read/write Hindsight memory
* [ ] Let Mimir use Hindsight for recurring issue reports

### Phase 8 — Optional n8n

* [ ] Decide if n8n is needed
* [ ] Add n8n only for third-party workflow glue
* [ ] Deploy n8n with PostgreSQL
* [ ] Protect n8n behind VPN or access control
* [ ] Keep n8n out of core ticket source-of-truth logic

### Phase 9 — Production Hardening

* [ ] Configure logrotate
* [ ] Configure backups
* [ ] Configure uptime checks
* [ ] Configure service health checks
* [ ] Add restore documentation
* [ ] Add disaster recovery test
* [ ] Add least-privilege service users
* [ ] Confirm `.env` files are `0600`
* [ ] Confirm Hermes gateways survive reboot
* [ ] Confirm Docker containers restart after reboot

## Current Status

Current milestone:

```text
Phase 1 completed.
Phase 2 baseline mostly completed.
```

Latest known status:

```text
Ansible bootstrap has completed on the VPS.
Hermes is installed under the non-root `hermes` user.
Docker is installed and available.
Initial Hermes profile files were generated.
Vault secrets are intentionally excluded from Git.
```

Next immediate milestone:

```text
Phase 3: Telegram gateway setup, starting with Eitri only.
```

## Next Steps

### 1. Verify Hermes profiles

On VPS:

```bash
sudo su - hermes
export PATH="$HOME/.local/bin:$HOME/.hermes/bin:$PATH"
hermes --version
ls -la ~/.hermes/profiles
```

Check each profile directory:

```bash
ls -la ~/.hermes/profiles/heimdall
ls -la ~/.hermes/profiles/tyr
ls -la ~/.hermes/profiles/mimir
ls -la ~/.hermes/profiles/eitri
```

### 2. Create Telegram bots

Create these bots using BotFather:

```text
heimdall_cs_bot
tyr_ticket_bot
mimir_report_bot
eitri_builder_bot
```

Save tokens in:

```text
ansible/inventories/prod/group_vars/all/vault.yml
```

Edit with:

```bash
cd ansible
ansible-vault edit inventories/prod/group_vars/all/vault.yml
```

### 3. Start with Eitri only

Eitri is the safest first live gateway because it should only be used in private admin or engineering chat.

Goal:

```text
Get one Hermes gateway working before enabling all support agents.
```

### 4. Add profile-specific gateway deployment

Update Ansible to:

* template per-profile `.env`
* template per-profile Telegram config
* install gateway services
* start only selected profiles first
* support enabling/disabling profiles by variable

Recommended first live profile:

```text
eitri
```

Do not start `heimdall` in customer chat until private and internal testing passes.

## Operational Notes

### Do not expose customer-facing automation too early

Recommended rollout:

```text
1. Eitri private admin bot
2. Mimir internal report bot
3. Tyr internal ticket bot
4. Heimdall shadow-mode complaint watcher
5. Heimdall limited customer-facing replies
```

### Do not use n8n as source of truth

n8n may be added later for glue workflows, but ticket state should live in the ticket API and PostgreSQL.

### Do not fine-tune yet

Current strategy:

```text
Prompt tuning + SOUL.md + AGENTS.md + skills + examples first.
Fine-tuning later after enough real reviewed examples.
```

### Memory strategy

Use Hindsight later as a memory/dataset sidecar.

Initial system should work without Hindsight.

## Useful Commands

Run full bootstrap:

```bash
cd ansible
ansible-playbook playbooks/bootstrap.yml --diff --ask-become-pass
```

Run check mode:

```bash
ansible-playbook playbooks/bootstrap.yml --check --diff --ask-become-pass
```

Edit vault:

```bash
ansible-vault edit inventories/prod/group_vars/all/vault.yml
```

View inventory for host:

```bash
ansible-inventory --host yggdrasil-prod-1 --yaml
```

SSH to server:

```bash
ssh ubuntu@YOUR_VPS_IP
```

Become Hermes user:

```bash
sudo su - hermes
```

Check Hermes:

```bash
export PATH="$HOME/.local/bin:$HOME/.hermes/bin:$PATH"
hermes --version
```

Check Docker:

```bash
docker ps
```

Check fail2ban:

```bash
sudo systemctl status fail2ban --no-pager
sudo fail2ban-client status
```

Check Docker service:

```bash
sudo systemctl status docker --no-pager
```

## Troubleshooting Notes

### Missing sudo password

Use:

```bash
ansible-playbook playbooks/bootstrap.yml --ask-become-pass
```

### Docker Signed-By conflict

Remove legacy Docker source files:

```bash
sudo rm -f /etc/apt/sources.list.d/docker.list
sudo rm -f /etc/apt/sources.list.d/download_docker_com_linux_ubuntu.list
sudo rm -f /etc/apt/keyrings/docker.gpg
sudo apt update
```

Then rerun Ansible.

### Vault variable undefined

Make sure vault file is located at:

```text
ansible/inventories/prod/group_vars/all/vault.yml
```

Not:

```text
ansible/inventories/prod/group_vars/vault.yml
```

### Hermes installer appears stuck

Check running processes:

```bash
ps aux | grep -E 'hermes|install.sh|curl|bash|uv|python|node|npm|playwright' | grep -v grep
```

If `npm`, `node`, or `python` is active, it is likely still installing.

## Design Principles

* Reproducible infrastructure first
* Non-root Hermes runtime
* One profile per responsibility
* One Telegram bot token per profile
* Docker backend for safer execution
* Ticket API as source of truth
* Telegram messages as evidence only
* No internal notes in customer chat
* Start with private/internal bots before customer-facing automation
* Keep fine-tuning for later
* Use Ansible to make rebuilds boring

## License

TBD.
