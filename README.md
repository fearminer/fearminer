# FearMiner

A GPU and CPU miner: several proof-of-work algorithms behind one binary, one
command line and one cockpit. Native CUDA kernels on NVIDIA, Metal on Apple
silicon, a Vulkan fallback elsewhere, a CPU engine. This repository carries
the releases; the full documentation is at
[fearminer.com/docs](https://fearminer.com/docs/).

## 1. Mine now

One line installs FearMiner, checks the download against the release key,
and runs it as a service that starts with the machine.

Linux, macOS:

```
curl -fsSL https://get.fearminer.com | sh
```

Windows (PowerShell):

```
irm https://get.fearminer.com/win | iex
```

| | |
|---|---|
| with your cockpit's code | `curl -fsSL https://get.fearminer.com \| sh -s -- fm1_...` (the cockpit's *Add a rig* gives the exact line); Windows: `$env:FEARMINER_ENROLL='fm1_...'; irm https://get.fearminer.com/win \| iex` |
| no cockpit: your wallet on your pool | `curl -fsSL https://get.fearminer.com \| sh -s -- --wallet YOUR_WALLET --pool stratum+ssl://POOL:PORT --worker rig1`; Windows: `$env:FEARMINER_WALLET='YOUR_WALLET'; $env:FEARMINER_POOL='stratum+ssl://POOL:PORT'; $env:FEARMINER_WORKER='rig1'; irm https://get.fearminer.com/win \| iex` |
| HiveOS | nothing to install: `--enroll fm1_...` in the flight sheet's extra config arguments; the flight sheet keeps deciding what the rig mines |

The check: `SHA256SUMS` against the release key (minisign, or OpenSSL 3),
then the archive against `SHA256SUMS`; without either tool,
download.fearminer.com and GitHub must agree (on Windows, always that). Nothing
runs before. The service is your account's (no password); `--system` installs
the machine's, with sudo. With neither a cockpit nor a wallet, the rig watches
its cards and waits.

### Cockpit

**[app.fearminer.com](https://app.fearminer.com)**: the whole farm on one
screen, from your phone or your PC. Free, no account, end-to-end encrypted:
the relay in the middle passes sealed messages it cannot read, cannot add a rig
and cannot sign a command. Your fleet is 12 words, written down once. The
fleet's hashrate and history, every rig and every card; mining sheets (what each
rig mines: the algorithm, the wallet, the pools, like a HiveOS flight sheet);
pause, resume and restart. A new wallet waits 10 minutes before it applies,
announced everywhere, and can be cancelled from any cockpit or with
`fearminer remote cancel`. HiveOS rigs are watched and commanded; their flight
sheet decides what they mine.

| Command | |
|---|---|
| `fearminer service install [OPTIONS]` | a run made permanent: the same options as a run (`fearminer service install -o stratum+ssl://POOL:PORT -u YOUR_WALLET -w rig1`); systemd on Linux, launchd on macOS, a task at logon on Windows |
| `fearminer service status`, `logs`, `stop`, `start`, `restart`, `uninstall` | as named; `uninstall` keeps the settings, the history and the keys |
| `fearminer enroll fm1_...` | join your cockpit's fleet, no restart; `--status`, `--leave` |
| `fearminer remote status`, `off`, `on`, `cancel` | the remote channel on this machine; `off` refuses every remote command until `on` |

### Prefer to download?

The same files are at
[download.fearminer.com/latest/](https://download.fearminer.com/latest/)
(`fearminer-linux-x86_64.tar.gz`, `fearminer-windows-x86_64.zip`,
`fearminer-macos-arm64.tar.gz`, `SHA256SUMS`, `SHA256SUMS.minisig`) and on
[GitHub](https://github.com/fearminer/fearminer/releases/latest), with every
earlier version.

`YOUR_WALLET` is a public receiving address from a wallet app or an exchange
account. Never a private key, a seed phrase or a password. The chain is read
off the address, so the wallet picks the algorithm; *Algorithms* below gives
the shape of each address. The pool is yours to choose: its host and port are
on the pool's own page, and FearMiner never picks one for you.

Then download, check the download with a tool that is not ours, unpack, and
run:

```
$ curl -LO https://github.com/fearminer/fearminer/releases/latest/download/fearminer-linux-x86_64.tar.gz
$ curl -LO https://github.com/fearminer/fearminer/releases/latest/download/SHA256SUMS
$ grep ' fearminer-linux-x86_64.tar.gz$' SHA256SUMS | sha256sum -c -
$ tar xzf fearminer-linux-x86_64.tar.gz && cd fearminer-*-linux-x86_64
$ ./fearminer -o stratum+ssl://POOL:PORT -u YOUR_WALLET -w rig1
```

Windows: fetch `fearminer-windows-x86_64.zip` and `SHA256SUMS` the same way,
compare `(Get-FileHash .\fearminer-windows-x86_64.zip -Algorithm SHA256).Hash`
with the matching line, unzip, then `.\fearminer.exe -o stratum+ssl://POOL:PORT -u YOUR_WALLET -w rig1`.
macOS: `fearminer-macos-arm64.tar.gz`, `grep ' fearminer-macos-arm64.tar.gz$' SHA256SUMS | shasum -a 256 -c -`,
unpack, `xattr -dr com.apple.quarantine .`, then `./fearminer -o stratum+ssl://POOL:PORT -u YOUR_WALLET -w rig1`.
Or open `start_quantus.sh` (or `start_randomx`, `start_verushash`, `start_pearl`; `.bat` on Windows) in a text editor, set `WALLET`,
`POOL` and `WORKER`, and run it.

A wallet always comes with its pool: a wallet without one is refused before
anything is sent (`E224`, exit code 2). The chain is read off the address when
`-a` is left out. Without a wallet and a pool the miner watches the devices
instead of mining.

**The first run is blocked on Windows and macOS.** The binaries are not
code-signed yet, and scanners file any miner under `HackTool` or `CoinMiner`.
Check the download first (*Verifying a download* below), then:

- **Windows**: SmartScreen says *Windows protected your PC*, *Unknown
  publisher*. **More info**, then **Run anyway**. Defender may quarantine the
  miner: add an exclusion for the unpacked folder (Windows Security, Virus &
  threat protection, Manage settings, Exclusions), or
  `Add-MpPreference -ExclusionPath <folder>` in an elevated PowerShell. Never
  a whole drive.
- **macOS**: Gatekeeper says *fearminer cannot be opened because the developer
  cannot be verified*. System Settings, Privacy & Security, **Open Anyway**;
  or `xattr -dr com.apple.quarantine .` in the unpacked folder, which the
  `start_<algo>.sh` launchers run for you.
- **Linux and HiveOS**: nothing happens. Unpack, `fearminer verify`, mine.

It works when the log reaches these lines. A GPU rig:

```
job 9dd830e6 diff 18.25G
✔ 1/1 · GPU0 · 18.25G · 24 ms
hashrate 618.4 MH/s (10m 611.9, session 609.8, eff 598.2 / 97.8 %)
```

A CPU rig, where the rows read `CPU`:

```
job 4b1c77a2 diff 40.0K
✔ 1/1 · CPU · 40.0K · 31 ms
hashrate 7.1 kH/s (10m -, session -, eff -)
```

`eff` and the averages read `-` until ten shares are accepted in the window.
To stop: `q` in the cockpit, or Ctrl+C. Either one stops cleanly, exit code 0.

**Where the money goes.** An accepted share is not a payment. The `pool` line
at start names the pool the miner is on, the one you gave with `-o`. Your balance lives on that pool's own
dashboard, under the address you gave, and it pays at the threshold and on the
schedule that pool sets, minus that pool's fee. Those are the pool's, not
FearMiner's; FearMiner's own fee is separate and shown in the cockpit header.

## 2. Change what matters

| | | |
|---|---|---|
| wallet | `-u WALLET.rig1` | The payout address, with an optional worker after a dot. Always given with a pool (`-o`). |
| pool | `-o stratum+ssl://host:port` | `stratum+ssl://` is a TLS port, `stratum+tcp://` or a bare `host:port` a plain one. Repeat `-o` for a backup. |
| algorithm | `-a quantus` | `quantus` (GPU and CPU), `randomx` (CPU), `verushash` (CPU), `pearl` (NVIDIA sm_86). Read off the wallet when left out. |
| cards | `-d 0,2` | Indices, PCI ids or UUIDs; `!1` excludes card 1. `--list-devices` prints them. Every discrete card by default; `--igpu` adds integrated ones. |
| CPU threads | `-t 8` | A count, `50%` or `+N`. One per physical core on RandomX and VerusHash; off beside a GPU on Quantus. |

```
fearminer -a quantus -o stratum+ssl://pool.example.com:3335 -u WALLET.rig1
fearminer -o stratum+ssl://pool.example.com:3335 -o stratum+tcp://backup.example.com:3333 -u WALLET.rig1
fearminer -a randomx -o stratum+ssl://pool.example.com:3335 -u WALLET.rig1 -t 50% --msr auto
fearminer -o pool.example.com:3333 -u WALLET -w rig1 -d 0,2
```

Every option can also be set as `FEARMINER_<OPTION>` in the environment, or in
a TOML file (`fearminer config example` prints one). The command line wins.

## 3. Everything else

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
| `--no-telemetry` | do not send the build-and-pool ping to `api.fearminer.com` |
| `--enroll <CODE>`, `--remote-relay <URL>`, `--remote-wallet-delay <MIN>` | join a cockpit's fleet at start (for a service, a HiveOS flight sheet or a script); a relay of your own before `wss://relay.fearminer.com`; the minutes a new wallet from a mining sheet waits (10 by default, 0 for none) |
| `--unrestricted-api <LEVEL>`, `--watch-config`, `--hook <EVENT:PATH>` | a running miner takes `pause`, `resume`, `toggle` and `retune` by default (the API, the cockpit keys `p` and `1`-`9`, `SIGUSR1`, or a file dropped in the state directory); `restart` and `stop` need `--unrestricted-api operate`. `SIGHUP`, `--watch-config` and `PUT /api/v1/config` reload the configuration without stopping the mining, validated whole before anything is applied. `--hook` runs a program of yours on one of ten events |
| `--history off`, `--history-retention <DAYS>`, `--history-max-size <MB>` | each rig keeps its own history in its state directory (10 s for a day, 1 min for a week, 5 min for a month, 1 h beyond, 90 days by default). `fearminer history` reads it offline, `GET /api/v1/history` serves it |
| `--background`, `--priority <0-5>` | run without a console (Unix; give `--log-file` with it), and the whole process's scheduling priority |
| `--no-tui`, `--no-color`, `-v` | plain log, no colour, debug log |

## Algorithms

| Algorithm | Coin | `-a` | Engines | Fee |
|---|---|---|---|---|
| Quantus, QPoW over Poseidon2 | QTC | `quantus` (also `qpow`, `qtc`) | CUDA, Vulkan, CPU | 2 % |
| RandomX (rx/0) | XMR | `randomx` (also `rx`, `monero`, `xmr`) | CPU | 0.85 % |
| VerusHash 2.2 | VRSC | `verushash` (also `verus`, `vrsc`) | CPU | 0.85 % |
| Pearl (pearlhash) | PRL | `pearl` (also `pearlhash`, `prl`) | CUDA, sm_86 only (RTX 3090, 3080 Ti) | 1 % |

RandomX wants about 2.3 GiB of RAM per rig (a 2080 MiB dataset, a 256 MiB
cache, 2 MiB of scratchpad per thread), plus huge pages and the MSR tweaks for
the full rate (the Linux archive's helper does both). VerusHash needs AES-NI
and CLMUL.

Pearl is a noised int8 matrix product mined on NVIDIA sm_86 cards only for now
(RTX 3090, 3080 Ti) with 4.7 GB free; no CPU engine (`-t` is refused), not on
macOS. Its rate is in T (10^12 multiply-accumulates a second), the unit its
pools credit. Every proof is verified twice before it is sent. The pool must
speak Pearl's `"type": "v2"` stratum and is your choice, as for every algorithm:
`fearminer -o stratum+ssl://POOL:PORT -u prl1... -w rig1`. The 1 % fee is a
ceiling; nothing is charged until the signed terms name a Pearl endpoint.

The wallet says which of them runs. Each address is checksum-verified at start;
`--ignore-wallet-check` sends what you typed.

| Chain | Wallet |
|---|---|
| QTC | an SS58 address of the Quantus network, prefix 189. A `+diff` suffix or a `solo:` prefix is passed to the pool as typed |
| XMR | 95 characters from `4` (standard) or `8` (subaddress), 106 from `4` (integrated). A testnet or stagenet address is refused by name |
| VRSC | `R...`, an identity address `i...`, or an identity name `name@` (`sub.name@` for a sub-identity), quoted when it holds spaces. A shielded `zs1...` is refused |
| PRL | `prl1...`, bech32m, checksum verified. The test networks' `tprl1...` and `rprl1...` are refused |

New algorithms arrive with their own launcher in the archive and their own row
above: nothing else in this file changes.

## Downloads

The current release is 1.4.1. Every version ships, on GitHub and at
[download.fearminer.com/latest/](https://download.fearminer.com/latest/)
(without the version in the file names there):

| File | For |
|---|---|
| `fearminer-<version>-windows-x86_64.zip` | Windows: `fearminer.exe`, one `start_<algo>.bat` per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer-<version>-linux-x86_64.tar.gz` | Linux: `fearminer`, `fearminer-helper`, one `start_<algo>.sh` per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer-<version>-macos-arm64.tar.gz` | macOS, Apple silicon: `fearminer` (native Metal), one `start_<algo>.sh` per algorithm, `readme.txt`, `openapi.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer_custom-<version>.tar.gz` | HiveOS custom miner package |
| `SHA256SUMS` | checksums of every file above |
| `SHA256SUMS.minisig` | signature of `SHA256SUMS` by FearMiner's release key (from 1.0.1) |
| `sbom.cdx.json` | every component inside the binary (CycloneDX), from 1.1.0 |

Each archive unpacks into a folder named like the archive. `fearminer-helper`
is the small privileged binary RandomX uses for the MSR tweaks and the huge
pages, through `sudo`; the miner itself never runs as root.

## HiveOS

Flight sheet: custom miner, the `fearminer_custom-<version>.tar.gz` asset's URL
as installation URL, the algorithm's HiveOS name (`qpow` for Quantus,
`randomx`, `verushash`, `pearlhash` for Pearl), wallet and pool as usual. `--enroll fm1_...` in the
extra config arguments adds the rig to your cockpit.

## Requirements

Three builds are published, and nothing else:

| Platform | |
|---|---|
| Linux | x86_64, built against glibc 2.35 (Ubuntu 22.04), so glibc 2.35 or newer. No ARM build |
| Windows | x86_64, cross-built with mingw-w64. No 32-bit build |
| macOS | Apple silicon only, built with the macOS 15.5 SDK. No Intel build |

For Quantus: an NVIDIA card of compute capability 7.0 or newer with 256 MB of
memory free, on a driver of the 550 series or newer (570+ for RTX 50). A card
under either floor is left out of the run with its code (`E310` for the memory,
`E308` for the driver). Other GPUs run on Vulkan at a fraction of the rate.
RandomX and VerusHash need no GPU at all. Pearl needs an NVIDIA sm_86 card
(RTX 3090, 3080 Ti) with 4.7 GB free.

Vulkan and Metal report no sensors, so there is no temperature, fan or power
reading on them and the thermal cut-off cannot act; it covers NVIDIA cards
through NVML and nothing else.

The miner refuses to run as root unless `--allow-root` (the HiveOS package
passes it).

From 1.1.1 every engine answers a known-answer test before the first share; a
card that answers wrong is left out, and GPU shares are re-checked on the CPU
before they are sent: every one while the check costs under 1 % of a core, one
in ten past that.

RandomX at full rate needs huge pages and the MSR tweaks, which need root. The
Linux archive ships `fearminer-helper`: the miner reaches it through `sudo -n`
before its first connection and never afterwards. It grows the huge-page pool,
applies the CPU's MSR preset (the `msr` kernel module, `modprobe msr`), puts
the original values back at exit and after a crash, and gives back the huge
pages a run reserved. Install it once:

```
sudo install -m 0755 fearminer-helper /usr/local/bin/
fearminer-helper install     # prints the sudoers line and the systemd drop-in
```

The miner looks for `/usr/local/bin/fearminer-helper` first, then for the copy
beside its own binary. `--helper PATH` names another place and is then the only
path tried. Without a helper the miner starts anyway and says what it costs.

Overclocking (core and memory clocks, power limit, fan) needs the NVIDIA driver
520 or newer for the clock offsets, and the miner as root (`--allow-root`).

## Fee

Per algorithm, listed by `--list-algorithms` and shown in the cockpit header.
It is mined in one-minute rounds, on a separate connection to the fee pool,
never on your session; the log says when a round starts and ends.

The rate printed is a ceiling built into the binary. The signed terms the miner
fetches can lower it, move it to another pool, or have it mined with another
algorithm of the same device class, never raise it. An algorithm the terms name
no fee pool for is mined with no fee. A higher rate takes a new release, and
this table changes with it. The miner is closed source at this stage.

## What the miner talks to

- Your pool, over `stratum+tcp` or `stratum+ssl`; through the SOCKS5 proxy when
  `--proxy` names one, which then resolves the pool names. The pool is never
  contacted directly while a proxy is configured. `--dns doh` resolves the
  names over DNS over HTTPS.
- The fee pool, during the fee rounds, through the same proxy.
- `cfg.fearminer.com`, for the signed fee terms: which pool the fee is mined on
  and the latest version. Read at start and every 20 minutes, directly, never
  through the proxy. Unreachable, it delays the start by five seconds at most,
  then the miner mines with the last terms it verified, or with the ones built
  in.
- `api.fearminer.com`, a ping with the build number and the pool address, at
  the same cadence, directly. No wallet, no worker name, nothing about the
  hardware. `--no-telemetry` (or `FEARMINER_NO_TELEMETRY=1`) turns it off; the
  terms are still fetched.
- `relay.fearminer.com`, once the rig is enrolled in a cockpit and not before:
  one outgoing connection (port 443) carrying sealed messages only your fleet
  reads, through the same proxy and resolver as the pools. `--remote-relay`
  puts a relay of your own first.

Nothing else. The `egress` line printed at start lists every host the miner
will talk to. The notifiers and the heartbeat reach only the URLs you give
them, directly; only the pool connections go through `--proxy`.

The stats endpoint listens on `127.0.0.1` unless `--api-bind` says otherwise.
Its three writes, `POST /api/v1/oc`, `POST /api/v1/commands` and
`PUT /api/v1/config`, need the token from anywhere (`fearminer token show`
prints it, `fearminer token rotate` replaces it), and what a command may do is
bounded by `--unrestricted-api`.

Outside its own folder the miner writes two caches: the GPU tuning result
(`~/.config/fearminer/tuning.json` on Linux, `~/Library/Caches/fearminer` on
macOS) and, under `~/.cache/fearminer/`, the last verified terms, the API
token, the crash counters, the watchdog log, this rig's history (`history.bin`,
bounded by `--history-max-size`), the certificates remembered by `--tls-tofu`
and what the cards read as before an overclock. Plus the log file when
`--log-file` asks for one. The helper keeps the original MSR values under
`/run/fearminer-helper/`, root-owned, until it puts them back.

## Verifying a download

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
that announces it here. Available from 1.1.0.

Without the binary, from 1.0.1:

```
minisign -Vm SHA256SUMS -P RWQR3owV+nRhfgNJeUBqcRK868S52NFG2BOrIzKpkQ5ey6lCEfM9+3Eg
sha256sum -c SHA256SUMS --ignore-missing
```

Signing the binaries themselves is planned, with no date promised: an
Authenticode certificate and an Apple developer account are a recurring bill
FearMiner pays out of what it earns rather than before. Until then the
signature that matters is the one on `SHA256SUMS`, which you can check
yourself, offline. The exact clicks per platform are at
[fearminer.com/docs](https://fearminer.com/docs/#unsigned).
