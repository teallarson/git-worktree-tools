---
name: worktree
description: Manage git worktrees with automatic .env file syncing for parallel branch development
---

# Git Worktree Management

## Purpose
Streamline working with git worktrees by providing easy access to worktree creation, management, and .env file syncing. Enables developers to work on multiple branches simultaneously without stashing or branch switching.

## When to Use
- User wants to work on multiple branches in parallel
- Need to test a hotfix without disrupting current work
- Running multiple branches for comparison or testing
- Setting up isolated environments with different configs
- User mentions "worktree", "parallel branches", or "multiple features"

## Prerequisites
The worktree management scripts should be available in your PATH:
- `worktree-create` - Creates worktrees with .env syncing
- `worktree-manage` - Manages existing worktrees

Users should install these tools from https://github.com/teallarson/git-worktree-tools and add them to their PATH, or use the full path to the scripts when executing commands.

## Core Concepts

### .env Syncing
By default, `.env*` files are **symlinked** from the main repo to worktrees:
- Changes to .env in any worktree sync automatically to all others
- No duplication of environment variables
- Always consistent across all worktrees

To make .env files independent, use the `materialize` command to convert symlinks to real files.

## Instructions

### Creating a Worktree

Use the `worktree-create` script:

```bash
worktree-create [OPTIONS] <branch-name>
```

**Common options:**
- `-b, --base <branch>` - Base branch to create from (default: main)
- `-d, --dir <path>` - Worktree root directory (default: .worktrees)
- `-c, --copy-env` - Copy .env files instead of symlinking
- `-h, --help` - Show help message

**Default behavior:**
- Creates worktree in `.worktrees/<branch-name>`
- Creates new branch from `main`
- Symlinks all `.env*` files from main repo

### Listing Worktrees

Use the `worktree-manage list` command:

```bash
worktree-manage list
```

Shows:
- All worktrees with their paths
- Number of .env files in each
- How many are symlinked vs real files

### Materializing Symlinks

Convert symlinked .env files to real, independent copies:

```bash
# Materialize one worktree
worktree-manage materialize <branch-name>

# Materialize all worktrees
worktree-manage materialize-all
```

Use when:
- Need different environment configs in each worktree
- Want to test with independent .env settings
- No longer want changes to sync across worktrees

### Removing Worktrees

Use the `worktree-manage remove` command:

```bash
worktree-manage remove <branch-name>
```

This will:
- Remove the worktree directory
- Prompt to delete the associated branch

## Common Workflows

### Workflow 1: Parallel Feature Development

**Scenario:** User wants to work on two features simultaneously

**Steps:**
1. Create worktrees for each feature:
   ```bash
   worktree-create feature-payments
   worktree-create feature-notifications
   ```

2. Work in each independently:
   ```bash
   cd .worktrees/feature-payments
   # Make changes, commit, etc.

   cd ../feature-notifications
   # Work on different feature
   ```

3. .env changes sync automatically across both

### Workflow 2: Urgent Hotfix

**Scenario:** User has uncommitted work but needs to fix production bug

**Steps:**
1. Create hotfix worktree from main:
   ```bash
   worktree-create -b main hotfix-urgent-bug
   ```

2. Fix bug in isolation:
   ```bash
   cd .worktrees/hotfix-urgent-bug
   # Fix, test, commit, push
   ```

3. Return to original work - nothing was disrupted

### Workflow 3: Testing with Different Configs

**Scenario:** User wants to test with different environment settings

**Steps:**
1. Create worktree with independent .env:
   ```bash
   worktree-create --copy-env experiment-config
   ```

2. Or create with synced .env, then materialize:
   ```bash
   worktree-create experiment-config
   worktree-manage materialize experiment-config
   ```

3. Modify .env in the worktree independently:
   ```bash
   cd .worktrees/experiment-config
   echo "DEBUG=true" >> .env
   ```

### Workflow 4: Cleanup

**Scenario:** User finished with worktrees and wants to clean up

**Steps:**
1. List what exists:
   ```bash
   worktree-manage list
   ```

2. Remove completed worktrees:
   ```bash
   worktree-manage remove feature-payments
   worktree-manage remove hotfix-urgent-bug
   ```

3. Prompts will ask about deleting branches too

## Tips & Best Practices

### When to Use Symlinked .env
- Most development scenarios
- Want consistency across all worktrees
- Single source of truth for environment config
- Team shares same environment setup

### When to Materialize .env
- Testing different configurations
- Experimentation with settings
- Debugging environment-specific issues
- Need isolation between worktrees

### Naming Conventions
Suggest clear, descriptive branch names:
- Feature work: `feature-<name>` or `feat-<name>`
- Bug fixes: `fix-<name>` or `bugfix-<name>`
- Hotfixes: `hotfix-<name>`
- Experiments: `experiment-<name>` or `exp-<name>`

### Directory Structure
Default structure:
```
main-repo/
  .env
  .worktrees/
    feature-auth/
      .env -> ../../.env
    feature-ui/
      .env -> ../../.env
```

Recommend adding `.worktrees/` to `.gitignore`.

## Troubleshooting

### Scripts Not Found
If scripts aren't found, verify they are in your PATH:
```bash
which worktree-create worktree-manage
```

If not in PATH, either:
- Add the installation directory to your PATH
- Use the full path to the scripts (e.g., `/path/to/git-worktree-tools/worktree-create`)

### Permission Denied
Make scripts executable:
```bash
chmod +x /path/to/git-worktree-tools/worktree-*
```

### Base Branch Doesn't Exist
User specified a base branch that doesn't exist. Check available branches:
```bash
git branch -a
```

### Worktree Already Exists
Directory already exists at the target location. Either:
- Choose a different name
- Remove the existing worktree first
- Use a different directory with `-d` flag

## Integration with Claude Code

When user requests worktree operations:
1. Determine which operation they need (create, list, materialize, remove)
2. Gather required parameters (branch name, base branch, etc.)
3. Execute the appropriate script with Bash tool
4. Explain what was done and next steps
5. Suggest related operations if helpful

Always show the full command being run for transparency.

## Related Commands

Standard git worktree commands (if scripts unavailable):
```bash
git worktree list
git worktree add <path> <branch>
git worktree remove <path>
```

The custom scripts enhance these with .env syncing and better UX.

## Examples

### Example 1: Simple Feature Branch
```
User: "I want to work on a new authentication feature"

Response: I'll create a worktree for the authentication feature.
[Execute: worktree-create feature-auth]

Your worktree is ready at .worktrees/feature-auth with .env files synced.
To start working:
  cd .worktrees/feature-auth
```

### Example 2: Hotfix from Main
```
User: "I need to fix a production bug but I have uncommitted changes"

Response: I'll create a hotfix worktree from the main branch so your current work isn't disrupted.
[Execute: worktree-create -b main hotfix-login-bug]

Your hotfix worktree is ready. You can work on the fix while your original work remains untouched.
```

### Example 3: List and Clean Up
```
User: "What worktrees do I have?"

Response: Let me list your worktrees.
[Execute: worktree-manage list]

You have 3 worktrees. Would you like to remove any that are no longer needed?
```

### Example 4: Independent Configs
```
User: "I want to test with different database settings"

Response: I'll create a worktree with independent .env files so you can modify settings without affecting other worktrees.
[Execute: worktree-create --copy-env experiment-db-config]

Your worktree has independent .env files. You can now modify environment variables without affecting other worktrees.
```

## Notes
- Source code: https://github.com/teallarson/git-worktree-tools
- Scripts require Git 2.5+ for worktree support
- Always verify the user is in a git repository before executing commands
- If scripts are not in PATH, Claude should use the full path provided by the user or check common installation locations
