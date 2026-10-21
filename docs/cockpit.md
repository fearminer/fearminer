# Cockpit

**[app.fearminer.com](https://app.fearminer.com)**: the whole farm on one
screen, from your phone or your PC. Free, no account, end-to-end encrypted:
the relay in the middle passes sealed messages it cannot read, cannot add a rig
and cannot sign a command. Your fleet is 12 words, written down once.

- The fleet's hashrate and history, every rig and every card.
- Mining sheets: what each rig mines (the algorithm, the wallet, the pools),
  like a HiveOS flight sheet.
- Pause, resume and restart.
- A new wallet applies at once by default; a rig that sets
  `--remote-wallet-delay` makes it wait, announced everywhere and cancellable
  from any cockpit or with `fearminer remote cancel`.
- HiveOS and mmpOS rigs are watched and commanded; their flight sheet or
  miner profile decides what they mine.
- A new release updates one rig or the whole fleet from the cockpit: the
  rig's service checks the release's signature itself, installs it, and puts
  the previous version back on its own if the new one does not come up
  (Linux services; HiveOS and mmpOS update their miner themselves).

## Add a rig

The cockpit's *Add a rig* gives the one line to paste on the rig
([install.md](install.md#one-line)). A rig that already runs FearMiner joins
with `fearminer enroll fm1_...`, or `--enroll fm1_...` on its command line,
in a service, a HiveOS flight sheet or a script.

| Command | |
|---|---|
| `fearminer enroll fm1_...` | join your cockpit's fleet, no restart; `--status`, `--leave` |
| `fearminer remote status`, `off`, `on`, `cancel` | the remote channel on this machine; `off` refuses every remote command until `on` |
| `--remote-relay <URL>` | a relay of your own before `wss://relay.fearminer.com` |
| `--remote-wallet-delay <MIN>` | the minutes a new wallet from a mining sheet waits (0 by default: at once) |

The rig talks to `relay.fearminer.com` only once it is enrolled
([security.md](security.md#what-the-miner-talks-to)).
