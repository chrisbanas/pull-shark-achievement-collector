# Pull Shark Achievement Collector

Automate the creation and merging of pull requests with GitHub Actions
to help progress toward the **Pull Shark** GitHub achievement.

This repository contains a GitHub Actions workflow that repeatedly:

1.  Creates a new branch
2.  Makes a small file change
3.  Opens a pull request into `main`
4.  Merges the pull request
5.  Repeats the process up to a configurable maximum

It also includes retry logic and GitHub API rate‑limit handling to make
long runs more stable.

------------------------------------------------------------------------

## What this does

The workflow automates a large number of merged pull requests in a
repository you control.

It uses:

-   GitHub Actions
-   GitHub CLI (`gh`)
-   `jq`
-   a Personal Access Token
-   rate limit checks and retry loops

The workflow writes to a file named `merged_prs.txt` so each pull
request contains a real change.

------------------------------------------------------------------------

## How it works

When you manually trigger the workflow, it will:

-   check out the repository
-   configure Git
-   install `gh` and `jq`
-   repeatedly create a branch with a unique name
-   append one line to `merged_prs.txt`
-   commit and push the branch
-   create a pull request into `main`
-   merge the pull request with squash merge
-   delete the branch
-   repeat until the configured maximum is reached

The script also pauses when GitHub API rate limits are low, then resumes
automatically after reset.

------------------------------------------------------------------------

## Repository requirements

Before using this workflow, make sure:

-   your repository has a `main` branch
-   GitHub Actions are enabled
-   branch protections do not block the workflow from merging
-   you understand and accept the rate‑limit and activity implications

This is best used in a **test or personal repository you control**.

------------------------------------------------------------------------

## Setup

### 1. Add the workflow file

Create:

    .github/workflows/pullshark.yml

Paste the workflow script into that file.

------------------------------------------------------------------------

### 2. Create a Personal Access Token

Generate a GitHub Personal Access Token with permissions to:

-   read/write repository contents
-   create and merge pull requests

Typical permission:

    repo

------------------------------------------------------------------------

### 3. Add the token as a repository secret

In your repository:

1.  Go to **Settings**
2.  Open **Secrets and Variables**
3.  Click **Actions**
4.  Create a new secret:

```{=html}
<!-- -->
```
    PERSONAL_ACCESS_TOKEN

Paste your token value.

------------------------------------------------------------------------

## Usage

The workflow is triggered manually using `workflow_dispatch`.

Steps:

1.  Open your repository
2.  Click the **Actions** tab
3.  Select **Automate Pull Requests and Merge for Pullshark Gold**
4.  Click **Run workflow**

The workflow will continue creating and merging PRs until the configured
limit is reached.

------------------------------------------------------------------------

## Workflow configuration

Inside the workflow:

    env:
      GH_TOKEN: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
      MAX: 1024
      CORE_THRESHOLD: 200
      ITERATION_DELAY: 30

### MAX

Maximum number of PRs created.

Example:

    MAX: 1024

For testing you should start smaller:

    MAX: 5

------------------------------------------------------------------------

### CORE_THRESHOLD

Minimum remaining API requests before the script pauses.

Example:

    CORE_THRESHOLD: 200

------------------------------------------------------------------------

### ITERATION_DELAY

Delay in seconds between iterations.

Example:

    ITERATION_DELAY: 30

------------------------------------------------------------------------

## File changes made

Each automated PR appends a new line to:

    merged_prs.txt

Example:

    Merged PR #57 at 2026-03-15 19:22:10 UTC

------------------------------------------------------------------------

## Rate limiting

The workflow checks:

-   GitHub **core API rate limits**
-   GitHub **search API limits**

If limits are low, it sleeps until reset before continuing.

It also retries:

-   branch pushes
-   pull request creation
-   merges

------------------------------------------------------------------------

## Important considerations

### Branch protections

If your `main` branch has protections enabled, merges may fail unless:

-   your token has permission
-   admin merge is allowed
-   required checks do not block the merge

The workflow merges with:

    gh pr merge --squash --delete-branch --admin

------------------------------------------------------------------------

### Commit identity

The workflow currently sets:

    git config --global user.name "chrisbanas"
    git config --global user.email "bananas595@gmail.com"

You should update these values to match your GitHub identity.

------------------------------------------------------------------------

### Public repository note

If this repo is public, anyone can see:

-   the workflow logic
-   the generated commit history
-   pull request activity

Never expose your token. Always use GitHub Secrets.

------------------------------------------------------------------------

## Recommended testing approach

Before running a large job:

    MAX: 5

Confirm:

-   PR creation works
-   merges succeed
-   branch deletion works
-   rate limits are handled

Then increase the value.

------------------------------------------------------------------------

## Security best practices

-   never hardcode tokens
-   always use repository secrets
-   use minimum token permissions
-   test in a throwaway repository first

------------------------------------------------------------------------

## Disclaimer

Use this workflow responsibly and only in repositories you control or
are authorized to automate.

You are responsible for complying with GitHub platform rules and
repository policies.
