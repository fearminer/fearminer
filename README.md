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

More are on the way; each arrives with its own launcher in the archive and
its own line here.

## Downloads

This repository carries the **releases**. Every version ships:

| File | For |
|---|---|
| `fearminer-<version>-windows-x86_64.zip` | Windows: `fearminer.exe`, one `fearminer-<algo>.bat` launcher per algorithm |
| `fearminer-<version>-linux-x86_64.tar.gz` | Linux: `fearminer/fearminer`, one `fearminer-<algo>.sh` launcher per algorithm |
| `fearminer-<version>.tar.gz` | HiveOS custom miner package |
| `SHA256SUMS` | checksums of every file above |

The same files are served from <https://download.fearminer.com/>.

## Quick start

Unpack, open the launcher of the algorithm you mine (`fearminer-quantus.bat`
on Windows, `fearminer-quantus.sh` on Linux) in a text editor, set `WALLET`
to your address and `WORKER` to a name for the machine, run it. Or by hand:

```
fearminer -a quantus -o stratum+ssl://pool.example.com:3335 -u WALLET.rig1
fearminer -a quantus -o pool.example.com:3334 -u WALLET -w rig1        (plain TCP)
fearminer --list-algorithms
fearminer --list-devices
fearminer --help
```

| Option | |
|---|---|
| `-a, --algo <ALGO>` | what to mine (`--list-algorithms`) |
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
miner, install URL above, the algorithm's HiveOS name (`qpow` for Quantus),
wallet and pool as usual.

## Requirements

An NVIDIA GPU with a driver of the 550 series or newer (570+ for RTX 50).
Without an NVIDIA driver the miner runs on Vulkan at a fraction of the rate;
CPU-only algorithms need no GPU at all.

## Fee

Per algorithm, listed by `--list-algorithms` and shown in the header of the
cockpit; mined in one-minute rounds. The miner is closed source at this stage.

## Verifying a download

```
sha256sum -c SHA256SUMS --ignore-missing
```
