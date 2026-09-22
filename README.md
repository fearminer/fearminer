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

RandomX wants 2 GiB of RAM per rig, and huge pages plus the MSR tweaks for the
full rate, which the Linux archive's helper takes care of (see below); VerusHash
needs AES-NI and CLMUL. More algorithms are on the way; each arrives with its
own launcher in the archive and its own line here. The full documentation,
every option and every feature, is at [fearminer.com/docs](https://fearminer.com/docs/).

## Downloads

This repository carries the **releases**. The current one is 1.3.0
(`fearminer-1.3.0-windows-x86_64.zip`, `fearminer-1.3.0-linux-x86_64.tar.gz`,
`fearminer-1.3.0-macos-arm64.tar.gz`, `fearminer_custom-1.3.0.tar.gz`, `SHA256SUMS`,
`SHA256SUMS.minisig`, `sbom.cdx.json`). Every version ships:

| File | For |
|---|---|
| `fearminer-<version>-windows-x86_64.zip` | Windows: `fearminer.exe`, one `start_<algo>.bat` launcher per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer-<version>-linux-x86_64.tar.gz` | Linux: `fearminer`, `fearminer-helper` (the small privileged helper RandomX uses for the MSR tweaks and the huge pages, through `sudo`; the miner itself never runs as root), one `start_<algo>.sh` launcher per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer-<version>-macos-arm64.tar.gz` | macOS, Apple silicon: `fearminer` (native Metal), one `start_<algo>.sh` launcher per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer_custom-<version>.tar.gz` | HiveOS custom miner package |
| `SHA256SUMS` | checksums of every file above |
| `SHA256SUMS.minisig` | signature of `SHA256SUMS` by FearMiner's release key (from 1.0.1) |
| `sbom.cdx.json` | the list of every component inside the binary (CycloneDX), from 1.1.0 |

Each archive unpacks into a folder named like the archive.

## The warning at the first run

The binaries are not code-signed yet, so Windows and macOS stop them the first
time you run one, and scanners routinely file any miner under `HackTool` or
`CoinMiner`. Check the download is ours first (see *Verifying a download*
below), then:

- **Windows**: SmartScreen shows *Windows protected your PC* with *Unknown
  publisher*; the Run button is behind the small link: **More info**, then
  **Run anyway**. Microsoft Defender may also quarantine the miner as a
  potentially unwanted application: add an exclusion for the folder the
  archive unpacked into (Windows Security, Virus & threat protection, Manage
  settings, Exclusions, or `Add-MpPreference -ExclusionPath <folder>` in an
  elevated PowerShell), never for a whole drive.
- **macOS**: Gatekeeper says *fearminer cannot be opened because the developer
  cannot be verified*, because the build is not notarised and the download
  carries the quarantine flag. Either System Settings, Privacy & Security,
  **Open Anyway**, or clear the flag yourself from the unpacked folder:
  `xattr -dr com.apple.quarantine .` (the `start_<algo>.sh` launchers run that
  line for you).
- **Linux and HiveOS**: nothing of the sort happens. Unpack, run
  `fearminer verify`, mine.

Signing is planned, with no date promised: an Authenticode certificate and an
Apple developer account are a recurring bill FearMiner pays out of what it
earns rather than before. Until then the signature that matters is the one on
`SHA256SUMS`, which you can check yourself, offline. The detail, with the
exact clicks per platform, is at
[fearminer.com/docs](https://fearminer.com/docs/#unsigned).

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
fearminer -a randomx -o stratum+ssl://pool.example.com:3334 -u WALLET.rig1 -t 50% --msr auto   (MSR tweaks and huge pages through fearminer-helper, see Requirements)
fearminer -o stratum+ssl://pool.example.com:3335 -u WALLET.rig1 --proxy socks5://127.0.0.1:9050   (every pool connection through Tor or any SOCKS5 proxy)
fearminer --list-algorithms
fearminer --list-devices
fearminer history --from -24h --step 5m                               (what this rig did, offline, from its own history)
fearminer token show                                                  (the API token a write or another machine must carry)
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
| `--tls-fingerprint <SHA256>`, `--tls-spki <sha256/BASE64>`, `--tls-tofu` | pin a self-signed pool certificate, by its fingerprint or its key, or trust it on first use |
| `--proxy <URL>` | reach every pool through a SOCKS5 proxy (`socks5://[user:pass@]host:port`; Tor works, the proxy resolves the names) |
| `--dns doh` | resolve the pool names over DNS over HTTPS (`--doh-url`); `--ip 4`, `--ip 6` pick the address family |
| `--submit-timeout <SECS>`, `--max-latency <MS>` | a share with no reply is unanswered after 10 s; a pool whose replies stay slow hands the session to a faster one of the list |
| `-d, --devices <LIST>` | GPUs to mine on, by index or PCI id, or `!1` to leave one out |
| `--config <PATH>` | a TOML configuration file (`fearminer config example` prints one) |
| `--log-file <PATH>` | also write the log to a file, rotated by size |
| `--notify-telegram`, `--notify-discord`, `--notify-url`, `--heartbeat-url` | alerts and a heartbeat |
| `-t, --threads <N>` | CPU threads (0 by default on a rig with a GPU); also `50%`, `-2`, or `randomx:16` per algorithm; `--cpu-affinity`, `--cpu-priority` place them |
| `--msr auto`, `--helper <PATH>` | RandomX: the MSR tweaks and the huge pages through `fearminer-helper` (`auto` by default: applied when the helper is there, skipped with a coded line otherwise, never a refusal to start) |
| `--cclock`, `--lock-cclock`, `--mclock`, `--lock-mclock`, `--pl`, `--fan` | overclocking (NVIDIA): one value for every card or one per card, read back after every set, put back at exit and after a crash; `fearminer oc show`, `fearminer oc reset`; without any of them no register is touched, `--no-oc` says so |
| `--api-bind <IP:PORT>` | stats endpoint (`/stats`, `/hive-stats`, `/api/v1/*`, `/healthz`), `127.0.0.1:4300` by default (bind `0.0.0.0:4300` for a dashboard on another machine, then a token is required: `fearminer token show`); `--no-api` turns it off |
| `--no-telemetry` | do not send the build-and-pool ping to `api.fearminer.com` |
| `--unrestricted-api <LEVEL>`, `--watch-config`, `--hook <EVENT:PATH>` | a running miner takes `pause`, `resume`, `toggle` and `retune` by default (the API, the cockpit keys `p` and `1`-`9`, `SIGUSR1`, or a file dropped in the state directory); `restart` and `stop` need `--unrestricted-api operate`. `SIGHUP`, `--watch-config` and `PUT /api/v1/config` reload the configuration without stopping the mining, the whole of it validated before anything is applied; `--hook` runs a program of yours on one of ten events |
| `--history off`, `--history-retention <DAYS>`, `--history-max-size <MB>` | each rig keeps its own history in its state directory (10 s for a day, 1 min for a week, 5 min for a month, 1 h beyond, 90 days by default); `fearminer history` reads it offline, `GET /api/v1/history` serves it |
| `--background`, `--priority <0-5>` | run without a console (Unix; give `--log-file` with it), and the whole process's scheduling priority |
| `--no-tui`, `--no-color`, `-v` | plain log, no colour, debug log |

Every option can also be set from the environment as `FEARMINER_<OPTION>`
or from the configuration file. The rest (pools with failover, backoff and
what happens when a pool misbehaves, proxy and DNS, TLS, the supervisor and
the watchdog, the thermal cut-off, RandomX and the helper, overclocking,
the commands and the hot reload, the hooks, the history, the error codes
and `fearminer explain`, the exit codes, the API) is on
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
VerusHash) need no GPU at all. From 1.1.1 every engine checks itself
against known answers before the first share; a card that gives wrong
answers is left out, and every GPU share is re-checked on the CPU before it
is sent. The miner refuses to run as root unless `--allow-root` (the HiveOS
package passes it).

RandomX runs at its full rate with huge pages and the MSR tweaks, which need
root: on Linux the archive ships `fearminer-helper`, a small separate
privileged binary the miner reaches through `sudo -n` before its first
connection and never afterwards; it grows the huge-page pool, applies the
CPU's MSR preset (the `msr` kernel module, `modprobe msr`), puts the
original values back at exit and after a crash, and gives back the huge
pages a run reserved when that run stops. Install it once
(`sudo install -m 0755 fearminer-helper /usr/local/bin/`, then
`fearminer-helper install` prints the sudoers line and the systemd drop-in;
it writes nothing itself): the miner looks for the installed
`/usr/local/bin/fearminer-helper` first, the path the sudoers line names,
then for the copy beside its own binary. `--helper PATH` names another
place and is then the only path tried. Without a helper the miner starts
anyway and says what it costs. Overclocking (core and memory clocks, power
limit, fan) needs the NVIDIA driver 520 or newer for the clock offsets and
the miner as root (`--allow-root`); without an OC option no register is
touched.

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

- Your pool, over `stratum+tcp` or `stratum+ssl`; through the SOCKS5 proxy
  when `--proxy` names one (the proxy then resolves the pool names; the pool
  is never contacted directly while a proxy is configured), with the names
  resolved over DNS over HTTPS when `--dns doh` says so.
- The fee pool, during the fee rounds, through the same proxy.
- `cfg.fearminer.com`, for the signed fee terms: which pool the fee is mined
  on and the latest version. Read at start and every 20 minutes, directly,
  never through the proxy; when it cannot be reached the start is delayed by
  five seconds at most, then the miner mines with the last terms it
  verified, or with the ones built in.
- `api.fearminer.com`, a ping carrying the build number and the pool
  address, at the same cadence, directly. No wallet, no worker name, nothing
  about the hardware. `--no-telemetry` (or `FEARMINER_NO_TELEMETRY=1`) turns
  it off; the terms are still fetched.

Nothing else, and the miner says so itself: the `egress` line printed at
start lists every host it will talk to, and the notifiers and the heartbeat
only reach the URLs you give them, directly. Only the pool connections go
through `--proxy`. The stats endpoint listens on `127.0.0.1` unless
`--api-bind` says otherwise; its three writes, `POST /api/v1/oc`,
`POST /api/v1/commands` and `PUT /api/v1/config`, need the token from
anywhere (`fearminer token show` prints it, `fearminer token rotate`
replaces it), and what a command may do is bounded by
`--unrestricted-api`.
Outside its own folder the miner writes two caches: the
GPU tuning result (`~/.config/fearminer/tuning.json` on Linux,
`~/Library/Caches/fearminer` on macOS) and, under `~/.cache/fearminer/`,
the last verified terms, the API token, the crash counters, the watchdog
log, this rig's own history (`history.bin`, bounded by
`--history-max-size`), the certificates remembered by `--tls-tofu` and what
the cards read as before an overclock; plus the log file when you ask for
one with `--log-file`. The helper keeps the original MSR values under
`/run/fearminer-helper/`, root-owned, until it puts them back.

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
