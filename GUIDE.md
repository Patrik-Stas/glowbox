# Glowbox Guide

Step-by-step guide for setting up and using Glowbox Lima VMs.

---

## Prerequisites

```bash
brew install lima
limactl --version    # should be 2.x
```

Requires macOS on Apple Silicon.

---

## 1. Initial Setup

### Configure

From inside your projects/working directory:

```bash
glowbox init
```

This prompts for two paths and writes `glowbox.yaml`:

- **Work mount** — your main working directory, mounted as `~/work` in VMs. Defaults to the current directory.
- **Data mount** — per-VM isolated storage, mounted as `~/data` in VMs. Defaults to `~/glowbox`.

Or configure manually:

```bash
cp glowbox.yaml.example glowbox.yaml
# edit glowbox.yaml
```

### Set up API keys (optional)

Create a `.env` file in your work directory:

```bash
echo 'export ANTHROPIC_API_KEY=sk-ant-your-key-here' > .env
```

This is auto-sourced on every VM login (available at `~/work/.env` inside the VM).

### Create the VM

```bash
glowbox create
```

This takes a few minutes. It will:
1. Download Ubuntu 24.04 ARM image (first time only)
2. Create and boot the VM
3. Install Docker, zsh, oh-my-zsh, neovim, mise, Claude Code
4. Wait for provisioning to complete

### Enter the VM

```bash
glowbox shell
```

---

## 2. Mounts Inside the VM

```
~/work/       ← your host working directory (shared across all VMs)
~/data/       ← per-VM isolated storage (unique to this VM)
```

- Edit files on your Mac in the work directory — changes appear instantly in `~/work/`
- Store VM-specific state (databases, build caches, etc.) in `~/data/`
- Both use virtiofs — instant sync, no lag

---

## 3. Daily Workflow

```bash
# Start + enter (shell auto-starts a stopped VM)
glowbox shell

# Inside the VM
cd ~/work                  # your synced working directory
cd ~/data                  # per-VM storage

# End of day
exit
glowbox stop
```

---

## 4. Running Microservices

### Port forwarding is automatic

Any port 1024–65535 the VM listens on is forwarded 1:1 to localhost on your Mac.

```bash
# Inside the VM:
docker run -d -p 8080:80 nginx
# → http://localhost:8080 on your Mac

docker compose up -d
# → all exposed ports just work
```

### Services must bind to 0.0.0.0

```bash
# Good — Lima can see this
node server.js --host 0.0.0.0 --port 3000

# Bad — Lima can't forward this
node server.js --host 127.0.0.1 --port 3000
```

---

## 5. Running Claude Code Agents

```bash
cd ~/work/my-project

# Interactive
claude

# Headless / unsupervised
claude -p "implement the feature described in TODO.md" --allowedTools "Edit,Write,Bash" --yes
```

---

## 6. Installing Tools with mise

```bash
mise use node@22
mise use python@3.12
mise use go@1.22
mise install              # from project .mise.toml / .tool-versions
```

---

## 7. Multiple VMs

```bash
glowbox create heavy --cpus 8 --memory 16GiB
glowbox create light --cpus 2 --memory 4GiB
glowbox shell heavy
glowbox shell light
glowbox list
```

All VMs share the same `~/work` mount. Each gets its own `~/data` directory (backed by `~/glowbox/<name>` on the host).

---

## 8. Command Reference

| Command | What it does |
|---------|-------------|
| `glowbox init` | Set up `glowbox.yaml` config in current directory |
| `glowbox create [name] [--cpus N] [--memory SIZE] [--disk SIZE]` | Create and provision a VM |
| `glowbox shell [name]` | Enter VM (auto-starts if stopped) |
| `glowbox start [name]` | Start a stopped VM |
| `glowbox stop [name]` | Stop a running VM |
| `glowbox list` | List all VMs |
| `glowbox status [name]` | Detailed VM info (JSON) |
| `glowbox logs [name]` | Tail provisioning log |
| `glowbox delete [name]` | Delete VM (with confirmation) |
| `glowbox nuke [name]` | Force delete |

Default name is `glowbox` — omit `[name]` to use it.

---

## 9. Troubleshooting

### Docker permission denied

Exit and re-enter the VM. If that doesn't work, stop/start it:
```bash
glowbox stop && glowbox start
glowbox shell
```

### Port not accessible from Mac

1. Check the service binds to `0.0.0.0` (not `127.0.0.1`)
2. Check the port isn't already taken on the host
3. Port must be in range 1024–65535

### Provisioning seems incomplete

```bash
glowbox logs
# or inside the VM:
sudo cloud-init status
```

### Need to start over

```bash
glowbox nuke
glowbox create
```

Work and data files are never affected — they live on the host.
