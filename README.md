# hsync - WIP

A lightweight file synchronization tool for working with remote HPC clusters. Built for personal use to streamline the workflow between local Neovim development and remote cluster execution.

> **Note:** This tool was created for my personal workflow and is shared as-is. It works well for my use case (local Neovim editing + remote HPC execution), but may need adaptation for yours.

## Features

- 🎯 **Smart auto-sync** - Automatically detects which direction to sync based on timestamps
- 🚀 **Fast & efficient** - Uses rsync with compression and smart exclusions
- 🔧 **Per-project configuration** - Each project maintains its own `.hsync.conf`
- 🔒 **Safe by default** - Dry-run mode to preview changes
- 📦 **Portable** - Single bash script, works anywhere
- 💻 **Perfect for Neovim** - Edit locally, execute remotely

## Use Case

I use this tool for HPC development where I:
1. Edit code locally in Neovim on my laptop
2. Sync changes to remote HPC clusters
3. Run computations and jobs on the cluster
4. Pull results back to local machine for analysis

The workflow is optimized for quick iteration: edit → sync → run → analyze.

## Quick Start

### Installation

```bash
# Install
git clone git@github.com:dnyse/hsync.git
chmod +x ~/.local/bin/hsync
```

Make sure `~/.local/bin` is in your `PATH`:
```bash
export PATH="$HOME/.local/bin:$PATH"
```

### Basic Usage

```bash
# Initialize in your project directory
cd my-project
hsync init

# Check sync status
hsync status

# Auto-detect and sync
hsync auto

# Or explicitly push/pull
hsync push  # Local → Remote
hsync pull  # Remote → Local
```

## Configuration

### SSH Config

First, configure your remote host in `~/.ssh/config`.

### Project Config

After running `hsync init`, edit `.hsync.conf` in your project root:

```bash
# Remote host (as defined in ~/.ssh/config)
REMOTE_HOST="fritz"

# Remote path (can use ~ for home directory or absolute path)
REMOTE_PATH="~/projects/my-project"
# Or use absolute path to avoid tilde expansion issues:
REMOTE_PATH="/home/username/projects/my-project"

# Additional excludes (one per line)
EXCLUDES+=(
    "data/"
    "*.log"
    "results/"
)

# Sync options
DELETE=true          # Delete files on destination that don't exist on source
COMPRESS=true        # Use compression during transfer
VERBOSE=false        # Verbose output

# Advanced: Custom rsync options
# RSYNC_OPTS="--partial --progress"
```

## Commands

| Command | Description |
|---------|-------------|
| `hsync init` | Initialize configuration in current directory |
| `hsync status` | Show sync status and suggest action |
| `hsync auto` | Auto-detect direction and sync (recommended) |
| `hsync push` | Push local changes to remote |
| `hsync pull` | Pull remote changes to local |
| `hsync help` | Show help message |

### Options

| Option | Description |
|--------|-------------|
| `-n, --dry-run` | Show what would be transferred without making changes |
| `-v, --verbose` | Verbose output |
| `--no-delete` | Don't delete files on destination |
| `--no-compress` | Don't use compression |

## Examples

### Basic Workflow

```bash
# Edit code locally in Neovim
nvim my_code.py

# Sync to HPC cluster
hsync push

# SSH to cluster and run job
ssh fritz
sbatch job.sh
exit

# Pull results back
hsync pull

# Or let hsync decide (recommended)
hsync auto
```

### Preview Changes

```bash
# Dry-run to see what would be synced
hsync push -n
hsync pull -n --verbose
```

### Custom Sync Options

```bash
# Verbose push without deleting remote files
hsync push -v --no-delete

# Pull without compression (faster for local networks)
hsync pull --no-compress
```

## Default Exclusions

hsync automatically excludes common temporary and build files:

- `.git/` - Git repository data
- `__pycache__/`, `*.pyc`, `*.pyo` - Python bytecode
- `*.o`, `*.out`, `*.so` - Compiled objects
- `slurm-*.out` - SLURM job output files
- `build/`, `venv/`, `.venv/` - Build and virtual environment directories
- `*.swp`, `*.swo`, `*~` - Editor temporary files
- `.DS_Store` - macOS metadata

Add project-specific exclusions in `.hsync.conf`.

### Tilde expansion issues

**Cause:** Remote home directory lookup requires SSH connection.

**Solution:** Use absolute paths in `.hsync.conf`:
```bash
# Instead of:
REMOTE_PATH="~/projects/my-project"

# Use:
REMOTE_PATH="/home/username/projects/my-project"
```

### Remote directory doesn't exist

**Solution:** hsync will create it automatically on first push:
```bash
hsync push
```

## How It Works
1. **Configuration** - Reads `.hsync.conf` from current directory
2. **Direction detection** - Compares last modification times (for `auto`)
3. **Sync** - Uses rsync with compression, exclusions, and safety options
4. **State tracking** - Saves sync metadata in `.hsync.state`
