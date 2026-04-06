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

### Re-running is safe
Ansible is idempotent — running the playbook multiple times will only install or configure what's missing. Already-installed tools show as `ok` instead of `changed`.

## Available Tools

| Tool               | Tag                | CachyOS | Fedora | Pop!_OS |
|--------------------|--------------------|---------|--------|---------|
| paru               | `paru`             | ✓       | —      | —       |
| Zsh (default shell)| `zsh`              | ✓       | ✓      | ✓       |
| Docker + Compose   | `docker`           | ✓       | ✓      | ✓       |
| JetBrains Toolbox  | `jetbrains-toolbox`| ✓       | ✓      | ✓       |
| Brave Browser      | `brave`            | ✓       | ✓      | ✓       |
| 1Password          | `1password`        | ✓       | ✓      | ✓       |
| VS Code            | `vscode`           | ✓       | ✓      | ✓       |
| asdf + Node.js     | `asdf`             | ✓       | ✓      | ✓       |
| CachyOS Gaming     | `gaming`           | ✓       | —      | —       |
| Discord            | `discord`          | ✓       | ✓      | ✓       |
| Spotify            | `spotify`          | ✓       | ✓      | ✓       |
| Flameshot          | `flameshot`        | ✓       | ✓      | ✓       |
| SSH Keys           | `ssh-keys`         | ✓       | ✓      | ✓       |
| VMware Workstation | `vmware`           | ✓       | ✓      | ✓       |
| Nerd Fonts         | `nerd-fonts`       | ✓       | ✓      | ✓       |

## Customization

### Per-host tool selection

Override the default tool list for a specific host in `host_vars/<hostname>.yml`:

```yaml
# host_vars/fedora-laptop.yml
tools:
  - zsh
  - docker
  - brave
  - vscode
  - 1password
  - ssh-keys
  - nerd-fonts
```

### SSH key profiles

Edit `group_vars/all.yml` to change key names or git hosts:

```yaml
ssh_key_profiles:
  - name: personal
    host: github.com
  - name: SC
    host: github.com
  - name: Spot2
    host: github.com
```

Each key generates a `~/.ssh/config` Host entry like `github.com-personal`, so you clone repos with:

```bash
git clone git@github.com-personal:you/repo.git
git clone git@github.com-SC:company/repo.git
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

### Nerd Fonts

Choose which fonts to install in `group_vars/all.yml`:

```yaml
nerd_fonts:
  - JetBrainsMono
  - Hack
  - FiraCode
  - Meslo
```

Full list of available fonts at [nerdfonts.com](https://www.nerdfonts.com/).

## Project Structure

```
workstation-ansible/
├── ansible.cfg              # Ansible configuration
├── playbook.yml             # Main playbook
├── inventory.ini            # Target machines
├── requirements.yml         # Galaxy collections needed
├── group_vars/
│   └── all.yml              # Shared variables
├── host_vars/
│   └── localhost.yml         # Per-host overrides
└── roles/
    ├── paru/                # Arch only — AUR helper
    ├── zsh/                 # Install zsh + set as default shell
    ├── docker/              # Docker Engine + Compose + Buildx + containerd
    │   └── handlers/        # Restart Docker handler
    ├── brave/               # Brave Browser
    ├── onepassword/         # 1Password (GPG key + repo setup)
    ├── vscode/              # Visual Studio Code
    ├── jetbrains_toolbox/   # JetBrains Toolbox (API download + background launch)
    ├── asdf/                # asdf v0.16+ binary + shell config + plugins
    ├── gaming/              # CachyOS gaming meta packages
    ├── discord/             # Discord (native on Arch, Flatpak on Fedora/Pop!_OS)
    ├── spotify/             # Spotify (native on Arch, Flatpak on Fedora/Pop!_OS)
    ├── flameshot/           # Flameshot screenshot tool
    ├── ssh_keys/            # SSH key generation + ~/.ssh/config
    ├── vmware/              # VMware Workstation Pro
    └── nerd_fonts/          # Nerd Fonts from GitHub releases
```

## Notes

- **Distro detection is automatic** — the playbook reads `ansible_distribution` and routes to the correct package manager. The detection runs with `tags: always` so it works even when filtering by tag.
- **Sudo password is verified upfront** — if you enter the wrong password, the playbook fails immediately instead of hanging.
- **Idempotent** — safe to re-run. Ansible skips anything already in the desired state.
- **Discord & Spotify on Fedora/Pop!_OS** use Flatpak since they don't have official repos for those distros.
- **Gaming role** is CachyOS-only (installs `cachyos-gaming-meta` + `cachyos-gaming-applications`).
- **SSH keys** role uses `community.crypto.openssh_keypair` — won't overwrite existing keys.
- **VMware on Fedora/Pop!_OS** requires manually downloading the `.bundle` from [Broadcom](https://support.broadcom.com) (free account required) and placing it in `~/Downloads`. On CachyOS it installs via AUR.
- **asdf** uses the v0.16+ Go rewrite. Shell config adds shims to `$PATH` based on your detected shell. The `asdf set -u` command sets global versions per the official docs.
- **JetBrains Toolbox** launches in the background after install so it doesn't block the playbook.
- **Nerd Fonts** are downloaded directly from GitHub releases — no package manager needed. Works identically on all distros.
- **Zsh** is installed early in the playbook so later tools (like asdf) detect it when configuring shell rc files.
