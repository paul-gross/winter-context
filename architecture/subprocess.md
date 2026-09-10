# Subprocess

## Rule

- All `subprocess.run` / `subprocess.Popen` usage is confined to a repository or adapter under `internal/` — see
  `./repository-pattern.md`.
- Always `capture_output=True`, `text=True`, `check=False`. Inspect `returncode` explicitly.
- Wrap non-zero exits and `OSError` into the feature's `RepoError` via the injected `RepoErrorFactory` (see
  `./error-handling.md`). Capture `subcommand`, `cmd_args`, `cwd`, `exit_code`, and `stderr` as structured fields, not
  as a concatenated message.
- Never `shell=True` for any command whose tokens come from a variable. Pass `cmd` as a `list[str]`.
- **Bounded exception** — a feature may let workspace configuration opt an entry into `shell=True`, per entry, when the
  command's tokens are text the operator wrote into that entry. The bound this opt-in actually enforces is narrower than
  "safe from command-derived values": it refuses, at resolution time, an entry whose `command` string contains a
  `${...}` reference to a command-derived key — naming the entry and the reference rather than running it. That refusal
  covers only the `${...}` substitution grammar. The process such an entry starts still receives the scope *visible to
  that entry* as its own environment, overlaid on winter's inherited environment — the full accumulated scope for a
  feature- or named-band entry (so any command-derived key already in that scope travels with it, as ordinary env vars),
  but only the restricted workspace-band view for a workspace-band one, which excludes every feature- or named-band key
  regardless of origin. `shell=True` hands the whole command line to a real shell, which performs its own
  environment-variable expansion independent of winter's `${...}` grammar — a declared command like `foo $DB_PASSWORD`
  reaches the shell with a command-derived value expanded into it wherever `$DB_PASSWORD` is in the entry's visible
  scope. A feature taking this opt-in accepts that residual surface, exactly as broad as the `${...}` refusal leaves it
  for a feature- or named-band entry; document it alongside the opt-in rather than presenting the `${...}` refusal as
  closing it.

## Why

`check=True` raises `CalledProcessError`, which leaks a subprocess-specific exception type into every caller. Wrapping
at the boundary lets services catch `RepoError` without importing `subprocess`, and centralizes the structured fields
the dashboard and CLI render.

`capture_output=True` + `text=True` keeps stdout and stderr decoded and available for the error wrapper. Streaming
subprocesses are a different shape — use the `ISubprocessRunner.popen` seam, not raw `Popen`.

`shell=True` with variable inputs is a command-injection footgun. The list form is safe by default and indistinguishable
in cost.

Workspace configuration the operator wrote is not the untrusted variable that rule guards against — it's text the
operator typed into a file they control, the same trust level as the code they'd otherwise write by hand. A value a
command produced is different in origin — data a prior invocation returned — so the opt-in refuses letting the
*operator's own declared command line* reach for one by name via `${...}`. It cannot also keep that value out of the
child process's environment: the entry needs its own visible scope to resolve its own `${...}` tokens, and any
command-derived key already in *that* scope travels with it — the full accumulated scope for a feature- or named-band
entry, but only the restricted workspace-band view for a workspace-band one, where no feature- or named-band key
(command-derived or not) is ever present to travel. A `shell=True` entry's shell can still read and expand a
command-derived value through ordinary shell syntax (`$NAME`, `` `cmd` ``, etc.) — that surface is accepted, not closed,
by the `${...}`-only refusal above.

## Do

```python
def fetch(self, cwd: Path, remote: str) -> None:
    completed = subprocess.run(
        ["git", "fetch", remote],
        cwd=str(cwd),
        capture_output=True,
        text=True,
        check=False,
    )
    if completed.returncode != 0:
        raise self._errors.from_subprocess(
            completed, f"fetch {remote} failed", cwd=cwd,
        )
```

The factory extracts `subcommand`, `cmd_args`, `exit_code`, and `stderr` off `completed.args` and attaches them to the
`RepoError` as structured fields — same shape as `from_git(exc, message, *, cwd)` in `./error-handling.md`. Callers pass
only the high-level `message` and don't repeat the extraction at every wrap site.

**Method-name convention:** the factory has one method per underlying transport — `from_git` for `git.GitCommandError`,
`from_subprocess` for `subprocess.CompletedProcess`, and so on. `from_subprocess` is the canonical shape for an adapter
that wraps raw `subprocess`.

## Don't

```python
# Leaks CalledProcessError; loses cwd/args structure; check=True hides exit_code.
subprocess.run(["git", "fetch", remote], cwd=cwd, check=True)

# shell=True with a variable — command injection if `remote` contains a space or `;`.
subprocess.run(f"git fetch {remote}", shell=True, check=False)

# Discards stderr — the wrap site has nothing to log or surface.
subprocess.run(["git", "fetch", remote], cwd=cwd)
```

## See also

- `./error-handling.md` — structured errors via the injected factory; `from_<transport>(exc, message, *, cwd)` canonical
  shape.
- `./repository-pattern.md` — why subprocess lives in `internal/`.
- `../standards/logging.md` — log levels for wrapped subprocess failures.
- `winter:/tools/winter-cli/src/winter_cli/core/internal/local_subprocess_runner.py` — the production
  `ISubprocessRunner` adapter (`run` + `popen` seams).
- `winter:/tools/winter-cli/src/winter_cli/modules/workspace/internal/repo_error_factory.py` — the production wrapping
  factory, implementing `from_git`, `from_subprocess`, and the catch-all `from_exception`.
