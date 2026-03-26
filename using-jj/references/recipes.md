# Copy-paste jj recipes for coding agents

These recipes assume:
- trunk bookmark is `main`
- default remote is `origin`
- one agent = one workspace

## Create a new agent workspace

```bash
jj git fetch
jj workspace add ../repo-agent-a --name agent-a -r main@origin -m "agent-a: start from main"
cd ../repo-agent-a
jj status
jj describe -m "feat: <task>"
```

## Finish a task in the current workspace

```bash
# edit files
jj diff
jj commit -m "feat: <task>"
```

## Show active workspaces and agent tips

```bash
jj workspace list
jj log -r 'main@origin | main | agent-a@ | agent-b@ | integrate@ | @ | @-'
```

## Inspect another workspace’s last committed change

```bash
jj show agent-a@-
jj diff --from main@origin --to agent-a@-
```

## Copy another agent’s work without rewriting it

```bash
jj duplicate agent-a@- -A @
```

## Adopt another agent’s work into the current change

```bash
jj squash --from agent-a@- --into @ --use-destination-message
```

## Rebase current finished change onto latest trunk

```bash
jj git fetch
jj rebase -r @- -o main@origin
```

## Split out a file-based subset

```bash
jj split src/new_parser.rs tests/parser_test.rs -m "extract parser"
```

## Publish the current finished change quickly

```bash
jj git push --change @-
```

## Publish with an explicit bookmark

```bash
jj bookmark set agent/feature-x -r @-
jj git push --bookmark agent/feature-x
```

## Start an integration workspace and land selected changes

```bash
jj git fetch
jj workspace add ../repo-integrate --name integrate -r main@origin -m "integrate selected agent work"
cd ../repo-integrate
jj squash --from agent-a@- --into @ --use-destination-message
jj squash --from agent-b@- --into @ --use-destination-message
jj commit -m "integrate selected agent work"
jj bookmark set main -r @-
```

## Push `main` only when asked

```bash
jj git push --bookmark main
```

## Repair a stale workspace

```bash
jj workspace update-stale
```

## Undo the last bad mutation

```bash
jj undo
```

## Restore an older repo state

```bash
jj op log
jj op restore <operation-id>
```
