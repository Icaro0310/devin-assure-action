# devin-assure-action

GitHub Action that audits AI coding agent sessions with
[`devin-qa-pack`](https://pypi.org/project/devin-qa-pack/) — the
claim-verification package from the
[devin-assure](https://github.com/Icaro0310/devin-assure) family of the
devin-* ecosystem.

It runs `devin-qa-pack audit` against the agent's `sessions.db` and fails
the job when any audited session's verdict is not `PASS`. Gate CI on
evidence, not narrative: a session that claims "tests passed, committed,
pushed" is checked against the recorded `tool_call_state`.

## Usage

```yaml
- uses: Icaro0310/devin-assure-action@v0.1.0
```

Audit one session, or a database at a non-default location:

```yaml
- uses: Icaro0310/devin-assure-action@v0.1.0
  with:
    sessions-db: path/to/sessions.db   # default: auto-detect
    session: 9f3c...                   # audit one session instead of --all
```

```yaml
- uses: Icaro0310/devin-assure-action@v0.1.0
  with:
    limit: "50"                          # max sessions with --all
    version: "devin-qa-pack==0.1.0"
```

## The two distinct failures

| Situation | Exit | Meaning |
|---|---|---|
| sessions.db missing/unreadable, zero sessions in scope | 2 | violated precondition — nothing to audit |
| audits ran, a session is `PARTIAL`/`UNVERIFIED` | 1 | a real evidence gap — claims not backed by tool_call_state |

## Inputs / outputs

- `sessions-db` — path to `sessions.db` (default: auto-detect).
- `session` — audit a single session ID instead of `--all`.
- `limit` — max sessions with `--all` (default `20`).
- `version` — pip specifier (default `devin-qa-pack>=0.1,<0.2`).
- outputs: `verdict` (`PASS`/`PARTIAL`/`UNVERIFIED`, worst across
  sessions), `sessions` (count), `disputed` (contradicted claims).

## Verdicts

| Verdict | Meaning | Fails job? |
|---|---|---|
| `PASS` | every claim verified against tool_call_state | no |
| `PARTIAL` | any disputed claim, or a mix of outcomes | yes |
| `UNVERIFIED` | no claims found, or none checkable | yes |

## Platform support

Runs on any runner with bash; Python 3.12 comes from
`actions/setup-python`. The audit needs a `sessions.db` — typically a
self-hosted runner with a Devin install, or a db artifact passed via
`sessions-db`.

## Notes

- Read-only by design — the audit inspects recorded state and never
  modifies it.
- The satellite repo is only the Action distribution vehicle; all audit
  logic lives in
  [`devin-qa-pack`](https://github.com/Icaro0310/devin-assure/tree/main/packages/qa-pack).
