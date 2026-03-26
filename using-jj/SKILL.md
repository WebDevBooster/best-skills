---
name: using-jj
description: Workspace-first Jujutsu workflow for coding agents. Use when working in a jj repo, coordinating parallel agent work, moving changes between workspaces, landing selected work onto trunk, publishing bookmarks, or recovering from rewrites and stale workspaces.
version_target: "0.39.x"
---

# Using jj for parallel coding agents

Use this skill when the repository is using **Jujutsu (`jj`) as the primary mutating VCS interface**.

This skill is opinionated. It is written for **coding agents**, not for humans exploring jj casually.

Default assumptions:
- trunk bookmark is `main`
- default remote is `origin`
- one agent = one `jj workspace`
- use `jj` for mutations, not `git`
- prefer non-interactive commands

If the repo clearly uses a different trunk bookmark, use that instead of `main`.

## Quick start

```bash
# 1) Sync remote state
jj git fetch

# 2) Create a dedicated workspace for this agent
jj workspace add ../repo-agent-<name> --name agent-<name> -r main@origin -m "agent-<name>: start from main"

# 3) Enter the workspace and inspect state
cd ../repo-agent-<name>
jj status
jj log -r 'main@origin | main | @ | @- | bookmarks()'

# 4) Start the change by describing it
jj describe -m "feat: implement <task>"

# 5) Edit files, then commit non-interactively
jj commit -m "feat: implement <task>"

# 6) Publish quickly if asked
jj git push --change @-
```

## Non-negotiable rules

1. Treat `jj workspace` as the unit of parallel agent work.
2. Use `jj` for mutations. Do not mix mutating `git` commands into a jj workflow unless the user explicitly asks for that.
3. Never assume there is a current bookmark. jj does not have one.
4. Use **descriptions first**. In normal agent work, start by naming the change with `jj describe -m ...`.
5. Prefer non-interactive commands. Add `-m` when supported.
6. After `jj commit -m ...`, the finished change is usually `@-`, not `@`.
7. When taking work from another workspace and you want to keep the source intact, use `jj duplicate`.
8. Use `jj squash --from ... --into ...` only when you intentionally want to adopt or consume that source change.
9. If a workspace becomes stale, repair it with `jj workspace update-stale`.
10. If a rewrite goes wrong, recover with `jj undo`. If needed, inspect `jj op log` and use `jj op restore`.

## The mental model you must use

### Think in changes, not branches

jj is not centered around “the current branch”. It is centered around **changes** and a **working-copy commit**.

- `@` = current workspace’s working-copy commit
- `@-` = parent of the current working-copy commit
- `<workspace>@` = another workspace’s working-copy commit
- `<workspace>@-` = that workspace’s most recently committed change in the common case

### Think describe-first

In this skill, treat jj as a **describe-first workflow**:
- first name the change with `jj describe -m "..."`
- then edit files
- then seal that state with `jj commit -m "..."`
- after commit, continue from the fresh working-copy commit at `@`

The practical shortcut is that `jj commit` without path arguments or `--interactive` is effectively `jj describe` followed by `jj new`, so change description comes first in the normal flow.

### Bookmarks are explicit public names

Bookmarks are for naming work that needs to be pushed, reviewed, or followed later.

Do not expect bookmarks to move automatically like Git branches. Move them explicitly with:
- `jj bookmark set ... -r ...`
- `jj bookmark move ... --to ...`
- `jj bookmark advance ... --to ...`

### Conflicts are state, not emergencies

jj can record conflicts directly in commits. That means a rebase can succeed while leaving a conflicted commit behind for later resolution.

Do not panic when conflicts appear. Inspect, resolve, continue.

## Standard flow for agent work

### 1) Inspect first

Run these before mutating anything important:

```bash
jj status
jj log -r 'main@origin | main | @ | @- | bookmarks()'
jj workspace list
```

### 2) Start the task in a dedicated workspace

Preferred:

```bash
jj git fetch
jj workspace add ../repo-agent-<name> --name agent-<name> -r main@origin -m "agent-<name>: start from main"
```

Then:

```bash
cd ../repo-agent-<name>
jj status
jj describe -m "feat: <task>"
```

### 3) Implement the work

```bash
# edit files
jj diff
jj commit -m "feat: <task>"
```

After that, the completed change is typically `@-`.

### 4) Reshape the work if needed

Use the least destructive command that fits the job.

#### Fixups across a stack

```bash
# edit files in the current workspace
jj absorb
```

Use this when you changed files that logically belong in earlier commits.

#### Move one whole change into another

```bash
jj squash --from <source> --into <destination> --use-destination-message
```

Use this when you intentionally want the destination to absorb the source.

#### Split a mixed change by files

```bash
jj split path/to/file1 path/to/file2 -m "extract <part>"
```

Use this only when the split can be described by filesets and done non-interactively.

#### Rebase onto fresh trunk

```bash
jj git fetch
jj rebase -r @- -o main@origin
```

Use this when your change should be replayed onto current trunk.

### 5) Take work from another workspace

#### Safe copy mode

Use when you want another agent’s work without rewriting their source change.

```bash
jj duplicate agent-a@- -A @
```

Or duplicate after trunk:

```bash
jj duplicate agent-a@- -A main@origin
```

#### Adopt mode

Use when you are intentionally taking ownership of the source change.

```bash
jj squash --from agent-a@- --into @ --use-destination-message
```

Warning: this can leave the source workspace stale.

### 6) Publish

Fast route:

```bash
jj git push --change @-
```

Named bookmark route:

```bash
jj bookmark set agent/<task> -r @-
jj git push --bookmark agent/<task>
```

### 7) Land selected work onto `main`

Use a dedicated integration workspace.

```bash
jj git fetch
jj workspace add ../repo-integrate --name integrate -r main@origin -m "integrate selected agent work"
cd ../repo-integrate
jj squash --from agent-a@- --into @ --use-destination-message
jj squash --from agent-b@- --into @ --use-destination-message
jj commit -m "integrate selected agent work"
jj bookmark set main -r @-
```

Push `main` only if explicitly requested:

```bash
jj git push --bookmark main
```

## Commands by job

### Inspect and understand

```bash
jj status
jj log -r 'main@origin | main | @ | @- | bookmarks()'
jj show @-
jj diff
jj evolog -r @-
jj workspace list
jj workspace root --name agent-a
```

### Start and name work

```bash
jj describe -m "feat: <task>"
jj new -m "next: <task>"
jj commit -m "feat: <task>"
```

### Move and reshape work

```bash
jj absorb
jj squash --from <source> --into <destination> --use-destination-message
jj split <filesets> -m "extract <part>"
jj duplicate <rev> -A @
jj rebase -r <rev> -o main@origin
```

### Workspaces

```bash
jj workspace add ../repo-agent-a --name agent-a -r main@origin -m "agent-a: start from main"
jj workspace list
jj workspace root --name agent-a
jj workspace update-stale
jj workspace forget agent-a
```

### Bookmarks and publishing

```bash
jj bookmark set agent/<task> -r @-
jj bookmark move main --to @-
jj bookmark advance main --to @-
jj git push --change @-
jj git push --bookmark agent/<task>
```

### Recovery

```bash
jj undo
jj redo
jj op log
jj op restore <operation-id>
jj resolve
```

## Decision rules

### Prefer `jj describe` when
- you are starting or renaming the current change
- you need to update the description without opening an editor

### Prefer `jj commit` when
- you want to finalize the current working-copy change and continue from a fresh working-copy commit

### Prefer `jj duplicate` when
- another workspace’s change should stay intact
- you want a safe local copy before integration

### Prefer `jj squash` when
- you want one change to be absorbed into another
- you are intentionally consuming the source change into the destination

### Prefer `jj absorb` when
- you made follow-up edits that logically belong to older commits

### Prefer `jj bookmark set` when
- you need to create or point a bookmark by name

### Prefer `jj bookmark advance` when
- a bookmark should follow rewritten work forward to a new target

## Hard “do not do this” rules

- Do not default to Git worktrees. Use jj workspaces.
- Do not assume `main` moved just because you committed. It did not.
- Do not point bookmarks at `@` after `jj commit` unless you really want the fresh empty working-copy commit.
- Do not mix `git commit`, `git rebase`, `git branch -f`, or `git worktree` into a normal jj workflow.
- Do not use `jj split` for a non-file-based split unless the user explicitly wants an interactive/editor flow.
- Do not run `jj git push --all` as a lazy default. Push the intended bookmark or change.
- Do not rewrite another agent’s work unless the workflow explicitly allows adopt mode.

## Specific tasks

- **Mental model and describe-first**: `references/mental-model.md`
- **Workspace setup and agent coordination**: `references/workspaces.md`
- **Landing work onto trunk**: `references/landing-on-main.md`
- **Recovery, stale workspaces and conflicts**: `references/recovery.md`
- **Compatibility and mixed Git/JJ warnings**: `references/compatibility-notes.md`
- **Copy-paste recipes**: `references/recipes.md`
