# Install

## One line

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
| HiveOS | nothing to install: `--enroll fm1_...` in the flight sheet's extra config arguments; the flight sheet keeps deciding what the rig mines ([mining-os.md](mining-os.md)) |
| mmpOS (beta) | the mmpOS package in a miner profile ([mining-os.md](mining-os.md)), `--enroll fm1_...` after its arguments; the profile keeps deciding what the rig mines |

The check: `SHA256SUMS` against the release key (minisign, or OpenSSL 3),
then the archive against `SHA256SUMS`; without either tool,
download.fearminer.com and GitHub must agree (on Windows, always that). Nothing
runs before. The service is your account's (no password); `--system` installs
the machine's, with sudo. With neither a cockpit nor a wallet, the rig watches
its cards and waits.

## The service

| Command | |
|---|---|
| `fearminer service install [OPTIONS]` | a run made permanent: the same options as a run (`fearminer service install -o stratum+ssl://POOL:PORT -u YOUR_WALLET -w rig1`); systemd on Linux, launchd on macOS, a task at logon on Windows, a background process in a container without systemd (Vast.ai, RunPod: `~/.local/bin/fearminer service start` in its start script) |
| `fearminer service status`, `logs`, `stop`, `start`, `restart`, `uninstall` | as named; `uninstall` keeps the settings, the history and the keys |
| `fearminer service update [VERSION]`, `fearminer service rollback` | the same update as the cockpit's, from the machine: the latest release by default, its signature checked, the previous version back if the new one does not come up; `rollback` puts the previous version back (Linux) |

## Download an archive

The same files are at
[download.fearminer.com/latest/](https://download.fearminer.com/latest/)
(without the version in the file names there) and on
[GitHub](https://github.com/fearminer/fearminer/releases/latest).

| File | For |
|---|---|
| `fearminer-<version>-windows-x86_64.zip` | Windows: `fearminer.exe`, one `start_<algo>.bat` per algorithm, `readme.txt`, `openapi.json`, `telemetry-schema.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer-<version>-linux-x86_64.tar.gz` | Linux: `fearminer`, `fearminer-helper`, one `start_<algo>.sh` per algorithm, `readme.txt`, `openapi.json`, `telemetry-schema.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer-<version>-macos-arm64.tar.gz` | macOS, Apple silicon: `fearminer` (native Metal), one `start_<algo>.sh` per algorithm, `readme.txt`, `openapi.json`, `telemetry-schema.json`, `THIRD-PARTY-NOTICES.txt` |
| `fearminer_custom-<version>.tar.gz` | HiveOS custom miner package |
| `fearminer_mmpos-<version>.tar.gz` | mmpOS custom miner package (beta) |
| `SHA256SUMS` | checksums of every file above |
| `SHA256SUMS.minisig` | signature of `SHA256SUMS` by FearMiner's release key (from 1.0.1) |
| `sbom.cdx.json` | every component inside the binary (CycloneDX), from 1.1.0 |

Each archive unpacks into a folder named like the archive. `fearminer-helper`
is the small privileged binary RandomX uses for the MSR tweaks and the huge
pages, through `sudo`; the miner itself never runs as root
([algorithms.md](algorithms.md#the-randomx-helper)).

`YOUR_WALLET` is a public receiving address from a wallet app or an exchange
account. Never a private key, a seed phrase or a password. The chain is read
off the address, so the wallet picks the algorithm;
[algorithms.md](algorithms.md#wallets) gives the shape of each address. The
pool is yours to choose: its host and port are on the pool's own page, and
FearMiner never picks one for you.

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
Or open `start_quantus.sh` (or `start_randomx`, `start_pearl`; `.bat` on Windows) in a text editor, set `WALLET`,
`POOL` and `WORKER`, and run it.

One line per coin, from the unpacked folder. Common Foundry addresses do not
name their chain, so give `-a commonfoundry`:

```
./fearminer -a quantus       -o stratum+ssl://POOL:PORT -u qz...   -w rig1
./fearminer -a randomx       -o stratum+ssl://POOL:PORT -u 4...    -w rig1 -t 50%
./fearminer -a pearl         -o stratum+ssl://POOL:PORT -u prl1... -w rig1
./fearminer -a tensorcash    -o wss://HOST/v1/ws        -u tc1...  -w rig1
./fearminer -a commonfoundry -o "cmfd+tls://IP:PORT?pin=PIN" -u 64_HEX_DIGITS -w rig1
./fearminer -a vearl         -o stratum+ssl://POOL:PORT -u vrl1... -w rig1
```

A wallet always comes with its pool: a wallet without one is refused before
anything is sent (`E224`, exit code 2). The chain is read off the address when
`-a` is left out. Without a wallet and a pool the miner watches the devices
instead of mining.

## First run on Windows and macOS

The binaries are not code-signed yet, and scanners file any miner under
`HackTool` or `CoinMiner`. Check the download first
([security.md](security.md#verify-a-download)), then:

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

Signing the binaries themselves is planned, with no date promised: an
Authenticode certificate and an Apple developer account are a recurring bill
FearMiner pays out of what it earns rather than before. Until then the
signature that matters is the one on `SHA256SUMS`, which you can check
yourself, offline. The exact clicks per platform are at
[fearminer.com/docs](https://fearminer.com/docs/#unsigned).

## Is it working?

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

## Getting paid

An accepted share is not a payment. The `pool` line at start names the pool
the miner is on, the one you gave with `-o`. Your balance lives on that
pool's own dashboard, under the address you gave, and it pays at the
threshold and on the schedule that pool sets, minus that pool's fee. Those
are the pool's, not FearMiner's; FearMiner's own fee is separate and shown in
the cockpit header.
