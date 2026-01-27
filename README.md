# Clawdbot Ansible Installer

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Ansible](https://img.shields.io/badge/Ansible-2.14+-blue.svg)](https://www.ansible.com/)
[![macOS](https://img.shields.io/badge/macOS-Docker%20Desktop-orange.svg)](https://www.docker.com/products/docker-desktop/)

Automated installation of [Clawdbot](https://clawd.bot) in a secure Docker Sandbox microVM on macOS.

## Why Docker Sandbox?

Docker Sandbox runs Clawdbot in an isolated microVM with its own Docker daemon. This provides **real security isolation** - not just user separation or firewall rules, but hypervisor-level containment.

Benefits:
- **Hypervisor isolation** - Separate kernel, can't access host resources
- **Private Docker daemon** - Clawdbot's containers are isolated from yours
- **Workspace sync** - Your project files sync bidirectionally
- **Host service access** - Connect to LM Studio, Ollama via `host.docker.internal`

## Requirements

- macOS 11 (Big Sur) or later
- Docker Desktop 4.58+ with Docker Sandbox enabled
- Ansible 2.14+

## Quick Start

```bash
# Clone this repo
git clone https://github.com/pasogott/clawdbot-ansible.git
cd clawdbot-ansible

# Install Ansible if needed
brew install ansible

# Run the playbook
ansible-playbook playbook.yml
```

This creates:
- A Docker Sandbox microVM for Clawdbot
- Clawdbot installed inside the sandbox
- A helper script at `~/.clawdbot/run-clawdbot.sh`

## Usage

### Start Clawdbot

```bash
~/.clawdbot/run-clawdbot.sh
```

Or manually:

```bash
docker sandbox exec -it <sandbox-name> bash
clawdbot onboard --install-daemon
```

### Access Host Services (LM Studio, Ollama)

From inside the sandbox, use `host.docker.internal` to reach services on your Mac:

```bash
# LM Studio API
curl http://host.docker.internal:1234/v1/models

# Ollama
curl http://host.docker.internal:11434/api/tags
```

Make sure your local LLM server binds to `0.0.0.0` (all interfaces), not just `127.0.0.1`.

### Manage Sandbox

```bash
# List sandboxes
docker sandbox ls

# Shell into sandbox
docker sandbox exec -it <sandbox-name> bash

# Stop sandbox (preserves state)
docker sandbox stop <sandbox-name>

# Remove sandbox (deletes everything inside)
docker sandbox rm <sandbox-name>
```

## Configuration

Edit variables before running:

```bash
ansible-playbook playbook.yml \
  -e clawdbot_install_mode=development \
  -e clawdbot_repo_branch=feature-branch
```

### Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `clawdbot_workspace` | `~/.clawdbot` | Host directory synced to sandbox |
| `clawdbot_install_mode` | `release` | `release` or `development` |
| `clawdbot_repo_url` | GitHub URL | Git repo for development mode |
| `clawdbot_repo_branch` | `main` | Branch for development mode |
| `nodejs_version` | `22` | Node.js major version |

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Your Mac                         │
│  ┌───────────────┐    ┌─────────────────────────┐  │
│  │  LM Studio    │    │  Docker Sandbox microVM │  │
│  │  :1234        │◄───│  ┌─────────────────────┐│  │
│  └───────────────┘    │  │    Clawdbot         ││  │
│                       │  │    + Node.js        ││  │
│  ┌───────────────┐    │  │    + pnpm           ││  │
│  │  ~/.clawdbot  │◄──►│  │                     ││  │
│  │  (workspace)  │sync│  └─────────────────────┘│  │
│  └───────────────┘    │  Private Docker daemon  │  │
│                       └─────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

- **Hypervisor isolation**: macOS virtualization.framework
- **Bidirectional sync**: Workspace files sync between host and sandbox
- **Network**: Internet access via proxy, host services via `host.docker.internal`

## Linux Support

Docker Sandbox microVMs are macOS/Windows only. For Linux servers, see the `legacy/` directory for the traditional container-based approach, or run Clawdbot directly on the host.

## Troubleshooting

### "docker sandbox" command not found

Ensure Docker Desktop 4.58+ is installed and running. Docker Sandbox is a Docker Desktop feature.

### Can't reach LM Studio from sandbox

1. Make sure LM Studio binds to `0.0.0.0:1234`, not `127.0.0.1:1234`
2. Use `http://host.docker.internal:1234` from inside the sandbox

### Sandbox creation fails

```bash
# Reset Docker Sandbox state
docker sandbox reset

# Try again
ansible-playbook playbook.yml
```

## License

MIT - see [LICENSE](LICENSE)

## Links

- [Clawdbot](https://clawd.bot)
- [Docker Sandbox Docs](https://docs.docker.com/ai/sandboxes/)
- [This installer](https://github.com/pasogott/clawdbot-ansible)
