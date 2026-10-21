# Security and privacy

## Verify a download

```
fearminer verify SHA256SUMS
```

The release key is built into the binary; nothing else is needed. It checks the
signature offline, then the SHA-256 of every listed file in the same folder,
one line per file: `ok` when it matches, `MISSING` when the sums name a file
you did not download, `MISMATCH` when a file is there and its sum differs. It
exits 0 when the signature is good and every file that is present matches, so
`MISSING` is expected and only `MISMATCH` or a bad signature is the failure. Given an archive instead, it looks for `SHA256SUMS` beside
it. The sums also cover `sbom.cdx.json`. The key changes only with a release
that announces it in the README. Available from 1.1.0.

Without the binary, from 1.0.1:

```
minisign -Vm SHA256SUMS -P RWQR3owV+nRhfgNJeUBqcRK868S52NFG2BOrIzKpkQ5ey6lCEfM9+3Eg
sha256sum -c SHA256SUMS --ignore-missing
```

The binaries themselves are not code-signed yet
([install.md](install.md#first-run-on-windows-and-macos)).

## What the miner talks to

- Your pool, over `stratum+tcp`, `stratum+ssl`, `wss` (TensorCash) or
  `cmfd+tls` (Common Foundry); through the SOCKS5 proxy when `--proxy` names
  one, which then resolves the pool names. The pool is never
  contacted directly while a proxy is configured. `--dns doh` resolves the
  names over DNS over HTTPS.
- `telemetry.fearminer.com`, an anonymous report once a run is under way,
  every 15 minutes and at a clean stop, through the same proxy and resolver as
  the pools: a random install id, the version, the system, the algorithm and
  its engine, the GPU and CPU models, a hashrate range, the features in use, an
  uptime range, the error codes seen and the first pool's host name. Never the
  wallet, the worker name, a password, the machine name, an address, nor
  anything of the cockpit. `fearminer telemetry show` prints exactly what goes,
  [fearminer.com/telemetry](https://fearminer.com/telemetry/) explains every
  field, and `--no-telemetry` (or `FEARMINER_NO_TELEMETRY=1`) turns all of it
  off.
- `download.fearminer.com`, the signed engine index at start and every hour,
  an engine when one is missing or newer, and a release when a service update
  is asked (from the cockpit or `fearminer service update`).
- `cfg.fearminer.com`, the signed fee terms, about every 20 minutes, through
  the same proxy and resolver as the pools; fetched with `--no-telemetry` too.
- `relay.fearminer.com`, once the rig is enrolled in a cockpit and not before:
  one outgoing connection (port 443) carrying sealed messages only your fleet
  reads, through the same proxy and resolver as the pools. `--remote-relay`
  puts a relay of your own first.

Nothing else. The `egress` line printed at start lists every host the miner
will talk to. The notifiers and the heartbeat reach only the URLs you give
them. With `--proxy`, all of the above goes through the proxy, the engines'
own connections and the notifications too; only a notification or a
heartbeat to the local network goes direct.

The stats endpoint listens on `127.0.0.1` unless `--api-bind` says otherwise.
Its three writes, `POST /api/v1/oc`, `POST /api/v1/commands` and
`PUT /api/v1/config`, need the token from anywhere (`fearminer token show`
prints it, `fearminer token rotate` replaces it), and what a command may do is
bounded by `--unrestricted-api`.

## What it writes

Outside its own folder the miner writes two caches: the GPU tuning result
(`~/.config/fearminer/tuning.json` on Linux, `~/Library/Caches/fearminer` on
macOS) and, under `~/.cache/fearminer/`, the API token, the crash counters,
the watchdog log, this rig's history (`history.bin`,
bounded by `--history-max-size`), the certificates remembered by `--tls-tofu`
and what the cards read as before an overclock. Plus the log file when
`--log-file` asks for one. The helper keeps the original MSR values under
`/run/fearminer-helper/`, root-owned, until it puts them back.

## Reporting a vulnerability

Privately, to security@fearminer.com: see [SECURITY.md](../SECURITY.md).
