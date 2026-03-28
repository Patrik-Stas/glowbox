# Glowbox — Lima VM for Claude Code Agent Development

Isolated Linux VMs for running unsupervised Claude Code agents and developing project microservices in Docker containers.

## Mounts

Each VM has two mount points connecting it to the host:

### Work mount (`~/work`)

Your main working directory — code, projects, whatever you need the VM to operate on. Configured via `work_dir` in `glowbox.yaml`. Shared across all VMs.

| | Path |
|---|---|
| **Host** | whatever you set as `work_dir` (defaults to pwd during `glowbox init`) |
| **VM** | `~/work` |

### Data mount (`~/data`)

Per-VM isolated storage. Each VM gets its own directory on the host, so VMs don't step on each other. Use this for VM-specific state, build artifacts, databases, etc.

| | Path |
|---|---|
| **Host** | `<data_dir>/<vm-name>` (e.g. `~/glowbox/myvm`) |
| **VM** | `~/data` |

Both use **virtiofs** — a shared filesystem, not copy/sync. Changes are instantly visible on both sides.

## Architecture Decisions

### Why Lima + vz (Apple Virtualization Framework)

- **vz** is Apple's native hypervisor — fastest option on Apple Silicon, no third-party kernel extensions
- Near-native performance for ARM64 Linux workloads
- **Rosetta** enabled for transparent x86_64 binary support (useful for Docker images without ARM variants)
- Lima wraps vz with ergonomic CLI tooling, cloud-init provisioning, and automatic port forwarding

### OS: Ubuntu 24.04 LTS

- Long-term support, broad package availability
- Best compatibility with Docker CE and cloud-init

### Port Forwarding: automatic 1:1, range 1024–65535

- Lima auto-detects when the guest starts listening on a port and forwards it to the same port on the host
- No need to pre-declare ports or restart the VM when adding services
- Services must bind to `0.0.0.0` inside the VM (not `127.0.0.1`) for Lima to detect and forward them
- `docker run -p 8080:80` works transparently — Docker binds on 0.0.0.0, Lima forwards it
- Host-side binding is `127.0.0.1` only (not exposed to LAN)
- If a port is already in use on the host, forwarding for that port is silently skipped

### Docker CE (not containerd)

- Docker CE installed directly via `get.docker.com` — standard Docker CLI experience
- Docker Compose v2 included as a plugin (`docker compose`)
- Lima's built-in containerd is disabled to avoid conflicts
- No `sudo` needed for docker commands

### Claude Code: native binary

- Installed via `https://claude.ai/install.sh` (native binary, not npm)
- Available as `claude` on PATH after provisioning

### SSH Agent Forwarding

- Your host's SSH keys (GitHub, etc.) are available inside the VM
- Enables `git clone/push` to private repos without copying keys

## Resources (defaults)

| Resource | Default | Override flag |
|----------|---------|---------------|
| CPUs     | 4       | `--cpus N`    |
| Memory   | 8 GiB   | `--memory 16GiB` |
| Disk     | 60 GiB  | `--disk 100GiB` |

Override at create time: `glowbox create myvm --cpus 8 --memory 16GiB`

## Included Tools

### System packages
`build-essential` `curl` `wget` `git` `zsh` `tmux` `htop` `tree` `neovim` `jq` `ripgrep` `fd-find` `bat` `python3`

### User-space
- **oh-my-zsh** — zsh framework with sensible defaults
- **mise** — polyglot tool version manager (node, python, rust, go, etc.)
- **Claude Code** — native binary

### Shell aliases

- **Git:** `g` `gch` `gchb` `gbr` `gadd` `gcommit` `gamend` `gf` `gstatus` `ggba` `log` `logg`
- **Docker:** `dps` `dpsa` `dimg` `dlogs` `dexec` `dc` `dcu` `dcd` `dcl` `kcontainers`
- **General:** `ll` `la` `vim`/`v` (neovim) `ca` (package.json scripts) `w`/`work` `data`
- **Network:** `checkport` `inspectports`

## Environment Variables

Create a `.env` file in your **work directory** on the host:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

It is auto-sourced by `.zshrc` on every VM shell login (mounted at `~/work/.env`).

## File layout

```
glowbox/
  glowbox                # Control script (chmod +x)
  template.yaml          # Lima VM template (generic, with placeholders)
  glowbox.yaml.example   # Example user config — copy to glowbox.yaml
  glowbox.yaml           # Your config (gitignored, contains personal paths)
  OVERVIEW.md            # This file
  GUIDE.md               # Step-by-step usage guide
  .gitignore
```

## Customization

- **Mount paths:** Set `work_dir` and `data_dir` in `glowbox.yaml`
- **Port range:** Adjust `guestPortRange` in `template.yaml`
- **Packages:** Add to the `apt-get install` line in the system provisioning script
- **Aliases:** Append to the echo block in the user provisioning script
- **Default resources:** Edit `cpus`, `memory`, `disk` in `template.yaml`
