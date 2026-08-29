# go-hostdb

Database dump, import and inspection for restricted shared-hosting accounts —
where the only access is a shell over SSH and no tooling can be assumed —
extracted from [rehost](https://github.com/pietervanleuven/rehost).

- `Dump` streams `mysqldump | gzip` while gunzipping in memory to verify the
  completion footer — the shell reports gzip's exit, not mysqldump's, so the
  footer is the truncation guard. `DumpPHP` is the same contract via a PHP
  helper (mysqli → PDO, gzip from PHP itself, including views, triggers and
  routines) for hosts without mysqldump.
- `Import` streams a verified local dump into the destination's client;
  `Inspect` learns server version, size, charset and table counts in one
  round trip.
- Passwords never touch argv or the environment: MySQL clients read a
  defaults file on stdin or over a mode-600 FIFO; PostgreSQL (libpq refuses
  FIFOs) gets a umask-077 pgpass file staged under `$HOME`/`StageDir` and
  removed on the same command line.
- Driver-aware throughout: `NormalizeDriver` folds config spellings to
  mysql/pgsql, `ResolveClientTools` picks mysql/mariadb/psql binary names
  from what the host actually has.

All remote I/O goes through the transport-free
[`go-ssh/remote`](https://github.com/pietervanleuven/go-ssh) contract — any
`Run`/`Stream` implementation works; nothing here dials SSH itself.

```go
import hostdb "github.com/pietervanleuven/go-hostdb"
```

Apache-2.0.
