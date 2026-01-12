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

## Claude Code Integration

If you use [Claude Code](https://claude.com/claude-code), you can install the included skill for an enhanced workflow:

```bash
# Copy the skill to your Claude Code skills directory
cp -r .claude/skills/worktree ~/.claude/skills/

# Restart Claude Code to load the skill
```

Now Claude can help you manage worktrees naturally:

```
"Create a worktree for my new authentication feature"
"Show me my current worktrees"
"Make the .env files independent in my experiment branch"
```

Or invoke the skill directly with `/worktree`

The skill provides Claude with:
- Automatic detection of when to use worktrees
- Knowledge of common workflows (parallel dev, hotfixes, experiments)
- Understanding of .env syncing and materialization
- Troubleshooting guidance

See `.claude/skills/worktree/SKILL.md` for full details.

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

**Q: Are the tools safe for concurrent operations?**
A: Yes! The scripts use file-based locking to prevent race conditions when running multiple operations simultaneously. See [Concurrent Operations](#concurrent-operations-and-locking) for details.

**Q: How do I secure my environment files?**
A: The tools validate permissions and warn you if `.env` files are world-readable. See [Security Considerations](#security-considerations) for best practices.

## Security Considerations

### Environment File Permissions

By default, `.env` files are symlinked from your main repository to all worktrees. This means:

- All worktrees share the same sensitive configuration
- File permissions apply to all worktrees
- Changes in one worktree affect all others

**Best Practices:**

```bash
# Set restrictive permissions on .env files
chmod 600 .env*

# Verify permissions
ls -la .env*

# Or use independent copies for sensitive worktrees
worktree-create --copy-env sensitive-feature
```

The `worktree-create` script will automatically check file permissions and warn you if `.env` files are world-readable (permissions > 600). You'll be prompted to continue or abort.

### Input Validation

Branch names are validated according to git's rules. The following characters are **prohibited**:
- `\` (backslash)
- `~` (tilde)
- `^` (caret)
- `:` (colon)
- `?` (question mark)
- `*` (asterisk)
- `[` `]` (brackets)
- `@{` (at-brace sequence)

Branch names also cannot:
- Start or end with `/` or `.`
- Contain `..` or `//` sequences
- Be empty or exceed 255 characters

### Concurrent Operations and Locking

The tools use file-based locking to prevent race conditions when multiple operations run simultaneously:

```bash
# Lock file location
.git/worktree-tools.lock

# If you see "lock file exists" error:
# 1. Check if another operation is running
ps aux | grep worktree

# 2. If not, remove the stale lock
rm .git/worktree-tools.lock
```

The lock is automatically released when operations complete or if the process is interrupted.

### Error Rollback

If worktree creation fails partway through (e.g., during `.env` file setup), the script automatically:
- Removes the partially created worktree
- Deletes any newly created branches
- Leaves your repository in a clean state

No manual cleanup required!

## Environment Variables

Customize the tools' behavior with environment variables:

### WORKTREE_ENV_PATTERN

Override which files get synced (default: `.env*`):

```bash
# Only sync the main .env file
WORKTREE_ENV_PATTERN='.env' worktree-create my-feature

# Sync config files instead
WORKTREE_ENV_PATTERN='config.*' worktree-create my-feature

# Also works with worktree-manage
WORKTREE_ENV_PATTERN='config.*' worktree-manage list

# Only sync production env files
WORKTREE_ENV_PATTERN='.env.production*' worktree-create prod-test

# Make it permanent in your shell profile
echo 'export WORKTREE_ENV_PATTERN=".env"' >> ~/.bashrc
```

## Troubleshooting

### "Lock file exists" Error

**Symptom:**
```
Error: Another worktree operation is in progress
If you're sure no other operation is running, remove: .git/worktree-tools.lock
```

**Cause:** Another worktree operation is running, or a previous operation crashed without releasing the lock.

**Solution:**
```bash
# Check if another operation is actually running
ps aux | grep worktree-create
ps aux | grep worktree-manage

# If nothing is running, remove the lock file
rm .git/worktree-tools.lock

# Try your operation again
```

### "Permission denied" on .env Files

**Symptom:**
```
Warning: .env is world-readable (permissions: 644)
```

**Cause:** Your `.env` files have loose permissions that could expose sensitive data.

**Solution:**
```bash
# Secure your .env files
chmod 600 .env*

# Verify the change
ls -la .env*

# Should show: -rw------- (600)
```

### Symlink Points to Wrong Location

**Symptom:** `.env` file in worktree points to incorrect path or broken symlink.

**Cause:** Worktree was moved, or original `.env` was deleted/renamed.

**Solution:**
```bash
# Option 1: Materialize to make it independent
worktree-manage materialize <branch-name>

# Option 2: Recreate the symlink manually
cd .worktrees/<branch-name>
rm .env
ln -sf ../../.env .env

# Option 3: Recreate the worktree
cd /path/to/main/repo
worktree-manage remove <branch-name>
worktree-create <branch-name>
```

### "Branch name invalid" Error

**Symptom:**
```
Error: Invalid branch name: feature/../../../etc/passwd
Branch names cannot contain: \ ~ ^ : ? * [ ] @{
```

**Cause:** Branch name contains prohibited characters or invalid sequences.

**Solution:**
```bash
# Use only alphanumeric characters, dashes, and underscores
worktree-create feature-auth     # ✓ Good
worktree-create feature_auth     # ✓ Good
worktree-create feature/auth     # ✓ Good (single slash OK)

# Avoid special characters
worktree-create "feature auth"   # ✗ Bad (spaces)
worktree-create feature:auth     # ✗ Bad (colon)
worktree-create feature..auth    # ✗ Bad (double dots)
```

### Worktree Shows Old/Stale Environment Variables

**Symptom:** Environment variables in worktree don't match the main repo.

**Cause:** `.env` file was materialized (converted from symlink to real file).

**Solution:**
```bash
# Check if it's a symlink
ls -la .worktrees/<branch-name>/.env

# If it's NOT a symlink (-rw instead of lrw), recreate it
cd .worktrees/<branch-name>
rm .env
ln -sf ../../.env .env

# Or recreate the worktree entirely
worktree-manage remove <branch-name>
worktree-create <branch-name>
```

## Requirements

- Bash 4.0+
- Git 2.5+ (for `git worktree` support)
- Standard Unix tools (find, ln, cp)

## Contributing

Found a bug or have an idea? Open an issue or PR!

## License

MIT License - see [LICENSE](LICENSE) file.
