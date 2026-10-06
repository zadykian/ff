# ff

A GitHub Action that lands pull requests by fast-forward. A comment `/ff` on a pull request pushes
its head commit to the default branch, so the branch holds the very commits the checks ran on.

GitHub has no such merge method: a merge commit, a squash and a rebase all write commits of their
own, a rebase setting each commit's committer anew.

## Usage

`.github/workflows/fast-forward.yml`:

```yaml
name: fast-forward

on:
  issue_comment:
    types: [created]

permissions: {}

jobs:
  fast-forward:
    if: github.event.issue.pull_request && startsWith(github.event.comment.body, '/ff')
    runs-on: ubuntu-latest
    timeout-minutes: 5
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: zadykian/ff@SHA # vX.Y.Z
```

Pin the action by a commit's hash, the release's tag in the comment. The job's `if` starts a
runner only for a `/ff` comment; the action checks the comment again, and ignores the others.

## What it does

The job runs the default branch's copy of the workflow, so it runs nothing of the pull request's:
it fetches its commits and pushes one. It refuses, and comments why on the pull request, where:

- the commenter cannot push to the repository;
- the pull request is closed, or into another branch than the default one;
- the pull request does not start from the head of the default branch: rebase it;
- the pull request changes `.github/workflows`, which `GITHUB_TOKEN` may not push. Push it by
  hand, `git push origin SHA:main`;
- GitHub refuses the push. The comment quotes what git printed.

The push starts no workflow, so the checks do not run again for a commit they have checked.

## The repository's settings

Leave GitHub's own merge methods nothing to merge with:

- Settings, Pull Requests: allow merge commits alone.
- A ruleset on the default branch:
  - Require a pull request before merging, with rebase as the only allowed merge method;
  - Require linear history, which refuses merge commits too;
  - Require status checks to pass, and conversation resolution where you want it;
  - Block force pushes, and restrict deletions.

The ruleset still judges the action's push. It passes only as a fast-forward to the head of a pull
request whose required checks pass on that commit.

## License

[MIT](LICENSE)
