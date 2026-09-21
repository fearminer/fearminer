# FearMiner

A GPU and CPU miner built to carry several proof-of-work algorithms behind
one binary, one command line and one cockpit. Native CUDA kernels on NVIDIA
cards, a Vulkan fallback elsewhere, a CPU engine. Pick the algorithm with
`-a`; `fearminer --list-algorithms` prints what a build mines and the fee of
each.

## Algorithms

| Algorithm | Coin | `-a` | Engines | Fee |
|---|---|---|---|---|
| Quantus, QPoW over Poseidon2 | QTC | `quantus` (also `qpow`, `qtc`) | CUDA, Vulkan, CPU | 2 % |
| RandomX (rx/0) | XMR | `randomx` (also `rx`, `monero`, `xmr`) | CPU | 0.85 % |
| VerusHash 2.2 | VRSC | `verushash` (also `verus`, `vrsc`) | CPU | 0.85 % |

RandomX wants 2 GiB of RAM per rig and huge pages for the full rate; VerusHash
needs AES-NI and CLMUL. More algorithms are on the way; each arrives with its
own launcher in the archive and its own line here. The full documentation,
every option and every feature, is at [fearminer.com/docs](https://fearminer.com/docs/).

## Downloads

This repository carries the **releases**. Every version ships:

| File | For |
|---|---|
| `fearminer-<version>-windows-x86_64.zip` | Windows: `fearminer.exe`, one `start_<algo>.bat` launcher per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer-<version>-linux-x86_64.tar.gz` | Linux: `fearminer`, one `start_<algo>.sh` launcher per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer-<version>-macos-arm64.tar.gz` | macOS, Apple silicon: `fearminer` (native Metal), one `start_<algo>.sh` launcher per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer_custom-<version>.tar.gz` | HiveOS custom miner package |
| `SHA256SUMS` | checksums of every file above |
| `SHA256SUMS.minisig` | signature of `SHA256SUMS` by FearMiner's release key (from 1.0.1) |
| `sbom.cdx.json` | the list of every component inside the binary (CycloneDX), from 1.1.0 |

Each archive unpacks into a folder named like the archive.

## Quick start

Unpack, open the launcher of the algorithm you mine (`start_quantus.bat` on
Windows, `start_quantus.sh` on Linux) in a text editor, put your wallet in
`WALLET` and a name for the machine in `WORKER`, run it. The launcher lists
a few public pools; `POOL` takes any stratum pool. Or by hand:

```
fearminer WALLET -w rig1                                              (the wallet alone: chain and pool chosen for you)
fearminer -a quantus -o stratum+ssl://pool.example.com:3335 -u WALLET.rig1
fearminer -a quantus -o pool.example.com:3334 -u WALLET -w rig1        (plain TCP)
fearminer -a randomx -o stratum+tcp://pool.example.com:3333 -u WALLET.rig1
fearminer --list-algorithms
fearminer --list-devices
fearminer explain E302
fearminer --help
```

From 1.1.0 the wallet alone is a complete command: the chain is read off the
address, the chain's public pools are probed and the fastest is used, the
others as backups. Without a wallet the miner starts in monitoring mode
(devices, cockpit and stats, no mining) and says so.

| Option | |
|---|---|
| `-a, --algo <ALGO>` | what to mine (`--list-algorithms`) |
| `-o, --url <URL>` | `stratum+tcp://host:port`, `stratum+ssl://host:port`, or `host:port`; repeated for backup pools |
| `-u, --user <WALLET[.WORKER]>` | wallet address, with an optional worker name |
| `-w, --worker <NAME>` | worker name, appended to `--user` when it has none |
| `-p, --pass <PASS>` | pool password, `x` by default |
| `--tls-fingerprint <SHA256>` | pin a self-signed pool certificate |
| `-d, --devices <LIST>` | GPUs to mine on, by index or PCI id, or `!1` to leave one out |
| `--config <PATH>` | a TOML configuration file (`fearminer config example` prints one) |
| `--log-file <PATH>` | also write the log to a file, rotated by size |
| `--notify-telegram`, `--notify-discord`, `--notify-url`, `--heartbeat-url` | alerts and a heartbeat |
| `-t, --threads <N>` | CPU threads (0 by default on a rig with a GPU) |
| `--api-bind <IP:PORT>` | stats endpoint (`/stats`, `/hive-stats`, `/api/v1/*`, `/healthz`), `127.0.0.1:4300` by default (bind `0.0.0.0:4300` for a dashboard on another machine, then a token is required: `fearminer token show`); `--no-api` turns it off |
| `--no-telemetry` | do not send the build-and-pool ping to `api.fearminer.com` |
| `--no-tui`, `--no-color`, `-v` | plain log, no colour, debug log |

Every option can also be set from the environment as `FEARMINER_<OPTION>`
or from the configuration file. The rest (pools with failover, the
supervisor and the watchdog, the thermal cut-off, the error codes and
`fearminer explain`, the exit codes, the API) is on
[fearminer.com/docs](https://fearminer.com/docs/).

## HiveOS

Custom miner package: the `fearminer_custom-<version>.tar.gz` asset of a
release. Flight sheet: custom miner, that asset's URL as installation URL,
the algorithm's HiveOS name (`qpow` for Quantus, `randomx`, `verushash`),
wallet and pool as usual.

## Requirements

An NVIDIA GPU with a driver of the 550 series or newer (570+ for RTX 50), or
an Apple silicon Mac (native Metal, macOS archive). Without either the miner
runs on Vulkan at a fraction of the rate; CPU-only algorithms (RandomX,
VerusHash) need no GPU at all. The miner refuses to run as root unless
`--allow-root` (the HiveOS package passes it).

## Fee

Per algorithm, listed by `--list-algorithms` and shown in the header of the
cockpit; mined in one-minute rounds, on a separate connection to the fee
pool, never on your session; the log says when a round starts and ends. The
rate printed is a ceiling built into the binary: the signed terms the miner
fetches can lower it, move it to another pool, or have it mined with another
algorithm of the same device class, never raise it. An algorithm the terms
name no fee pool for is mined with no fee. A higher rate takes a new
release, and this table changes with it. The miner is closed source at this
stage.

## What the miner talks to

- Your pool, over `stratum+tcp` or `stratum+ssl`.
- The fee pool, during the fee rounds.
- `cfg.fearminer.com`, for the signed fee terms: which pool the fee is mined
  on and the latest version. Read at start and every 20 minutes; when it
  cannot be reached the start is delayed by five seconds at most, then the
  miner mines with the last terms it verified, or with the ones built in.
- `api.fearminer.com`, a ping carrying the build number and the pool
  address, at the same cadence. No wallet, no worker name, nothing about
  the hardware. `--no-telemetry` (or `FEARMINER_NO_TELEMETRY=1`) turns it
  off; the terms are still fetched.

Nothing else, and the miner says so itself: the `egress` line printed at
start lists every host it will talk to, and the notifiers and the heartbeat
only reach the URLs you give them. The stats endpoint listens on
`127.0.0.1` unless `--api-bind` says otherwise. Outside its own folder the
miner writes two caches: the GPU tuning result
(`~/.config/fearminer/tuning.json` on Linux, `~/Library/Caches/fearminer`
on macOS) and, under `~/.cache/fearminer/`, the last verified terms, the
API token, the crash counters and the watchdog log; plus the log file when
you ask for one with `--log-file`.

## Verifying a download

```
sha256sum -c SHA256SUMS --ignore-missing
```

From 1.0.1, `SHA256SUMS` is signed with [minisign](https://jedisct1.github.io/minisign/)
by FearMiner's release key, so the checksums can be trusted even if the
download did not come from this page:

```
minisign -Vm SHA256SUMS -P RWQR3owV+nRhfgNJeUBqcRK868S52NFG2BOrIzKpkQ5ey6lCEfM9+3Eg
```

From 1.1.0 the miner checks its own downloads, with the key built in and
nothing else needed:

```
fearminer verify SHA256SUMS
```

It checks the signature offline, then the SHA-256 of every listed file in
the same folder, one line per file (`ok`, `MISSING`, `MISMATCH`), and exits 0
only when everything is right. Given an archive instead, it looks for
`SHA256SUMS` beside it. The sums also cover `sbom.cdx.json`. The key changes
only with a release that announces it here.
