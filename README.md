# LarzOS

**A Debian-based Linux distribution with a money-native scripting language,
Larzscript, as its native tongue — the config engine, the shell, the
package tool and the AI router are all `.lz` files, and spending is a
language-level idea rather than a library bolted on after the fact.**

Official site: <https://larzos.com/larzos-linux/>
This repo is a personal showcase of the OS's niche — not the OS source tree.

## The niche

Most distros treat "how much did this script spend on an API this month"
as an application concern, if they think about it at all. LarzOS treats it
as an OS concern:

- `wallet`, `price`, `to`, `pay` and `require` are **reserved words** in
  Larzscript — money is a first-class citizen of the language, not an SDK.
- Every machine has a wallet (`larz wallet`), monthly spend caps per
  category (`larz budget`), and an append-only, receipted ledger for every
  debit and credit (`larz spend`).
- The AI router (`larz-aid`) checks the wallet's budget **before** every
  gateway completion and meters actual token usage back into it after —
  see [`examples/budget_guard.lz`](examples/budget_guard.lz).
- The whole machine — hostname, users, packages, services, AI routing,
  spend caps — is one declarative `system.lz` spec, applied like a NixOS
  generation: `larz plan` → `sudo larz apply` → `larz rollback` to undo
  instantly. See [`examples/system.lz`](examples/system.lz).

## The `larz` CLI

```
  software
    larz install|remove|search <pkg>     larz update      larz upgrade      larz list
  this machine
    larz plan [system.lz]                larz apply       larz rollback     larz status
    larz snapshot [name]                 larz generations larz switch <n>   larz diff [<n>]
  money
    larz wallet                          larz pay <who> <amount>            larz topup <amount>
    larz budget [<category> <amount>]    larz spend [category]
  ai
    larz ai <text>                       larz do "<goal>"   larz explain [file]
    larz why <topic>                     larz ask <question>
    larz code [args]                     Claude Code, the terminal coding agent
  fleet
    larz fleet [add|rm <name> <target>]  larz fleet apply|run <...>
    larz share                           larz adopt <id|url>
  other
    larz doctor    larz rebrand [status]    larz version    larz apt <args>
```

`larz doctor` gives a one-screen health check: identity, engine version,
whether the AI router is running, apt sources, wallet balance and monthly
AI spend, spec generations, disk, and failed systemd units.

## Larzscript

See [`docs/language.md`](docs/language.md) for a quick tour of the
language itself — functions, control flow, the standard library, and the
`larzos/wallet` module that every money-aware tool builds on.

```larzscript
import "larzos/wallet" as wal

let check = wal.check("ai", 0.02)
if check["ok"] {
  wal.debit("ai", 0.02, "drlarz-fast", "one completion")
} else {
  print("blocked: " + check["reason"])
}
```

## Examples

| File | Shows |
|---|---|
| [`examples/hello.lz`](examples/hello.lz) | The language basics |
| [`examples/budget_guard.lz`](examples/budget_guard.lz) | Check-then-spend against a wallet budget, the pattern behind `larz-aid` |
| [`examples/pay.lz`](examples/pay.lz) | Sending money as a one-line, language-level operation |
| [`examples/system.lz`](examples/system.lz) | A whole machine — packages, services, spend caps — as one declarative spec |

Run any of them with `larzscript examples/<file>.lz` on a LarzOS box.
`budget_guard.lz` and `pay.lz` use the `larzos/wallet` standard module, so
run those two with the OS library on the path:
`LARZSCRIPT_PATH=/usr/lib/larzos larzscript examples/budget_guard.lz`.

## Getting LarzOS

Install and image instructions live at
<https://larzos.com/larzos-linux/>. On an existing box:

```sh
larz doctor      # health check
larz --help      # full command reference
```
