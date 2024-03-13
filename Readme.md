A GitHub Action for R packages that runs `atime::atime_pkg` and comments the generated results on pull requests to help identify potential performance regressions introduced from the incoming changes.

Contents of the comment consist of:
- A plot comparing different versions<sup>1</sup> of the package on all the test cases (portrayed side-by-side) that are defined in the `inst/atime/tests.R` file within the respository. <sup>1</sup>Versions include base (the target branch), HEAD (the PR/source branch), merge-base (their common ancestor), and CRAN (yup, your package needs to be there :).
- The SHA for the commit that triggered the workflow (or as an end result, generated that comment) within the PR.
- A download link to retrieve a zipped file containing the `atime`-generated results.
- The time it took to install `atime` and run its operations.

To avoid flooding the pull request on every push to it, only one comment will exist in the PR thread at all times. After the initial comment generated from the first commit on the pull request branch, subsequent pushes will simply update the comment's body.