---
name: using-jj
description: Use this skill for any task involving jj or Jujutsu, especially multi-agent workflows with parallel workspaces and landing selected changes onto main. Trigger when the user mentions jj, jujutsu, revsets, change IDs, bookmarks, oplog, workspaces, stale workspaces, absorb, squash, split, rebase, or parallel agent work.
version_target: "0.39.x"
---

# JJ workflow for coding agents

This skill assumes:
- trunk bookmark is `main`
- default remote is `origin`
- one agent = one `jj workspace`
- local work should prefer `jj` commands over `git` commands

Read `references/recipes.md` for copy-paste command sequences.

## Hard rules

1. Prefer `jj workspace`, not Git worktrees.
2. Always use non-interactive commands. Add `-m` to `jj new`, `jj desc` / `jj describe`, `jj commit`, `jj squash`, and `jj split` when applicable.
3. Never assume a current bookmark exists. jj does not have one. Move or set bookmarks explicitly.
4. Treat bookmarks as public names for pushing and PRs. For local coordination, prefer workspace names, change IDs and revsets.
5. Use `@` for the current working-copy commit. After `jj commit -m "..."`, the finished work is usually `@-` and the new `@` is a fresh working-copy commit.
6. When importing another agent's work and you want to keep their original intact, use `jj duplicate`. Do not use `jj squash --from other@-` unless you intentionally want to consume or rewrite the original change.
7. If another workspace rewrote your current change and commands complain that the working copy is stale, run `jj workspace update-stale`.
8. Prefer `jj absorb` for fixups across a stack. Prefer `jj squash` for deliberate movement of a whole change.
9. Keep stacks shallow. Squash or abandon junk early.
10. If a rewrite goes wrong, recover with `jj undo`. If the repo is badly wrong, inspect `jj op log` and use `jj op restore <operation-id>`.

## Mental model

- `@` is the current working-copy commit.
- `@-` is the parent of the current working-copy commit.
- `<workspace>@` is another workspace's current working-copy commit.
- `<workspace>@-` is usually that workspace's most recently committed change.
- Change IDs stay stable across rewrites. Commit IDs do not.
- Conflicts are state, not emergencies. jj stores them in commits and lets you resolve them later.

## Default workflow for agent work

### 1) Start or refresh an agent workspace
From an existing workspace in the repo:

```bash
jj git fetch
jj workspace add ../repo-agent-a --name agent-a -r main@origin -m "agent-a: start from main"
```

If `main@origin` is not available locally yet, use `main` after a fetch.

### 2) Do the work
Inside the new workspace:

```bash
jj st
jj desc -m "feat: implement X"
# edit files
jj commit -m "feat: implement X"
```

After `jj commit`, the change to share is usually `agent-a@-`.

### 3) Inspect other agents' work

```bash
jj log -r 'main@origin | agent-a@ | agent-b@ | agent-c@'
jj show agent-a@-
jj diff --from main@origin --to agent-a@-
```

### 4) Import another agent's work

#### Safe copy mode
Use this when you want to test or integrate another agent's change without rewriting their original:

```bash
jj duplicate agent-a@- -A @
```

You can also copy it directly after trunk:

```bash
jj duplicate agent-a@- -A main@origin
```

#### Adopt mode
Use this when you are taking ownership of the change and it is fine for the source workspace to become stale:

```bash
jj squash --from agent-a@- --into @ --use-destination-message
```

After adopt mode, the source workspace may need `jj workspace update-stale`.

### 5) Curate the result

- Use `jj absorb` when you fixed several older commits from the top of a stack.
- Use `jj squash` to move a whole change into its destination.
- Use `jj split <filesets> -m "..."` only when the split is file-based and can be done non-interactively.
- Avoid interactive editor flows unless the user explicitly wants them.

### 6) Publish

Fastest route for one change:

```bash
jj git push --change @-
```

Named bookmark route:

```bash
jj bookmark set agent/feature-x -r @-
jj git push --bookmark agent/feature-x
```

### 7) Land selected work onto `main`

Prefer a curated integration workspace. Make one empty integration change on top of trunk, adopt the selected agent changes into it, then commit and move `main` to the landed commit.

```bash
jj workspace add ../repo-integrate --name integrate -r main@origin -m "integrate selected agent work"
cd ../repo-integrate
jj squash --from agent-a@- --into @ --use-destination-message
jj squash --from agent-b@- --into @ --use-destination-message
jj commit -m "integrate selected agent work"
jj bookmark set main -r @-
```

Only push `main` if the user explicitly wants that:

```bash
jj git push --bookmark main
```

## Common mistakes to avoid

- Do not use `master` in examples. Use `main` unless the repo clearly uses another trunk bookmark.
- Do not assume `jj git push --all` will push unpublished work. It pushes bookmarks, not arbitrary revisions.
- Do not point a bookmark at `@` right after `jj commit` unless you really want the fresh working-copy commit instead of the finished change in `@-`.
- Do not use Git worktree commands as the main coordination mechanism in a jj repo. Use `jj workspace`.
- Do not panic when a rebase creates conflicts. Resolve them in the affected commit and continue working.

## Recommended troubleshooting sequence

```bash
jj st
jj log -r 'main@origin | main | @ | @- | bookmarks()'
jj op log --limit 10
```

If the working copy is stale:

```bash
jj workspace update-stale
```

If the last rewrite was wrong:

```bash
jj undo
```

If you need a previous repo state:

```bash
jj op log
jj op restore <operation-id>
```
