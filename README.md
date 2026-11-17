# shared-workflows

GitHub Actions workflows shared across PinMeTo repositories. This repo is
public, so it holds generic logic only: no repo-specific settings, exceptions
or secrets.

## Workflows

### Dependency review

[`dependency-review.yml`](.github/workflows/dependency-review.yml) runs on
every PR in repos covered by the org ruleset. Nothing needs to be added to a
repo for it to run.

It fails a PR that adds or changes a runtime dependency with a known high or
critical vulnerability. Vulnerable packages that are already on the base branch
and not touched by the PR are not flagged.

To accept an advisory in one repo, add it to that repo's
`.github/dependency-review-config.yml`, in the PR that introduces it, with a
comment on why and when to remove it:

```yaml
allow-ghsas:
  # some-package 1.2.3 via other-package, no patched release yet.
  # Remove when other-package drops it.
  - GHSA-xxxx-xxxx-xxxx
```

The severity threshold, scope and the check itself are fixed in the shared
workflow; a repo's config file can only add exceptions.

### Risk label

[`check-risk-label.yml`](.github/workflows/check-risk-label.yml) fails a PR
that has none of the labels `Low risk`, `Medium risk` or `High risk`. Call it
from the repo:

```yaml
name: Risk label

on:
  pull_request:
    types: [opened, reopened, labeled, unlabeled, synchronize]

jobs:
  risk-label:
    uses: PinMeTo/shared-workflows/.github/workflows/check-risk-label.yml@main
```

### Changelog

[`changelog.yml`](.github/workflows/changelog.yml) checks that `CHANGELOG.md`
follows the version in `package.json`, compared with the PR's base branch:

- Version changed: it must be higher than the base, and the top section of
  `CHANGELOG.md` must be `## [X.Y.Z] - YYYY-MM-DD` for that version.
- Version unchanged: the top section must be unchanged too. With
  `require-bump: true`, the PR fails instead.

The changelog follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Call it from the repo:

```yaml
name: Changelog

on:
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  changelog:
    uses: PinMeTo/shared-workflows/.github/workflows/changelog.yml@main
    with:
      package-dir: server/package # default: repo root
      require-bump: true # default: false
```

## Changing a workflow

Callers use `@main`, so a merge here takes effect in every repo on its next PR
run. The org ruleset also runs `dependency-review.yml` from `main`. Test a
change by pointing one caller at your branch (`@your-branch`) before merging.
