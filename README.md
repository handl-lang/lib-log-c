# lib-log-c

C backend of the `log` contract — leveled records to **stderr** over
`ffi.call("libc.so.6", ...)`, no posix dep, no shell.

`src/index.hnl` implements the full `lib-log` surface as a single
module: the pure logger algebra (`make`/`with_level`/`child`/
`should_log`/`level_rank`), the canonical `format` renderer
(`LEVEL ISO-8601-UTC message key=value ...`, slog-style quoting), and
the effectful emission path.

Sink: libc stdio's `stderr` FILE* via `fwrite` — unbuffered under the
C stdio model, so each record reaches the kernel promptly and stdout
stays clean for piped program output. Timestamp is
`clock_gettime(CLOCK_REALTIME)` -> epoch ms -> proleptic Gregorian
civil date (the `lib-datetime` algorithm) as `YYYY-MM-DDTHH:MM:SS.mmmZ`,
always UTC.

Syslog sink: `make_syslog(min, attrs, opts)` builds a logger with
`sink: "syslog"` — records go to `syslog(3)` via
`openlog(ident, LOG_PID, facility)` + `syslog(facility | level, "%s",
msg)`. The variadic problem is sidestepped: the format is the constant
`"%s"` and the rendered message rides as the vararg, so the ffi's fixed
3-arg signature suffices (the `curl_easy_setopt` precedent). The
syslog line is `msg key=value ...` — no LEVEL/timestamp prefix, since
syslog stamps its own and encodes the level in the priority word.
`opts` = `{ facility, ident }` (defaults `LOG_USER`=8 / `"handl"`).

One implementation note: the module exports an arrow named `error`
(the error-level convenience), which shadows the host `error` module
inside the file — so `error.raise` is imported as `raise` up front.

## Layout

```text
src/index.hnl       the .hnl sources (all contract arrows)
src/index.test.hnl  contract tests — 64 cases, all passing natively
pkg.hnl             package manifest — modules, deps, description, license
```

Dependencies: log (contract), grade

## Gate

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

## License

MIT.
