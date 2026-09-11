# Larzscript, quickly

Larzscript is the language LarzOS itself is written in — the config engine
(`larz-system`), the login shell (`larzsh`), the package tool (`larz-pkg`),
the AI router (`larz-aid`) and the `larz` CLI are all `.lz` files. There's no
separate "OS layer" in a different language underneath the scripting layer.

## Basics

```larzscript
import "json" as json
import "larzos/wallet" as wal

let name = "world"

fn greet(who) {
  return "hello, " + who
}

if name == "world" {
  print(greet(name))
} else if name == "" {
  print("who?")
} else {
  print(greet(name) + "!")
}

for line in read_file("/etc/hostname").strip().split("\n") {
  print(line)
}
```

- `fn name(args) { ... }`, `let x = ...`, `import "mod" as m`
- `run(cmd)` spawns a command and inherits stdio (returns exit code);
  `capture(cmd)` runs it and returns stdout as a string
- dicts (`{"k": v}`), lists (`[1, 2, 3]`), `d.has(k)`, `d.get(k, default)`,
  `keys(d)`, `len(x)`, `type(x)`
- strings: `.strip()`, `.split(sep)`, `.replace(a, b)`, `.contains(s)`,
  `.starts_with(s)`, slicing `s[0:60]`
- ternary: `cond ? a : b`
- `env(name, default)`, `file_exists(path)`, `read_file(path)`,
  `write_file(path, content)`, `exit(code)`, `args` (CLI argv)

## The money-native part

`wallet`, `price`, `to`, `pay` and `require` are **reserved words** — you
cannot use them as variable names, because the language treats spending as
a first-class thing every program might do, not an API some programs
happen to call. In practice, day to day, that shows up as the
`larzos/wallet` standard module:

```larzscript
import "larzos/wallet" as wal

let check = wal.check("ai", 0.02)      # would this spend breach the budget?
if check["ok"] {
  wal.debit("ai", 0.02, "drlarz-fast", "one completion")
} else {
  print("blocked: " + check["reason"])
}

wal.pay("alice", 0.05, "thanks!")      # send money - ledger entry today,
                                        # a real payout once a provider
                                        # (cryptolarz / larzpay) is wired in
```

Every spend is capped by `/etc/larzos/wallet.toml` + `/etc/larzos/budget.toml`
(both edited declaratively via `system.lz`, see [`examples/system.lz`](../examples/system.lz)),
logged to an append-only ledger, and receipted to a human-readable file —
so "how much did this script spend this month" is always a real, auditable
answer, not a support ticket.

## Running things

```sh
larzscript file.lz              # run a program
larzscript -e "print(1 + 1)"    # run a snippet
larzscript repl                 # interactive REPL
larzscript --check file.lz      # syntax-check (for editors / CI)
larzscript fmt file.lz          # canonical formatting
larzscript --emit-c file.lz     # compile to C
```

Full docs: <https://larzos.com/larzos-linux/>
