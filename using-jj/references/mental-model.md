# Mental model for jj agents

This file explains the minimum mental model an agent must use to work safely in jj.

## 1) The current thing is a working-copy commit

Do not think “I am on branch X”.

Think:
- `@` is my current working-copy commit
- `@-` is the parent of that working-copy commit
- after `jj commit -m ...`, the finished change is usually `@-`
- `@` becomes a fresh working-copy commit for continued work

## 2) Describe first

Normal agent flow:

```bash
jj describe -m "feat: implement parser"
# edit files
jj commit -m "feat: implement parser"
```

That is the default because the important unit in jj is the **change description plus diff**, not the branch name.

## 3) Change IDs are stable enough for reasoning, commit IDs are not

Rewrites are normal in jj. Rebases, squashes and other history edits can change the commit ID while preserving the logical identity of the change.

So:
- reason about **changes**
- do not treat raw commit hashes as the primary mental handle unless required
- use revsets, workspace names and bookmarks when possible

## 4) Bookmarks are explicit names, not a current branch

jj does not have a current bookmark.

That means:
- committing does not magically move `main`
- publishing is a separate step
- naming a change for review is a separate step

Common patterns:

```bash
jj bookmark set agent/feature-x -r @-
jj git push --bookmark agent/feature-x
```

Or:

```bash
jj git push --change @-
```

## 5) Workspaces are first-class

Each workspace has its own working-copy commit and can be referenced directly.

Examples:
- `agent-a@`
- `agent-a@-`
- `integrate@`

This is how agents should think about each other’s live state.

## 6) Conflicts can live inside commits

A conflict in jj is not always a stop-the-world failure.

A commit can exist in a conflicted state. You can then:
- inspect the diff
- edit files directly
- run `jj resolve`
- continue with later work if that is the intended workflow

## 7) Operations are recoverable

jj tracks repo-level operations.

Use:

```bash
jj undo
jj op log
jj op restore <operation-id>
```

This is stronger than relying only on commit-level history when an agent made a bad rewrite.
