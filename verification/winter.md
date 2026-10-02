# Verifiability matrix — winter

An inventory of the verification methods available for the winter ecosystem. Each entry is one way a skill or agent may
assert a winter change is correct.

Method ids use the following scheme: commands and manual methods are `<scope>:<method>` (a manual method's method name
is `manual`, optionally suffixed with a subject, as in `winter-context:manual-shared-core` and `winter:manual-tracing`);
`cli-probe:*` and `markdown:*` are category scopes — the workspace-level `winter` CLI probes and the mechanical markdown
gates that every adopting repo runs identically; tools are unscoped under a flat `tool:`. The `winter:*` Python-QA rows
run from `tools/winter-cli/` inside the `winter` repo worktree; each sibling project's rows run from that project's own
worktree; `cli-probe:*` run from a configured workspace root. Choosing a scope for a new winter method: Python QA of
winter's own source is `winter:*`; a behavioral probe of the installed CLI against a workspace is `cli-probe:*`.

## Commands

Verification that runs as a single command — exit 0 is the pass signal unless noted.

| Method                          | Command                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| winter:build                    | `uv sync` — installs all dependencies including the `dev` group (pytest, ruff, pyright). Exit 0 = environment ready.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| winter:unit-test                | `uv run pytest` — runs the full pytest suite under `tests/`. Shorthand: `mise run test`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| winter:lint                     | `uv run ruff check .` — ruff linting over `src/` and `tests/`. Shorthand: `mise run lint`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| winter:format                   | `uv run ruff format --check .` — every Python file in `src/` and `tests/` matches the format ruff's `[tool.ruff]` config declares; `uv run ruff format .` (`mise run format`) writes the fix.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| winter:typecheck                | `uv run pyright` — pyright type-checking over `src/` and `tests/` in standard mode. Shorthand: `mise run typecheck`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| winter-service-tmux:build       | `uv sync` — install deps in `alpha/winter-service-tmux/`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| winter-service-tmux:unit-test   | `uv run pytest` (`mise run test`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| winter-service-tmux:lint        | `uv run ruff check .` (`mise run lint`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| winter-service-tmux:typecheck   | `uv run pyright` (`mise run typecheck`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| winter-service-docker:build     | `uv sync` — install deps in `alpha/winter-service-docker/`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| winter-service-docker:unit-test | `uv run pytest` (`mise run test`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| winter-service-docker:lint      | `uv run ruff check .` (`mise run lint`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| winter-service-docker:typecheck | `uv run pyright` (`mise run typecheck`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| winter-plugin-api:build         | `uv sync` — install deps in `alpha/winter-plugin-api/`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| winter-plugin-api:unit-test     | `uv run pytest` (`mise run test`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| winter-plugin-api:lint          | `uv run ruff check .` (`mise run lint`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| winter-plugin-api:typecheck     | `uv run pyright` (`mise run typecheck`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| winter-context:unit-test        | `cd agent-context/scripts && python3 -m unittest discover` — runs the stdlib test suites for all four documentation lints: `test_doclint` (the three semantic lints, against the fixture tree) and `test_markdown_style` (the style check, against stub `dprint` / `rumdl` binaries, so neither tool need be installed).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| winter-context:lint             | Run `python3 agent-context/scripts/lint_path_notation.py --repo .`, `python3 agent-context/scripts/lint_doc_references.py --repo . --orphan-severity fail`, and `python3 agent-context/scripts/lint_link_anchors.py --repo .` from the winter-context worktree. The scripts follow the `winter lint` contract and always exit 0; no NDJSON findings on stdout = pass. The fourth contributed check is `markdown:format` / `markdown:lint` below — run those directly rather than through `lint_markdown_style.py`, which only re-emits their output.                                                                                                                                                                                                                                                                                                                                                                                                        |
| winter-canon:lint               | From the winter-context worktree with `CANON_REPO` set to an alpha winter-canon worktree, run `python3 agent-context/scripts/lint_path_notation.py --repo "$CANON_REPO"`, `python3 agent-context/scripts/lint_doc_references.py --repo "$CANON_REPO" --orphan-severity fail`, and `python3 agent-context/scripts/lint_link_anchors.py --repo "$CANON_REPO"`. The scripts always exit 0; no NDJSON findings on stdout = pass. This directly verifies canon path notation, routing references, and anchors against the selected worktree.                                                                                                                                                                                                                                                                                                                                                                                                                     |
| winter-workflow:unit-test       | `python3 tests/test_lint_agents.py && python3 tests/test_lint_methodology.py` — exercises both contributed lint checks and their real-tree assertions.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| winter-workflow:lint            | Run `python3 scripts/lint-agents.py .` and `python3 scripts/lint-methodology.py .` from the winter-workflow worktree. Then, from the winter-context worktree with `WORKFLOW_REPO` set to that workflow worktree, run `python3 agent-context/scripts/lint_path_notation.py --repo "$WORKFLOW_REPO"`, `python3 agent-context/scripts/lint_doc_references.py --repo "$WORKFLOW_REPO" --orphan-severity fail`, and `python3 agent-context/scripts/lint_link_anchors.py --repo "$WORKFLOW_REPO"`. The scripts follow the `winter lint` contract and always exit 0; no NDJSON findings on stdout = pass. The last three directly verify workflow path notation, routing reachability, and anchors rather than relying on a workspace-wide dispatcher run.                                                                                                                                                                                                         |
| winter-docs:build               | `npm run build` — builds the Astro site and generated agent-readable outputs under `dist/`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| markdown:format                 | `dprint check` — every markdown file matches the format `dprint.json` declares; `dprint fmt` writes the fix. Run from the root of any repo carrying `dprint.json`: `winter`, `winter-canon`, `winter-docs`, `winter-context`, `winter-service-docker`, `winter-service-tmux`, `winter-workflow`. Needs the `dprint` binary on `PATH`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| markdown:lint                   | `rumdl check .` — the structural markdown lint `.rumdl.toml` declares; `rumdl check . --fix` applies the autofixable subset. Same seven repos as `markdown:format`. Needs the `rumdl` binary on `PATH`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| cli-probe:doctor                | `winter doctor` — built-in preflight probes for the workspace and every installed extension; pass/warn/fail per probe with remediation hints. Exit 1 = any probe failed (warnings allowed). The command that turns the always-exit-0 introspection probes below into a failing gate.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| cli-probe:status                | `winter ws status <env>` — git status across all worktrees in a feature env: untracked files, uncommitted changes, ahead/behind counts.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| cli-probe:graph                 | `winter graph` — prints the extension dependency graph; verifies every declared extension resolves and the wiring is coherent.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| cli-probe:lint                  | `winter lint` — runs all registered lint checks (built-in extractability plus harness-registered path-notation, doc-reference, and anchor checks). Emits NDJSON findings; non-zero on any finding.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| cli-probe:capabilities          | `winter capabilities` — lists every capability slot, its bound extension, how the binding resolved, and whether each candidate entrypoint exists on disk. Always exits 0 — assert on the output (or `--json`); `winter doctor` is what fails on misconfiguration.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| cli-probe:agents                | `winter agents` — read-only; lists the resolved model and effort for every installed agent × harness, with the layer each value came from. Assert on `--json` rather than exit status; `winter:/context/winter-cli/usage/agents.md` owns its exit codes and schema, and `winter doctor` is what fails on misconfiguration.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| cli-probe:ext-verify            | `winter ext verify <extension>` — runs the capability-spec conformance checks (accepts-action / refuses-unknown / forwards-params) against a provider's entrypoint. Exit 0 = conforms. Accepts a local path, so it verifies an in-progress provider worktree: `winter ext verify ./alpha/winter-service-tmux`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| cli-probe:env                   | `winter env <scope>` — prints the computed runtime env vars (port base, env-band entries) for `<env>` or `workspace`. Assert the ports and injected vars match expectation. Without `--resolve`, a command entry's own declared key prints its masked placeholder and the command never runs; a reference to a key a multi-value command entry only imports instead exits 1 with no output, since that key is never known while the command is gated off — either way nothing runs, so this default form stays read-only and is safe to point at the live workspace via `tool:winter-core-override`. **`--resolve` runs every configured command entry for real, so it is never safe to point at a live workspace regardless of what that workspace's config happens to declare today — probe it from a scratch workspace whose own `.winter/config.toml` declares a command entry instead, never the live workspace edited to grow one for the occasion.** |
| cli-probe:provision-plan        | `winter provision <env> --dry-run` — resolves and prints the provision handler plan without running anything or starting a service. Validates handler wiring non-destructively; add `--json` for a structured plan.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| cli-probe:clean-plan            | `winter clean <env> --dry-run` — resolves and prints the clean plan without running anything or starting a service. Validates handler wiring non-destructively; add `--json` for a structured plan.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| cli-probe:restack-plan          | `winter ws restack <chain> <base> --dry-run --json` — resolves and prints the restack plan (every link, each participating repo's boundary sha and source) without rebasing anything; `RestackHandler.run` never reaches the execute service on a dry run. Read-only by construction, so — unlike every other `ws restack` scenario in `winter:manual` above — it's safe to point at a live workspace via `tool:winter-core-override`. **Exception to this table's exit-0-is-pass default:** a refused chain exits 1 even though `--dry-run` did exactly what this probe promises — previewed without mutating; treat the presence of `refusals` in the printed JSON, not the exit code alone, as this probe's pass signal.                                                                                                                                                                                                                                 |
| cli-probe:service-describe      | `winter service describe` — the bound provider reports its declared service catalog; confirms the service manifest parses and services are discoverable.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |

## Manual testing

Verification no single command performs — it needs a running stack, spans many invocations, or rests on judgment. Any of
these can be pointed at in-progress code with the override Tools below.

### winter-context:manual-shared-core — cold shared-core reach and behavior

From a spawning session at the configured workspace root, give a fresh agent with no session history this scope
preamble: "Use `alpha/winter-context` as the harness root, resolve `winter-canon:/` against `alpha/winter-canon`, and do
not inspect the installed `.winter/ext` clones." Then give it this cue: "Design the winter file layout and
responsibility split for an operation that both a session skill and an isolated agent execute. The operation consumes
project constraints and sometimes needs a human decision." The scope preamble binds the in-progress dependency without
naming the convention the agent must discover; do not add a convention path to the cue.

Observe and record the two verdicts separately. **Reached:** the agent traverses `alpha/winter-context/index.md` →
`agent-context/index.md` → `methodology-packaging.md` and follows its universal-owner citation into the alpha canon's
`facts-vs-methodology.md`. **Behaved:** it selects the shared-core shape, places one caller-neutral process and its
assets under the workflow methodology root, keeps constraints canonically addressed at the target, makes the skill and
agent thin adapters with their distinct input/preamble responsibilities, and expresses the human decision as a semantic
runtime port rather than a native tool call. Pass requires both verdicts to be yes. This is one hand-driven cold smoke
run following `winter-canon:/evaluating-harness-changes.md`; there is no automated eval runner, so record the cue,
transcript evidence, and verdicts manually.

### winter-service-tmux:manual — service orchestration end-to-end (tmux)

Surface: the `winter-service-tmux` provider bound to the `service` slot. Bring a feature env's services up, then confirm
state: `winter service up <env>` followed by `winter service status <env>`. Pass: every declared pane appears in the
named tmux session and each service reports running; `winter service down <env>` tears the session down cleanly.
Requires `[capabilities] service = "winter-service-tmux"`, a valid service manifest, and `tmux` on the host. **Gap**: no
automated harness drives a real tmux session — exercised manually in a development workspace. Verifier hygiene: match
the full unique command string when probing pids (never a short prefix that can match your own shell process); confirm
the pids you act on are the intended ones; let the session settle before reading status.

### winter-service-docker:manual — service orchestration end-to-end (docker)

Surface: the `winter-service-docker` provider. Same up/status/down gestures, but the provider reports real container
health, so `winter service up <env> --wait` is a genuine readiness gate. Pass: containers reach healthy,
`winter service status` maps container state to winter state, and per-env isolation holds — distinct
`COMPOSE_PROJECT_NAME` and `WSD_PORT_*` host ports let two envs run side by side without collision. Requires the docker
daemon and compose v2 (checked by `winter doctor`). **Gap**: no automated harness drives real containers. Bounded
follow: `winter service logs '<glob>' -f` blocks until SIGINT — bound it
(`timeout -s INT 10 winter service logs '*/backend' -f`) or run it backgrounded and cancel when done.

### winter:manual — feature-environment lifecycle (ws init/destroy)

`winter ws init <env>` creates every project worktree declared in `.winter/config.toml` by cloning or adding git
worktrees; `winter ws destroy <env>` removes them. Exit 0 plus the expected worktrees on disk verifies the init/destroy
path against real git remotes. **Gap**: no dedicated CI job runs this against a throwaway workspace — exercised as part
of normal feature development.

### winter:manual — destructive branch-moving verbs (ws checkout / ws reset)

`winter ws checkout <env> <feature-branch>` force-attaches HEAD onto the env branch and force-moves it;
`winter ws reset PATTERNS... REF [--hard]` moves a matched worktree's branch pointer directly. Both mutate real
worktrees and neither can be scoped below `PATTERNS`/`ENV` — a wrong pattern reaches every worktree it matches, not just
the one under test. Verify against a throwaway env (`tool:winter-ws-init`) or a scratch workspace, never a live env
holding unpushed work; use `--dry-run`/`--json` first to confirm the plan, then re-run for real and confirm the per-repo
report matches (branch attached where expected, no worktree left dirty or detached that shouldn't be). Pass: the refusal
guards (`refused-missing-ref`, `refused-dirty`, `refused-abandonment`, `refused-ambiguous-sha`) fire on the scenarios
that should trip them and the all-or-nothing property holds — one refused repo blocks every repo, none partially
mutated. **Gap**: no dedicated CI job runs either command against real git worktrees — exercised via the real-git
regression tests under `tools/winter-cli/tests/modules/workspace/test_env_checkout_service_detached_head.py` plus manual
verification in a scratch or throwaway workspace.

### winter:manual — destructive file-removing verb (ws clean)

`winter ws clean PATTERNS...` removes untracked files and untracked directories from every matched non-pinned worktree
(`git clean -fd`). Held separate from the branch-moving row above because nothing it does is a ref move: its blast
radius is measured in files, so the branch-attached / not-detached audit that verifies `checkout`/`reset` cannot detect
a bad clean, and it carries **no refusal guards at all** — there is no precondition to violate, so the confirmation
prompt is the only gate. No reflog stands behind a deleted untracked file, and unlike `ws destroy` — also irreversible —
a bad clean leaves the env looking intact, so nothing announces it. Verify against a throwaway env
(`tool:winter-ws-init`) or a scratch workspace, never a live env holding untracked work. Pass: `--dry-run` lists paths
and removes nothing; preview and removal apply the same selection rules (both derive from `git clean`, `-nd` and `-fd`),
including for an **empty untracked directory**, which `git ls-files --others` cannot see — note they are enumerated at
different moments, so a file created in between is removed and reported rather than the two sets being frozen equal;
ignored files (`.venv`, `node_modules`, `*.pyc`) survive every mode; an untracked nested git repository is neither
listed nor removed; tracked modifications and deletions are untouched; the prompt fires at any worktree count unless
`--force`; `--json` without `--force`/`--dry-run` is refused rather than prompting onto the NDJSON stream; and a
**partial failure** (make a directory unreadable so `git clean -fd` deletes one file, warns, and exits 1) still reports
the paths already removed, names the worktree it stopped on, leaves later worktrees untouched, and exits non-zero. For a
cheap external assertion compare per-repo `untracked` counts from `winter ws status --json` before and after — but note
the units differ: status counts one entry per *file* recursively and cannot see an empty untracked directory, while
clean counts one entry per *directory*, so the two numbers will not match and a `0 → 0` reading does not prove nothing
was removed. **Gap**: no CI job drives the command end-to-end — covered by real-git adapter tests in
`tools/winter-cli/tests/modules/workspace/internal/test_write_repo_repository.py` plus manual verification in a
throwaway workspace.

### winter:manual — destructive artifact-removing verb (clean)

`winter clean PATTERNS...` runs each matched handler's declared `clean` command across the sub-target chain
(`dependency` → `resource` → `data`, same within-sub-target order as `provision`) in the handler's own cwd — a
project-declared removal command, not a git-level operation: unlike `ws clean` above (`git clean -fd` over untracked
files), this verb touches only what a handler's `clean` command names, so it can remove build output, caches, or other
disposable artifacts a `.gitignore` entry would hide from `ws clean` entirely. Verify against a throwaway env
(`tool:winter-ws-init`) or a scratch workspace, never a live env holding build state you still need — but note a
throwaway env only contains `feature-environment`/`feature-worktree`-scope handlers; a `workspace`-scope handler's
`clean` command runs at the live workspace root regardless of which env you name it against (see
`workspace:/context/worktree-ops.md#verifying-destructive-commands-safely`), so verifying one of those needs the fully
scratch workspace instead — which is why the fixture below declares its handlers at `workspace` scope, on a scratch
workspace's own config, matching the canonical example in `workspace:/context/winter-cli/configuration/provision.md`.

**Fixture — no handler in this workspace's own manifest declares `clean`** (verified: no `clean` key appears anywhere in
`.winter/config.toml` or any installed extension's `winter-ext.toml`), so a throwaway env built straight from
`tool:winter-ws-init` genuinely exercises only the no-op path — every sub-target reports `no_handlers`, no command runs
— which is exactly the `no_handlers` criterion below and nothing else. To exercise a real `clean` command, add scratch
`[[provision.*]]` entries to a scratch workspace's own `.winter/config.toml` (or a throwaway extension) before running:

```toml
[[provision.dependency]]
scope = "workspace"
name  = "clean-fixture-fail"
apply = "true"
clean = "exit 1"                                             # the deliberately failing handler

[[provision.dependency]]
scope = "workspace"
name  = "clean-fixture-dep-sibling"
apply = "true"
clean = "touch $WINTER_WORKSPACE_DIR/.dep-sibling-cleaned"   # sibling in the same sub-target as the failure

[[provision.resource]]
scope = "workspace"
name  = "clean-fixture-resource"
apply = "true"
clean = "touch $WINTER_WORKSPACE_DIR/.resource-cleaned"      # a later sub-target

[[provision.resource]]
scope = "workspace"
name  = "clean-fixture-no-clean"
apply = "true"                                                # declares no `clean` — the `--name` warn target

[[provision.data]]
scope = "workspace"
name  = "clean-fixture-data"
apply = "true"
clean = "touch $WINTER_WORKSPACE_DIR/.data-cleaned"           # the last sub-target
```

Preview first — `winter clean <env> --dry-run` (`cli-probe:clean-plan`) — then re-run for real and confirm the
per-handler report matches. Pass: the command runs to completion with stdin closed — there is no confirmation prompt to
begin with. `--force` is rejected as an unknown option: `winter clean <env> --force` fails with
`Error: No such option: --force`. Every handler declaring `clean` for the matched sub-targets has its command executed —
confirmed by `.dep-sibling-cleaned`, `.resource-cleaned`, and `.data-cleaned` all appearing despite
`clean-fixture-fail`'s non-zero exit; a sub-target where nothing declares `clean` reports `no_handlers` and starts no
service, i.e. is a true no-op — verified against the unmodified throwaway env above, since this workspace's own manifest
declares no `clean` anywhere; an explicit `--name` selector naming a handler with no declared `clean` produces a warn
line rather than silently doing nothing — `winter clean <env> --name workspace.clean-fixture-no-clean` against the
fixture above; the dependency trees a handler's `clean` command does not itself name (installed packages, container
images, and the like) survive the run untouched; and a failing clean ends the run non-zero **without stopping it** —
`clean-fixture-fail`'s failure does not prevent `clean-fixture-dep-sibling` (same sub-target), `clean-fixture-resource`
(a later sub-target), or `clean-fixture-data` (the last sub-target) from running, each handler gets its own reported
result line, and the process still exits non-zero. **Gap**: no CI job drives the command end-to-end — covered by the
clean-selection and clean-run-path tests in `tools/winter-cli/tests/modules/provision/test_provision_service.py` and
`tools/winter-cli/tests/modules/provision/test_provision_execution_service.py` plus manual verification in a throwaway
workspace.

### winter:manual — history-rewriting chain verb (ws restack)

`winter ws restack ENV... BASE [--cut ENV_OR_REF] [--dry-run] [--json]` rebases a locally-existing chain of env
worktrees across every non-pinned project repo, all-or-nothing, the same reach as `checkout` / `reset` / `clean` — but
it rewrites history in place rather than just moving a branch pointer or deleting files, so a bad run is recoverable
only from each worktree's own reflog.

Verify in a **fully scratch workspace** — its own `.winter/config.toml` and its own throwaway git repos, outside any
live workspace tree — with the working directory pinned to that scratch root for every invocation. Reach the in-progress
CLI either through `tool:winter-core-override` or by running the built virtualenv entry point directly; what matters is
the working directory, not which of the two you pick. `--winter=<path>` selects which CLI *code* runs and does not
itself create a sandbox: the workspace is resolved from the directory the command is invoked in, so the scratch working
directory is what keeps a destructive verb off live worktrees
(`workspace:/context/worktree-ops.md#verifying-destructive-commands-safely`). Give the scratch config deliberately
distinctive values (an unusual `service_prefix` and `base_port`) and confirm `ws status --json` reports
`workspace.root_path` as the scratch directory before running anything that mutates, so a misresolution is conspicuous
rather than silent. Declare two project repos, so every condition below is observed twice independently. Create the
chain's own env worktrees with `tool:winter-ws-init` (one per chain element the scenario needs, e.g. `env2` and `env3`)
before staging anything — restack reads each element's own worktree branch and never infers parentage otherwise. Stage
the rewrite locally — force-move one env's own branch to simulate the appended, amended, or squashed commit the command
exists to recover from — so no second remote is needed.

Preview before mutating anything: run the full chain with `--dry-run --json` first and read the printed plan — every
link in execution order, each participating repo's boundary sha and source — then re-run the identical command without
`--dry-run` for real and confirm the report matches.

**Reset before each scenario below.** Every scenario shares this one scratch workspace, its two project repos, and the
same env names — leftover state from one scenario poisons the next. Most visibly: the conflict scenario in Pass
deliberately leaves a repo mid-rebase, and a worktree left mid-rebase trips `refused-rebase-in-progress` on every other
scenario run after it, making the refusal-matrix condition ("each guard fires on its own scenario and not on the
others") impossible to observe. Before each scenario: record every branch tip, run `git rebase --abort` in any worktree
still mid-rebase, and force-move each branch back to its recorded tip.

Pass:

- A three-env chain (`env3 env2 base`) restacks base-ward first: the report and `--json`'s `links` render the
  `env2`-onto-`base` link before the `env3`-onto-`env2` link — the reverse of the argument order, the one part of
  base-ward execution a hand verifier can actually observe; the order links run in, not just the order they're listed
  in, isn't independently visible. After the run, `git merge-base --is-ancestor env2 env3` exits 0 in every
  participating repo — the restacked `env2` is an ancestor of `env3`, so the top env ends up reachable through the
  restacked middle env, which is what a base-ward run guarantees and a top-ward one wouldn't.
- A `--cut squashed-env` run at the bottom link drops the squashed env's own history. Before the run,
  `git merge-base --is-ancestor squashed-env env` exits 0 in the repos where the cut applies. After the run,
  `git merge-base --is-ancestor squashed-env env` exits 1 in those same repos — none of the squashed env's own commits
  remain in the restacked env's ancestry.
- Re-running the identical chain and the identical `--cut` after a link has already landed reports that link
  `up-to-date` in every repo where it already ran, and does not move it again.
- Each refusal guard fires on the one scenario built to trip it, and does not fire on the others: `refused-missing-ref`,
  `refused-dirty`, `refused-rebase-in-progress`, `refused-detached-head`, `refused-unknown-boundary`, and
  `refused-cut-not-ancestor`.
- `refused-inverted-order` has two independent triggers, each its own scenario — a chain that trips only one must still
  refuse:
  - **repeated element** — a chain element (including the `--cut` ref) repeats another chain element or `BASE`.
  - **ancestry inversion** — no repeated element, but a link is inverted at the chain level: the env carries commits
    past its frozen boundary in every participating repo, and its tip is a proper ancestor of the predecessor's pre-run
    tip in every one of them. Build this one without a repeated ref, or a regression in the ancestry check alone passes
    the method.
- When any participating repo would be refused, no repo in any link is mutated: record every branch tip before the run
  and confirm each is unchanged afterwards, in every repo of every link, and that the command exited 1.
- A conflict staged on a middle link stops the run there: that repo is left mid-rebase, no repo later in that same link
  is touched, and no link above it runs at all.

Post-run audit, once every link reports clean with no repo left mid-conflict:

- Every participating worktree's `HEAD` is attached to its own env branch — `git symbolic-ref -q HEAD` succeeds and
  names that branch, not a detached commit.
- No commit was silently dropped by the replay, or by a `--cut` that excluded too much — the failure mode a rewriting
  verb actually risks. A sha-presence check does not catch it: `git rebase --onto` abandons the *original* commits by
  design, so `git branch --contains <replayed-sha> --all` prints nothing for a healthy run and fails it, while the
  post-run sha sits on the env branch by construction and passes vacuously; the CLI also emits no sha at all on a clean
  run — `RestackConflict.replayed_commit` is populated only on a conflict, and only a conflict's report ever carries it.
  Instead, for every repo of every link that reported **`rebased`**: before the run, record its env branch's own commit
  subjects past that link's boundary — `git log --format=%s <boundary>..<env>` (the same boundary the `--dry-run --json`
  preview above already printed). After the run, `git log --format=%s <new-base>..<env>` in that same repo —
  `<new-base>` being the predecessor's post-run tip — must name the identical set of subjects.

  **Scope this comparison to `rebased` repos only — audit an `up-to-date` repo by branch-tip stability instead.** This
  is the second attempt at this audit condition: the first (comparing the same two ranges for every participating repo,
  unconditionally) cannot pass on a healthy run either. Two legitimate cases diverge without a dropped commit in sight:
  a `cut already done` repo does get a frozen boundary — the cut's own tip — but by definition that tip is *not* an
  ancestor of the env branch there, so the pre-run range lists strictly more subjects than the post-run range while the
  run correctly reports `up-to-date` and moves nothing; and an env that absorbed its predecessor via `winter ws merge`
  before the restack has a fork point well below the predecessor's tip, so the two ranges differ again with nothing
  moved. For an `up-to-date` repo, audit instead by recording its env branch's tip before the run and confirming it is
  unchanged after — that is the promise `up-to-date` actually makes.

**Gap**: no dedicated CI job runs this command against real git worktrees — exercised via the real-git regression tests
under `tools/winter-cli/tests/modules/workspace/test_env_restack_plan_service_real_git.py` and
`tools/winter-cli/tests/modules/workspace/test_env_restack_service_real_git.py`, plus manual verification in a throwaway
workspace.

### winter:manual — dashboard screens (headless tmux)

Surface: the real `winter dashboard` screens rendering a real workspace. The textual pilot tests under
`tools/winter-cli/tests/modules/tui/` render fixtures or a synthetic workspace; this exercise checks that a screen
renders live data. Run the dashboard detached on a private tmux socket, so no user session is touched. Drive it with
`send-keys` and read the rendered screen with `capture-pane`:

```bash
tmux -L winter-verify new-session -d -s dash -x 260 -y 90 "winter --winter=./alpha/winter dashboard"
tmux -L winter-verify send-keys -t dash M        # the screen's open key (M = Agent matrix)
tmux -L winter-verify capture-pane -p -t dash    # -e keeps the ANSI colors
tmux -L winter-verify kill-server
```

Run it from the workspace root through `tool:winter-core-override`. It is read-only as long as you press only a screen's
open key and `q`; the Agent matrix's `i` runs `winter ws init` and rewrites the live agent files, so don't press it. The
workspace screen only reads git status, and the Agent matrix screen reads config and the on-disk agent copies. A plugin
action key can shell out, so don't press one. The first refresh and each screen's background load take seconds, so poll
`capture-pane` until the expected text appears instead of sleeping for a fixed time. For the Agent matrix, check that:

- the three tables and the legend render;
- the `[stale]` / `[missing]` markers and the unmatched-override warning line agree with `winter agents --json`
  (`cli-probe:agents`);
- `q` returns to the workspace screen.

Always `kill-server` the private socket, even when a check fails. **Gap**: no CI job drives the real dashboard.

### winter:manual-tracing — OpenTelemetry tracing end to end (shim, export, propagation)

Surface: the opt-in tracing of `winter` commands (`workspace:/context/winter-cli/tracing.md`) through the real launcher
shim, a real OTLP/HTTP receiver, and real child processes. The unit tests cover the adapter against an in-process
receiver; this exercise covers what they cannot — the shim's `WINTER_LAUNCH_TIME`, a collector that the verifier can
read the span back from, and the process-level exit cost. Run everything from the workspace root, in one shell session
(bash 5 or newer, for `$EPOCHREALTIME`), by executing the alpha shim **file** directly, so the installed shim at
`~/.local/bin/winter` is never replaced. Every command it runs is non-mutating: `winter provision alpha --dry-run` and
the read-only `winter ws status alpha` through the shim, the dashboard on a private tmux socket with `q` as the only key
pressed, and every provision and service step inside a scratch workspace.

Setup:

```bash
SHIM=./alpha/winter/tools/winter-cli/bin/winter
SCRATCH=$(mktemp -d)

# A known caller context, and a fresh trace id for each run so one trace holds exactly one span.
unset OTEL_SERVICE_NAME OTEL_RESOURCE_ATTRIBUTES
fresh_parent() {
  TRACE_ID=$(python3 -c 'import secrets; print(secrets.token_hex(16))')
  CALLER_SPAN=00f067aa0ba902b7
  export TRACEPARENT=00-$TRACE_ID-$CALLER_SPAN-01
}

# Host ports: Jaeger's OTLP/HTTP receiver, its query API and UI, and the hanging endpoint. A local collector often
# already holds 4318; when the check below names a port in use, set a free one here and rerun the setup from this line.
OTLP_PORT=4318 QUERY_PORT=16686 HANG_PORT=4319
RESPONSIVE=http://127.0.0.1:$OTLP_PORT HANGING=http://127.0.0.1:$HANG_PORT
export JAEGER=http://127.0.0.1:$QUERY_PORT   # the read-back helpers below query Jaeger here
port_free() { python3 -c 'import socket, sys; sys.exit(socket.socket().connect_ex(("127.0.0.1", int(sys.argv[1]))) == 0)' "$1" \
  || { echo "SETUP FAILED: port $1 is in use" >&2; return 1; }; }

# A local Jaeger. Tracing is switched on only once this Jaeger answers, so no span reaches any other collector.
if port_free "$OTLP_PORT" && port_free "$QUERY_PORT" && port_free "$HANG_PORT" \
  && docker run --rm -d --name winter-jaeger -p "127.0.0.1:$QUERY_PORT:16686" -p "127.0.0.1:$OTLP_PORT:4318" \
    jaegertracing/jaeger:latest; then
  for _ in $(seq 60); do curl -sf "$JAEGER/api/v3/services" >/dev/null && break; sleep 1; done
  curl -sf "$JAEGER/api/v3/services" >/dev/null && export WINTER_OTEL_EXPORTER_OTLP_ENDPOINT=$RESPONSIVE \
    || echo "SETUP FAILED: Jaeger is not answering on $JAEGER" >&2
fi

# A hanging endpoint: a localhost socket that accepts and never answers.
python3 -c 'import socket, sys
s = socket.socket()
s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
s.bind(("127.0.0.1", int(sys.argv[1])))
s.listen(64)
held = []
while True:
    held.append(s.accept()[0])' "$HANG_PORT" &
HANG_PID=$!

# Read one span back from Jaeger: name, parent span id, service.name, start time in microseconds.
cat > "$SCRATCH/span.py" <<'PY'
import json, os, sys, time, urllib.request
url = f"{os.environ['JAEGER']}/api/traces/{sys.argv[1]}"
for _ in range(40):
    try:
        trace = json.load(urllib.request.urlopen(url))["data"][0]
        break
    except Exception:
        time.sleep(0.5)
else:
    sys.exit(f"no trace {sys.argv[1]} in Jaeger")
(span,) = trace["spans"]
print(json.dumps({
    "name": span["operationName"],
    "parent": [r["spanID"] for r in span["references"] if r["refType"] == "CHILD_OF"],
    "service": trace["processes"][span["processID"]]["serviceName"],
    "start_us": span["startTime"],
}))
PY

# Read a whole trace back: one entry per span, with its parent span id and its tags as a dict.
cat > "$SCRATCH/spans.py" <<'PY'
import json, os, sys, time, urllib.request
url = f"{os.environ['JAEGER']}/api/traces/{sys.argv[1]}"
spans, stable = [], 0
for _ in range(40):                      # the trace is whole once its span count stops growing
    try:
        trace = json.load(urllib.request.urlopen(url))["data"][0]
    except Exception:
        time.sleep(0.5)
        continue
    stable = stable + 1 if len(trace["spans"]) == len(spans) else 0
    spans = trace["spans"]
    if stable == 2:
        break
    time.sleep(0.5)
else:
    sys.exit(f"no stable trace {sys.argv[1]} in Jaeger")
print(json.dumps([{
    "id": s["spanID"],
    "name": s["operationName"],
    "parent": next((r["spanID"] for r in s["references"] if r["refType"] == "CHILD_OF"), None),
    "service": trace["processes"][s["processID"]]["serviceName"],
    "start_us": s["startTime"],
    "tags": {t["key"]: t["value"] for t in s["tags"]},
} for s in spans]))
PY

# Read a dashboard session back by its unique service name: the `dashboard <purpose>` roots, each with its
# trace id, parent, span links and the number of `git status` spans in its trace, and the session span.
# Argument 2 is "roots" (poll up to 12 s for a root holding git status spans) or "session" (poll up to 20 s
# for the session span).
cat > "$SCRATCH/dash.py" <<'PY'
import datetime, json, os, sys, time, urllib.parse, urllib.request
service, want = sys.argv[1], sys.argv[2]

def read():
    now = datetime.datetime.now(datetime.timezone.utc)
    query = urllib.parse.urlencode({
        "query.service_name": service,
        "query.start_time_min": (now - datetime.timedelta(hours=1)).strftime("%Y-%m-%dT%H:%M:%SZ"),
        "query.start_time_max": (now + datetime.timedelta(hours=1)).strftime("%Y-%m-%dT%H:%M:%SZ"),
    })
    spans = []
    for line in urllib.request.urlopen(f"{os.environ['JAEGER']}/api/v3/traces?{query}").read().splitlines():
        for rs in json.loads(line)["result"]["resourceSpans"]:
            for ss in rs["scopeSpans"]:
                spans += ss["spans"]
    return spans

def summarize(spans):
    return {
        "roots": [{
            "name": r["name"],
            "trace": r["traceId"],
            "parent": r.get("parentSpanId") or None,
            "link": [(l["traceId"], l["spanId"]) for l in r.get("links", [])],
            "git_status": sum(1 for s in spans if s["name"] == "git status" and s["traceId"] == r["traceId"]),
        } for r in spans if r["name"].startswith("dashboard ")],
        "session": [{
            "trace": s["traceId"],
            "id": s["spanId"],
            "parent": s.get("parentSpanId") or None,
        } for s in spans if s["name"] == "winter dashboard"],
    }

for _ in range(24 if want == "roots" else 40):
    try:
        out = summarize(read())
    except Exception:
        out = {"roots": [], "session": []}
    if any(r["git_status"] for r in out["roots"]) if want == "roots" else out["session"]:
        break
    time.sleep(0.5)
print(json.dumps(out))
PY

# The assertions of the inner-span and dashboard checks; each prints `ok: ...` or fails with the broken expectation.
cat > "$SCRATCH/check.py" <<'PY'
import json, pathlib, sys, tomllib
CALLER = "00f067aa0ba902b7"

def one(spans, name):
    (span,) = [s for s in spans if s["name"] == name]
    return span

def logged_span_id(lines, prefix):       # the span id in the TRACEPARENT of the one log line that starts with prefix
    (line,) = [l for l in lines if l.startswith(prefix)]
    return line.split("TRACEPARENT=")[1].split("-")[2]

def status(trace, output, env="alpha"):
    spans, lines = json.load(open(trace)), open(output).read().splitlines()
    shown, in_table = set(), False       # the repos the env table lists: first column, up to the first blank line
    for line in lines:
        if not in_table:
            in_table = line.startswith("REPO")
        elif not line.strip():
            break
        else:
            shown.add(line.split()[0])
    on_disk = {p.name for p in pathlib.Path(env).iterdir() if (p / ".git").exists()}
    assert shown == on_disk, f"output lists {sorted(shown)}, worktrees on disk {sorted(on_disk)}"
    root = one(spans, "winter ws status")
    assert root["parent"] == CALLER, root["parent"]
    git = {s["id"]: s for s in spans if s["name"].startswith("git ")}
    assert git, "no git spans"
    for s in git.values():
        assert s["parent"] == root["id"] or s["parent"] in git, f"{s['name']} is parented on {s['parent']}"
        assert "winter.repo" in s["tags"], f"{s['name']} has no winter.repo"
    reads = [s for s in git.values() if s["name"] == "git status"]
    have = {s["tags"]["winter.repo"] for s in reads if s["tags"].get("winter.env") == env}
    assert have == on_disk, f"no git status span with winter.env={env} for {sorted(on_disk - have)}"
    assert any("winter.env" not in s["tags"] for s in reads), "no git status span without an env (checkouts, standalones)"
    others = set(tomllib.load(open(".winter/state.toml", "rb"))["env_index"]) - {env}
    seen = {s["tags"]["winter.env"] for s in git.values() if "winter.env" in s["tags"]} - {env}
    assert not others or seen, f"no git span for the other envs {sorted(others)}"
    print(f"ok: {len(git)} git spans; {len(have)} repos read in {env}; other envs seen {sorted(seen)}")

def service(trace, provider_log):
    spans, lines = json.load(open(trace)), open(provider_log).read().splitlines()
    up, down = one(spans, "winter service up"), one(spans, "winter service down")
    assert up["parent"] == down["parent"] == CALLER
    cells = [s for s in spans if s["name"] == "service provider up"]
    assert {s["tags"]["winter.scope"] for s in cells} == {"workspace", "alpha"}
    assert all(s["parent"] == up["id"] for s in cells)
    wait = one(spans, "service readiness wait")
    assert wait["parent"] == up["id"]
    assert wait["tags"]["winter.service.patterns"] == 1 and wait["tags"]["winter.ready"] is True
    cell_down = one(spans, "service provider down")
    assert cell_down["parent"] == down["id"] and cell_down["tags"]["winter.scope"] == "alpha"
    providers = {s["tags"].get("winter.provider") for s in cells + [cell_down]}
    assert len(providers) == 1 and not providers & {None, ""}, providers
    assert logged_span_id(lines, "down alpha") == cell_down["id"], "down's TRACEPARENT is not the provider down span"
    assert logged_span_id(lines, "status") == wait["id"], "the readiness poll's TRACEPARENT is not the wait span"
    assert all("TRACEPARENT=<unset>" in l for l in lines if l.startswith("up "))
    print(f"ok: provider {providers.pop()}")

def provision(trace, output, handler_log, trace_id):
    spans, text, logged = json.load(open(trace)), open(output).read(), open(handler_log).read().split()
    root, handler = one(spans, "winter provision"), one(spans, "provision handler apply")
    assert handler["parent"] == root["id"]
    tags = handler["tags"]
    assert tags["winter.env"] == "alpha" and tags["winter.exit_code"] == 0 and "winter.repo" not in tags
    assert tags["winter.handler"] and tags["winter.handler"] in text, "winter.handler is not the label the output shows"
    assert [l.split("-")[1:3] for l in logged] == [[trace_id, handler["id"]]], logged
    print(f"ok: {tags['winter.handler']}")

def dashboard(live, end, trace_id):
    live, end = json.load(open(live)), json.load(open(end))
    assert any(r["git_status"] for r in live["roots"]), "no root holding git status spans before quitting"
    assert not live["session"], "the session span was exported before quitting"
    (session,) = end["session"]
    assert session["trace"] == trace_id and session["parent"] == CALLER, session
    assert "dashboard refresh workspace" in {r["name"] for r in end["roots"]}
    for r in end["roots"]:
        assert r["parent"] is None and r["trace"] != trace_id, r
        assert r["link"] == [[trace_id, session["id"]]], r
    print(f"ok: {len(end['roots'])} roots, all linked to the session span")

{"status": status, "service": service, "provision": provision, "dashboard": dashboard}[sys.argv[1]](*sys.argv[2:])
PY
```

Teardown, always, even when a check fails:

```bash
tmux -L winter-trace kill-server 2>/dev/null
kill "$HANG_PID"; docker rm -f winter-jaeger; rm -rf "$SCRATCH"
unset WINTER_OTEL_EXPORTER_OTLP_ENDPOINT TRACEPARENT PROVIDER_LOG HANDLER_LOG JAEGER   # later commands run untraced
```

Checks:

- **Span content.** Run the command, then read the span from the trace JSON Jaeger serves at `/api/traces/<trace_id>`
  (the helper fetches it):

  ```bash
  fresh_parent
  "$SHIM" --winter=./alpha/winter provision alpha --dry-run; echo "exit $?"
  python3 "$SCRATCH/span.py" "$TRACE_ID"
  ```

  Pass: the command prints `exit 0`; `name` is `winter provision`, `parent` is `["00f067aa0ba902b7"]` (the caller's span
  id), `service` is `winter`. The helper fails unless the trace holds exactly one span. Open the `$JAEGER` URL to see
  the same trace in the UI.

- **Start time.** Run the shim under `bash -x`; the trace prints the exported value:

  ```bash
  fresh_parent
  bash -x "$SHIM" --winter=./alpha/winter provision alpha --dry-run >/dev/null 2>"$SCRATCH/xtrace.txt"
  launch=$(sed -n 's/^+ export WINTER_LAUNCH_TIME=//p' "$SCRATCH/xtrace.txt")
  echo "launch=${launch/[.,]/}"
  python3 "$SCRATCH/span.py" "$TRACE_ID"
  ```

  Pass: `start_us` equals `launch` with its decimal separator removed (both are microseconds since the epoch).

- **Exit cost.** The same command against two endpoints — the responsive Jaeger receiver and the hanging socket — so the
  import and setup path is identical and the difference is the export cost. The cap is 100 ms, and the pass threshold
  adds 50 ms of measurement margin:

  ```bash
  timed_run() {   # prints wall ms; fails on a non-zero exit or any stderr output
    local s e
    s=$EPOCHREALTIME
    WINTER_OTEL_EXPORTER_OTLP_ENDPOINT=$1 "$SHIM" --winter=./alpha/winter provision alpha --dry-run \
      >/dev/null 2>"$SCRATCH/stderr.txt" || { echo "exit code $?" >&2; return 1; }
    e=$EPOCHREALTIME
    [[ ! -s "$SCRATCH/stderr.txt" ]] || { echo "stderr not empty" >&2; return 1; }
    echo $(( (${e/[.,]/} - ${s/[.,]/}) / 1000 ))
  }
  samples() { for _ in $(seq 11); do timed_run "$1" || return 1; done; }

  timed_run "$RESPONSIVE" >/dev/null     # warm-up: the first run may build the venv
  samples "$RESPONSIVE" > "$SCRATCH/responsive.txt" && samples "$HANGING" > "$SCRATCH/hanging.txt" \
    || echo "FAIL: a run exited non-zero or wrote to stderr"
  responsive=$(sort -n "$SCRATCH/responsive.txt" | sed -n 6p)
  hanging=$(sort -n "$SCRATCH/hanging.txt" | sed -n 6p)
  echo "exit cost: $((hanging - responsive)) ms (pass threshold 150)"
  ```

  Pass: all 22 runs exit 0 with empty stderr, and the hanging median minus the responsive median is at most 150 ms (the
  100 ms cap plus 50 ms).

- **Shim without `EPOCHREALTIME`.** A stub `mise` first on `PATH` prints its environment and exits, so the shim runs to
  its `exec` and nothing else. Run it under the host bash, then under bash 4.4, which has no `EPOCHREALTIME`:

  ```bash
  mkdir -p "$SCRATCH/nm/ws/.winter" "$SCRATCH/nm/ws/tools/winter-cli" "$SCRATCH/nm/stub"
  touch "$SCRATCH/nm/ws/.winter/config.toml"
  printf '#!/bin/sh\nenv\n' > "$SCRATCH/nm/stub/mise" && chmod +x "$SCRATCH/nm/stub/mise"
  SHIM_ABS=$(realpath "$SHIM")

  # host bash (5 or newer)
  (cd "$SCRATCH/nm/ws" && env -u WINTER_LAUNCH_TIME PATH="$SCRATCH/nm/stub:$PATH" bash "$SHIM_ABS" ws status) \
    >"$SCRATCH/host.out" 2>"$SCRATCH/host.err"; echo "host exit $?"
  grep -E '^(WINTER_LAUNCH_TIME|WINTER_INVOCATION_CWD)=' "$SCRATCH/host.out"; cat "$SCRATCH/host.err"

  # bash 4.4: mount the shim and the minimal workspace
  docker run --rm bash:4.4 bash -c 'echo "EPOCHREALTIME=${EPOCHREALTIME:-unset}"'     # unset
  docker run --rm -v "$SCRATCH/nm:/scr" -v "$SHIM_ABS:/scr/shim:ro" -w /scr/ws \
    -e PATH=/scr/stub:/usr/local/bin:/usr/bin:/bin bash:4.4 bash /scr/shim ws status \
    >"$SCRATCH/old.out" 2>"$SCRATCH/old.err"; echo "bash 4.4 exit $?"
  grep -E '^(WINTER_LAUNCH_TIME|WINTER_INVOCATION_CWD)=' "$SCRATCH/old.out"; cat "$SCRATCH/old.err"
  ```

  Pass: `host.err` and `old.err` are both empty. The host run exits 0 and prints `WINTER_LAUNCH_TIME=<seconds>.<micros>`
  (or with a `,`). The bash 4.4 run exits 0 and prints `WINTER_INVOCATION_CWD=` (the shim reached the stub) with no
  `WINTER_LAUNCH_TIME=` line and no `unbound variable` message.

- **Version-1 fallback.** Extract the version-1 shim from the parent of the commit that introduced version 2, and run it
  the same way as the version-2 shim. It exports no launch time, so its span starts at command dispatch, after the
  launcher gap. Measure each span's start offset from the instant recorded just before the shim is spawned:

  ```bash
  V2=$(git -C alpha/winter log --format=%H -S'WINTER_SHIM_VERSION=2' -- tools/winter-cli/bin/winter | tail -1)
  git -C alpha/winter show "$V2^:tools/winter-cli/bin/winter" > "$SCRATCH/shim-v1" && chmod +x "$SCRATCH/shim-v1"
  grep '^WINTER_SHIM_VERSION=' "$SCRATCH/shim-v1" "$SHIM"      # 1 and 2

  start_offset_ms() {   # span start minus the instant before spawning, in ms
    fresh_parent
    local before=$EPOCHREALTIME start_us
    "$1" --winter=./alpha/winter provision alpha --dry-run >/dev/null || return 1
    start_us=$(python3 "$SCRATCH/span.py" "$TRACE_ID" | python3 -c 'import json, sys; print(json.load(sys.stdin)["start_us"])')
    echo $(( (start_us - ${before/[.,]/}) / 1000 ))
  }
  : > "$SCRATCH/v1.txt"; : > "$SCRATCH/v2.txt"
  for _ in 1 2 3 4 5; do
    start_offset_ms "$SHIM" >> "$SCRATCH/v2.txt"
    start_offset_ms "$SCRATCH/shim-v1" >> "$SCRATCH/v1.txt"
  done
  v2=$(sort -n "$SCRATCH/v2.txt" | sed -n 3p); v1=$(sort -n "$SCRATCH/v1.txt" | sed -n 3p)
  echo "v1 offset $v1 ms, v2 offset $v2 ms, difference $((v1 - v2)) ms (need at least 100)"
  ```

  Pass: the version-1 median offset exceeds the version-2 median offset by at least 100 ms. The threshold sits
  deliberately below the launcher gap `workspace:/context/winter-cli/tracing.md` documents (a few hundred milliseconds),
  so a version-1 shim cannot pass by noise alone.

- **Service launches carry no trace context.** A scratch workspace whose scratch provider records the `TRACEPARENT` it
  was handed shows that `service up` and `service restart` pass none, whether tracing is on or off. The provider writes
  to a log file, because its stdout is winter's wire format, and answers `status` with a one-service healthy document
  for the readiness wait in the service-span check below. The CLI runs through `uv run --project` from inside the
  scratch workspace, not through the shim: the shim re-roots Python in the checkout's own `tools/winter-cli/`
  (`mise -C`), so the CLI would resolve the live workspace and its registry instead of the scratch one. The scratch
  `state.toml` registers env `alpha`; without it the registry is empty, no `alpha` cell matches, and the provider is
  never called for `alpha`:

  ```bash
  W=$(realpath alpha/winter)
  mkdir -p "$SCRATCH/ws/.winter" "$SCRATCH/ws/alpha" "$SCRATCH/prov"
  touch "$SCRATCH/ws/.winter/config.toml"
  printf '[env_index]\nalpha = 1\n' > "$SCRATCH/ws/.winter/state.toml"
  printf 'name = "scratch-provider"\nprefix = "scr"\nprovides.service = "orchestrate"\n' > "$SCRATCH/prov/winter-ext.toml"
  printf '{"envs": [{"env": "alpha", "session": null, "port_base": null, "services": [{"name": "api", "state": "running", "health": "healthy", "ports": [], "handle": null, "log_path": null, "since": null}]}]}\n' > "$SCRATCH/prov/status.json"
  printf '#!/usr/bin/env bash\necho "$* TRACEPARENT=${TRACEPARENT-<unset>}" >> "${PROVIDER_LOG:?}"\n[[ $1 == status ]] && cat "$(dirname "$0")/status.json"\nexit 0\n' > "$SCRATCH/prov/orchestrate"
  chmod +x "$SCRATCH/prov/orchestrate"

  export PROVIDER_LOG=$SCRATCH/provider.log; : > "$PROVIDER_LOG"
  fresh_parent
  for endpoint in "" "$RESPONSIVE"; do          # tracing off, then on
    echo "== tracing $([[ -n $endpoint ]] && echo on || echo off)" >> "$PROVIDER_LOG"
    for action in up restart down; do
      (cd "$SCRATCH/ws" && WINTER_OTEL_EXPORTER_OTLP_ENDPOINT=$endpoint uv run --project "$W/tools/winter-cli" winter \
        --service-orchestrator="$SCRATCH/prov" service "$action" alpha) >/dev/null 2>&1
    done
  done
  cat "$PROVIDER_LOG"
  echo "caller: $TRACEPARENT"
  ```

  Pass: the log holds, in order, these lines per mode (`<caller>` is the printed `caller:` value):

  | Mode        | `up workspace`            | `up alpha`                | `restart alpha`           | `down alpha`                                                      |
  | ----------- | ------------------------- | ------------------------- | ------------------------- | ----------------------------------------------------------------- |
  | tracing off | `... TRACEPARENT=<unset>` | `... TRACEPARENT=<unset>` | `... TRACEPARENT=<unset>` | `... TRACEPARENT=<caller>`, unchanged                             |
  | tracing on  | `... TRACEPARENT=<unset>` | `... TRACEPARENT=<unset>` | `... TRACEPARENT=<unset>` | `... TRACEPARENT=` the caller's trace id with a different span id |

  The `up` and `restart` lines must read `<unset>` in both modes. The `down` line is the control, since `down` follows
  winter's own environment: a `down` line equal to the caller's value with tracing on, or missing, fails the check. An
  empty log, a log without the `up alpha`, `restart alpha`, and `down alpha` lines for a mode, or a mode marker with no
  lines under it is a setup failure (the scratch registry or provider did not take effect), not a pass.

- **Inner spans: `ws status`.** The one command here that runs through the alpha shim against the live workspace,
  because `ws status` only reads. It exits 0 or 1 (1 reports a dirty worktree), so only a code above 1 is a failure:

  ```bash
  fresh_parent
  "$SHIM" --winter=./alpha/winter ws status alpha > "$SCRATCH/status.out" 2>/dev/null; echo "exit $?"
  python3 "$SCRATCH/spans.py" "$TRACE_ID" > "$SCRATCH/trace.json"
  python3 "$SCRATCH/check.py" status "$SCRATCH/trace.json" "$SCRATCH/status.out"
  ```

  Pass: the exit code is 0 or 1 and `check.py` prints `ok`, which means all of the following:
  - The trace holds one `winter ws status` root, parented on the caller's span.
  - Every `git` span carries `winter.repo` and is parented on the root or on another `git` span, so the thread pools
    keep the context.
  - Every worktree on disk under `alpha/` has a row in the env table of the command's output, and a `git status` span
    with that `winter.repo` and `winter.env` equal to `alpha`. The output check matters because a git span opens even
    when the worktree path is missing, so the span alone does not prove git ran.
  - `git status` spans without a `winter.env` appear (the main-branch checkouts and standalones), and so do spans
    carrying each other env that `.winter/state.toml` registers, because status reads them before filtering.

- **Service spans.** Reuses the scratch workspace and scratch provider of the previous check, from inside the scratch
  workspace with `uv run --project`, never through the shim. One trace holds both commands:

  ```bash
  scratch_winter() { (cd "$SCRATCH/ws" && uv run --project "$W/tools/winter-cli" winter --service-orchestrator="$SCRATCH/prov" "$@"); }
  export PROVIDER_LOG=$SCRATCH/provider.log; : > "$PROVIDER_LOG"
  fresh_parent
  scratch_winter service up alpha --wait >/dev/null 2>&1; echo "up exit $?"
  scratch_winter service down alpha >/dev/null 2>&1; echo "down exit $?"
  python3 "$SCRATCH/spans.py" "$TRACE_ID" > "$SCRATCH/trace.json"
  python3 "$SCRATCH/check.py" service "$SCRATCH/trace.json" "$PROVIDER_LOG"
  ```

  Pass: both commands exit 0 and `check.py` prints `ok`, which means all of the following:
  - Under the `winter service up` root: two `service provider up` spans, one for the `workspace` scope (the implicit
    cell `up` dispatches first) and one for `alpha`, and a `service readiness wait` span with `winter.service.patterns`
    1 and `winter.ready` true.
  - Under the `winter service down` root: a `service provider down` span for `alpha`.
  - Every provider span carries the same non-empty `winter.provider`.
  - The span id in the `TRACEPARENT` that the provider logged for `down` equals the `service provider down` span's id,
    and the one the `status` poll logged equals the `service readiness wait` span's id. The `up` lines still read
    `<unset>`.

- **Provision spans.** Give the scratch workspace's own config a handler whose `apply` logs the `TRACEPARENT` it
  receives, then run `provision alpha` from inside the scratch workspace with `uv run --project`, never through the
  shim. The handler is `feature-environment` scoped, so it runs in `$SCRATCH/ws/alpha` and nothing outside the scratch
  workspace is touched:

  ```bash
  printf 'echo "$TRACEPARENT" >> "$HANDLER_LOG"\n' > "$SCRATCH/handler.sh"
  printf '[[provision.dependency]]\nscope = "feature-environment"\napply = "sh %s/handler.sh"\n' "$SCRATCH" > "$SCRATCH/ws/.winter/config.toml"
  export HANDLER_LOG=$SCRATCH/handler.log; : > "$HANDLER_LOG"
  fresh_parent
  (cd "$SCRATCH/ws" && uv run --project "$W/tools/winter-cli" winter provision alpha) > "$SCRATCH/provision.out" 2>&1; echo "exit $?"
  python3 "$SCRATCH/spans.py" "$TRACE_ID" > "$SCRATCH/trace.json"
  python3 "$SCRATCH/check.py" provision "$SCRATCH/trace.json" "$SCRATCH/provision.out" "$HANDLER_LOG" "$TRACE_ID"
  ```

  Pass: the command exits 0 and `check.py` prints `ok`, which means: a `provision handler apply` span under the
  `winter provision` root, carrying `winter.env` `alpha`, `winter.exit_code` 0 and a `winter.handler` equal to the label
  the provision output shows, and no `winter.repo` (the handler's directory is the env root, not a project worktree).
  The span id in the `TRACEPARENT` that the handler logged equals the span's id.

- **Exit cost with inner spans.** The hanging endpoint against a baseline of the same `ws status alpha` run in an
  unsampled caller context (sampled flag `00`). The baseline does the same git work and loads the same tracing setup but
  records no span and sends no request, so the hanging run's extra cost is recording and encoding every span plus the
  exporter cap. Do not take the baseline from the same command run against the responsive receiver: both runs encode the
  spans, so the encoding cost cancels out of the difference and the check cannot see it. Do not take it from a command
  that emits only the root span either, such as `provision alpha --dry-run`: its own work is a fraction of
  `ws status`'s, so the difference measures the commands, not the export. The cap is 100 ms, and the pass threshold adds
  50 ms of measurement margin:

  ```bash
  timed_status() {   # $1 endpoint, $2 sampled flag; prints wall ms; fails on an exit code above 1 or any stderr output
    local s e rc
    s=$EPOCHREALTIME
    TRACEPARENT=00-$TRACE_ID-$CALLER_SPAN-$2 WINTER_OTEL_EXPORTER_OTLP_ENDPOINT=$1 \
      "$SHIM" --winter=./alpha/winter ws status alpha >/dev/null 2>"$SCRATCH/stderr.txt"; rc=$?
    e=$EPOCHREALTIME
    (( rc <= 1 )) || { echo "exit code $rc" >&2; return 1; }
    [[ ! -s "$SCRATCH/stderr.txt" ]] || { echo "stderr not empty" >&2; return 1; }
    echo $(( (${e/[.,]/} - ${s/[.,]/}) / 1000 ))
  }

  fresh_parent
  timed_status "$RESPONSIVE" 01 >/dev/null 2>&1     # warm-up: the first run may rebuild the venv
  : > "$SCRATCH/baseline.txt"; : > "$SCRATCH/hanging-inner.txt"
  for _ in $(seq 11); do
    timed_status "$RESPONSIVE" 00 >> "$SCRATCH/baseline.txt" && timed_status "$HANGING" 01 >> "$SCRATCH/hanging-inner.txt" \
      || { echo "FAIL: a run exited above 1 or wrote to stderr"; break; }
  done
  baseline=$(sort -n "$SCRATCH/baseline.txt" | sed -n 6p)
  hanging=$(sort -n "$SCRATCH/hanging-inner.txt" | sed -n 6p)
  echo "exit cost with inner spans: $((hanging - baseline)) ms (pass threshold 150)"
  ```

  Pass: all 22 runs exit 0 or 1 with empty stderr, and the hanging median minus the baseline median is at most 150 ms
  (the 100 ms cap plus 50 ms). The ordinary-size claim in `workspace:/context/winter-cli/tracing.md` covers this
  command's roughly 100 spans.

- **Dashboard.** Start `winter dashboard` headless on a private tmux socket, as in
  `winter:manual — dashboard screens (headless tmux)`, with tracing on, and press `q` and no other key, so the live
  workspace is only read. Each run gets its own `service.name`, so the reader finds that run's traces without knowing
  their ids (the session roots are traces of their own), and a fresh caller context, so the session span lands in a
  trace whose id is known. A run is `dash_start` (the dashboard is up and has drawn its first screen), then reads, then
  `quit_ms`:

  ```bash
  dash_start() {   # $1 endpoint; sets DASH_SVC and a fresh TRACE_ID, and starts the dashboard
    DASH_SVC=winter-dash-$RANDOM$RANDOM
    fresh_parent
    tmux -L winter-trace new-session -d -s dash -x 260 -y 90 \
      "env OTEL_SERVICE_NAME=$DASH_SVC WINTER_OTEL_EXPORTER_OTLP_ENDPOINT=$1 $SHIM --winter=./alpha/winter dashboard"
    for _ in $(seq 60); do
      tmux -L winter-trace capture-pane -p -t dash 2>/dev/null | grep -q 'Winter Dashboard' && return 0
      sleep 0.5
    done
    echo "dashboard did not draw" >&2; return 1
  }
  quit_ms() {   # presses q; prints ms until the dashboard process has exited and the tmux session with it
    local s
    s=${EPOCHREALTIME/[.,]/}
    tmux -L winter-trace send-keys -t dash q
    while tmux -L winter-trace has-session -t dash 2>/dev/null; do
      (( ${EPOCHREALTIME/[.,]/} - s < 10000000 )) || { echo "dashboard did not exit" >&2; return 1; }
    done
    echo $(( (${EPOCHREALTIME/[.,]/} - s) / 1000 ))
  }

  dash_start "$WINTER_OTEL_EXPORTER_OTLP_ENDPOINT"
  python3 "$SCRATCH/dash.py" "$DASH_SVC" roots > "$SCRATCH/dash-live.json"     # before quitting
  quit_ms
  python3 "$SCRATCH/dash.py" "$DASH_SVC" session > "$SCRATCH/dash-end.json"     # after quitting
  tmux -L winter-trace kill-server 2>/dev/null
  python3 "$SCRATCH/check.py" dashboard "$SCRATCH/dash-live.json" "$SCRATCH/dash-end.json" "$TRACE_ID"
  ```

  Pass: `check.py` prints `ok`, which means all of the following:
  - Within about 12 s of starting, and before quitting, the receiver holds a `dashboard <purpose>` trace that contains
    `git status` spans. The session span is not there yet: it is sent only at exit.
  - After `q`, the `winter dashboard` session span appears in the run's own trace, parented on the caller's span.
  - Every `dashboard <purpose>` root, `dashboard refresh workspace` among them, has no parent, lives in a trace of its
    own, and carries exactly one span link, to the session span.

  Quit cost: the time from `send-keys q` until the dashboard's process has exited. Take 5 runs each against the
  responsive receiver and the hanging endpoint, after the dashboard has run past one 5 s flush interval, so a flush has
  happened before the quit:

  ```bash
  : > "$SCRATCH/quit-responsive.txt"; : > "$SCRATCH/quit-hanging.txt"
  for _ in 1 2 3 4 5; do
    dash_start "$RESPONSIVE" && sleep 6 && quit_ms >> "$SCRATCH/quit-responsive.txt"
    tmux -L winter-trace kill-server 2>/dev/null
    dash_start "$HANGING" && sleep 6 && quit_ms >> "$SCRATCH/quit-hanging.txt"
    tmux -L winter-trace kill-server 2>/dev/null
  done
  wc -l "$SCRATCH"/quit-*.txt                                                 # 5 lines each, or a run failed
  responsive=$(sort -n "$SCRATCH/quit-responsive.txt" | sed -n 3p)
  hanging=$(sort -n "$SCRATCH/quit-hanging.txt" | sed -n 3p)
  echo "quit cost: $((hanging - responsive)) ms (pass threshold 150)"
  ```

  Pass: five lines in each file, and the hanging median exceeds the responsive median by at most 150 ms (the 100 ms cap
  plus 50 ms). Always `kill-server` the private socket, even when a check fails.

**Gap**: no CI job runs the shim, a live collector, a hanging endpoint, or the dashboard under tracing — the unit tests
drive the adapter against an in-process receiver, and this method covers the rest by hand.

### winter-test-service:manual — full-stack app exercise

Stand up `winter-test-service` (web + api + worker + Postgres + RabbitMQ) under a feature env to exercise orchestration,
provisioning, port-band isolation, and log capture against a real multi-service application rather than a single
command. Drive its built-in diagnostic controls — induced API crash, slow boot, error output — to put services into
known failure and health states and confirm the orchestrator observes and reports them. See `tool:winter-test-service`
below for the app and its controls.

## Tools

Setup an agent uses to stand up the scenario a verification needs — not assertions of correctness themselves.

| Tool                               | Use                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| tool:winter-ws-init                | `winter ws init <env>` / `winter ws destroy <env>` — create or remove a throwaway feature environment to verify against.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| tool:winter-provision              | `winter provision <env>` runs dependency → resource → data to bring an env to a working state. Put state in a known shape: `winter provision <env> --stage data` (wipe-and-reload baseline), `winter provision <env> --stage resource --reset` (destroy + recreate databases / queues / buckets), `winter provision <env> --stage resource --seed` (resources then data). Handlers are idempotent.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| tool:winter-core-override          | `winter --winter=<path> …` runs the CLI from `<path>/tools/winter-cli` against **the current, live workspace** — not a sandbox; every worktree it touches is a real one in `alpha/`, `beta/`, etc. Read-only probes (`ws status`, `doctor`, `graph`, `--dry-run`/`--json` forms) are safe to point at the live workspace this way. A **destructive** verb (`ws checkout`, `ws reset --hard`, `ws clean`, `ws destroy`, `ws restack`, `winter clean`) run through this override still mutates the live worktree or env it targets — none of them has a repo-scoping flag beyond `PATTERNS`/`ENV`, so a wrong or missing pattern can hit worktrees or envs you didn't mean to touch. Route destructive-verb probes — including `winter clean` — at a throwaway env (`tool:winter-ws-init` builds one) or a fully scratch workspace (its own config + throwaway git repos), never at an env holding work you haven't pushed. `winter clean`'s only gate is `--dry-run`/`--json`: no prompt, no `--force`, no refusal guard. Which of the other verbs prompt, and under what pattern/count conditions, is per-verb and already drifts if restated here — see `workspace:/context/worktree-ops.md#verifying-destructive-commands-safely` for the gate each one actually has. |
| tool:service-orchestrator-override | `winter --service-orchestrator=<path-or-name> service …` redirects `service` dispatch at a worktree provider or a registered name. Combine with `--winter` when the change touches the status path (env enumeration lives in core): `winter --winter=./alpha/winter --service-orchestrator=./alpha/winter-service-docker service status alpha`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| tool:direct-entrypoint             | The override redirects the `winter service …` door but not every door. **tmux env-root door — Gap ([winter-service-tmux#26](https://github.com/paul-gross/winter-service-tmux/issues/26)):** the env-root `./up` / `./down` / `./status` symlinks ignore `WINTER_EXT_DIR`, so to run worktree code through them you must repoint the symlink at the worktree copy, run it, then **restore the symlink** — mandatory; a leftover override silently routes every later call in that env through worktree code. Once #26 lands, an exported `WINTER_EXT_DIR` covers this door too and the repoint goes away. **docker (no env-root door):** to invoke the entrypoint without the CLI, export `WINTER_WORKSPACE_DIR` / `WINTER_EXT_DIR` / `WINTER_EXT_CONFIG_DIR` and run `workflow/service <action>` under `PYTHONPATH=$WINTER_EXT_DIR/src`.                                                                                                                                                                                                                                                                                                                                                                                                                               |
| tool:winter-test-service           | A full-stack sample app (React web, FastAPI api, background worker, Postgres, RabbitMQ) winter can manage. Stand it up as the workload behind `winter-test-service:manual`; its diagnostic controls trigger crashes, slow boots, and error output on demand, creating the known failure and health states an orchestration check observes.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |

## See also

- [`../standards/testing.md`](../standards/testing.md) — pytest layout, conftest scoping, and fake-vs-mock guidance for
  unit-test rows.
- [`../standards/linting.md`](../standards/linting.md) — ruff configuration and `mise run lint` / `mise run format`
  convention.
- [`../standards/typechecking.md`](../standards/typechecking.md) — pyright configuration and `mise run typecheck`
  convention.
- `workspace:/context/winter-cli/root-flags.md` — the `--winter` and `--service-orchestrator` override flags in full.
- `winter-service-tmux:/context/orchestrator-dev-loop.md` and `winter-service-docker:/context/dev-loop.md` — the
  direct-entrypoint dev loops for exercising changed orchestrator code.
- `workspace:/context/winter-cli/usage/provision.md` — the full `winter provision` surface for resource and seed-data
  setup.
- `workspace:/context/winter-cli/usage/clean.md` — the full `winter clean` surface, owner of the clean-verb command
  reference.
- `workspace:/context/winter-cli/tracing.md` — the opt-in tracing behavior `winter:manual-tracing` verifies: the span,
  propagation, the launcher gap, and the exit cap.
