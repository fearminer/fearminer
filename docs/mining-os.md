# HiveOS and mmpOS

## HiveOS

In the Flight Sheet, pick the coin, the wallet and the pool, then the miner
**Custom** and **Setup Miner Config**:

| Field | Value |
|---|---|
| Miner name | filled in from the installation URL: `fearminer_custom` |
| Installation URL | `https://github.com/fearminer/fearminer/releases/download/v<version>/fearminer_custom-<version>.tar.gz`, the release asset as it is |
| Hash algorithm | the coin's HiveOS name below (`hive` in `fearminer --list-algorithms --json`) |
| Wallet and worker template | `%WAL%.%WORKER_NAME%` |
| Pool URL | as below |
| Pass | `x`, or what the pool asks |
| Extra config arguments | any FearMiner option: `-d 0,2`, `-t 0`, `--tls-fingerprint <sha256>`, `--enroll fm1_...` (adds the rig to your cockpit), the coin's own options below |

HiveOS reads the miner name and the version off the file name, split at its
last dash, so the URL must be the versioned asset: the `latest/download`
alias or a renamed file installs under another name, and the Flight Sheet
fails with `Custom miner name should be ...`. A new version is a new URL.

| Coin | Hash algorithm | Pool URL | Runs on | Extra arguments |
|---|---|---|---|---|
| QTC | `qpow` | `stratum+ssl://host:port` | NVIDIA, CPU with `-t N` | |
| XMR | `randomx` | `stratum+tcp://host:port` or `stratum+ssl://host:port` | CPU | `-t N` |
| PRL | `pearlhash` | `stratum+ssl://host:port` | NVIDIA sm_86, sm_89, sm_120 | |
| TSC | `tensorcash` | `wss://host/v1/ws` | NVIDIA, 16 GB of RAM, about 11 GB of VRAM | `--tsc-precision gptq`, `gptq-ilv`, `q8` or `bf16`; a pool that asks for a token: `-p token-file=/path` |
| CMFD | `commonfoundry` | `cmfd+tls://IPv4:port?pin=<64 hex digits>`, an address and the pool's certificate pin, never a name | NVIDIA, 8 GB of RAM, about 7 GB of VRAM | |
| VRL (testnet) | `vearl` | `stratum+ssl://host:port` | NVIDIA from the RTX 30 series, CPU with `-t N` | |

The image needs glibc 2.35 or newer (HiveOS on Ubuntu 22.04; `ldd --version`
says which): an older image refuses the binary with `GLIBC_2.3x not
found`. The 550 driver runs the CUDA 12.4 engines the package carries;
TensorCash needs the 575 driver or newer. An
algorithm that mines with a model downloads it at its first start, several
GB, into `/root/.cache/fearminer`. The miner's log is on its screen,
`screen -r miner` (Ctrl+A then D to leave it running).

## mmpOS (beta)

**Beta.** Built to mmpOS's published custom-miner guide and checked against
it, not yet run on a real mmpOS rig: say what does not work, on the
[Telegram channel](https://t.me/FearMiner) or in an issue.

Miner profile: the miner `Custom miner`, the `fearminer_mmpos-<version>.tar.gz`
asset's URL as its download URL (a new version is a new URL: mmpOS fetches a
URL once), the coin by its ticker (`QTC`, `XMR`, `PRL`, `TSC`, `CMFD`: a name `fearminer --list-algorithms` gives), and the arguments
`./mmp-launch.sh --coin %coin% %pool_protocol% --pool %pool_server%:%pool_port% --user %user% --password %password% --api-port %api_port%`,
then any FearMiner option: `--enroll fm1_...` adds the rig to your cockpit.
