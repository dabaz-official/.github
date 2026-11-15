# `.github`

GitHub configuration shared by the [dabaz-official](https://github.com/dabaz-official) projects.

## Reusable web CI

`.github/workflows/web-ci.yml` installs dependencies with pnpm and runs lint, typecheck,
test, and build, each behind a flag. A repository calls it from its own workflow:

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
    with:
      test: false # until package.json has a test script
```

The check appears as `<caller job id> / checks`, so a caller that keeps the job id `ci`
produces the context `ci / checks`. Use that exact string when a branch requires checks.

pnpm comes from the caller's `package.json` → `packageManager`. Node.js defaults to 24;
pass `node-version` to move a repository deliberately.

## Adding a repository

1. Add the caller workflow above at `.github/workflows/ci.yml`.
2. Push a branch and open a pull request; the new workflow runs on it, so a broken setup
   shows up before merge.
3. Require the checks on the default branch once the pull request is green.

## While a repository is still behind

The flags exist because the projects are not equally covered yet: `lint`, `typecheck`,
`test`, and `build` are on by default, and a project turns off only what it cannot run
yet. Moving a flag back to on is part of adding that script.

`@main` means a change here takes effect on every caller without review. That is the
trade for one canonical workflow; treat edits to this directory as production changes.
