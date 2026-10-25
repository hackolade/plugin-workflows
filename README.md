# plugin-workflows

Reusable GitHub workflows shared by the [Hackolade Studio](https://hackolade.com/) plugin repositories.

| Workflow | Purpose |
|---|---|
| [`jira-link-pr.yml`](./.github/workflows/jira-link-pr.yml) | Adds the Jira issue found in a pull request's branch or title to its description and labels |
| [`trigger-teamcity-build.yml`](./.github/workflows/trigger-teamcity-build.yml) | Queues a TeamCity build for the calling pull request (`pull/<number>`) |
| [`notify-direct-push.yml`](./.github/workflows/notify-direct-push.yml) | Posts a direct push to Slack |

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

## Inputs

**`jira-link-pr.yml`**: `project-key` (`HCK`), `base-url` (`https://hackolade.atlassian.net`), `use` (`both`: branch and title), `skip-branches` (`^(develop|main|master)$`), `fail-when-issue-not-found` (`false`). Secret: `jira-token`.

**`trigger-teamcity-build.yml`**: `teamcity-url` and `build-type-id` (required), `pass-build-branch` (`false`; `true` passes the source branch as the `BUILD_BRANCH` property). Secret: `teamcity-token`. The job fails when TeamCity rejects the request.

**`notify-direct-push.yml`**: `channel-id` (`develop-direct-pushes`). Secrets: `slack-bot-token`, and `commit-push-source`, the pusher name that identifies a direct push.

## Releasing

Changes are released as tags, with the major tag `v1` moved to the latest release:

```bash
git tag v1.0.1 && git tag -f v1 && git push origin v1.0.1 && git push -f origin v1
```

Third-party actions are pinned by commit SHA and kept up to date by Dependabot.
