# dzil-actions

This repository contains actions for building and testing Perl CPAN
distributions that use [Dist::Zilla](https://dzil.org/).

## Workflows

### `dzil-matrix`

Maybe the most useful thing in this repository is the `dzil-matrix` workflow,
which you can call from your own workflow like this:

```yaml
jobs:
  build-and-test:
    uses: rjbs/dzil-actions/.github/workflows/dzil-matrix.yaml@main
```

This will use Dist::Zilla to build a tarball from your repository, then run its
tests with all the major versions of perl that it claims to support.  It works
by combining the `build` action and `multiperl-test-matrix` workflow, both
described below.

### `multiperl-test-matrix`

This uses the `rjbs/dzil-actions/test-tarball` action across a matrix of
`perldocker/perl-tester` containers to test the given tarball.

The tarball name is currently required to be `${REPONAME}.tar.gz` (except
expanded).  This should become an input to the workflow, but isn't yet.

Inputs to this workflow:

* `perl-versions-json`: This must be JSON string encoding an array of perl
  versions.  It will be used as the set of versions of the `perl-tester`
  container for testing.  An example value is: `"[\"5.8\", \"5.10\"]"`

## Actions

# `build`

This action builds a CPAN-style tarball from your Dist::Zilla-style repository.

In brief, it will:
* check out the repository
* set up a working Dist::Zilla installation
* install the prereqs for building your distribution archive
* use `dzil build` to build an archive file
* upload that file as an artifact

Inputs to this action:

* `dist-name`:  This the name of the directory the dist files will be built
  into, and then part of the name of the final tarball.  You probably never
  need to provide this input.  It defaults to the name part of your repository.
  So, if your repo is `rjbs/Sub-Exporter`, the dist name will default to
  `Sub-Exporter` and the uploaded artifact will be `Sub-Exporter.tar.gz`.

There are two outputs:

* `minimum-perl` is a string containing the minimum version of perl for this
  distribution, from the generated `META.json` file.  If none was found, it
  will use `v5.8`.
* `perl-versions` is a JSON string, from the
  [perl-actions/perl-versions](https://github.com/perl-actions/perl-versions)
  action, generated using the `minimum-perl` output (above) as the `since-perl`
  parameter.

# `test-tarball`

This action runs the tests for a CPAN-style tarball and produces a simple
summary report.

In brief, it will:
* fetch the dist tarball (as an artifact download)
* extract it
* install the dependencies, plus some test tools
* configure the distribution (currently assuming a `Makefile.PL`)
* run the tests
* publish a test report

Inputs to this action:

* `dist-name`.  This is the basename of the tarball we're expecting to find.
  If you provide `Sub-Exporter`, the artifact `Sub-Exporter.tar.gz` is
  expected, and it's expected to have a top-level `Sub-Exporter` directory.
  You probably never need to provide this input.  It defaults to the name part
  of your repository.  So, if your repo is `rjbs/Sub-Exporter`, the dist name
  will default to `Sub-Exporter`.
