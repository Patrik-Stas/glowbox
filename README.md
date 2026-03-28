# Glowbox

Sandboxed Linux VMs for running unsupervised [Claude Code](https://docs.anthropic.com/en/docs/claude-code) agents on macOS.

One command gives you an isolated Ubuntu VM with Docker, zsh, and Claude Code — ready for AI-driven development without risking your host machine.

## Why

Running autonomous AI agents directly on your Mac means giving them access to your entire filesystem, running processes, and system configuration. Glowbox puts them in a sandbox:

- **Isolated** — the VM only sees directories you explicitly mount. Nothing else on your Mac is accessible.
- **Disposable** — nuke and recreate in minutes. Break things freely.
- **Port-forwarded** — any port the VM listens on (1024–65535) is automatically available on localhost. No config needed.
- **Fast** — runs on Apple's native Virtualization framework (vz). Near-native performance.

## Quick Start

```bash
brew install lima
git clone https://github.com/user/glowbox.git
cd glowbox
```

```bash
# From your projects directory:
./glowbox init          # configure mount paths (defaults to current dir)
./glowbox create        # create and provision the VM (~5 min first time)
./glowbox shell         # enter the VM
```

Inside the VM:

```bash
cd ~/work               # your host directory, synced instantly
claude                  # Claude Code is ready to go
docker compose up -d    # Docker works out of the box
```

## How It Works

Glowbox creates [Lima](https://lima-vm.io/) VMs using Apple's Virtualization framework with two mount points:

| Mount | VM path | Host path | Purpose |
|-------|---------|-----------|---------|
| **Work** | `~/work` | your working directory | Shared code/projects — same across all VMs |
| **Data** | `~/data` | `~/glowbox/<vm-name>` | Per-VM isolated storage — build caches, databases |

Both use virtiofs — changes are instant on both sides, no sync lag.

## What's Included

| Category | Tools |
|----------|-------|
| Shell | zsh, oh-my-zsh, tmux, neovim |
| Docker | Docker CE, Compose v2 (no sudo needed) |
| AI | Claude Code (native binary) |
| Search | ripgrep, fd, bat, jq |
| Runtimes | [mise](https://mise.jdx.dev/) (install any version of node, python, go, rust, etc.) |
| Build | build-essential, python3, git |

SSH agent forwarding is enabled — your GitHub keys work inside the VM without copying them.

## Commands

```
glowbox init                    Set up config (interactive)
glowbox create [name] [flags]   Create and start a VM
glowbox shell  [name]           Enter a VM (auto-starts if stopped)
glowbox stop   [name]           Stop a VM
glowbox start  [name]           Start a stopped VM
glowbox list                    List all VMs
glowbox status [name]           Show VM details
glowbox logs   [name]           Tail provisioning log
glowbox delete [name]           Delete with confirmation
glowbox nuke   [name]           Force delete
```

Default VM name is `glowbox`. Create multiple VMs with different resource profiles:

```bash
./glowbox create heavy --cpus 8 --memory 16GiB
./glowbox create light --cpus 2 --memory 4GiB
```

## Configuration

Run `glowbox init` or copy the example:

```bash
cp glowbox.yaml.example glowbox.yaml
```

```yaml
# Host directory mounted as ~/work in VMs
work_dir: ~/my/projects

# Per-VM storage base directory (default: ~/glowbox)
# data_dir: ~/glowbox
```

### Environment variables

Place a `.env` file in your work directory:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

It's auto-sourced on every VM login.

## Requirements

- macOS on Apple Silicon
- [Lima](https://lima-vm.io/) 2.x (`brew install lima`)

## Docs

- **[GUIDE.md](GUIDE.md)** — step-by-step setup and usage walkthrough
- **[OVERVIEW.md](OVERVIEW.md)** — architecture decisions, included tools, customization

## License

MIT
