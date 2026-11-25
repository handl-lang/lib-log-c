# lib-log-c

C backend of the `log` contract — leveled records to **stderr** (or
syslog) over `ffi.call("libc.so.6", ...)`. Pure `.hnl`, no posix dep,
no shell.

## Layout

| path | role |
| --- | --- |
| `src/index.hnl` | the full `lib-log` surface as a single module |
| `src/index.test.hnl` | the .hnl test suite |
| `pkg.hnl` | manifest — `deps: { log: "branch:main" }` |
| `scripts/hnl-gate.sh`, `workflows/gate.yaml` | the gate / bupha entry |

## Contract

- Implements the pure logger algebra (`make`/`with_level`/`child`/
  `should_log`/`level_rank`), the canonical `format` renderer
  (`LEVEL ISO-8601-UTC message key=value ...`, slog-style quoting),
  and the effectful emission path — same as `log-js`.
- Sink: libc `stderr` FILE* via `fwrite` (unbuffered; stdout stays
  clean). Timestamp: `clock_gettime(CLOCK_REALTIME)` → epoch ms →
  proleptic Gregorian civil date, always UTC.
- Syslog sink: `make_syslog(min, attrs, opts)` → `syslog(3)` via
  `openlog`/`syslog`; format is constant `"%s"` so the 3-arg ffi
  signature suffices (the `curl_easy_setopt` precedent). `opts` =
  `{ facility, ident }` (defaults `LOG_USER`=8 / `"handl"`).
- Gotcha: the module exports an arrow named `error` (the error-level
  convenience), which shadows the host `error` module — `error.raise`
  is imported as `raise` up front.

## Verification

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

## License

MIT.

<!-- myco:agent-awareness -->

## Agent awareness

- Session start / any work discussion: run `myco status --prompt`
  before proposing next steps — know the board first.
- User describes new work → offer to capture it (`myco todo add`,
  `issue new`, `proposal new`). Don't wait to be asked.
- Picking up a task: `myco resume <ID>` is the briefing.
- Wrapping up: `myco handoff <ID>` anything resumable; keep statuses
  honest (claimed → in-progress → blocked/done).

<!-- /myco:agent-awareness -->
