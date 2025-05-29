A GitHub Action for R packages that runs `atime::atime_pkg` and comments the generated results on pull requests to help identify potential performance regressions introduced from the incoming changes.

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
  pull_request:
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
    container: ghcr.io/iterative/cml:0-dvc2-base1
    env:
      GITHUB_PAT: ${{ secrets.GITHUB_TOKEN }}
      repo_token: ${{ secrets.GITHUB_TOKEN }}
    steps:
      - uses: Anirban166/Autocomment-atime-results@v1.4.3
```
Emplace the contents in `.github/workflows/<workflowName>.yml`. The example I provided above can be customized further as needed, as long as a few things are kept intact:
- The workflow runs on a `pull_request` event
- `GITHUB_PAT` is supplied (required to authenticate git operations, have higher rate limits, etc.)
- The `container` and `repo_token` fields are specified as I did above (required for `cml` functionality)
