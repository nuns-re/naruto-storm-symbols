# CyberConnect2 game function names for modders

Function names for CyberConnect2's NARUTO SHIPPUDEN: Ultimate Ninja STORM games on PC (Storm 1, 2, 3 HD, 3 Full
Burst, Revolution, 4, Connections), the Xbox 360 retail Storm Generations, and a partial, high-confidence set for
JoJo's Bizarre Adventure: All-Star Battle R. Load them into Ghidra, IDA, Binary Ninja, x64dbg, WinDbg or Visual
Studio, and `FUN_1401a2b30` becomes `ccPlayer::Update`.

The Storm builds contain 140,038 names across 10 builds, and JoJo's Bizarre Adventure: All-Star Battle R
adds 4,117 more, for **144,155 across 11 builds**. The Storm names were recovered by:

- matching against the Nintendo Switch *Storm Trilogy* build, which shipped with C++ symbols;
- bridging those symbols through the stripped Switch builds of Storm 4 and Connections (same compiler) to the PC
  builds of the same games;
- the exported C++ symbols of the Storm 1 mobile (Android arm64) build;
- the developers' own function-name strings left in debug and client builds (log and assert text);
- matching the PC builds against each other;
- rebuilding the open-source libraries the games link (Lua, Bullet, curl, zlib, tolua) and the Microsoft C/C++
  runtime, then matching against those;
- RTTI, vtables and export tables.

One further build was named by a different route: a later CyberConnect2 title on the same engine, *JoJo's Bizarre
Adventure: All-Star Battle R*, was named from its own task-registration strings and by matching it against eight
Storm builds, keeping only names whose evidence measured about 90% correct in a blind review (see below).

The names are CyberConnect2's own, taken from those sources. Nothing is invented, and placeholder names
(`FUN_...`) are never emitted.

## Bring your own game, no game files here

This repository contains **names, addresses and match metadata only**: no executables, game data, Ghidra or IDA
databases, or decompiled code. You need your own copy of each game.

Names apply only to the **exact** exe they were made for. Each build folder is named after the exe's SHA-256, and
every script and tool checks it before applying anything. To find your exe's hash:

```bat
certutil -hashfile NSUNS4.exe SHA256
```

PowerShell: `Get-FileHash NSUNS4.exe`. macOS or Linux: `shasum -a 256 NSUNS4.exe`.

If your hash is not in the table below, you have a different version (another patch or region). Those names would
be wrong at almost every address, so the scripts refuse to apply them.

## Struct layouts

`structs/` holds reconstructed C11 struct layouts for Storm 1, 2, 3 HD and Storm 4 release (562 headers, 3,535 struct types, inferred from static analysis; see `structs/README.md`). New in 1.3.0; common-group coverage completed in 1.3.1.

## Coverage

| Build | Folder (`games/…`) | Functions | Named | % | High | Medium | Low | Self-named |
|---|---|---|---|---|---|---|---|---|
| Storm 1 PC (NSUNS1.exe) | `storm1/10231093af9c…` | 25,921 | 13,585 | 52% | 6,787 | 5,398 | 1,400 | 0 |
| Storm 1 PC debug build (NSUNS1(debug).exe) | `storm1/d209f8d0c035…` | 30,939 | 13,652 | 44% | 6,576 | 5,134 | 1,942 | 298 |
| Storm 2 PC (NSUNS2.exe) | `storm2/697488011750…` | 35,327 | 17,547 | 50% | 7,280 | 8,564 | 1,703 | 0 |
| Storm 3 HD PC (NSUNS3.exe) | `storm3/55b726e94dad…` | 43,144 | 22,230 | 52% | 10,524 | 9,334 | 2,372 | 0 |
| Storm 3 Full Burst PC 2014 (NS3FB.exe) | `storm3fb/5373bfd6308c…` | 34,315 | 12,097 | 35% | 5,252 | 4,652 | 2,193 | 60 |
| Storm Revolution PC (NSUNSR.exe) | `revolution/a123d39ed684…` | 33,896 | 9,840 | 29% | 4,375 | 3,841 | 1,624 | 56 |
| Storm 4 PC debug 1.06 | `storm4/8324cc6d2e0f…` | 71,105 | 14,588 | 21% | 7,900 | 4,060 | 2,628 | 543 |
| Storm 4 PC release (NSUNS4.exe) | `storm4/147e52a3e0d7…` | 61,105 | 14,407 | 24% | 7,526 | 4,525 | 2,356 | 0 |
| Storm Connections PC (NSUNSC.exe) | `connections/cecf0405b5ac…` | 105,435 | 18,932 | 18% | 8,700 | 7,246 | 2,986 | 380 |
| Storm Generations Xbox 360 retail (default.xex, USA) | `generations/0d09e0b47270…` | 21,633 | 3,160 | 15% | 1,239 | 1,267 | 654 | 10 |
| JoJo's Bizarre Adventure: All-Star Battle R PC (JJASBR.exe) | `jjasbr/4d6044d50cc4…` | 73,928 | 4,117 | 5.6% | 4,117 | 0 | 0 | 139 |

*Self-named* means the exe itself contains the function's name string, so it is the official name. The full
SHA-256 of each build is its folder name and is also in its `meta.json`.

The JJASBR entry is partial and high-confidence only. JJASBR has no RTTI for its own classes, so the vtable
route used for Connections is not available. The names come from two sources:

- **Its own strings (139, self-named).** The game registers per-frame tasks by name with a function pointer
  (`RegisterTask(obj, "ccPlayer::StoreDamage", prio, mask, obj, &fn)`), and its UI classes pass their own
  `"<Class>::Ctrl"` / `"<Class>::Draw"` task names to their constructors, which name both. These cover
  JoJo's own code too (`ccUiTitle::ccUiTitle`, `ccPlayerExclude::CalculateCtrl`).
- **Matching against eight Storm builds (3,869).** Storm 1, 2, 3 HD, 4 release and debug, Revolution, Connections
  and the Switch Trilogy, by unique shared strings, BSim and call-graph position, then a vote across games. A
  name is kept only if it rests on a shared string, on BSim similarity ≥ 0.9 with the same name in 2+ builds,
  or on a BSim match of a function of 64+ bytes whose class is shared engine or library code, or exists in
  JJASBR's own source-file list.

Both games pass `__FILE__` to their allocator, so each exe lists its source files; paths under `system\` are
the shared engine (`nucc`, `ccFramework`). Names of classes that exist only in Storm game code are dropped, as are
constructors and destructors without a string anchor (sibling constructors compile to identical code), names
from libraries JJASBR does not link (Bullet, tolua++) and libc++ spellings from the Switch build. Any name
landing on more than one function is dropped. Most names are CRIWARE, `nucc` engine, `cc*` framework, Lua,
curl, Recast/Detour and NintendoWare (`numm`/`nus3`).

## Confidence: what gets applied how

| Confidence | Applied as |
|---|---|
| high | function renamed |
| medium | renamed, plus a comment `[nucc medium <methods>]` |
| low | name **not** applied; comment `[nucc low] maybe <name> (<methods>)` |
| `+llm-disputed` (in methods) | an AI review judged the match wrong; always kept low, comment only |

Every tool takes a minimum confidence. If you want fewer but surer names, use `high`.

## How to use it

Download the latest release zip (or clone the repository), then pick your tool. In the commands below,
`games/storm4/<sha256>` stands for your build's folder.

### Ghidra (12.x)

1. Copy `scripts/ApplyNuccNames.java` into one of your Ghidra script folders (Window → Script Manager → Manage
   Script Directories).
2. Open and analyse your exe, then run **ApplyNuccNames** from the Script Manager (category *NUCC*) and pick
   `games/<game>/<sha256>/functions.csv`.

Qualified names such as `ccPlayer::Update` become real Ghidra namespaces and classes. Headless:

```sh
analyzeHeadless <project_dir> <project> -process NSUNS4.exe -postScript ApplyNuccNames.java /path/to/functions.csv high
```

The last argument is the minimum confidence (`high`, `medium` or `low`, default `low`).

### IDA Pro (7.4+, 8.x, 9.x)

File → Script file… → `scripts/apply_nucc_names_ida.py`, then pick the build's `functions.csv`. Edit
`MIN_CONFIDENCE` at the top of the script to apply only `medium` or `high`. You can also load a generated PDB
(see below) with File → Load file → PDB file.

### Binary Ninja (4.x)

File → Run Script… → `scripts/binja_apply_nucc_names.py`, then pick the `functions.csv`. **Untested in Binary Ninja
itself.** It was written against the documented API and checked against a stub, so please open an issue if it
misbehaves.

### Debuggers: x64dbg, WinDbg, Visual Studio (via a PDB)

Generate a PDB from your own exe. Python 3.8+ is needed; nothing else is.

```sh
python tools/make_pdb.py games/storm4/<sha256> "C:/Games/NARUTO STORM 4/NSUNS4.exe"
```

This writes the PDB next to the exe, under the file name the exe's debug record asks for (for example
`narutoNext4_Release_x64.pdb`), with the same GUID and age. Debuggers then accept it as the game's own PDB. It holds
public function symbols only (no types or line numbers), high and medium by default; `--min-confidence low` adds
low-confidence names as `maybe_<name>`.

- **x64dbg:** x64dbg looks for the PDB in the exe's folder under the debug record's file name, so the names show up
  when the module loads (Symbols tab). For comments as well as names, use the x64dbg database instead (next
  section).
- **WinDbg:** `.sympath+ C:\Games\NARUTO STORM 4`, then `.reload /f NSUNS4.exe`. Try `x NSUNS4!ccPlayer::*` or
  `ln <address>`.
- **Visual Studio:** Debug → Options → Debugging → Symbols, then add the folder. Or right-click the module in the
  Modules window and choose Load Symbols.

All the PC exes we could check (Storm 1, 2, 3 HD, 4 debug and 4 release) carry a CodeView record. Their PDBs load as
matched in LLDB, and the Storm 1 and Storm 4 release PDBs were checked in Microsoft's DbgHelp, the symbol library
debuggers on Windows use, where addresses resolve to names and names to addresses.

**Builds without a CodeView record.** If `make_pdb.py` warns that the exe has none, no debugger will pick the PDB up
automatically. Load it by hand where the tool allows a mismatched PDB (IDA's Load PDB, Ghidra's Load PDB File), or
use the x64dbg database or symbol map instead.

### x64dbg database (names and comments)

```sh
python tools/export_x64dbg.py games/storm4/<sha256> --exe "C:/Games/NARUTO STORM 4/NSUNS4.exe"
```

This writes `NSUNS4.exe.dd64` (`.dd32` for the 32-bit 2014 builds). In x64dbg, while the game is being debugged, use
File → Database → Import database, or the command `dbload <path>`. Alternatively, copy the file into x64dbg's `db`
folder before you start debugging. High and medium names become labels; medium and low names get comments, as in the
table above. `--module` must match the exe's file name if you renamed it (for example `--module "NSUNS4 - Copy.exe"`).

### Anything else: a plain symbol map

```sh
python tools/export_symbol_map.py games/storm4/<sha256> -o NSUNS4.map
```

This writes one `ADDRESS name` line per function, sorted, with absolute addresses (`--rva` for RVAs). It is for
Cheat Engine, ReClass, radare2/rizin, Frida scripts, your own tools, and also the Xbox 360 build.

## Data format (`games/<game>/<sha256>/`)

- `meta.json` holds the build label, `exe_sha256`, `image_base` and `address_kind`. PC builds list **RVAs** (add
  `image_base`). The Xbox 360 build lists **absolute VAs**.
- `functions.csv` has one row per named function:
  - `rva`: address (see `address_kind`)
  - `size`: function size in bytes
  - `name`: qualified name, Trilogy `nsN::` namespaces removed
  - `namespace` and `short_name`: `name` split at the last `::`
  - `confidence`: `high`, `medium` or `low`
  - `self_named`: the exe contains this name as a string
  - `methods`: how the name was found (see below; `;`-separated, `a>b` means found by `a` in another build and
    carried here by `b`)
  - `sources`: number of independent kinds of evidence that agree. Other builds of the same game don't count, since
    they share the same matching mistakes.
  - `conflict_with`: the strongest competing name, if any
  - `evidence`: addresses in the source build(s) and notes
- `counterparts.csv` (Generations only) gives the same function's address in the other builds, including prototype
  and demo builds the names were bridged through.

| Method | What it is |
|---|---|
| `selfname` | the exe contains the function's own name string (official) |
| `switch-string` / `switch-bsim` / `switch-bsim+neighbours` / `switch-callgraph` | matched to the Switch Trilogy build by unique shared strings, Ghidra BSim similarity, BSim backed by matched neighbours, or call-graph position |
| `nx1-…` / `nx4-…` / `nxc-…` (for example `nx1-bsim`, `nx4-bsim`) | in the Storm builds: Switch Trilogy symbols matched onto a stripped Switch build of Storm 4 (`nx4`, EUR update v196608) or Connections (`nxc`, update v65536), arm64 with the same compiler, then carried to that game's PC build by BSim, strings or call graph |
| `register-task` / `register-ctor` (JJASBR) | the exe registers the function as a task under this name, or the function is the constructor its callers hand their own `"<Class>::Ctrl"`/`"<Class>::Draw"` task names (official) |
| `<ref>-string` / `<ref>-bsim` / `<ref>-similar` / `<ref>-callgraph` (JJASBR) | matched to reference build `<ref>` — `s1`, `s2`, `s3hd`, `s4` (release), `s4d` (debug 1.06), `rev`, `conn`, `switch` — by shared unique string, BSim, call-graph similarity voting or call-graph position; `evidence` holds the reference address. One row lists every build that agreed; `sources` counts them |
| `bsim-consensus` (JJASBR) | BSim similarity ≥ 0.9 to functions with the same name in 2+ reference builds, and no competing name |
| `jojo-switch-bridge` (JJASBR) | an existing lower-confidence match was reached a second way: Switch Trilogy to JoJo All-Star Battle R on Switch to this exe, with the same name |
| `mobile-symbol` | the Storm 1 mobile (Android arm64) build's exported C++ symbols, carried to the PC builds by strings, BSim or call-graph position |
| `vtable` | virtual function: the PC vtable lined up slot by slot with the Switch build's named vtable |
| `rtti-vtable` | the same, with the PC vtable's class known from its own RTTI (Connections) |
| `export` | the exe's own export table (exact) |
| `runtime-msvcr120`, `runtime-msvcp100`, … | Microsoft C/C++ runtime, matched against the same runtime version with Microsoft's public symbols |
| `lib-lua`, `lib-bullet`, `lib-tolua`, `lib-zlib`, `lib-zlib1211`, `lib-curlfull`, `lib-curl7501` (older data: `lib-curl`) | open-source library built from source with the same compiler, matched by BSim: Lua, Bullet, tolua++, zlib 1.2.7 / 1.2.11, curl 7.29.0 with every protocol the games ship, curl 7.50.1 (Connections). Each build is matched only against the libraries it links (see below) |
| `fill-callees` / `fill-caller` | call-graph fill: unnamed callees paired in call order when both sides have the same count, or a lone unnamed caller on each side |
| `fill-similar` | call-graph fill by voting: matched callers and callees vote for pairings of unnamed neighbours, accepted only when the pair is mutual-best, wins by a margin and has 2+ votes (or 1 vote plus BSim similarity ≥ 0.5); tolerates inlining. `evidence` holds the Switch counterpart's address |
| `… > string` / `> bsim` / `> callgraph` / `> similar` | carried over from another PC build of the series, by shared unique strings, BSim, call-graph position or code similarity |
| `manual` | hand-verified (the evidence column says how) |
| `+llm-verified` | an AI review compared the two decompilations and agreed |
| `+llm-disputed` | an AI review judged the match wrong; kept as a low-confidence comment, because many such calls turned out to be version differences |

### Accuracy

Measured on known names (a share of names hidden, then re-derived):

| Method | medium | low |
|---|---|---|
| vtable alignment | 85% | 77% |
| RTTI vtable alignment (Connections) | 91% | 83% |
| call-graph fill (`fill-callees` / `fill-caller`) | 97% | 83% |
| call-graph fill by voting (`fill-similar`) | 95% | 89% |
| Switch matching (hand review, Storm 1) | ~92% overall | |

Across different games (for example Storm 3 code matched into Storm 4), string-only matches are weak (about 50%).
So they are demoted unless another method agrees.

The JJASBR set was measured by blind review rather than hidden names: an AI reviewer compared each JJASBR
function's decompilation with its reference function without seeing the method or confidence. On 150 sampled
matches, string anchors were 100% correct, BSim consensus 97%, BSim 81% and call-graph position alone 57%; functions
under 64 bytes were right 55% of the time. The keep rules above were set from that sample. On a separate held-out
sample of 80 kept names, 72 were judged correct, 7 wrong and 1 unsure (90%); 2 of the 7 were right matches whose
reference build carries a wrong name. About 2,640 lower-confidence JJASBR names (40–60% in review) are not
published. The 109 `jojo-switch-bridge` names were medium-confidence matches that a separate route through JoJo
All-Star Battle R on Switch confirmed; 15 of 16 sampled were judged correct, a small sample.

### Which libraries each build links

Checked from version and error strings, zlib's inflate/deflate tables and Bullet's `1e18f` constant. Library names
are matched only into builds that link the library.

| Build | Libraries |
|---|---|
| Storm 1, Storm 1 debug | Bullet |
| Storm 2, Storm 3 HD | Lua, tolua++, Bullet |
| Storm 3 Full Burst, Revolution, Generations | Lua, tolua++ |
| Storm 4 (debug and release) | Lua, tolua++, Bullet, zlib 1.2.7, curl 7.29.0 |
| Connections | Lua, tolua++, Bullet, zlib 1.2.7 + 1.2.11, curl 7.50.1 (Schannel) |
| JoJo's All-Star Battle R | Lua 5.1.4, zlib 1.2.11, curl 7.79.0-DEV, Recast/Detour, rapidxml, CRIWARE; no Bullet (no `1e18f` constant) and no tolua++ |

## Platform-call annotations (`platform-calls/`)

A companion data set for the nine Storm PC builds; JoJo's All-Star Battle R is not covered. It holds annotations, not
names, in three files per build:

- `imports.csv`: every call into an imported DLL function (D3D11/D3D9, DXGI, XInput, DirectInput, Steam, Win32,
  sockets). Exact.
- `calls.csv`: Direct3D COM method calls whose object type is proven. Deliberately few; precision first.
- `steam.csv`: Steamworks interface calls such as `ISteamMatchmaking::CreateLobby`, with the interface version used.
  1,113 calls across 9 builds.

Apply them as comments with `platform-calls/scripts/ApplyNuccD3D11.java` (Ghidra: pick the build folder) or
`platform-calls/scripts/apply_nucc_d3d11_ida.py` (IDA: pick `calls.csv`). Call sites get `[d3d11]`, `[steam]` or
`[import]` comments, and nothing is renamed. The 2014 builds (Full Burst, Revolution) use Direct3D 9 and have no COM
calls. See [platform-calls/README.md](platform-calls/README.md) for per-build counts and limits.

## The builds and the engine

| Build | Built | Compiler | Source tree (from paths left in the exe) |
|---|---|---|---|
| Storm 3 Full Burst PC | 2014-01 | VS2010, x86 | `narutoNext3` |
| Storm Revolution PC | 2014-11 | VS2010 SP1, x86 | `narutoNext35` |
| Storm 4 PC debug 1.06 | 2016-06 | VS2013 Update 4, x64 | `STORM4_ROM/narutoNext4` + `system/nucc` + `system/ccFramework` |
| Storm 1 / 2 / 3 HD PC (Trilogy) | 2017-03 | VS2013, x64 | `STORMT_PKG/narutoNext1,2,3` + `system/nucc` + `system/ccFramework` |
| Storm 4 PC release | 2020-06 | VS2013 Update 5, x64 | `Update_SVN/narutoNext4_trunk` |
| Storm Connections PC | 2026-09 patch | VS2019, x64 | `next5/…/trunk/narutoNext4` |
| JoJo's All-Star Battle R PC | 2024-01 | VS2017 15.9 (linker 14.16), x64 | `asb_client/asb_prog/prog` + `asb_client/system/nucc` + `asb_client/system/ccframework` |
| (name source) Switch Trilogy | 2020 | Clang, AArch64 | `narutoNext123`: games in namespaces ns1/ns2/ns3 |

All of them share one engine (`nucc`, the NU Library) and one game framework (`ccFramework`, the `cc*` classes),
under per-game code (`narutoNextN/prog`). Storm Generations is a branch of Storm 2's code, and Storm 3 branched from
Storm 2 separately. Revolution is "3.5". The 2017 Trilogy remasters sit on the Storm 4-era engine, and Connections is
a branch of Storm 4's trunk (its game code folder is still `narutoNext4`). JoJo's All-Star Battle R is a separate
title that reuses the same engine and framework under its own source tree (`asb_client`).

## Contributing

Found a name, or a wrong one? See [CONTRIBUTING.md](CONTRIBUTING.md). It describes a six-column CSV and a validator
(`tools/validate_contribution.py`) that checks it against the build before review. Run the tool tests with
`python -m unittest discover -s tests`; no game files are needed.

## How this was made: AI assistance

This project was built with AI assistance, using Anthropic's **Claude** through Claude Code. A human directed the
work and decided what to trust. Claude wrote most of the matching pipeline, the apply scripts, the exporters, the
tests and this documentation, and ran the analysis.

A Claude model (Haiku) also reviewed low-confidence matches by comparing decompilations. Those verdicts appear in
the data as `+llm-verified` / `+llm-disputed`. A verified match could be promoted, and a disputed one is kept as a
low-confidence comment. The names themselves come from the sources listed at the top, not from the model. Treat
medium and low names as leads, and report mistakes.

## Legal

- **Not affiliated with or endorsed by** CyberConnect2, Bandai Namco Entertainment, Nintendo, Microsoft or Valve.
  NARUTO and all related names are trademarks of their respective owners.
- The repository contains no game code, assets or executables. It holds function names, addresses and facts about
  the binaries, shared for interoperability, research and modding of copies you own.
- **Licences:** the scripts and tools (`scripts/`, `platform-calls/scripts/`, `tools/`, `tests/`) are MIT (see
  [LICENSE](LICENSE)). The data (`games/`, `platform-calls/games/`) is dedicated to the public domain under CC0 1.0
  (see [LICENSE-DATA](LICENSE-DATA)).
