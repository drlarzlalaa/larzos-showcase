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

## More from this account

The same author's other repositories. Larzscript is the language LarzOS is
written in, so most of these are written in it too; the last group is
ordinary Python, for the jobs where its ecosystem is the right tool. All
100 are MIT-licensed and have a test suite that runs in CI. The newer ones also
spell out what they do **not** do, and most compare their output with an
independent Python implementation or with published reference data before the
tests were written.

### LarzOS wallet tools (Larzscript)

| Repo | What it does |
|---|---|
| [`larz-meter`](https://github.com/drlarzlalaa/larz-meter) | Wraps any command with wallet metering: check the budget, run it, debit only on success |
| [`larz-budget-watch`](https://github.com/drlarzlalaa/larz-budget-watch) | Warns before a wallet budget is blown, with Nagios-style exit codes for cron |
| [`larz-receipts`](https://github.com/drlarzlalaa/larz-receipts) | Renders the wallet ledger as a report: by month, by category, or as JSON |

### General-purpose tools (Larzscript)

| Repo | What it does |
|---|---|
| [`larz-cronlint`](https://github.com/drlarzlalaa/larz-cronlint) | Expands a crontab for one day and shows which minutes many jobs start at once |
| [`larz-dupes`](https://github.com/drlarzlalaa/larz-dupes) | Finds duplicate files by content, read-only: groups by size, then SHA-256, and totals the wasted space |
| [`larz-diff`](https://github.com/drlarzlalaa/larz-diff) | A line diff with the shortest possible edit script; unified output that `patch` can apply, plus edit distance |
| [`larz-calc`](https://github.com/drlarzlalaa/larz-calc) | A calculator with real operator precedence (shunting-yard), fractional powers and scientific notation; checked against Python's arithmetic |
| [`larz-hash`](https://github.com/drlarzlalaa/larz-hash) | SHA-256, SHA-1, MD5, CRC-32 and Adler-32 in pure Larzscript (no bit operators); identical to hashlib and zlib on 725+ digests |
| [`larz-encode`](https://github.com/drlarzlalaa/larz-encode) | Base64, Base64url, Base32, hex and URL encoding with strict decoding; identical to Python on 780+ lines |
| [`larz-date`](https://github.com/drlarzlalaa/larz-date) | Calendar arithmetic in whole numbers: weekdays, date differences, adding days, calendars and Easter; identical to Python's datetime on 18,000+ operations |
| [`larz-csvstat`](https://github.com/drlarzlalaa/larz-csvstat) | Column summaries, frequency tables and text histograms for any CSV file |
| [`larz-logstat`](https://github.com/drlarzlalaa/larz-logstat) | Visitors, top paths, hourly load, errors and bots from Apache/Nginx access logs |
| [`larz-textstat`](https://github.com/drlarzlalaa/larz-textstat) | Word counts, Flesch readability, reading time, top words and longest sentences |

### Science and mathematics projects (Larzscript)

Each one is a single `.lz` file with tests and a README built from real
program output. Several are teaching tools, and say so.

| Repo | What it does |
|---|---|
| [`larzscript-superpowers`](https://github.com/drlarzlalaa/larzscript-superpowers) | What physics says about tunnelling through walls, time travel and levitation, plus the GPS check |
| [`larzscript-worldweight`](https://github.com/drlarzlalaa/larzscript-worldweight) | Weight, jumps and orbits on 17 Solar System bodies, checked against NASA's fact sheet |
| [`larzscript-moonphase`](https://github.com/drlarzlalaa/larzscript-moonphase) | Lunar phases to the minute (Meeus), checked against eclipse times |
| [`larzscript-transferwindow`](https://github.com/drlarzlalaa/larzscript-transferwindow) | An interplanetary launch-window planner: Hohmann transfers and a real ephemeris |
| [`larzscript-integrators`](https://github.com/drlarzlalaa/larzscript-integrators) | Euler, leapfrog and RK4 on an orbit: which conserves energy, which keeps time |
| [`larzscript-projectile`](https://github.com/drlarzlalaa/larzscript-projectile) | A thrown object with and without air resistance: the exact vacuum formulas, then drag and the best launch angle |
| [`larzscript-queue`](https://github.com/drlarzlalaa/larzscript-queue) | M/M/c queues: exact Erlang C waiting times, a customer-by-customer simulation, and the hockey-stick effect near full load |
| [`larzscript-elo`](https://github.com/drlarzlalaa/larzscript-elo) | The Elo rating system plus a tournament simulation: how well ratings recover hidden true strength, and the K trade-off |
| [`larzscript-birthday`](https://github.com/drlarzlalaa/larzscript-birthday) | The birthday paradox: exact chances, a simulation, and how many items until a collision is likely for any number of slots |
| [`larzscript-pendulum`](https://github.com/drlarzlalaa/larzscript-pendulum) | A pendulum's period against its amplitude: the exact formula and a simulation agree, the textbook one does not |
| [`larzscript-heat`](https://github.com/drlarzlalaa/larzscript-heat) | Heat diffusion: the explicit scheme blows up past r = 1/2, Crank-Nicolson stays bounded but not always accurate |
| [`larzscript-lorenz`](https://github.com/drlarzlalaa/larzscript-lorenz) | Deterministic chaos: the Lorenz system, the butterfly effect and the Lyapunov exponent |
| [`larzscript-fourier`](https://github.com/drlarzlalaa/larzscript-fourier) | The fast Fourier transform: spectra, spectral leakage (rectangular vs Hann window), Parseval's theorem, DFT vs FFT cost |
| [`larzscript-halflife`](https://github.com/drlarzlalaa/larzscript-halflife) | Radioactive decay, radiocarbon-style ages and decay chains to secular equilibrium |
| [`larzscript-hyperspace`](https://github.com/drlarzlalaa/larzscript-hyperspace) | The geometry of higher dimensions: ball volumes, n-cubes and a rotating tesseract |
| [`larzscript-neuron`](https://github.com/drlarzlalaa/larzscript-neuron) | A spiking-neuron lab: Hodgkin-Huxley action potentials and Izhikevich firing patterns |
| [`larzscript-connectome`](https://github.com/drlarzlalaa/larzscript-connectome) | The real C. elegans wiring diagram: hubs, touch-reflex circuits, signal spread |
| [`larzscript-wormsim`](https://github.com/drlarzlalaa/larzscript-wormsim) | Can the wiring diagram alone predict the touch reflex? An honest negative result |
| [`larzscript-genelab`](https://github.com/drlarzlalaa/larzscript-genelab) | The genetic code and point mutations, on the real HBB gene and the sickle-cell variant |
| [`larzscript-allelefreq`](https://github.com/drlarzlalaa/larzscript-allelefreq) | Hardy-Weinberg, carrier frequencies and genetic drift in small populations |
| [`larzscript-epidemic`](https://github.com/drlarzlalaa/larzscript-epidemic) | The SIR epidemic model, checked against its exact peak and final-size solutions |
| [`larzscript-montecarlo`](https://github.com/drlarzlalaa/larzscript-montecarlo) | Estimating pi, the 1/sqrt(N) error law, the birthday problem and Monty Hall against their exact answers |
| [`larzscript-bayes`](https://github.com/drlarzlalaa/larzscript-bayes) | Why a positive test is usually less alarming than it sounds: Bayes' rule, a simulated population and repeat tests |
| [`larzscript-knapsack`](https://github.com/drlarzlalaa/larzscript-knapsack) | The knapsack problem: dynamic programming vs brute force, and how often greedy loses |
| [`larzscript-gambler`](https://github.com/drlarzlalaa/larzscript-gambler) | The gambler's ruin problem: exact formulas vs simulation, and why a small roulette edge makes doubling your money unlikely |
| [`larzscript-secretary`](https://github.com/drlarzlalaa/larzscript-secretary) | The secretary problem: reject the first 37% and the chance of hiring the best tends to 1/e, exact and simulated |
| [`larzscript-markov`](https://github.com/drlarzlalaa/larzscript-markov) | Markov chains: stationary distributions by iteration and by simulation, and the chains that never settle |
| [`larzscript-pagerank`](https://github.com/drlarzlalaa/larzscript-pagerank) | PageRank on a small link graph, checked against an exact linear-system solution |
| [`larzscript-paths`](https://github.com/drlarzlalaa/larzscript-paths) | Shortest paths: Dijkstra and Bellman-Ford with negative cycles, checked against Floyd-Warshall and a path verifier |
| [`larzscript-cipher`](https://github.com/drlarzlalaa/larzscript-cipher) | The Caesar and Vigenère ciphers and how to break them (chi-squared, index of coincidence); a history lesson, not security |
| [`larzscript-kmeans`](https://github.com/drlarzlalaa/larzscript-kmeans) | k-means clustering with a k-means++ start and the elbow method, with ASCII plots; identical to a Python reference on 64 settings |
| [`larzscript-regex`](https://github.com/drlarzlalaa/larzscript-regex) | A regular-expression engine (Thompson NFA, no backtracking blow-up), checked against Python's `re` on over 900 pattern/text pairs |
| [`larzscript-sudoku`](https://github.com/drlarzlalaa/larzscript-sudoku) | A Sudoku solver and uniqueness checker (fewest-candidates-first search), checked against an independent Python solver |
| [`larzscript-queens`](https://github.com/drlarzlalaa/larzscript-queens) | The n-queens puzzle: all solutions and symmetry classes, matching the published sequences and a bitmask solver |
| [`larzscript-huffman`](https://github.com/drlarzlalaa/larzscript-huffman) | Huffman coding: optimal prefix codes, the entropy bound they can't beat, and a decode check |
| [`larzscript-bwt`](https://github.com/drlarzlalaa/larzscript-bwt) | The Burrows-Wheeler transform (the core of bzip2) with its exact inverse, move-to-front and run-length; identical to a Python reference |
| [`larzscript-lzw`](https://github.com/drlarzlalaa/larzscript-lzw) | Lempel-Ziv-Welch compression and decompression, identical to an independent Python implementation, including the full-dictionary case |
| [`larzscript-loan`](https://github.com/drlarzlalaa/larzscript-loan) | Loan payments, amortization schedules, extra-payment savings and savings growth in integer cents; identical to a Decimal reference |
| [`larzscript-bigint`](https://github.com/drlarzlalaa/larzscript-bigint) | Arbitrary-precision integers: add, subtract, multiply, divide, gcd, powers, factorials, Fibonacci; digit-for-digit identical to Python ints on 1,000+ operations |
| [`larzscript-matrix`](https://github.com/drlarzlalaa/larzscript-matrix) | Exact linear algebra with fractions: solve, determinant, inverse, rank; refuses rather than risk an inexact value; checked against Python Fractions |
| [`larzscript-tsp`](https://github.com/drlarzlalaa/larzscript-tsp) | The travelling salesman problem: exact brute force, nearest neighbour and 2-opt, with a study of how good the heuristics are |
| [`larzscript-astar`](https://github.com/drlarzlalaa/larzscript-astar) | BFS, A* and greedy best-first search through grid mazes, with ASCII paths and a comparison study; identical to a Python heapq reference |
| [`larzscript-flow`](https://github.com/drlarzlalaa/larzscript-flow) | Maximum flow, minimum cut and bipartite matching (Edmonds-Karp); checked against brute-force cuts, with printed flows validated |
| [`larzscript-ga`](https://github.com/drlarzlalaa/larzscript-ga) | A genetic algorithm that evolves a text (tournament selection, crossover, mutation, elitism); identical to a Python reference |
| [`larzscript-sat`](https://github.com/drlarzlalaa/larzscript-sat) | A DPLL SAT solver: DIMACS input, verified models, model counting, random 3-SAT; every verdict checked against brute force |
| [`larzscript-bloom`](https://github.com/drlarzlalaa/larzscript-bloom) | A Bloom filter: the exact false-positive theory against a simulation, and sizing a filter for a target rate |
| [`larzscript-mandelbrot`](https://github.com/drlarzlalaa/larzscript-mandelbrot) | The Mandelbrot set in text and an estimate of its area; every pixel matches a Python reference |
| [`larzscript-primes`](https://github.com/drlarzlalaa/larzscript-primes) | Miller-Rabin, factoring, Carmichael numbers, toy RSA, with exactness limits enforced |
| [`larzscript-automata`](https://github.com/drlarzlalaa/larzscript-automata) | Conway's Life (the R-pentomino takes 1,103 generations) and Wolfram's elementary rules |
| [`larzscript-sortlab`](https://github.com/drlarzlalaa/larzscript-sortlab) | Six sorting algorithms, counted: comparisons and moves on four kinds of input |

### Python tools

Zero dependencies, standard library only, tested on Python 3.9 to 3.13.

| Repo | What it does |
|---|---|
| [`py-emailhygiene`](https://github.com/drlarzlalaa/py-emailhygiene) | Finds typo domains, bad syntax, disposable providers, role accounts and duplicates in an email list |
| [`py-bounceparse`](https://github.com/drlarzlalaa/py-bounceparse) | Reads bounce messages and reports who failed, why, and how many per hour |
| [`py-sitemapcheck`](https://github.com/drlarzlalaa/py-sitemapcheck) | Validates sitemap.xml files and indexes offline, and refuses entity bombs |
| [`py-robotscheck`](https://github.com/drlarzlalaa/py-robotscheck) | Parses robots.txt and tests URLs against it following RFC 9309, with a linter |
| [`py-redirectlint`](https://github.com/drlarzlalaa/py-redirectlint) | Finds chains, loops, conflicts and dead ends in a redirect map |
| [`py-utmlint`](https://github.com/drlarzlalaa/py-utmlint) | Checks campaign links (UTM tags) for typos and spelling drift, and builds correct ones |
| [`py-emailauth`](https://github.com/drlarzlalaa/py-emailauth) | Lints SPF, DMARC and DKIM records offline: the 10-lookup limit, policy tags and RSA key size |
| [`py-headercheck`](https://github.com/drlarzlalaa/py-headercheck) | Lints HTTP security headers offline: HSTS, CSP, cookies, framing, referrer policy |
| [`py-feedcheck`](https://github.com/drlarzlalaa/py-feedcheck) | Validates RSS 2.0 and Atom feeds offline: required elements, RFC 822/3339 dates, duplicate IDs, enclosures |
| [`py-hreflangcheck`](https://github.com/drlarzlalaa/py-hreflangcheck) | Validates hreflang annotations across the pages of a multilingual site: codes, self-references, return links |
| [`py-pageaudit`](https://github.com/drlarzlalaa/py-pageaudit) | Audits HTML pages offline: title, description, canonical, headings, alt text, robots, Open Graph, JSON-LD |
| [`py-safeedit`](https://github.com/drlarzlalaa/py-safeedit) | Bulk regex edits across a tree with a dry run, backup, additive proof, per-file lint and restore |
| [`py-logscan`](https://github.com/drlarzlalaa/py-logscan) | Summarises Apache and nginx access logs offline: status classes, top paths, 404s, bots, hourly traffic |
| [`py-jsonldcheck`](https://github.com/drlarzlalaa/py-jsonldcheck) | Validates schema.org JSON-LD in HTML pages offline: Article, Organization, BreadcrumbList, FAQPage |
| [`py-linkaudit`](https://github.com/drlarzlalaa/py-linkaudit) | Audits a static site's internal links offline: broken pages, images and scripts, missing #anchors, orphan pages |
| [`py-csvdoctor`](https://github.com/drlarzlalaa/py-csvdoctor) | Diagnoses CSV files offline: encoding, BOM, delimiter, ragged rows, duplicate headers, stray whitespace, mixed line endings |
| [`py-htaccesslint`](https://github.com/drlarzlalaa/py-htaccesslint) | Lints Apache .htaccess files offline: unbalanced containers, 2.2 access rules, headers without `always`, HSTS and CSP mistakes, typos |
| [`py-envcheck`](https://github.com/drlarzlalaa/py-envcheck) | Lints .env files and compares them with .env.example offline; never prints a value |
| [`py-semver`](https://github.com/drlarzlalaa/py-semver) | Semantic Versioning 2.0.0: validate, compare, bump and match ranges, following the specification |
| [`py-mdcheck`](https://github.com/drlarzlalaa/py-mdcheck) | Checks Markdown offline: unclosed fences, heading levels, broken relative links and #anchors, table column counts |
| [`py-mailcopycheck`](https://github.com/drlarzlalaa/py-mailcopycheck) | Checks email copy before sending, offline: subject and body heuristics, link and unsubscribe problems, header mismatches |
| [`py-zonediff`](https://github.com/drlarzlalaa/py-zonediff) | Lints DNS zone files and diffs two versions offline, flagging risky changes to MX, NS, SPF, DMARC and DKIM |
| [`py-redact`](https://github.com/drlarzlalaa/py-redact) | Scrubs secrets and personal data from logs before you share them: keys, tokens, passwords, emails, IPs, card numbers; consistent placeholders |
| [`py-rellint`](https://github.com/drlarzlalaa/py-rellint) | Audits outbound links offline: affiliate and sponsored links without rel=sponsored, missing disclosure wording, target=_blank |
| [`py-sitemapdiff`](https://github.com/drlarzlalaa/py-sitemapdiff) | Compares two sitemap.xml versions offline and warns on bulk lastmod stamping, backwards or future dates and sudden drops |
| [`py-jsonlint`](https://github.com/drlarzlalaa/py-jsonlint) | Lints, formats, minifies and queries JSON offline: line and column errors with hints, duplicate keys, integers beyond 2^53 |
| [`py-mailtrace`](https://github.com/drlarzlalaa/py-mailtrace) | Diagnoses a received email from its headers, offline: delivery path and delays, SPF/DKIM/DMARC results, alignment, unsubscribe headers |
| [`py-cachecheck`](https://github.com/drlarzlalaa/py-cachecheck) | Checks HTTP caching headers offline: Cache-Control, ETag, Vary, cookies, compression and Cloudflare cache status, judged by content type |
| [`py-csvdiff`](https://github.com/drlarzlalaa/py-csvdiff) | Compares two CSV files by key: added, removed and changed rows with cell-level changes; ignores row and column order |
| [`py-placeholders`](https://github.com/drlarzlalaa/py-placeholders) | Finds unfilled template placeholders and missing merge fields in emails before sending, in every common syntax |
| [`py-sqlcheck`](https://github.com/drlarzlalaa/py-sqlcheck) | Catches dangerous and mistaken SQL before you run it: UPDATE or DELETE without WHERE, = NULL, DROP and TRUNCATE, unbalanced quotes; executes nothing |
| [`py-diskreport`](https://github.com/drlarzlalaa/py-diskreport) | A read-only report of where the disk space went: biggest folders, files and types, with old, empty and same-size hints |
| [`py-manifest`](https://github.com/drlarzlalaa/py-manifest) | Creates, verifies and compares checksum manifests in the sha256sum format: changed, missing and unlisted files |
| [`py-urlnorm`](https://github.com/drlarzlalaa/py-urlnorm) | Normalizes URLs to a canonical form and finds duplicates: RFC 3986 rules plus optional tracking-parameter, www, trailing-slash and fragment cleanup |
| [`py-readability`](https://github.com/drlarzlalaa/py-readability) | Flesch, Flesch-Kincaid, Gunning fog, SMOG, Coleman-Liau and ARI readability scores for English text |
| [`py-cidr`](https://github.com/drlarzlalaa/py-cidr) | CIDR arithmetic for IPv4 and IPv6: info, contains, merge, subtract and split, verified against the standard library |
| [`py-cronnext`](https://github.com/drlarzlalaa/py-cronnext) | Next run times of a cron expression (lists, ranges, steps, names, aliases, the day-of-month/weekday OR rule), checked against brute force |

## Getting LarzOS

Install and image instructions live at
<https://larzos.com/larzos-linux/>. On an existing box:

```sh
larz doctor      # health check
larz --help      # full command reference
```
