# bcov

`bcov` measures Go branch coverage by counting every outcome of every decision.

A decision is an `if` or `for` condition, or a `switch` or type switch:

- `if` and `for` have two outcomes: the condition is false, or it is true.
- `switch` and type switch have one outcome per clause. A switch without a `default`
  gets a synthetic default, so "no clause matched" is an outcome too.

Go's built-in coverage counts statements only. `bcov` adds the branch outcomes that
statement coverage misses, such as the false path of an `if` without an `else`.

## Usage

```bash
go run github.com/fupacat/bcov@v0.1.0 -root . -out reports/_bcov
BCOV_OUT=$PWD/reports/_bcov/hits go test -vet=off -count=1 \
  -overlay reports/_bcov/overlay.json ./...
```

The first command writes instrumented copies of each package under `-out`, plus:

- `overlay.json`: maps original files to instrumented copies for `go test -overlay`.
- `decisions.json`: each decision's file, line, kind and outcome indexes.

The second command runs the tests against the copies. Each package's test hook writes
its outcome counts to `$BCOV_OUT/<package>.txt` when the tests finish. Each line is
`index count`, where `index` matches `base + i` for outcome `i` in `decisions.json`.

The repository tree is never modified. The instrumented copies are applied only through
`-overlay`.

## Limits

- Packages with their own `TestMain` are not supported. The tool adds one per package.
- `select` statements are rejected with an error.
- `-vet=off` is needed because `go test`'s vet step cannot read overlaid files.
- Packages with no tests report zero hits for every outcome.

## Flags

- `-root` repository root (default `.`)
- `-out` output directory (default `reports/bcov`)
- `-scope` comma-separated directories under root to measure (default `cmd,internal`)

## Private repository

While the repository is private, Go needs access to fetch it:

```bash
export GOPRIVATE=github.com/fupacat/bcov
```

and git credentials for `github.com/fupacat`.
