# Workstation Setup — Ansible

Automated dev workstation provisioning for **CachyOS**, **Fedora**, and **Pop!_OS**.

## Quick Start

```bash
# 1. Install Ansible
# CachyOS / Arch
paru -S ansible

# Fedora
sudo dnf install ansible

# Pop!_OS / Ubuntu
sudo apt install ansible

# 2. Install required collections
ansible-galaxy install -r requirements.yml

# 3. Run on the current machine
ansible-playbook playbook.yml --connection=local --ask-become-pass
```

## Usage

### Install everything (default)
```bash
ansible-playbook playbook.yml --connection=local --ask-become-pass
```

### Install specific tools using tags
```bash
ansible-playbook playbook.yml --connection=local --ask-become-pass --tags "docker,brave,vscode"
```

### Run against remote machines
```bash
# Edit inventory.ini with your hosts, then:
ansible-playbook playbook.yml --ask-become-pass
```

## Available Tools

| Tool               | Tag               | CachyOS | Fedora | Pop!_OS |
|--------------------|-------------------|---------|--------|---------|
| paru               | `paru`            | ✓       | —      | —       |
| Docker + Compose   | `docker`          | ✓       | ✓      | ✓       |
| JetBrains Toolbox  | `jetbrains-toolbox`| ✓      | ✓      | ✓       |
| Brave Browser      | `brave`           | ✓       | ✓      | ✓       |
| 1Password          | `1password`       | ✓       | ✓      | ✓       |
| VS Code            | `vscode`          | ✓       | ✓      | ✓       |
| asdf + Node.js     | `asdf`            | ✓       | ✓      | ✓       |
| CachyOS Gaming     | `gaming`          | ✓       | —      | —       |
| Discord            | `discord`         | ✓       | ✓      | ✓       |
| Spotify            | `spotify`         | ✓       | ✓      | ✓       |
| Flameshot          | `flameshot`       | ✓       | ✓      | ✓       |
| SSH Keys           | `ssh-keys`        | ✓       | ✓      | ✓       |

## Customization

### Per-host tool selection

Override the default tool list for a specific host in `host_vars/<hostname>.yml`:

```yaml
# host_vars/fedora-laptop.yml
tools:
  - docker
  - brave
  - vscode
  - 1password
  - ssh-keys
```

### SSH key profiles

Edit `group_vars/all.yml` to change key names or hosts:

```yaml
ssh_key_profiles:
  - name: personal
    host: github.com
  - name: SC
    host: github.com
  - name: Spot2
    host: github.com
```

### asdf plugins

Add more language runtimes in `group_vars/all.yml`:

```yaml
asdf_plugins:
  - name: nodejs
    version: latest
  - name: python
    version: latest
  - name: ruby
    version: latest
```

## Project Structure

```
workstation-ansible/
├── ansible.cfg              # Ansible configuration
├── playbook.yml             # Main playbook
├── inventory.ini            # Target machines
├── requirements.yml         # Galaxy collections needed
├── group_vars/
│   └── all.yml              # Shared variables (tools, SSH keys, asdf plugins)
├── host_vars/
│   └── localhost.yml        # Per-host overrides
└── roles/
    ├── paru/                # Arch only — AUR helper
    ├── docker/              # Docker Engine + Compose + Buildx
    ├── brave/               # Brave Browser
    ├── onepassword/         # 1Password
    ├── vscode/              # Visual Studio Code
    ├── jetbrains_toolbox/   # JetBrains Toolbox
    ├── asdf/                # asdf + shell config + plugins
    ├── gaming/              # CachyOS gaming meta packages
    ├── discord/             # Discord
    ├── spotify/             # Spotify
    ├── flameshot/           # Flameshot screenshot tool
    └── ssh_keys/            # SSH key generation + config
```

## Notes

- **Distro detection is automatic** — the playbook reads `ansible_distribution` and routes to the correct package manager.
- **Idempotent** — safe to re-run. Ansible skips anything already in the desired state.
- **Discord & Spotify on Fedora/Pop!_OS** use Flatpak since they don't have official repos for those distros.
- **Gaming role** is CachyOS-only (installs `cachyos-gaming-meta` + `cachyos-gaming-applications`).
- **SSH keys** role uses `community.crypto.openssh_keypair` for proper idempotency — won't overwrite existing keys.
