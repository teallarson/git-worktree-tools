# Git Worktree Tools

A pair of bash scripts to streamline working with git worktrees and environment files.

## What's This For?

Git worktrees let you work on multiple branches simultaneously without stashing or switching. These tools make it easy to:

- Create worktrees with a single command
- Automatically sync `.env` files (and other config) between worktrees
- Manage multiple worktrees efficiently
- Convert symlinked config files to independent copies when needed

Perfect for:
- Working on multiple features in parallel
- Testing hotfixes without disrupting your main work
- Running different branches for comparison
- Experimenting without affecting your main codebase

## Installation

```bash
# Clone this repo
git clone https://github.com/yourusername/git-worktree-tools.git
cd git-worktree-tools

# Make scripts executable (if not already)
chmod +x worktree-create worktree-manage

# Optional: Add to your PATH
# Add this to your ~/.bashrc or ~/.zshrc:
export PATH="$PATH:/path/to/git-worktree-tools"
```

## Quick Start

```bash
# In your git repository
cd /path/to/your/repo

# Create a worktree for a new feature
/path/to/worktree-create feature-auth

# Your new worktree is ready at .worktrees/feature-auth
# with .env files automatically synced!
cd .worktrees/feature-auth
```

## Usage

### Creating Worktrees

```bash
# Basic usage - creates worktree from 'main'
worktree-create my-feature

# Create from a different base branch
worktree-create -b develop my-feature

# Use custom worktree location
worktree-create -d ../worktrees my-feature

# Copy .env files instead of symlinking
worktree-create --copy-env my-feature
```

**Options:**
- `-b, --base <branch>` - Base branch to create from (default: main)
- `-d, --dir <path>` - Worktree root directory (default: .worktrees)
- `-c, --copy-env` - Copy .env files instead of symlinking
- `-h, --help` - Show help message

### Managing Worktrees

```bash
# List all worktrees with .env status
worktree-manage list

# Remove a worktree
worktree-manage remove my-feature

# Convert symlinked .env files to real files in one worktree
worktree-manage materialize my-feature

# Convert symlinked .env files in ALL worktrees
worktree-manage materialize-all
```

## How .env Syncing Works

By default, `.env*` files are **symlinked** from your main repo to worktrees:

```
main-repo/
  .env                    ← original file
  .worktrees/
    feature-auth/
      .env                ← symlink → ../../.env
    feature-ui/
      .env                ← symlink → ../../.env
```

**Benefits:**
- Changes to `.env` in any worktree sync automatically
- No duplication of environment variables
- Always consistent across all worktrees

**When you need independence:**

Use `worktree-manage materialize <name>` to convert symlinks to real files:

```bash
# Make .env files independent in one worktree
worktree-manage materialize feature-auth

# Now you can modify .env without affecting other worktrees
cd .worktrees/feature-auth
echo "DEBUG=true" >> .env  # Only affects this worktree
```

## Examples

### Example 1: Parallel Feature Development

```bash
# Working on main branch
cd ~/projects/myapp

# Need to start two new features
worktree-create feature-payments
worktree-create feature-notifications

# Work on payments
cd .worktrees/feature-payments
# ... make changes ...

# Switch to notifications (no stashing needed!)
cd ../feature-notifications
# ... make changes ...

# Check all worktrees
worktree-manage list
```

### Example 2: Hotfix While Feature In Progress

```bash
# Currently working on a large feature
cd ~/projects/myapp/.worktrees/big-feature
# ... lots of uncommitted changes ...

# Production bug! Need to fix immediately
cd ~/projects/myapp
worktree-create -b main hotfix-login-bug

# Fix the bug in isolation
cd .worktrees/hotfix-login-bug
# ... fix, commit, push ...

# Back to your feature work
cd ../big-feature
# All your changes still there, untouched!
```

### Example 3: Testing with Different Configs

```bash
# Create worktree with synced .env
worktree-create experiment-a

# Create another with independent .env
worktree-create --copy-env experiment-b

# Or materialize later
worktree-create experiment-c
worktree-manage materialize experiment-c

# Now you can test different env configs simultaneously
```

### Example 4: Cleanup

```bash
# List what you have
worktree-manage list

# Remove worktrees you're done with
worktree-manage remove feature-auth
worktree-manage remove experiment-a

# Removes the worktree and prompts to delete the branch too
```

## FAQ

**Q: What files get synced?**
A: Any file matching `.env*` recursively in your repo - `.env`, `.env.local`, `.env.production`, etc., even in subdirectories.

**Q: Can I use a different location for worktrees?**
A: Yes! Use `-d` flag: `worktree-create -d ~/my-worktrees feature-name`

**Q: What happens if I edit a symlinked .env file?**
A: Changes apply to the original file and all other worktrees that symlink it.

**Q: Can I convert back from a real file to a symlink?**
A: Not automatically, but you can manually: `ln -sf ../../.env .env`

**Q: Do I need to commit .worktrees/ to git?**
A: No! Add `.worktrees/` to your `.gitignore`.

**Q: What if my team uses a different base branch?**
A: Use `-b`: `worktree-create -b develop my-feature`

## Requirements

- Bash 4.0+
- Git 2.5+ (for `git worktree` support)
- Standard Unix tools (find, ln, cp)

## Contributing

Found a bug or have an idea? Open an issue or PR!

## License

MIT License - see [LICENSE](LICENSE) file.
