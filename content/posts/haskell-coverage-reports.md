---
title: "Haskell Coverage Reports"
date: 2021-12-22
tags: [haskell]
---

Test coverage is a tricky topic. I'm of the opinion that 100% test coverage is
not a useful goal. Some take this opinion and discard the idea of tracking
coverage altogether. However, there are interesting things you can do with test
coverage information in the context of a Pull Request, such as asking if the
lines introduced in the Pull Request are covered by tests, or if the Pull
Request significantly helps or hurts the overall project coverage. This post
describes how to track test coverage in a Haskell project and utilize just such
information in Pull Requests.

## Approach

My setup will be described in three parts:

1. Generate a Haskell test coverage report (in its bespoke format)
1. Convert that report to a standardized format
1. Upload that report to a software-as-a-service coverage product

Finally, we'll wrap all this up into a GitHub Actions Workflow to bring the
coverage details into the context of the Pull Request they represent.

## Generate Coverage

Haskell supports coverage reports in a format called [HPC][]. In a Stack-based
project, these can be generated trivially by passing the `--coverage` flag to
`stack build`. The artifacts will end up in `local-hpc-root`, which `stack path`
can give you:

[hpc]: https://wiki.haskell.org/Haskell_program_coverage

```console
% tree "$(stack path --local-hpc-root)"
...
├── combined
│   └── all
│       ├── all.tix     <-- re-format/upload this to other tooling
│       ├── ...
│       └── ...
│           ├── ...
│           └── ...
├── index.html          <-- open this in $BROWSER to view the report
└── ...
```

Opening `index.html` (and navigating to the "all" report) shows:

![](/images/haskell-coverage-reports.png)

The data we're interested in is contained in the `.tix` files, with `all.tix`
representing the overall project. Before utilizing it in any other tooling, we
need to get it into a standardized format.

## Re-format as LCOV

[LCOV][] is the most commonly-supported "lingua franca" between various coverage
tools I could find. Lots of languages ([Ruby][], [Go][], [JavaScript][]) can
generate their coverage reports in this format and lots of products
([Coveralls][], [Codecov][], [Code Climate][]) can ingest this format.

[LCOV]: https://github.com/linux-test-project/lcov#readme
[ruby]: https://github.com/fortissimo1997/simplecov-lcov
[go]: https://github.com/jandelgado/gcov2lcov
[javascript]: https://jestjs.io/docs/configuration#coveragereporters-arraystring--string-options

[coveralls]: https://github.com/okkez/coveralls-lcov
[codecov]: https://about.codecov.io/tool/lcov/
[code climate]: https://docs.codeclimate.com/docs/configuring-test-coverage#supported-languages-and-formats

For Haskell, we have the [hpc-lcov][] project:

[hpc-lcov]: https://github.com/LeapYear/hpc-lcov#readme

```console
stack install --copy-compiler-tool hpc-lcov
```

To convert your `all.tix` into an `lcov.info`, run:

```console
stack exec -- \
  hpc-lcov --file "$(stack path --local-hpc-root)"/combined/all/all.tix
```

## Upload Coverage

With your `lcov.info` in hand, you can upload it to any number of destinations.
For example, reporting to Code Climate can be done through their
[`cc-test-reporter`][cc-test-reporter] CLI:

[cc-test-reporter]: https://github.com/codeclimate/test-reporter

```sh
# Write-only API key available in settings
export CC_TEST_REPORTER_ID={...}

# Move the file to a location Code Climate expects
mkdir -p coverage
mv lcov.info coverage/lcov.info

cc-test-reporter after-build
```

If you'd rather not move the file around, you can drop down to lower-level
commands so you can specify things explicitly:

```console
cc-test-reporter format-coverage --input-type lcov --output - lcov.info |
  cc-test-reporter upload-coverage --input -
```

If you're doing the format and upload in the same location (e.g. the same CI
Job), you could even skip the intermediate file altogether:

```console
stack exec -- hpc-lcov --file "$tix" --output /dev/stdout |
  cc-test-reporter format-coverage --input-type lcov --output - /dev/stdin |
  cc-test-reporter upload-coverage --input - --id {...}
```

Such unix, so wow.

![](/images/haskell-coverage-reports-cc-source.png)

This file's coverage is not great.

## GitHub Actions

Putting this all together in a Workflow looks as you might expect:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: freckle/stack-cache-action@v1

      - id: stack
        uses: freckle/stack-action@v3
        with:
          stack-arguments: --coverage

      - run: |
          stack --no-terminal install --copy-compiler-tool hpc-lcov
          stack --no-terminal exec -- \
            hpc-lcov --file '${{ steps.stack.outputs.local-hpc-root }}/combined/all/all.tix'

      - uses: codecov/codecov-action@v2
        with:
          files: ./lcov.info
```

Here, I'm uploading to Codecov instead of Code Climate. Their treatment of my
36% coverage is a little less hostile:

![](/images/haskell-coverage-reports-codecov-source.png)


What I really like is how Codecov will annotate introduced lines when they are
not covered:

![](/images/haskell-coverage-reports-codecov-annotation.png)

Now that's useful!
