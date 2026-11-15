# `.github`

GitHub configuration shared by the [dabaz-official](https://github.com/dabaz-official) projects.

## Reusable web CI

`.github/workflows/web-ci.yml` installs dependencies with pnpm and runs `pnpm check`.
The gate is the project's own script, so each repository decides what it composes
(format check, lint, typecheck, test, build) and this workflow never restates it.

A repository calls it from its own workflow:

```yaml
name: CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

jobs:
  ci:
    uses: dabaz-official/.github/.github/workflows/web-ci.yml@main
```

The check appears as `<caller job id> / checks`, so a caller that keeps the job id `ci`
produces the context `ci / checks`. Use that exact string when a branch requires checks.

pnpm comes from the caller's `package.json` → `packageManager`. Node.js defaults to 24;
pass `node-version` to move a repository deliberately.

## Adding a repository

1. Make sure `package.json` has `check`, and that it passes locally.
2. Add the caller workflow above at `.github/workflows/ci.yml`.
3. Push a branch and open a pull request; the new workflow runs on it, so a broken
   setup shows up before merge.
4. Require `ci / checks` on the default branch once the pull request is green.

## While a repository is still behind

A project that cannot run every gate yet composes `check` from what it has and records
the missing piece (no tests, no ESLint config) in its `AGENTS.md` under Exceptions.
Adding the piece means extending `check`, not editing this workflow.

`@main` means a change here takes effect on every caller without review. That is the
trade for one canonical workflow; treat edits to this directory as production changes.
