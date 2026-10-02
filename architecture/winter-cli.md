# winter-cli architecture

The architecture of the real `winter` (winter-cli) codebase — how the generic conventions in `../architecture/*.md` are
realized here, plus the CLI's own argument conventions. Read this before adding or modifying a `winter` subcommand, so
you build with the existing structure instead of reverse-engineering it.

This is a **reference**, not a CLAUDE.md — it is not auto-loaded. Open it on demand from
`winter-context:/architecture/winter-cli.md` (reached via `architecture/index.md`).

## Layout

```text
tools/winter-cli/src/winter_cli/
├── cli.py                 # click entry point — wires subcommand groups
├── cli_context.py         # shared CLI context object
├── container.py           # DI container — binds Protocol seams to concrete adapters
├── config/                # config loading (.winter/config.toml + config.local.toml)
├── core/                  # cross-cutting Protocol seams (≥2-feature usage)
│   ├── cli_output_service.py            # ICliOutputService — TUI/CLI output abstraction
│   ├── cli_input_validation_service.py  # ICliInputValidationService — click-bound validators
│   ├── config_file.py                   # IConfigFileReader — TOML loader seam
│   ├── context_thread_pool.py           # ContextThreadPoolExecutor — thread pool that carries the submitter's context
│   ├── filesystem.py                    # IFilesystem — file/dir read/write seam
│   ├── subprocess_runner.py             # ISubprocessRunner — process execution seam
│   ├── tracing.py                       # ICommandTracer / IOperationTracer — span seams for tracing
│   └── internal/                        # adapters for the core Protocols
├── modules/               # feature packages
│   ├── workspace/         # everything reachable from `winter ws *` and `winter repo *`
│   │   ├── command.py             # click commands — thin wrappers over handlers
│   │   ├── handlers/              # CLI-shaped output formatting + arg parsing
│   │   │   ├── init_handler.py        # `winter ws init`
│   │   │   ├── destroy_handler.py     # `winter ws destroy`
│   │   │   ├── restack_handler.py     # `winter ws restack` — its own file because it plans then
│   │   │   │                          #   executes through two separate services (see below), not
│   │   │   │                          #   the single omnibus service every other row here shares
│   │   │   ├── workspace_handler.py   # every other `winter ws *` surface (list, status, connect,
│   │   │   │                          #   disconnect, checkout, reset, clean, fetch, pull, push,
│   │   │   │                          #   merge, update, prune, index, diff, worktrees) — an open
│   │   │   │                          #   list; a new `ws` verb lands here unless it earns its own file
│   │   │   └── repo_handler.py        # `winter repo {list,add,remove}`
│   │   ├── *_service.py           # domain orchestration (init / destroy / workspace (omnibus) /
│   │   │                          #   prune / env checkout / env reset / env clean / sync / push / merge /
│   │   │                          #   env_restack_plan (read-only planning) / env_restack (execution))
│   │   ├── *_reporter.py          # stream / json reporters for lifecycle events
│   │   ├── reporter_factory.py    # picks stream-vs-json reporter from --json flag
│   │   ├── repository_factory.py  # builds per-repo IWriteRepoRepository instances
│   │   ├── models/                # domain + service models (enums, dataclasses)
│   │   ├── repo_repository.py     # IReadRepoRepository / IWriteRepoRepository (Protocols)
│   │   ├── workspace_repository.py # IReadWorkspaceRepository (Protocol)
│   │   └── internal/              # concrete adapters: git_ops_service, gitpython_repository, git_operation, repo_error_factory, …
│   └── tui/               # textual-based dashboard (`winter dashboard`)
├── plugins/               # plugin loader — discovers extension click commands + TUI plugins
└── util.py
tools/winter-cli/tests/    # pytest; DI-friendly via injected fixtures (see tests/conftest.py)
```

The layout instantiates the `winter-context:/architecture/*.md` rules at once:

- **`./service-architecture.md`** — behavior lives in injected service classes (`*_service.py`), not module-level free
  functions; the other three rules below are facets of this one.
- **`./repository-pattern.md`** — `git`, `subprocess`, and other I/O libraries are confined to
  `modules/<feature>/internal/*.py` (and `core/internal/` for cross-cutting seams like `local_subprocess_runner.py`).
  The Protocol surface (`repo_repository.py`, `workspace_repository.py`) imports nothing from those libraries.
- **`./dependency-injection.md`** — every service receives its collaborators via constructor injection; everything is
  wired in `container.py`.
- **`./module-layout.md`** — Protocols at the feature root, adapters in `internal/`, cross-cutting Protocols in `core/`.
  Handlers may live as a flat `handler.py` or as a `handlers/` subpackage once the feature grows past one CLI surface
  (see [Handlers: flat vs. subpackage](#handlers-flat-vs-subpackage) below).
- **`./error-handling.md`** — library exceptions are wrapped at the call site via an injected concrete
  `RepoErrorFactory`, never via ad-hoc `raise X from Y`.

## Handlers: flat vs. subpackage

`modules/workspace/` outgrew a single `handler.py` and split into a `handlers/` subpackage organized by CLI surface
(init, destroy, restack, the workspace omnibus, repo). The split rule: keep a flat `handler.py` while a feature has one
cohesive handler; promote to `handlers/<surface>_handler.py` files when distinct CLI surfaces start sharing little code.
Re-export the handler classes from `handlers/__init__.py` so callers import from the subpackage root.

`./module-layout.md` shows the flat form as the default. This exemplar is the canonical reference for the split form.

**Declared exception:** `restack_handler.py` is the first `ws` verb to earn its own file *and* its own pair of services
(`env_restack_plan_service.py`, `env_restack_service.py`) rather than extending `workspace_handler.py` /
`WorkspaceService`. The split is a property of the domain, not just file size: planning (read-only, refuses before any
mutation) and execution (write-capable, resolves each `--onto` target and `up-to-date` verdict fresh at run time) are
different collaborators with different seams (`IReadRepoRepository` vs. `IWriteRepoRepository`), so `RestackHandler`
takes both services directly instead of routing through the omnibus. Follow this precedent — plan/execute services, plus
a dedicated handler — for any future `ws` verb whose pre-flight validation and mutation genuinely need to reason about
state at two different times, rather than folding it into the omnibus by default.

## Argument conventions

Read this before designing a new command's signature — the existing `winter ws` family is uniform, and a new command
joins it.

**Target selection is a positional, segment-aware glob `PATTERN` over `<env>/<repo>`.** Every command that acts on
worktrees (`status`, `pull`, `push`, `merge`, `fetch`, `diff`, …) takes its targets as positional `PATTERNS`, not a
flag. The glob is segment-aware: `*` does not cross `/`, and a bare env name with no `/` is treated as `<env>/*`.

```bash
winter ws status                 # all environments
winter ws status alpha           # alpha's worktrees (== alpha/*)
winter ws status alpha/winter    # one specific worktree
winter ws status '*/winter'      # every env's winter worktree
winter ws status '*/*'           # every env's every worktree (explicit)
```

A new command that selects worktrees **reuses this positional `PATTERNS` form** — it does not introduce a `--env NAME`
or `--name` flag for a target the positional pattern already expresses. The pattern already covers single-env,
single-worktree, and cross-env selection; a parallel flag fractures the surface and can't express `*/winter`. Match
`winter ws status` / `winter ws pull` (`[PATTERNS]...`, defaulting to all) for read-shaped commands; match
`winter ws merge` (a leading required positional like `SOURCE_REF`, then `[PATTERNS]...` with no implicit "all" default)
when an action needs an explicit target.

**Declared exception:** `winter ws restack`'s positionals are an ordered chain of literal env names, never a glob — see
`workspace:/context/winter-cli/usage/ws/patterns.md#winter-ws-restack--ordered-literal-chain-not-patterns`.

Reserve `--flags` for *modifiers* on the selected set, not for selection itself — `--json`, `--standalone`, `--all`,
`--exclude-pinned`, `--rebase`. The positional answers *which worktrees*; flags answer *how to act on them*.

## Adding a new `winter ws foo` subcommand

Follow this order — each step builds on the previous:

1. **click command** in `modules/workspace/command.py` — thin wrapper that parses click args and calls a handler.
2. **Handler** in `modules/workspace/handlers/<surface>_handler.py` (or a new `foo_handler.py` if `foo` is its own
   surface) — receives parsed args, calls a service, renders output via `ICliOutputService` (or returns structured JSON
   for `--json`).
3. **Service** — either extend `WorkspaceService` for read-shaped or env-spanning operations, or add
   `modules/workspace/foo_service.py` for a top-level lifecycle action (like `init` and `destroy`). Behavior goes in the
   service class, not module-level free functions — see `./service-architecture.md` (enforced by
   `tests/conventions/test_service_based_behavior.py`). Services depend on Protocols, not concretes.
4. **New I/O seam** (only if needed) — Protocol at `modules/workspace/<seam>.py`, concrete adapter at
   `modules/workspace/internal/<seam>.py`. Apply the I-prefix rule (enforced by
   `tests/conventions/test_protocol_naming.py`).
5. **Bind** the service and any new adapters in `container.py`. Services consume domain objects, not `WorkspaceConfig`
   directly — see `./dependency-injection.md` for the carve-outs (enforced by
   `tests/conventions/test_no_whole_config_injection.py`).
6. **Unit test** under `tests/modules/workspace/` (service tests) or `tests/modules/workspace/internal/` (adapter tests)
   — inject fakes for the Protocols. See `tests/modules/workspace/internal/test_git_ops_service.py` and
   `tests/modules/workspace/internal/test_write_repo_repository.py` for the fixture pattern.
7. **Surface the new command** in the docs so agents discover it from the docs, not from `--help`. Start at the usage
   index `workspace:/context/winter-cli/usage/index.md` and follow it to the right per-topic file — for a `winter ws`
   subcommand that's its own file under `workspace:/context/winter-cli/usage/ws/` (e.g. `usage/ws/checkout.md`), added
   to the `winter ws` hub's command table. If it's a whole new topic, add the file under `usage/` and a row routing to
   it from `usage/index.md`.

## Startup latency: lazy imports

`winter` is invoked per-command (e.g. the editor's worktrees picker shells out to `winter ws worktrees --json`), so
import cost on the hot path is felt directly. Two seams keep the cold imports off the `winter ws` path; respect both
when adding commands.

- **`LazyGroup` (cli.py).** The root group is a `LazyGroup` whose `_LAZY_SUBCOMMANDS` maps each top-level command name
  to a `"module:attribute"` reference, imported only when that command is dispatched (`--help` still lists them all
  without importing). **Adding a new top-level command** (a sibling of `ws` / `doctor` / `dashboard`, not a
  `winter ws foo` subcommand) means adding an entry here — don't `add_command` an eagerly-imported object.
- **`_lazy()` providers (container.py).** The DI `Container` is built on *every* invocation, so a module-top `import` of
  a command-specific tree (the `doctor`, `lint`, and `tui`/textual trees) would load it for `winter ws` too. Those
  providers use `providers.Factory(_lazy("module:Class"), ...)`, which imports the class on first resolution instead.
  **When binding a provider whose class drags in a heavy tree only one command needs**, wrap it in `_lazy(...)` rather
  than importing at the top; the workspace/core seams that `ws` itself needs stay eagerly imported.

`cli.py` also sets `sys.pycache_prefix` (a per-user cache dir) instead of `sys.dont_write_bytecode = True`, so the core
package keeps a warm `.pyc` cache across runs while plugin extension source trees stay free of `__pycache__/`. The
ordering — set before importing `click` / `winter_cli.*` — is load-bearing (hence the `E402` ignore for `cli.py`).

## Reporters as lifecycle event sinks

Services don't print. They emit lifecycle events to an injected reporter Protocol. Three reporter Protocols exist today:

- `IInitReporter` — used by both `winter ws init` and `winter ws destroy` (the destroy action vocabulary fits the init
  event shape).
- `IFetchReporter` — used by `winter ws fetch`.
- `IPullReporter` — used by `winter ws pull` and `winter ws push`.

For each Protocol there is a `Stream*Reporter` (human output) and `Json*Reporter` (`--json` mode). `ReporterFactory`
picks one based on the `--json` flag.

When adding a new lifecycle action, **extend an existing reporter's event vocabulary first** — don't fork a new reporter
Protocol unless the events truly don't overlap with init/fetch/pull. The `IInitReporter` reuse for destroy is the
precedent.

## Testing pattern

Full testing conventions — directory layout, conftest scoping, fake-vs-mock guidance, and per-layer assertion patterns —
live in `../standards/testing.md`. The winter-cli tree under `tests/` is its working reference; start at
`tests/conftest.py` and `tests/modules/workspace/test_init_service.py`. Run the suite with `mise run test` from the
package root (`mise run lint` / `mise run typecheck` likewise).

## Tracing

Opt-in OpenTelemetry tracing — the behavior is owned by `workspace:/context/winter-cli/tracing.md`. The active span
lives in `contextvars`, which a new thread does not inherit, so the thread rule keeps that context intact across a seam
that would otherwise drop it; the other rules keep the SDK and span sites behind the tracing seam, put every git span at
the one place git runs, and give every TUI worker thread a trace of its own. Each rule names what enforces it.

### Thread pools

Build every thread pool with `ContextThreadPoolExecutor` (`core/context_thread_pool.py`), never with
`concurrent.futures.ThreadPoolExecutor`.

- **Why.** A plain pool starts each worker with an empty context, so a task loses the span that was active when it was
  submitted: its inner spans parent on the wrong span, and a child process it starts carries the wrong `TRACEPARENT`.
  The helper runs each task in its own copy of the submitter's context, so the task sees the submitter's span and
  nothing the task sets leaks back.
- **Do.** Construct `ContextThreadPoolExecutor(max_workers=...)` where the pool is built, in place — it owns no I/O and
  needs no seam. `GitOpsService.executor()` hands the same pool to its callers.
- **Don't.** Import or name `ThreadPoolExecutor` anywhere in `src/` outside the helper. A bare `threading.Thread` whose
  work should inherit the submitter's span has the same gap; copy the context into it with
  `contextvars.copy_context().run`. A TUI thread follows the session-root rule below instead, and the tracer adapter's
  own flusher thread is exempt.
- **Enforced by** `tests/conventions/test_thread_pools_carry_context.py`.

### OpenTelemetry stays in its adapter

Import `opentelemetry` only in `core/internal/otel_command_tracer.py`.

- **Why.** A process with tracing off must load no OpenTelemetry code (see
  [Startup latency: lazy imports](#startup-latency-lazy-imports)): the container reaches the adapter lazily, and the
  no-op adapter imports nothing from the SDK. An SDK import anywhere else loads it on every command, and ties a span
  site to a vendor instead of the Protocol.
- **Do.** Reach tracing through the Protocols in `core/tracing.py`. Anything that needs a new tracing capability extends
  a Protocol and implements it in all three adapters (OTel, no-op, unavailable).
- **Don't.** Write `import opentelemetry...` or `from opentelemetry... import ...` anywhere else in `src/`, including
  under `TYPE_CHECKING` or inside a function.
- **Enforced by** `tests/conventions/test_opentelemetry_only_in_its_adapter.py`.

### Spans open through the injected Protocols

Open a span with `IOperationTracer.operation(name, attributes)`, taking `IOperationTracer` as a required constructor
parameter and binding it to the container's one `command_tracer` provider.

- **Why.** The tracer decides the parent (the span current when the operation opens), makes the span current for its
  body so child processes and nested spans parent on it, and turns an escaping exception into error status and the
  exception class name. A site that builds its own span gets none of that, and a default tracer silently drops spans
  where a construction site forgets to pass one.
- **Do.** Depend on the narrowest tracing Protocol: `IOperationTracer` for a span site, `ISessionTracer` for the TUI.
  Keep attribute values to names, counts, exit codes and booleans, so a span can carry its operation's identity and
  nothing else.
- **Don't.** Put free text on a span: an exception message, stderr, command output, argv or a config value. Don't give
  the tracer parameter a default, and don't depend on `ICommandTracer` beyond the CLI boundary that owns the command
  span.
- **Enforced by** review; no mechanical check applies, because span sites are the code's own operations.

### Git spans at the GitPython boundary

Open a GitPython repository only in the workspace adapters (`modules/workspace/internal/`), and declare the git
operation on every function there that opens one.

- **Why.** Every git command winter runs through GitPython runs on a repository that `ReadRepoRepository`,
  `WriteRepoRepository`, `GitPythonRepository` or `ReadWorkspaceRepository` opened. Declaring the operation at the
  opener gives each git call its `git <operation>` span by construction, with no list of verbs to keep current. Code
  that receives the open repository, such as a helper that takes `r`, runs inside the opener's span and needs none of
  its own.
- **Do.** Decorate each opener with `@GitOperationDeclaration("<subcommand>")`
  (`modules/workspace/internal/git_operation.py`), naming the git subcommand the function performs (`fetch`,
  `worktree add`, `status`). It opens the span through the adapter's injected `IOperationTracer`, so every adapter takes
  that tracer as a required constructor parameter. The declaration is a callable class, not a free function, because it
  acts on an instance's tracer and free functions are reserved for pure helpers (`./service-architecture.md`);
  `EnvTargetDeclaration` is the precedent. The one function that opens a repository without a span, env discovery's
  worktree listing, carries `@GitOperationExemption(reason)` instead.
- **Attributes come from the call's own arguments.** A `FeatureWorktree` argument supplies `winter.repo` and
  `winter.env`; a `ProjectRepository` or `StandaloneRepository` supplies `winter.repo` only. A path-taking private
  opener takes the `repo_name` and `env` it should carry, and its public callers pass them from the domain object they
  hold. Every method of `IGitRepository` but `list_worktrees` takes a required keyword `repo_name`, which each caller
  passes from the repo object it holds, and all but `clone` also take a required keyword `env`, which each caller sets
  to the env it acts in or to `None`.
- **Don't.** Derive the env from a path: a standalone repo may be configured at a path that looks like a worktree. Open
  `git.Repo(...)` or `git.Repo.clone_from(...)` outside `modules/workspace/internal/`, and don't give an adapter's
  tracer a default.
- **Enforced by** `tests/conventions/test_git_spans_at_the_gitpython_boundary.py`, which fails on a function in the
  adapter package that opens a repository with neither declaration, on any repository opened outside that package, and
  on a class with a declared method whose `__init__` does not assign `self._tracer`.

### TUI workers open session roots

Open a session root as the whole body of every TUI thread worker: each `@work(thread=True)` function in `modules/tui/`,
and each function a raw `threading.Thread` there runs.

- **Why.** A Textual thread worker runs on an executor thread that starts with an empty context, so it holds no span:
  everything it does would trace as spans under the dashboard's long-lived session span, or as unparented fragments, and
  a refresh could not be told from the one before it. `session_root` starts a new trace per refresh or user action,
  linked to the session span, so a slow refresh is one short trace that shows which git call took the time. The agent
  matrix's raw thread is the same boundary, because it is also not a pool thread and carries no context.
- **Do.** Make the function body, after an optional docstring, one
  `with self._session_tracer.session_root("dashboard <purpose>"):` block, taking `ISessionTracer` as a required
  constructor parameter that the container binds to the one `command_tracer` provider. Name the purpose for what the
  worker does (`refresh workspace`, `load repo detail`, `plugin action`); the name is a fixed phrase, never a repo, env
  or path.
- **Don't.** Start a thread worker without a root, put work outside the `with`, or give the screen's tracer parameter a
  default. A one-shot command never starts a session or its background export; only the `dashboard` command does.
- **Enforced by** `tests/conventions/test_tui_workers_open_a_session_root.py`, which fails on a decorated worker or a
  thread target whose body is not one `session_root` block.

## Network resilience

`GitOpsService` wraps every `git fetch` / `pull` / `push` and retries up to 3 times with jittered exponential backoff
when the stderr matches a transient pattern:

```text
Connection closed by ... port 22
kex_exchange_identification
remote end hung up
Connection timed out
```

Anything else is a hard failure on the first attempt. `is_transient_git_error` in
`modules/workspace/internal/git_ops_service.py` is the source of truth for the substring list — extend it there when a
new transient class appears.

## Cross-references

- Conventions this codebase instantiates: `./service-architecture.md`, `./dependency-injection.md`,
  `./repository-pattern.md`, `./error-handling.md`, `./module-layout.md`.
- Repository-pattern reference implementation: `exemplars/python/repo_pattern.py`.
- User-facing CLI command reference (hub): `workspace:/context/winter-cli/index.md`.
- Installation: `workspace:/context/winter-cli/setup.md`. Extension hook contract:
  `workspace:/context/winter-cli/configuration/extensions.md#extension-hooks`.
