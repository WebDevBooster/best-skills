# Recovery, stale workspaces and conflicts

Use this file when something went wrong.

## First inspection pass

```bash
jj status
jj log -r 'main@origin | main | @ | @- | bookmarks()'
jj op log --limit 10
```

## Undo the last bad operation

```bash
jj undo
```

Use this first when the most recent mutation was a mistake.

## Move forward again if needed

```bash
jj redo
```

## Restore an older repo state

```bash
jj op log
jj op restore <operation-id>
```

Use this when the bad state is not just the immediately previous operation.

## Inspect change evolution

```bash
jj evolog -r @-
```

Use this when you need to understand how a change moved or was rewritten over time.

## Repair a stale workspace

```bash
jj workspace update-stale
```

Use this when:
- the workspace’s working-copy commit was rewritten elsewhere
- an adopt/squash operation consumed a change that another workspace still referenced

## Resolve conflicts

Direct file editing is allowed.

```bash
jj resolve
```

Also valid:
- edit conflict markers directly in files
- inspect with `jj show` or `jj diff`
- continue once the conflicted commit is resolved

## Common bad situations

### I committed and now my bookmark did not move

That is normal in jj.

Fix it explicitly:

```bash
jj bookmark set <name> -r @-
```

### I accidentally rewrote another agent’s change

Best first step:

```bash
jj undo
```

Then inspect with:

```bash
jj op log
jj evolog -r <change>
```

### My integration workspace looks wrong after a squash

Inspect:

```bash
jj show @
jj show @-
jj op log --limit 10
```

Then either:

```bash
jj undo
```

or restore a known-good operation.
