# FearMiner

GPU miner for Quantus (QTC), for any stratum pool. Native CUDA kernels on
NVIDIA cards, a Vulkan fallback elsewhere, a CPU engine, and a full-screen
cockpit that tells you what the rig is doing.

This repository carries the **releases**. Every version ships:

| File | For |
|---|---|
| `fearminer-<version>-windows-x86_64.zip` | Windows: `fearminer.exe`, `fearminer-quantus.bat` to edit and run |
| `fearminer-<version>-linux-x86_64.tar.gz` | Linux: `fearminer/fearminer`, `fearminer-quantus.sh` to edit and run |
| `fearminer-<version>.tar.gz` | HiveOS custom miner package |
| `SHA256SUMS` | checksums of every file above |

The same files are served from <https://download.fearminer.com/>.

## Quick start

Unpack, open `fearminer-quantus.bat` (Windows) or `fearminer-quantus.sh`
(Linux) in a text editor, set `WALLET` to your Quantus address and `WORKER`
to a name for the machine, run it. Or by hand:

```
fearminer -a quantus -o stratum+ssl://qtc.kryptex.network:8049 -u WALLET.rig1
fearminer -a quantus -o pool.example.com:3334 -u WALLET -w rig1        (plain TCP)
fearminer --list-devices
fearminer --list-algorithms
fearminer --help
```

| Option | |
|---|---|
| `-a, --algo <ALGO>` | what to mine (`--list-algorithms`); `quantus` by default |
| `-o, --url <URL>` | `stratum+tcp://host:port`, `stratum+ssl://host:port`, or `host:port` |
| `-u, --user <WALLET[.WORKER]>` | wallet address, with an optional worker name |
| `-w, --worker <NAME>` | worker name, appended to `--user` when it has none |
| `-p, --pass <PASS>` | pool password, `x` by default |
| `--tls-fingerprint <SHA256>` | pin a self-signed pool certificate |
| `-d, --devices <LIST>` | GPUs to mine on, by index |
| `-t, --threads <N>` | CPU threads (0 by default on a rig with a GPU) |
| `--api-bind <IP:PORT>` | stats endpoint (`/stats`, `/hive-stats`), `0.0.0.0:4300` by default; `--no-api` turns it off |
| `--no-tui`, `--no-color`, `-v` | plain log, no colour, debug log |

Every option can also be set from the environment as `FEARMINER_<OPTION>`.

## HiveOS

Custom miner package: `https://download.fearminer.com/fearminer-hive.tar.gz`,
or the `fearminer-<version>.tar.gz` asset of a release. Flight sheet: custom
miner, install URL above, algorithm `qpow`, wallet and pool as usual.

## Requirements

An NVIDIA GPU with a driver of the 550 series or newer (570+ for RTX 50).
Without an NVIDIA driver the miner runs on Vulkan at a fraction of the rate.

## Fee

2 % of mining time, in one-minute rounds, shown in the header of the cockpit
and in `--list-algorithms`. The miner is closed source at this stage.

## Verifying a download

```
sha256sum -c SHA256SUMS --ignore-missing
```
