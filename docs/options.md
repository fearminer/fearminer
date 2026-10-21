# Options

`fearminer --help` lists every option; the complete reference is at
[fearminer.com/docs](https://fearminer.com/docs/).

## The ones that matter

| | | |
|---|---|---|
| wallet | `-u WALLET.rig1` | The payout address, with an optional worker after a dot. Always given with a pool (`-o`). |
| pool | `-o stratum+ssl://host:port` | `stratum+ssl://` is a TLS port, `stratum+tcp://` or a bare `host:port` a plain one. Repeat `-o` for a backup. |
| algorithm | `-a quantus` | `quantus` (GPU and CPU), `randomx` (CPU), `pearl`, `tensorcash`, `commonfoundry` (NVIDIA); `--list-algorithms` lists them. Read off the wallet when left out. |
| cards | `-d 0,2` | Indices, PCI ids or UUIDs; `!1` excludes card 1. `--list-devices` prints them. Every discrete card by default; `--igpu` adds integrated ones. |
| CPU threads | `-t 8` | A count, `50%` or `+N`. One per physical core on RandomX; off beside a GPU on Quantus. |

```
fearminer -a quantus -o stratum+ssl://pool.example.com:3335 -u WALLET.rig1
fearminer -o stratum+ssl://pool.example.com:3335 -o stratum+tcp://backup.example.com:3333 -u WALLET.rig1
fearminer -a randomx -o stratum+ssl://pool.example.com:3335 -u WALLET.rig1 -t 50% --msr auto
fearminer -o pool.example.com:3333 -u WALLET -w rig1 -d 0,2
```

Every option can also be set as `FEARMINER_<OPTION>` in the environment, or in
a TOML file (`fearminer config example` prints one). The command line wins.

## Everything else

Pools and failover, TLS pinning, the proxy and DNS modes, the supervisor and
the watchdog, the thermal cut-off, RandomX and the helper, overclocking, the
commands and the hot reload, the hooks, the history, the API, the error codes
and the exit codes are all at
[fearminer.com/docs](https://fearminer.com/docs/). The short list:

| Option | |
|---|---|
| `-a, --algo <ALGO>` | what to mine (`--list-algorithms`) |
| `-o, --url <URL>` | `stratum+tcp://host:port`, `stratum+ssl://host:port`, or `host:port`; repeated for backup pools |
| `-u, --user <WALLET[.WORKER]>` | wallet address, with an optional worker name |
| `-w, --worker <NAME>` | worker name, appended to `--user` when it has none |
| `-p, --pass <PASS>` | pool password, `x` by default |
| `--tls-fingerprint <SHA256>`, `--tls-spki <sha256/BASE64>`, `--tls-tofu` | pin a self-signed pool certificate, by its fingerprint or its key, or trust it on first use |
| `--tls-insecure` | a pool behind a TLS port forward or a proxy: its certificate is accepted unchecked, the session encrypted but no longer authenticated (`E120` at every start); a pool with a pin keeps it |
| `--proxy <URL>` | every pool connection through a SOCKS5 proxy (`socks5://[user:pass@]host:port`; Tor works, the proxy resolves the names) |
| `--dns doh` | resolve pool names over DNS over HTTPS (`--doh-url`); `--ip 4`, `--ip 6` pick the address family |
| `--submit-timeout <SECS>`, `--max-latency <MS>` | a share with no reply is unanswered after 10 s; a pool whose replies stay slow hands the session to a faster one of the list |
| `-d, --devices <LIST>` | GPUs to mine on, by index or PCI id, or `!1` to leave one out |
| `--config <PATH>` | a TOML configuration file (`fearminer config example` prints one) |
| `--log-file <PATH>` | also write the log to a file, rotated by size |
| `--notify-telegram`, `--notify-discord`, `--notify-url`, `--heartbeat-url` | alerts and a heartbeat |
| `-t, --threads <N>` | CPU threads (0 by default on a rig with a GPU); also `50%`, `-2`, or `randomx:16` per algorithm; `--cpu-affinity`, `--cpu-priority` place them |
| `--msr auto`, `--helper <PATH>` | RandomX: the MSR tweaks and the huge pages through `fearminer-helper`. `auto` applies them when the helper is there, skips them with a coded line otherwise, and never refuses to start |
| `--cclock`, `--lock-cclock`, `--mclock`, `--lock-mclock`, `--pl`, `--fan` | overclocking (NVIDIA): one value for every card or one per card, read back after every set, put back at exit and after a crash. `fearminer oc show`, `fearminer oc reset`. Without any of them no register is touched, and `--no-oc` says so |
| `--api-bind <IP:PORT>` | stats endpoint (`/stats`, `/hive-stats`, `/api/v1/*`, `/healthz`), `127.0.0.1:4300` by default. Bind `0.0.0.0:4300` for a dashboard on another machine; a token is then required (`fearminer token show`). `--no-api` turns it off |
| `--no-telemetry` | send no telemetry at all ([security.md](security.md#what-the-miner-talks-to)); no feature depends on it. `--telemetry-interval <MIN>` sets its cadence, 15 by default, 5 to 1440 |
| `--enroll <CODE>`, `--remote-relay <URL>`, `--remote-wallet-delay <MIN>` | join a cockpit's fleet at start (for a service, a HiveOS flight sheet or a script); a relay of your own before `wss://relay.fearminer.com`; the minutes a new wallet from a mining sheet waits (0 by default: at once) |
| `--unrestricted-api <LEVEL>`, `--watch-config`, `--hook <EVENT:PATH>` | a running miner takes `pause`, `resume`, `toggle` and `retune` by default (the API, the cockpit keys `p` and `1`-`9`, `SIGUSR1`, or a file dropped in the state directory); `restart` and `stop` need `--unrestricted-api operate`. `SIGHUP`, `--watch-config` and `PUT /api/v1/config` reload the configuration without stopping the mining, validated whole before anything is applied. `--hook` runs a program of yours on one of ten events |
| `--history off`, `--history-retention <DAYS>`, `--history-max-size <MB>` | each rig keeps its own history in its state directory (10 s for a day, 1 min for a week, 5 min for a month, 1 h beyond, 90 days by default). `fearminer history` reads it offline, `GET /api/v1/history` serves it |
| `--engine <ALGO=VERSION>` | pin an engine version ([algorithms.md](algorithms.md#engines)) |
| `--background`, `--priority <0-5>` | run without a console (Unix; give `--log-file` with it), and the whole process's scheduling priority |
| `--allow-root` | run as root (the HiveOS package passes it); needed for overclocking |
| `--no-tui`, `--no-color`, `-v` | plain log, no colour, debug log |
