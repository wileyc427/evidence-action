# Agentic Evidence: GitHub Action

Records evidence for each pull request: the project's saved demos (`.evidence/demos/*.json`) played against the PR's head commit in a real browser and recorded, plus a page check. Uploads the run to your evidence server, posts an **Evidence** check run on the commit, and keeps one comment on the PR up to date with the verdict, each demo, and links to the report and recordings.

```yaml
name: Evidence
on: pull_request
permissions:
  contents: read
  checks: write
  pull-requests: write
  id-token: write
jobs:
  evidence:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 0
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - uses: wileyc427/evidence-action@v0
        with:
          server: https://evidence.example.com
```

**Uploading:** with `server` and no key, the run is uploaded with the job's GitHub Actions OIDC token (`id-token: write` above), which proves to the server which repository's CI recorded it. The repository has to be linked: the server's GitHub App installed on it (sign in to the server's dashboard and choose **Link a repository**). No secrets to store. A server without the GitHub App takes an upload key instead: `key-id` and `key-secret`.

Check out the PR's **head** commit (`ref` above): evidence belongs to the code that was written, not to the merge commit `actions/checkout` makes by default. `fetch-depth: 0` lets the run list the files changed since the base.

Without a server, the run stays in the job; keep it with `actions/upload-artifact` (`path: ${{ steps.<id>.outputs.run-dir }}`).

The app runs in the project's Docker Compose environment (`.evidence/config.yml` names the file), with Docker on GitHub's runners; Google Chrome on the runner is used for the browser.

**An app that's already running** (started by an earlier step, or a preview deployment): pass `url`, and the action starts nothing. The data isn't reset between demos, so each demo should set up what it needs, and the report notes that the run didn't start the app from the commit. To run more than one evidence job on a PR, give each its own `check-name`; each gets its own check run and comment.

