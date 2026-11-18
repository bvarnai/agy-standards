# Go engineering standards

These standards apply to Go CLIs, tools, and backend services whenever `go.mod` is present in the workspace.

## Architecture & package boundaries
1. **Entry point & inward layering (`internal/`)**:
   - Keep `main.go` / `cmd/<app>/main.go` minimal: wire `signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)`, invoke an `app.Run(...)` or server entrypoint, and exit cleanly (`os.Exit`).
   - Never call `os.Exit`, `log.Fatal`, or `panic` inside `internal/`; return explicit errors up to the entrypoint. Avoid `init()` side effects and mutable package-level globals.
   - Keep application code inside `internal/` with inward dependency direction: pure domain types and error categories (`domain`, `errcode`) <- execution/infrastructure packages <- CLI or transport wiring (`cli`, `http`). Infrastructure packages must never import CLI/handler layers.
2. **Small interfaces, streams & minimal dependencies**:
   - Follow **"Accept interfaces, return concrete structs"**; define narrow interfaces at the consumer boundary rather than mirroring Java-style `Impl` hierarchies.
   - Accept `io.Reader` / `io.Writer` (`stdin`, `stdout`, `stderr`) in CLI handlers rather than writing directly to `os.Stdout`/`os.Stderr`. Reserve `stdout` for primary output, machine-readable payloads (`JSON`/`YAML`), and `--help`/`--version`; write logs, progress, warnings, and errors to `stderr` (via [`log/slog`](https://pkg.go.dev/log/slog) for services/structured logs, or a mutex-guarded logger for interactive CLIs respecting `NO_COLOR` and `TERM=dumb`).
   - Prefer the Go standard library (`flag`, `encoding/json`, `net/http`, `slices`, `maps`, `context`); add lightweight focused libraries (e.g., **[Cobra](https://github.com/spf13/cobra)** for multi-subcommand CLIs, `yaml.v3`) only when complexity warrants it, and avoid heavyweight wrappers or shelling out to text utilities (`jq`, `sed`, `awk`, `grep`).
3. **Build metadata**:
   - Inject release metadata (`Version`, `Commit`, `Date`) via `-ldflags` with `"dev"` fallbacks, supplemented by `runtime/debug.ReadBuildInfo()` (`vcs.revision`, `vcs.time`, `vcs.modified`) rather than hardcoding versions in source.

## Code discipline, errors & concurrency
1. **Style & idioms**:
   - Follow the **[Google Go Style Guide](https://google.github.io/styleguide/go/guide)**, **[Effective Go](https://go.dev/doc/effective_go)**, and canonical `gofmt` (`gofmt -s`) formatting; write GoDoc comments starting with the symbol name on all exported types and functions.
   - Keep subcommand and HTTP handlers thin by separating flag/request binding from core execution; use `defer` for deterministic resource and transactional cleanup (checking errors on writable `Close()` paths).
2. **Errors & CLI exit codes**:
   - Wrap internal errors with lowercase context without trailing punctuation (`fmt.Errorf("reading config %s: %w", path, err)`) and inspect with `errors.Is` / `errors.As` (using `errors.Join` or `Unwrap() []error` where multiple causes apply). Never discard errors (`_ = ...`) without an explicit inline comment explaining why it is safe.
   - Map sentinel or structured domain errors deterministically to POSIX exit codes at the CLI boundary: `0` (success), `1` (general runtime error), `2` (usage/flag syntax error), domain-specific `3–N` codes where needed, and `130` (`SIGINT`/`SIGTERM` cancellation).
3. **Context, concurrency & subprocesses**:
   - Pass `context.Context` as the first parameter on all I/O, network, database, and subprocess calls, checking `ctx.Done()` in loops and workers.
   - Bound goroutine lifetimes with `context.Context`, `sync.WaitGroup`, or [`errgroup`](https://pkg.go.dev/golang.org/x/sync/errgroup); never use `http.DefaultClient` or `http.ListenAndServe` without explicit timeouts.
   - Isolate subprocess calls behind a dedicated boundary using `exec.CommandContext` without an intermediate shell (unless executing user-configured hooks). Set a `WaitDelay` with graceful `SIGTERM` cancellation, sanitize child environments (e.g., `LC_ALL=C`, non-interactive prompt flags), pass `--` before positional arguments/paths, and parse structured or NUL-delimited (`-z`) output over human formatting.

## Verification & testing standards
1. **Table-driven unit tests**:
   - Write table-driven tests (`tests := []struct{ name string; ... }`) with `t.Run(tt.name, ...)` against injected interfaces or `httptest`, using standard checks / [`go-cmp`](https://pkg.go.dev/github.com/google/go-cmp/cmp) or **[testify](https://github.com/stretchr/testify)** (`assert`/`require`) consistently with the repository.
2. **Isolated integration tests & determinism**:
   - Isolate filesystem/CLI integration tests using `t.TempDir()`, `tb.Helper()`, and sanitized environment overrides (e.g., `HOME`, tool-specific config paths) so tests never read or mutate host `~/.gitconfig`, `~/.ssh`, or user state.
   - Never use `time.Sleep` for synchronization; coordinate via channels, `sync.WaitGroup`, `context.WithTimeout`, and `t.Cleanup`.
3. **Static analysis & CI parity**:
   - Require zero unformatted files (`gofmt -l .`), verified modules (`go mod verify`), clean builds (`go build ./...`), `go vet ./...`, and `go test -race ./...` (plus **[`golangci-lint`](https://golangci-lint.run/)** and **[`govulncheck`](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)** when configured).
