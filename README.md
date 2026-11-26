# .github

Organization-level files for [Kibanya](https://github.com/kibanyadev).

This repository must stay public. GitHub applies the default community health files below to every
repository in the organization only when they come from a public `.github` repository.

| Path | What it does |
|---|---|
| [`profile/README.md`](./profile/README.md) | The public profile shown on the [org page](https://github.com/kibanyadev). |
| [`SECURITY.md`](./SECURITY.md) | Default security policy: report vulnerabilities privately. |
| [`.github/ISSUE_TEMPLATE/`](./.github/ISSUE_TEMPLATE) | Default issue forms (bug, feature, task); blank issues off. |
| [`.github/pull_request_template.md`](./.github/pull_request_template.md) | Default pull request template. |
| [`.github/workflows/`](./.github/workflows) | The reusable CI workflows (`reusable-*.yml`) and this repo's own CI. |
| [`workflow-templates/`](./workflow-templates) | Starter workflows offered in the Actions tab; they call the reusable workflows here, pinned by commit SHA. |
| [`brand/`](./brand) | The 基盤屋 seal (SVG, PNG) and the favicon. Red is `#C52E13`. |

A repository that ships its own version of one of these files overrides the default.

## Reusable workflows

Public, so repos of any owner (private ones included) can call them. Each file only
exposes `on: workflow_call`; triggers, schedules and path filters live in the caller.
A called job never gets more permissions than the caller job grants, so the caller job
must declare the permissions listed below. Callers set `permissions: {}` at the top.

The check name a caller sees is `<caller job id or name> / <called job name>`.

Adopting a reusable workflow changes the check context from the bare job name to
`<caller job> / <called job name>`. A repo whose ruleset or branch protection requires the
old bare context must update its required contexts in the same change (list both old and
new during the transition, then drop the old one).

Pin the call by full commit SHA, with the branch or tag in a comment. Dependabot
(`github-actions` ecosystem) keeps the SHA current. The `@<sha> # main` snippets below
are placeholders: callers must replace `<sha>` with the real full 40-character SHA of
`main` after this repo's change is merged.

| Workflow | Called job name | Caller job permissions |
|---|---|---|
| `reusable-actionlint.yml` | GitHub Actions Schema Check | `contents: read` |
| `reusable-zizmor.yml` | GitHub Actions Security Audit | `contents: read` |
| `reusable-site-ci.yml` | Site Checks | `contents: read` |

### reusable-actionlint.yml

Runs actionlint 1.7.12 (pinned image digest) over the repo's workflows. No inputs.

```yaml
jobs:
  actionlint:
    permissions:
      contents: read
    uses: kibanyadev/.github/.github/workflows/reusable-actionlint.yml@<sha> # main
```

### reusable-zizmor.yml

Runs zizmor over the repo's workflows (no code-scanning upload).

| Input | Default | Meaning |
|---|---|---|
| `version` | `1.32.0` | zizmor version |

```yaml
jobs:
  zizmor:
    permissions:
      contents: read
    uses: kibanyadev/.github/.github/workflows/reusable-zizmor.yml@<sha> # main
```

### reusable-site-ci.yml

pnpm workspace CI: checkout, pnpm (version from `packageManager`), Node from a version
file with the pnpm cache, `pnpm install --frozen-lockfile`, optional Playwright Chromium,
then each script in order (names must start with a letter, digit or underscore; an empty list fails). Playwright results upload on failure.

| Input | Default | Meaning |
|---|---|---|
| `node-version-file` | `.nvmrc` | Node version file, path from the repo root |
| `working-directory` | `.` | Directory with `package.json` and `pnpm-lock.yaml` |
| `scripts` | `verify` | Space-separated pnpm scripts, run in order (`lint typecheck test build`) |
| `playwright-filter` | empty | `pnpm --filter` selector to install Chromium for; empty skips Playwright and the trace upload |
| `traces-path` | empty | Glob uploaded on failure. Resolved from the workspace root, not `working-directory` (`uses:` steps ignore `defaults.run`). Empty means `<working-directory>/apps/*/test-results/` |
| `artifact-name` | `playwright-test-results` | Traces artifact name; set a distinct one when a run calls the workflow more than once |
| `timeout-minutes` | `15` | Job timeout |

```yaml
jobs:
  site:
    permissions:
      contents: read
    uses: kibanyadev/.github/.github/workflows/reusable-site-ci.yml@<sha> # main
    with:
      playwright-filter: '@scope/app'
```

## Own CI

`CI: Actionlint` and `CI: Zizmor` run the reusable workflows above on this repo
(push and PR to `main`, daily, manual; never path-filtered).
