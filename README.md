<h3 align="center">Create an Issue Action</h3>
<p align="center">A GitHub Action that creates a new issue using a template file.<p>
<p align="center"><a href="https://github.com/dblock/create-a-github-issue"><img alt="GitHub Actions status" src="https://github.com/dblock/create-a-github-issue/workflows/Node%20CI/badge.svg"></a> <a href="https://codecov.io/gh/dblock/create-a-github-issue/"><img src="https://codecov.io/gh/dblock/create-a-github-issue/branch/main/graph/badge.svg?token=9wHcESNsRI" alt="Codecov"></a></p>

- [Fork](#fork)
- [Usage](#usage)
  - [Dates](#dates)
  - [Custom templates](#custom-templates)
  - [Inputs](#inputs)
  - [Outputs](#outputs)

## Fork

This is a maintained fork of [JasonEtco/create-an-issue](https://github.com/JasonEtco/create-an-issue). See [CHANGELOG](CHANGELOG.md) for fixes and features.

## Usage

This GitHub Action creates a new issue based on an issue template file. Here's an example workflow that creates a new issue any time you push a commit:

```yaml
# .github/workflows/issue-on-push.yml
on: [push]
name: Create an issue on push
permissions:
  contents: read
  issues: write 
jobs:
  stuff:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: dblock/create-a-github-issue@v3
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

This reads from the `.github/ISSUE_TEMPLATE.md` file. This file should have front matter to help construct the new issue:

```markdown
---
title: Someone just pushed
assignees: JasonEtco, matchai
labels: bug, enhancement
---
Someone just pushed, oh no! Here's who did it: {{ payload.sender.login }}.
```

You'll notice that the above example has some `{{ mustache }}` variables. Your issue templates have access to several things about the event that triggered the action. Besides `issue` and `pullRequest`, you have access to all the template variables [on this list](https://github.com/JasonEtco/actions-toolkit#toolscontext). You can also use environment variables:

```yaml
- uses: dblock/create-a-github-issue@v3
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    ADJECTIVE: great
```

```markdown
Environment variables are pretty {{ env.ADJECTIVE }}, right?
```

Note that you can only assign people matching given [conditions](https://help.github.com/en/github/managing-your-work-on-github/assigning-issues-and-pull-requests-to-other-github-users).

### Dates

Additionally, you can use the `date` filter and variable to show some information about when this issue was created:

```markdown
---
title: Weekly Radar {{ date | date('dddd, MMMM Do') }}
---
What's everyone up to this week?
```

This example will create a new issue with a title like **Weekly Radar Saturday, November 10th**. You can pass any valid [Moment.js formatting string](https://momentjs.com/docs/#/displaying/) to the filter.

### Custom templates

Don't want to use `.github/ISSUE_TEMPLATE.md`? You can pass an input pointing the action to a different template:

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: dblock/create-a-github-issue@v3
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    with:
      filename: .github/some-other-template.md
```

### Inputs

Want to use Action logic to determine who to assign the issue to, to assign a milestone or to update an existing issue with the same title? You can pass an input containing the following options:

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: dblock/create-a-github-issue@v3
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    with:
      assignees: JasonEtco, octocat
      milestone: 1
      update_existing: true
      search_existing: all
```

* The `assignees` and `milestone` speak for themselves.
* The `update_existing` param can be passed and set to `true` when you want to update an open issue with the **exact same title** when it exists and `false` if you don't want to create a new issue, but skip updating an existing one.
* The `search_existing` param lets you specify whether to search `open`, `closed`, or `all` existing issues for duplicates (default is `open`).

### Outputs

If you need the number or URL of the issue that was created or updated for another Action, you can use the `number`, `url` and `status` outputs, respectively. For example:

```yaml
steps:
  - uses: dblock/create-a-github-issue@v3
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    id: create-issue
  - run: 'echo number: ${{ steps.create-issue.outputs.number }}'
  - run: 'echo status: ${{ steps.create-issue.outputs.status }}'
  - run: 'echo url: ${{ steps.create-issue.outputs.url }}'
```

* The `number` and `url` outputs speak for themselves.
* The `status` is one of `created`, `updated` or `found`.
