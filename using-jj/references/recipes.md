# jj recipes for multi-agent workflows

These assume:
- trunk bookmark is `main`
- remote is `origin`
- each agent has its own named workspace

## Create an agent workspace

```bash
jj git fetch
jj workspace add ../repo-agent-a --name agent-a -r main@origin -m "agent-a: start from main"
```

## List workspaces

```bash
jj workspace list
```

## Show another workspace's root path

```bash
jj workspace root --name agent-a
```

## See all active agent tips and trunk

```bash
jj log -r 'main@origin | main | agent-a@ | agent-b@ | agent-c@'
```

## Inspect the committed result from another workspace

```bash
jj show agent-a@-
jj diff --from main@origin --to agent-a@-
```

## Copy another agent's work without rewriting it

```bash
jj duplicate agent-a@- -A @
```

## Adopt another agent's work into the current change

This rewrites the source revision and may stale the source workspace.

```bash
jj squash --from agent-a@- --into @ --use-destination-message
```

## Rebase an agent's change onto latest trunk

Use only when you intend to rewrite that change.

```bash
jj git fetch
jj rebase -r agent-a@- -o main@origin
```

## Start an integration workspace and land selected agent work onto main

```bash
jj git fetch
jj workspace add ../repo-integrate --name integrate -r main@origin -m "integrate selected agent work"
cd ../repo-integrate
jj squash --from agent-a@- --into @ --use-destination-message
jj squash --from agent-b@- --into @ --use-destination-message
jj commit -m "integrate selected agent work"
jj bookmark set main -r @-
```

Push only if requested:

```bash
jj git push --bookmark main
```

## Publish one change quickly without naming a bookmark yourself

```bash
jj git push --change @-
```

## Publish with an explicit bookmark

```bash
jj bookmark set agent/feature-x -r @-
jj git push --bookmark agent/feature-x
```

## Move a bookmark forward explicitly

Use this when a bookmark should follow rewritten work.

```bash
jj bookmark advance main --to @-
```

## Fixups across a stack

```bash
# edit files in the current working copy
jj absorb
```

## File-based split, non-interactive

```bash
jj split src/new_module.rs tests/new_module_test.rs -m "extract module"
```

## Recover from a bad rewrite

```bash
jj undo
```

## Recover an older repo state

```bash
jj op log
jj op restore <operation-id>
```

## Repair a stale workspace

```bash
jj workspace update-stale
```

## Useful quick checks

```bash
jj st
jj log -r '@ | @- | main | main@origin | bookmarks()'
```
