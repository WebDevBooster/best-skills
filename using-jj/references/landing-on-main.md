# Landing selected work onto main

This file gives the approved ways to get agent work back onto trunk.

## Principle

Do not land directly from a random agent workspace unless the workflow explicitly allows it.

Preferred pattern:
- fetch remote state
- create or refresh a dedicated integration workspace
- bring in selected changes
- curate the result
- point `main` at the finished landed commit
- push only if asked

## Safest default: integration workspace

```bash
jj git fetch
jj workspace add ../repo-integrate --name integrate -r main@origin -m "integrate selected agent work"
cd ../repo-integrate
```

Inspect target state:

```bash
jj status
jj log -r 'main@origin | main | agent-a@ | agent-b@ | @ | @-'
```

## Landing by adoption

Use this when the integration workspace should absorb the selected source changes.

```bash
jj squash --from agent-a@- --into @ --use-destination-message
jj squash --from agent-b@- --into @ --use-destination-message
jj commit -m "integrate selected agent work"
jj bookmark set main -r @-
```

This is the normal “curate and land” recipe.

## Landing by safe copy first

Use this when you want to test or curate without touching the original source changes.

```bash
jj duplicate agent-a@- -A @
jj duplicate agent-b@- -A @
# optionally reshape here
jj commit -m "integrate selected agent work"
jj bookmark set main -r @-
```

Use this when you need a conservative handoff path.

## Rebase before landing

If a source change should be replayed onto current trunk before integration:

```bash
jj git fetch
jj rebase -r agent-a@- -o main@origin
```

Do this only if rewriting that source change is acceptable.

## Publish options

### Push a generated review bookmark

```bash
jj git push --change @-
```

### Push an explicit bookmark

```bash
jj bookmark set agent/feature-x -r @-
jj git push --bookmark agent/feature-x
```

### Push `main`

Only when explicitly requested:

```bash
jj git push --bookmark main
```

## Adopt vs duplicate

Use **duplicate** when:
- the source change should remain intact
- you want to experiment
- multiple agents may still depend on the original change

Use **squash/adopt** when:
- you are taking ownership of the source change
- rewriting the source is acceptable
- the integration workspace is the new source of truth

## Bookmark rules during landing

- Do not assume `main` moved automatically.
- Point `main` explicitly with `jj bookmark set main -r @-` or move it intentionally.
- Remember that after `jj commit`, `@-` is usually the landed result.
