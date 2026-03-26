# Compatibility notes and mixed Git/JJ warnings

This skill assumes jj is the main mutating interface.

## Do not default to mixed mutation

Avoid normal mixes like:
- `git commit` plus `jj rebase`
- `git worktree` plus `jj workspace`
- `git branch -f` plus `jj bookmark move`

If the user explicitly wants a mixed setup, slow down and verify every step.

## Why this matters

Interleaving mutating `git` and `jj` commands can create confusing branch conflicts or divergent states.

## Repos that need extra caution

Be careful if the repo depends on features jj does not fully support in the same way.

Known limitations to keep in mind:
- Git hooks are not a jj feature
- Git submodules are not supported as a normal working-copy feature
- Git LFS is not supported

If the repo critically depends on those, prefer a conservative workflow and tell the user what the limitation is.

## Colocation note

A repo can be colocated with Git, but that does not mean agents should freely alternate between mutating Git and mutating jj commands.

Use Git mainly as:
- remote transport
- hosting integration
- external tooling that only needs read-only Git state

Use jj for:
- commits and descriptions
- rebases
- squashes
- duplication
- workspaces
- bookmark movement
- recovery
