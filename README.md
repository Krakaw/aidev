# aidev

AI-powered parallel development tool that combines git worktrees with tmux to enable developers to work on multiple features simultaneously with AI coding assistants.

## Overview

`aidev` streamlines the workflow of developing multiple features in parallel by:

- Creating isolated git worktrees for each feature branch
- Managing tmux windows for each development session
- Automatically launching AI coding tools (opencode, amp, claude, etc.) in each session
- Enabling parallel development without context switching between branches

## Prerequisites

- **Git** - For worktree management
- **tmux** - For terminal multiplexing/window management
- **An AI coding tool** - Such as [opencode](https://opencode.ai), amp, claude, etc.

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/Krakaw/aidev.git
   ```

2. Make the script executable and add to your PATH:
   ```bash
   chmod +x aidev/aidev
   ln -s "$(pwd)/aidev/aidev" /usr/local/bin/aidev
   ```

## Usage

```bash
aidev <command> [options]
```

### Commands

| Command | Description |
|---------|-------------|
| `new\|add <feature> [base-branch]` | Create new worktree, tmux window, and launch AI tool |
| `list\|ls` | List all active feature sessions |
| `attach\|a <feature>` | Attach to an existing feature session |
| `cleanup\|clean\|rm <feature> [--force]` | Remove worktree and close tmux window |
| `cleanup-all\|clean-all\|rm-all [--force]` | Remove all worktrees and windows |
| `help` | Show help message |

### Examples

```bash
# Start working on a new feature
aidev new add-user-auth

# Start a feature based on a different branch
aidev new hotfix-login release/v2

# List all active features
aidev list

# Switch to an existing feature
aidev attach add-user-auth

# Clean up when done
aidev cleanup add-user-auth

# Force cleanup without confirmation
aidev cleanup add-user-auth --force

# Clean up all features
aidev cleanup-all
```

## Workflow

1. Navigate to your git repository
2. Run `aidev new <feature>` to start working on a feature
3. A tmux session starts (if not already running), a worktree is created, and your AI tool launches
4. Work on multiple features by running `aidev new` for each one
5. Use `aidev list` to see all active features
6. Use `aidev attach <feature>` to switch between features
7. When done, merge your changes via git/GitHub and run `aidev cleanup <feature>`

## Configuration

Configure `aidev` using environment variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `AIDEV_TOOL` | `opencode` | AI tool to launch in each session |
| `AIDEV_BASE_BRANCH` | auto-detected | Base branch for new worktrees (detects from `origin/HEAD`) |
| `AIDEV_WORKTREE_DIR` | `../<repo>-worktrees` | Directory for worktrees |
| `AIDEV_SESSION_NAME` | `aidev` | Tmux session name |
| `AIDEV_SESSION_PREFIX` | `aidev` | Prefix for tmux window names |

### Example Configuration

```bash
# Add to your .bashrc or .zshrc
export AIDEV_TOOL=amp
export AIDEV_BASE_BRANCH=develop
```

## Hooks

`aidev` supports a pre-hook that runs after worktree creation but before the AI tool launches.

### Setting Up a Pre-Hook

1. Create a file named `.aidev-pre` in your repository root
2. Make it executable: `chmod +x .aidev-pre`
3. (Optional) Add `.aidev-pre` to your `.gitignore`

### Hook Arguments

The pre-hook receives the following arguments:

| Argument | Description |
|----------|-------------|
| `$1` | Worktree path |
| `$2` | Feature/branch name |
| `$3` | Git root (original repo) |

These are also available as environment variables:
- `AIDEV_WORKTREE_PATH`
- `AIDEV_FEATURE_NAME`
- `AIDEV_GIT_ROOT`

### Example Pre-Hook

```bash
#!/usr/bin/env bash
cd "$1"

# Copy environment files
cp "$3/.env" .env 2>/dev/null

# Install dependencies
npm install
```

## How It Works

1. **Worktree Creation**: When you run `aidev new <feature>`, it creates a new git worktree in a sibling directory (e.g., `../myrepo-worktrees/feature-name/`)
2. **Branch Management**: A new branch is created from the base branch (or an existing branch is reused)
3. **Tmux Integration**: A new tmux window is created within the aidev session
4. **AI Tool Launch**: Your configured AI coding tool is launched in the worktree directory
5. **Cleanup**: When done, `aidev cleanup` removes the worktree and closes the tmux window

## License

See [LICENSE](LICENSE) for details.
