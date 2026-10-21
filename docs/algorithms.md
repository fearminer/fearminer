# Algorithms

| Algorithm | Coin | `-a` | Engines | Fee |
|---|---|---|---|--:|
| Quantus, QPoW over Poseidon2 | QTC | `quantus` (also `qpow`, `qtc`) | CUDA, Metal, Vulkan, CPU | 2 % |
| RandomX (rx/0, rx/2) | XMR | `randomx` (also `rx`, `rx/2`, `monero`, `xmr`) | CPU | 0.85 % |
| Pearl (pearlhash) | PRL | `pearl` (also `pearlhash`, `prl`) | CUDA (sm_86, sm_89, sm_120) | 2 % |
| TensorCash, proof of inference | TSC | `tensorcash` (also `tsc`, `poi`) | CUDA, a large NVIDIA card | 2 % |
| Common Foundry, ProductionV4 | CMFD | `commonfoundry` (also `cmfd`, `productionv4`) | CUDA, NVIDIA | 3 % |

`fearminer --list-algorithms` prints the same list, from the signed engine
index; `--json` adds the requirements and the HiveOS names.

RandomX wants about 2.3 GiB of RAM per rig (a 2080 MiB dataset, a 256 MiB
cache, 2 MiB of scratchpad per thread), plus huge pages and the MSR tweaks for
the full rate (the Linux archive's helper does both, see
[below](#the-randomx-helper)).

Pearl is a noised int8 matrix product mined on NVIDIA cards (sm_86, sm_89,
sm_120) with about 4.7 GB free; no CPU engine (`-t` is refused), not on
macOS. Its rate is in T (10^12 multiply-accumulates a second), the unit its
pools credit. Every proof is verified twice before it is sent. The pool must
speak Pearl's `"type": "v2"` stratum and is your choice, as for every algorithm:
`fearminer -o stratum+ssl://POOL:PORT -u prl1... -w rig1`.

TensorCash (TSC) is a proof of inference on a large NVIDIA card: its pool is a
WebSocket broker (`-o wss://HOST/v1/ws`), and the engine downloads its model on
the first start. Common Foundry (CMFD) is a proof of useful work on NVIDIA:
its pool is an IPv4 address and the pin of its certificate
(`-o "cmfd+tls://IP:PORT?pin=PIN"`), and the engine downloads its model bank
on the first start. Both are NVIDIA-only, with no CPU engine.

## Wallets

The wallet says which of them runs. Each address is checksum-verified at start;
`--ignore-wallet-check` sends what you typed.

| Chain | Wallet |
|---|---|
| QTC | an SS58 address of the Quantus network, prefix 189. A `+diff` suffix or a `solo:` prefix is passed to the pool as typed |
| XMR | 95 characters from `4` (standard) or `8` (subaddress), 106 from `4` (integrated). A testnet or stagenet address is refused by name |
| PRL | `prl1...`, bech32m, checksum verified. The test networks' `tprl1...` and `rprl1...` are refused |
| TSC | `tc1q...`, the TensorCash address |
| CMFD | 64 hexadecimal characters (an Ed25519 public key); give `-a commonfoundry`, the address alone does not name the chain |

## Fee

FearMiner charges a dev fee per algorithm, the rate in the table above. It is
mined in one-minute rounds on a separate connection, never on your session,
and shown in the cockpit header. TensorCash is the exception: its engine
mines the fee itself, on the same pool. The rate is a ceiling from the signed engine
index: the signed terms from `cfg.fearminer.com` may lower it, never raise
it. An algorithm with no fee pool in the terms is mined free, said once at
start. The miner is closed source at this stage.

## Engines

New algorithms arrive by themselves: each one is a signed engine the miner
downloads and keeps up to date. The miner hashes nothing itself: each
algorithm is an engine of its own, `fearminer-engine-<algo>`, published at
`download.fearminer.com/algos/` and listed in a signed `index.json` (names,
coin, unit, requirements, versions, each build's SHA-256).

| | |
|---|---|
| checked | the index against the algorithms key compiled into the miner (`E321`), each engine against the index's SHA-256 (`E323`), before every start |
| cache | `<state dir>/engines/`: the index and the two latest versions of each algorithm |
| offline | the index cannot be fetched (`E322`): the cached one, else the seed in the release archive's `engines/` folder, so a first start without network mines too |
| updates | every hour; a newer engine is checked, started beside the running one, self-tested, then takes over between two jobs (`E326`) |
| rollback | a new engine that fails its self-test or mines 5 % slower over ten minutes is rolled back until the next index (`E327`) |
| pinning | `--engine quantus=1.0.6` (repeatable, `FEARMINER_ENGINE`, or `engine` in the configuration file): that version runs, no update moves it |
| no engine | no index anywhere or no build for this machine (`E324`), or a login family this miner does not speak (`E325`): it mines nothing and watches the cards |
| crash | an engine that dies is restarted, its jobs handed back (`E320`) |

A new algorithm of a login family the miner speaks is mined without a miner
release. Codes `E320` to `E329`: `fearminer explain E324`.

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
RandomX needs no GPU at all. Pearl needs an NVIDIA card (sm_86, sm_89,
sm_120) with about 4.7 GB free. TensorCash needs a large NVIDIA card (about
11 GB free), a driver of the 575 series or newer and 16 GB of system RAM,
Common Foundry an NVIDIA card (about 7 GB free) and 8 GB of system RAM.
Pearl, TensorCash and Common Foundry run on Linux and Windows.

Vulkan and Metal report no sensors, so there is no temperature, fan or power
reading on them and the thermal cut-off cannot act; it covers NVIDIA cards
through NVML and nothing else.

The miner refuses to run as root unless `--allow-root` (the HiveOS package
passes it).

From 1.1.1 every engine answers a known-answer test before the first share; a
card that answers wrong is left out, and GPU shares are re-checked on the CPU
before they are sent: every one while the check costs under 1 % of a core, one
in ten past that.

Overclocking (core and memory clocks, power limit, fan) needs the NVIDIA driver
520 or newer for the clock offsets, and the miner as root (`--allow-root`).

## The RandomX helper

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
