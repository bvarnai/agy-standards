# Python engineering standards

These standards apply to Python CLIs, tools, and services whenever `pyproject.toml`, `setup.py`, or `requirements.txt` is present in the workspace.

## Architecture & package boundaries
1. **Entry point, `src/` layout & inward layering**:
   - Use a standard `src/<package>/` layout configured via `pyproject.toml` (`[project.scripts]`) so tests always run against the installed package rather than implicit `CWD` imports.
   - Keep `__main__.py` / `cli.py` entry points minimal: parse arguments, handle top-level `KeyboardInterrupt` / `SIGTERM` and domain exceptions, invoke application logic, and return an integer exit code to `sys.exit(main())`.
   - Never call `sys.exit()` or `print()` inside core/domain modules; enforce inward dependency direction: pure domain models and exceptions (`domain`, `errors`) <- execution/infrastructure adapters <- CLI or transport wiring (`cli`). Keep `__init__.py` free of side effects, heavy imports, and mutable module-level globals.
2. **Protocol boundaries, streams & minimal dependencies**:
   - Define narrow structural interfaces with `typing.Protocol` at the consumer boundary (`Runner`, `Inspector`, `Repository`) for dependency injection and deterministic testing rather than deep ABC inheritance trees or pervasive `unittest.mock.patch` monkeypatching.
   - Accept `TextIO` streams (`stdin`, `stdout`, `stderr`) at CLI handler boundaries instead of writing directly to `sys.stdout`/`sys.stderr`. Reserve `stdout` for primary output, machine-readable payloads (`JSON`/`TOML`), and `--help`/`--version`; route diagnostics, progress, warnings, and errors to `stderr` (respecting `NO_COLOR` and `TERM=dumb`).
   - Prefer the Python 3.12+ standard library (`argparse`, `pathlib`, `json`, `tomllib`, `dataclasses`, `subprocess`, `contextlib`, `urllib`); add focused libraries (e.g., **[Click](https://click.palletsprojects.com/)** or **[Typer](https://typer.tiangolo.com/)** for multi-subcommand CLIs, **[Rich](https://rich.readthedocs.io/)** for TUI rendering, **[Pydantic](https://docs.pydantic.dev/)** for complex external boundary schemas) only when complexity warrants it.
3. **Packaging & version metadata**:
   - Declare dependencies, tool configurations (`ruff`, `mypy`/`pyright`, `pytest`), and `requires-python` in a single `pyproject.toml` (PEP 621), managed with a lockfile (`uv.lock` via **[uv](https://docs.astral.sh/uv/)** or repository-standard installer).
   - Read runtime version metadata via `importlib.metadata.version("<dist-name>")` (with a `"0.0.0+dev"` `PackageNotFoundError` fallback) rather than duplicating version strings across source files.

## Code discipline, typing, errors & subprocesses
1. **Style, modern baseline (3.12+) & strict typing**:
   - Follow the **[Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)** and **[PEP 8](https://peps.python.org/pep-0008/)** (enforced via **[Ruff](https://docs.astral.sh/ruff/)**). Write concise docstrings on public modules, classes, and functions.
   - Require full type annotations on all function signatures, methods, and class attributes (`from __future__ import annotations`, modern `X | None` union syntax, `collections.abc` abstract generics). Avoid `Any` outside untyped third-party adapter boundaries.
   - Model value objects, DTOs, and configuration snapshots as immutable `@dataclass(frozen=True, slots=True)` (or `NamedTuple` / `enum.Enum` / `enum.StrEnum`) rather than untyped `dict[str, Any]` bags; never use mutable default arguments (`def fn(items: list[str] | None = None)`).
2. **Exceptions & CLI exit codes**:
   - Define a rooted domain exception hierarchy (`class AppError(Exception)`) carrying structured context and exit-code mapping. Chain underlying causes explicitly with `raise ConfigError(f"reading {path}: {exc}") from exc`; never use bare `except:` or silently swallow `Exception`.
   - Map domain exceptions deterministically to POSIX exit codes at the CLI boundary: `0` (success), `1` (general runtime error), `2` (usage/argument syntax error), domain-specific `3–N` codes where specified, and `130` (`SIGINT` / `KeyboardInterrupt`).
3. **Resources, filesystem & subprocess safety**:
   - Manage files, locks, temp directories, and network sessions with `with` context managers (`contextlib.contextmanager` / `ExitStack`). Use `pathlib.Path` with explicit `encoding="utf-8"` on all text reads/writes, and perform atomic file updates via `tempfile.NamedTemporaryFile(dir=target.parent, delete=False)` + `os.replace()`.
   - Isolate subprocess calls behind a dedicated boundary using `subprocess.run([...], shell=False, text=True, capture_output=True, timeout=...)` (never `shell=True` or `os.system` unless executing user-configured shell hooks). Sanitize child environments (`LC_ALL=C`, non-interactive flags), pass `--` before positional paths/refs, and inspect `CompletedProcess.returncode` explicitly.

## Verification & testing standards
1. **Parametrized unit tests**:
   - Write table-driven tests with **[pytest](https://docs.pytest.org/)** (`@pytest.mark.parametrize`) against injected `Protocol` fakes/stubs or pure domain functions; avoid brittle deep-path `unittest.mock.patch` targets on internal functions.
2. **Isolated integration tests & determinism**:
   - Isolate filesystem and CLI tests using `pytest` fixtures (`tmp_path`, `monkeypatch.setenv`, `monkeypatch.chdir`, `capsys`) with sanitized environment overrides (`HOME`, `XDG_CONFIG_HOME`, tool config vars) so tests never read or mutate host `~/.gitconfig`, `~/.ssh`, or user state.
   - Inject clocks or callables for time/retry logic rather than calling `time.sleep` in unit tests.
3. **Static analysis & CI parity**:
   - Require zero lint or formatting violations (`ruff check .`, `ruff format --check .`), zero type errors (`mypy --strict src tests` or `pyright`), and a clean test suite (`pytest`).
