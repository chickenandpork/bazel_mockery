# Bazel rules for using mockery

This is a relatively quick-and-dirty approach to try and bazelify Golang mocks generation using the
third-party [Mockery](https://github.com/vektra/mockery) tool.

Contributions welcome! Feel free to open up PRs or raise issues.

## Required dependencies

The [`gomockery.bzl`](./gomockery.bzl) file provides two rules that allow for Bazel-generated mock
files for your Golang interfaces. In order for these rules to work correctly you will need to add
the following dependencies to your own `WORKSPACE` file if they are not already present.

```python
load("@bazel_tools//tools/build_defs/repo:git.bzl", "git_repository")
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")


http_archive(
    name = "io_bazel_rules_go",
    sha256 = "7904dbecbaffd068651916dce77ff3437679f9d20e1a7956bff43826e7645fcc",
    urls = [
        "https://mirror.bazel.build/github.com/bazelbuild/rules_go/releases/download/v0.25.1/rules_go-v0.25.1.tar.gz",
        "https://github.com/bazelbuild/rules_go/releases/download/v0.25.1/rules_go-v0.25.1.tar.gz",
    ],
)

http_archive(
    name = "bazel_gazelle",
    sha256 = "222e49f034ca7a1d1231422cdb67066b885819885c356673cb1f72f748a3c9d4",
    urls = [
        "https://mirror.bazel.build/github.com/bazelbuild/bazel-gazelle/releases/download/v0.22.3/bazel-gazelle-v0.22.3.tar.gz",
        "https://github.com/bazelbuild/bazel-gazelle/releases/download/v0.22.3/bazel-gazelle-v0.22.3.tar.gz",
    ],
)

load("@io_bazel_rules_go//go:deps.bzl", "go_register_toolchains", "go_rules_dependencies")
load("@bazel_gazelle//:deps.bzl", "gazelle_dependencies", "go_repository")

go_rules_dependencies()

go_register_toolchains(version = "1.16")

gazelle_dependencies()

go_repository(
    name = "com_github_vektra_mockery",
    importpath = "github.com/vektra/mockery",
    tag = "e78b021dcbb558a8e7ac1fc5bc757ad7c277bb81",
)

go_repository(
    name = "com_github_stretchr_testify",
    importpath = "github.com/stretchr/testify",
    tag = "363ebb24d041ccea8068222281c2e963e997b9dc",
)
```

## Rules

The rules that [`gomockery.bzl`](./gomockery.bzl) provides are:

- `go_mockery` - Which generates the mocks for the interfaces exposed by the specified package. The
    arguments taken by this rule are:
    - `name` _(optional)_ Name of the rule on which other `go_*` rules should depend if they want
        to include the generated mocks files. Default is `go_default_mocks`.
    - `src` _(required)_ Label of the `go_library` in the sources of which should be looked for the
        interfaces to mock.
    - `interfaces` _(required)_ Explicit list of the names of interfaces for which mocks should be
        generated.
    - `case` _(optional)_ Casing of the file names that will be generated and contain the mocks
        (suppported values are `underscore`, `camel`, `snake`). Default is `underscore`.
    - `outpkg` _(optional)_ Name of the package that will contain the mocks. Default is `mocks`.
    - `mockery_tool` _(optional)_ Alternative label that builds the `mockery` binary that should be
        used instead of the default `@com_github_vektra_mockery//cmd/mockery:mockery`.
- `go_mockery_with_library` - Which generates the mocks for the interfaces exposed by the specified
    package and compiles them into a `go_library` as well that can be depended on by other `go_*`
    rules. Besides exposing the same arguments as the `go_mockery` tool this rule also takes:
    - `name` _(optional)_ Name of the rule that will correspond to the generated `go_library`.
        Default will be `go_default_library`.
    - `mocks_name` _(optional)_ Name of the rule on which other `go_*` rules should depend if they
        want to include the generated mocks files. Default is `go_default_mocks`.
    - `importpath` _(required)_ The Go package path with which the generated mocks package can be
        included from other Go files. It is passed **as is** through to the underlying `go_library`
        rule.
    - `testify_mock_lib` _(optional)_ Alternative label that builds the
        `github.com/stretchr/testify/mocks` package on which the Go mocks library will need to
        depend. Will replace the default `@com_github_stretchr_testify//mocks:go_default_library`.
