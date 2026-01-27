# Agent Guidelines

## Project Overview

Ansible playbook for installing Clawdbot in a Docker Sandbox microVM on macOS.

## Architecture

This playbook creates a Docker Sandbox (microVM) that provides hypervisor-level isolation:

```
Host (macOS with Docker Desktop)
└── Docker Sandbox microVM
    ├── Private Docker daemon
    ├── Clawdbot + Node.js + pnpm
    └── Workspace synced from ~/.clawdbot
```

## Key Files

```
roles/clawdbot/
├── tasks/
│   ├── main.yml      # Entry point (macOS check, includes sandbox.yml)
│   ├── sandbox.yml   # Docker Sandbox setup, Clawdbot installation
│   └── legacy/       # Old Linux/bare-metal tasks (archived)
├── defaults/
│   └── main.yml      # Configuration variables
└── handlers/
    └── main.yml      # Service handlers (not used in sandbox mode)
```

## What the Playbook Does

1. Verifies Docker Desktop and Docker Sandbox are available
2. Creates workspace directory (`~/.clawdbot`)
3. Creates a Docker Sandbox microVM with the workspace mounted
4. Configures network proxy bypass for `host.docker.internal`
5. Installs Node.js and pnpm inside the sandbox
6. Installs Clawdbot (release or development mode)
7. Creates a helper script to run Clawdbot

## Development Guidelines

### Ansible
- Use `ansible.builtin.command` for `docker sandbox` commands (no Ansible module exists)
- Check `changed_when` carefully for idempotency
- All tasks run on the host, exec into sandbox via `docker sandbox exec`

### Docker Sandbox
- Sandboxes persist until explicitly removed with `docker sandbox rm`
- Use `docker sandbox exec <name> <command>` to run commands inside
- Network: internet works, host services via `host.docker.internal`
- No Tailscale inside sandbox (no /dev/net/tun in microVM)

### Testing

```bash
# Syntax check
ansible-playbook playbook.yml --syntax-check

# Dry run (limited - can't simulate docker sandbox commands)
ansible-playbook playbook.yml --check

# Full run
ansible-playbook playbook.yml

# Verify sandbox
docker sandbox ls
docker sandbox exec <name> clawdbot --version

# Test host service access from sandbox
docker sandbox exec <name> curl http://host.docker.internal:1234/v1/models
```

### Cleanup

```bash
# Remove sandbox (deletes all installed packages, images, etc.)
docker sandbox rm <sandbox-name>

# Remove workspace
rm -rf ~/.clawdbot

# Re-run playbook for fresh install
ansible-playbook playbook.yml
```

## Legacy Support

The `legacy/` directory contains the old bare-metal installation tasks for:
- Linux (Debian/Ubuntu) with UFW firewall
- macOS with user-scoped Homebrew

These are archived for reference. The Docker Sandbox approach is preferred for security.

## Common Issues

### "docker sandbox" not found
Docker Desktop 4.58+ required. Sandbox is a Docker Desktop feature.

### Can't reach host services
Use `host.docker.internal:PORT`. Ensure the service binds to `0.0.0.0`, not `127.0.0.1`.

### Tailscale in sandbox
Not supported - microVM lacks `/dev/net/tun`. Use Tailscale Funnel on host instead.
