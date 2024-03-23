# staticcheck-action

This action runs [Staticcheck](https://staticcheck.io) to find bugs and other problems in your Go code.

## Usage

At its simplest, just add `dominikh/staticcheck-action` as a step in your existing workflow.
A minimal workflow might look like this:

```yaml
name: "CI"
on: ["push", "pull_request"]

jobs:
  ci:
    name: "Run CI"
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 1
    - uses: dominikh/staticcheck-action@v1
      with:
        version: "latest"
```

A more advanced example that runs tests, go vet and Staticcheck on multiple OSs and Go versions looks like this:

```yaml
name: "CI"
on: ["push", "pull_request"]

jobs:
  ci:
    name: "Run CI"
    strategy:
      fail-fast: false
      matrix:
        os:  ["windows-latest", "ubuntu-latest", "macOS-latest"]
        go:  ["1.18.x", "1.19.x"]
        dir: ["server", "client"]
    runs-on: ${{ matrix.os }}
    steps:
    - uses: actions/checkout@v2
      with:
        fetch-depth: 1
    - uses: WillAbides/setup-go-faster@v1.14.0
      with:
        go-version: ${{ matrix.go }}
    - run: "go test ./..."
    - run: "go vet ./..."
    - uses: dominikh/staticcheck-action@v1
      with:
        version: "latest"
        install-go: false
        cache-key: ${{ matrix.go }}
        working-directory: ${{ matrix.dir }}
```



Please see [GitHub's documentation on Actions](https://docs.github.com/en/actions) for extensive
documentation on how to write and tweak workflows.

## Options

### `version`

Which version of Staticcheck to use.
Because new versions of Staticcheck introduce new checks that may break your build,
it is recommended to pin to a specific version and to update Staticheck consciously.

It defaults to `latest`, which installs the latest released version of Staticcheck.

### `min-go-version`

Minimum version of Go that Staticcheck will report diagnostics for. That is, some
diagnostics don't apply to older versions of Go.

If unset, this will default to the Go version specified in your go.mod. The default is
usually what you want.

See https://staticcheck.io/docs/running-staticcheck/cli/#go for more information.

### `build-tags`

Go build tags that get passed to Staticcheck via the `-tags` flag.

### `install-go`

Whether the action should install the latest version of Go to install and run Staticcheck.
If Staticcheck is the only action in your job, this option can usually be left on its default value of `true`.
If your job already installs Go prior to running Staticcheck, for example to run unit tests, it is best to set this option to `false`.

The latest release of Staticcheck works with the last minor release of Go at the time.
The action itself requires at least Go 1.16.

### `cache-key`

String to include in the cache key, in addition to the default, which is `runner.os`.
This is useful when using multiple Go versions.

### `working-directory`

Relative path to the working directory Staticcheck should be executed in.
This is useful when dealing with multiple projects within one repository.

Can be easily combined with a directory [`matrix`](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategymatrix),
see the advanced example above.

