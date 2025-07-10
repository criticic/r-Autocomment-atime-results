A GitHub Action for R packages that runs `atime::atime_pkg` and comments the generated results on pull requests to help identify potential performance regressions introduced from the incoming changes.

This action now includes **caching** for R packages and system dependencies to significantly speed up subsequent workflow runs. Due to this, there are significant changes in how the action is used, as detailed [below](#caching).

Contents of the comment consist of:
- A plot comparing different versions<sup>1</sup> of the package on all the test cases (portrayed side-by-side) that are defined in the `.ci/atime/tests.R` file within the respository. <sup>1</sup>Versions include base (the target branch), HEAD (the PR/source branch), merge-base (their common ancestor), and CRAN (yup, your package needs to be there :).
- Points regarding potential slowdowns in HEAD for applicable test cases.
- The SHA for the commit that triggered the workflow (or as an end result, generated that comment) within the PR.
- A download link to retrieve a zipped file containing the `atime`-generated results.
- A table enlisting:
  - The elapsed time from the job's start to the completion of the standard installation steps
  - The time it took to install the different git versions of the package being tested
  - The time taken to run and plot all the tests

To avoid flooding the pull request on every push to it, only one comment will exist in the PR thread at all times. After the initial comment generated from the first commit on the pull request branch, subsequent pushes will simply update the comment's body.

## Usage

For any GitHub repository of an R package, one can use this template:
```yml
name: Autocomment atime-based performance regression analysis on PRs

on:
  pull_request_target:
    types:
      - opened
      - reopened
      - synchronize
    # Modify path filters as needed:
    paths:
      - 'R/**'
      - 'src/**'
      - '.ci/atime/**'

jobs:
  comment:
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Run atime performance analysis
        uses: Anirban166/Autocomment-atime-results@v1.5.0
        with:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
Emplace the contents in `.github/workflows/<workflowName>.yml`. The example above can be customized, but a few things are important for it to work correctly:
- The workflow should run on a **`pull_request_target`** event. This allows the action to run in the context of the base repository, which is necessary for it to have permission to post comments securely.
- The job needs the **`permissions: pull-requests: write`** block to grant it explicit permission to write comments.
- The **`REPO_TOKEN`** input must be supplied with `${{ secrets.GITHUB_TOKEN }}`. This is required for the action to comment on the pull request.

## Caching

This action automatically caches R packages and system dependencies (`libgit2`) to make your jobs run faster. To get the most benefit, you can "warm up" the cache by running the workflow on your main branch. This is because PRs cannot share caches with each other, but they can access caches created from the base branch (e.g., `main`).

You can add a `push` or `workflow_dispatch` trigger to your workflow file like so:
```yml
on:
  # Run on pushes to the main branch to create/update the cache
  push:
    branches:
      - main
      - master
    # Modify path filters as needed:
    paths:
      - 'R/**'
      - 'src/**'

  # For manual runs from the Github Web UI
  workflow_dispatch:
  
  # Keep the PR trigger
  pull_request_target:
    types: [opened, reopened, synchronize]
```
When a commit is pushed to `main`, the workflow will run, install all dependencies, and save them to the cache. Subsequent PRs will then be able to download this cache, skipping the slow installation steps.