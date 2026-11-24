# setup-plum

[![test](https://github.com/plumlang/setup-plum/actions/workflows/test.yml/badge.svg)](https://github.com/plumlang/setup-plum/actions/workflows/test.yml)

Install [Plumlang](https://plum.elixpo.com), the `plum` command, in GitHub
Actions. Works on Linux, macOS and Windows runners, on x64 and arm64.

The action picks the release asset for the runner, downloads it from
[Plumlang's releases](https://github.com/plumlang/plumlang/releases), checks
it against the release's `SHA256SUMS` (and fails on a mismatch), unpacks it
into the runner's tool cache, puts `plum` on the `PATH` and caches it for
later runs. It is a composite action: there is nothing to build and no
Node.js code.

## Usage

```yaml
name: ci
on: [push, pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: plumlang/setup-plum@v1
      - run: plum check main.plum        # every error, without running
      - run: plum fmt --check .          # lists files that would change
      - run: plum test                   # every test_ function in *_test.plum files
```

`plum check` takes one file (or a folder with a `plum.toml`), and
`plum fmt --check` takes files and folders. `plum test` runs the `test_`
functions in every `*_test.plum` file under the current directory and
exits 1 if one fails ([testing](https://github.com/plumlang/plumlang/blob/main/docs/test.md)).
It needs Plumlang 2.2.0 or later; with 2.1.x, run a program that calls
`assert` instead (`plum run tests/main.plum`). See the
[docs](https://plum.elixpo.com/docs/) for the language and the `plum`
command.

### A version

```yaml
- uses: plumlang/setup-plum@v1
  with:
    version: "2.1"   # the newest 2.1.x
```

`version` is `latest` (the default), an exact version (`2.1.4` or `v2.1.4`),
or a prefix (`2.1` or `2`) for the newest release that matches it.
Prereleases are only installed when named exactly.

### A matrix

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, ubuntu-24.04-arm, macos-latest, windows-latest]
        plum: [latest, "2.1"]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: plumlang/setup-plum@v1
        id: plum
        with:
          version: ${{ matrix.plum }}
      - run: echo "Using Plumlang ${{ steps.plum.outputs.version }}"
      - run: plum run tests/main.plum
```

## Inputs

| Name | Default | Description |
|---|---|---|
| `version` | `latest` | `latest`, an exact version (`2.1.4`, `v2.1.4`) or a prefix (`2.1`, `2`). |
| `token` | `${{ github.token }}` | A GitHub token for the releases API, to avoid its rate limits. Only sent to `api.github.com`. |
| `cache` | `true` | Cache the installed `plum` with `actions/cache`, keyed by version, OS and architecture. `false` always downloads afresh. |

## Outputs

| Name | Description |
|---|---|
| `version` | The exact version installed, such as `2.1.4`. |
| `path` | The directory holding the `plum` binary (added to `PATH`). |

## Platforms

| Runner | Release asset |
|---|---|
| Linux x64, arm64 (and riscv64 self-hosted) | `plum-VERSION-linux-ARCH.tar.gz` |
| macOS x64, arm64 | `plum-VERSION-macos-universal.tar.gz` |
| Windows x64, arm64 | `plum-VERSION-windows-ARCH.zip` |

These are the assets that [install.sh and install.ps1](https://plum.elixpo.com/docs/)
install. Self-hosted runners need `bash`, `curl`, `tar` and `sha256sum` or
`shasum`, plus `jq` or `python3` when `version` is not exact; Windows runners
use Git Bash.

## Licence

Code under `MIT WITH Plumlang-trademarks`, docs under
`CC-BY-4.0 WITH Plumlang-trademarks`; see [LICENSE](LICENSE) and
[NOTICE](NOTICE). Plumlang itself is under the licence in each release.
