# plugin-workflows

> [!IMPORTANT]
> **Read-only.** This repository is a published copy, generated automatically at each release from Hackolade's internal source repository. It is only meant to be consumed: do not open pull requests or push changes here, they are not merged and are overwritten by the next release.
>
> **Hackolade team:** make changes in the internal source repository (its README describes the update and release process). Everyone else: report problems to your Hackolade contact, or privately as described in [SECURITY.md](./SECURITY.md).

Reusable GitHub workflows and actions shared by the public repositories of [Hackolade](https://hackolade.com/), such as the Studio plugins.

| Workflow | Purpose |
|---|---|
| [`jira-link-pr.yml`](./.github/workflows/jira-link-pr.yml) | Adds a link to the Jira issue found in a pull request's title or branch, with its summary, at the top of its description |
| [`trigger-teamcity-build.yml`](./.github/workflows/trigger-teamcity-build.yml) | Queues a TeamCity build for the calling pull request (`pull/<number>`) or another branch, and can wait for its result |
| [`notify-direct-push.yml`](./.github/workflows/notify-direct-push.yml) | Posts a direct push to Slack |
| [`setup-node`](./setup-node/action.yml) (action) | Installs the pinned Node.js (**24.22.1**) and npm (**11.7.0**) |

The workflows hold no secrets: each caller passes its own secrets and variables.

## Usage

`.github/workflows/jira_link_to_pr.yml`

```yaml
name: Link the Jira issue

on:
  pull_request:
    types: [opened, reopened]

permissions:
  contents: read
  pull-requests: write

jobs:
  jira:
    uses: hackolade/plugin-workflows/.github/workflows/jira-link-pr.yml@v1
    secrets:
      jira-token: ${{ secrets.JIRA_TOKEN }}
```

`.github/workflows/trigger-pr-tests-plugins.yml`

```yaml
name: Trigger PR tests (Plugins)

on:
  pull_request:
    types: [auto_merge_enabled]

permissions:
  contents: read

jobs:
  trigger-pr-tests-plugins:
    uses: hackolade/plugin-workflows/.github/workflows/trigger-teamcity-build.yml@v1
    with:
      teamcity-url: ${{ vars.TEAMCITY_API_URL }}
      build-type-id: ${{ vars.TEAMCITY_BUILD_ID_FOR_TESTING_PLUGIN_PR }}
      pass-build-branch: true
    secrets:
      teamcity-token: ${{ secrets.TEAMCITY_TRIGGER_TESTS_TOKEN }}
```

`.github/workflows/notif-push-to-slack.yml`

```yaml
name: notify-push

on:
  push:
    branches:
      - develop

permissions:
  contents: read

jobs:
  notify:
    uses: hackolade/plugin-workflows/.github/workflows/notify-direct-push.yml@v1
    secrets:
      slack-bot-token: ${{ secrets.SLACK_BOT_TOKEN }}
      commit-push-source: ${{ secrets.COMMIT_PUSH_SOURCE }}
```

`setup-node`

```yaml
steps:
  - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
  - uses: hackolade/plugin-workflows/setup-node@v1
    with:
      registry-url: https://registry.npmjs.org
```

## Inputs

**`setup-node`**: `node-version` (`24.22.1`), `npm-version` (`11.7.0`, installed over the bundled npm; empty keeps the bundled one), `cache`, `cache-dependency-path`, `package-manager-cache` (`true`), `registry-url`, `architecture`. Outputs: `node-version`, `npm-version`. Exact versions on purpose: a range such as `'24'` lets `actions/setup-node` reuse an older cached 24.x.


**`jira-link-pr.yml`**: `project-key` (`HCK`), `base-url` (`https://hackolade.atlassian.net`), `use` (`both`: title first, then branch), `skip-branches` (`^(develop|main|master)$`), `fail-when-issue-not-found` (`false`). Secret: `jira-token`.

**`trigger-teamcity-build.yml`**: `teamcity-url` (HTTPS) and `build-type-id` (required), `branch` (empty: `pull/<number>`), `pass-build-branch` (`false`; `true` passes the source branch as the `BUILD_BRANCH` property), `properties` (JSON object of strings, `{}`), `comment`, `rebuild-dependencies` (`false`), `wait` (`false`; `true` waits for the build, fails unless it succeeds, and cancels it when the run is cancelled or times out), `timeout-minutes` (`10`). Secret: `teamcity-token`. Output: `build-url`. The job fails when TeamCity rejects the request.

**`notify-direct-push.yml`**: `channel-id` (`develop-direct-pushes`). Secrets: `slack-bot-token`, and `commit-push-source`, the pusher name that identifies a direct push.

## Security

- Third-party actions are pinned by commit SHA, the actionlint image by digest; updates are reviewed before each release.
- Every workflow and job declares its permissions, starting from none; only `jira-link-pr.yml` needs `pull-requests: write`.
- Actions run on a supported Node.js runtime (no Node.js 20); the Jira step is a script (`curl`, `jq`, `gh`) rather than a third-party action holding the Jira token.
- `jira-link-pr.yml` and `trigger-teamcity-build.yml` skip pull requests from forks, which receive no secrets.
- Inputs reach shell scripts through environment variables, never through `${{ }}` interpolation.
- Every pull request runs actionlint and [zizmor](https://docs.zizmor.sh/) (auditor persona).

Report vulnerabilities as described in [SECURITY.md](./SECURITY.md).

## Versions

Each release is tagged `v1.x.y`, and the major tag `v1` moves to the latest one. Reference reusable workflows with `@v1`; pin the `setup-node` action to the commit of a release (`@<sha> # v1.x.y`).
