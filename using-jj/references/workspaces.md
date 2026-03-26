# Working with jj workspaces for parallel agents

Use one workspace per agent.

## Default policy

- one agent = one workspace
- workspace names should be short and stable
- use `agent-<name>` naming by default
- use a separate `integrate` workspace for curated landing work

## Create a workspace

From an existing workspace in the repo:

```bash
jj git fetch
jj workspace add ../repo-agent-a --name agent-a -r main@origin -m "agent-a: start from main"
```

Then:

```bash
cd ../repo-agent-a
jj status
```

## List and inspect workspaces

```bash
jj workspace list
jj workspace root --name agent-a
jj log -r 'main@origin | main | agent-a@ | agent-b@ | integrate@'
```

## Referencing another workspace

Use revsets based on workspace name.

Examples:
- `agent-a@` = current working-copy commit of that workspace
- `agent-a@-` = most recent committed change in the common case

Inspect another agent’s change:

```bash
jj show agent-a@-
jj diff --from main@origin --to agent-a@-
```

## Taking work from another workspace

### Safe copy mode

Use when you want the work but do not want to rewrite the source workspace.

```bash
jj duplicate agent-a@- -A @
```

### Adopt mode

Use when you intentionally want the current change to absorb the source change.

```bash
jj squash --from agent-a@- --into @ --use-destination-message
```

This can make the source workspace stale.

## Stale workspace recovery

If commands complain that the workspace is stale, repair it with:

```bash
jj workspace update-stale
```

Typical cause:
- another workspace rewrote the commit this workspace was based on
- an integration step consumed or rebased a shared change

## Forget a workspace

When a workspace is no longer needed:

```bash
jj workspace forget agent-a
```

This stops tracking the workspace in the repo. It does not delete files on disk.

## Agent operating rules for workspaces

- Do not share a workspace between agents.
- Do not treat workspaces as branches.
- Do not mutate another workspace by cd-ing into it unless the workflow explicitly says to.
- Prefer duplicating another workspace’s change before experimenting.
