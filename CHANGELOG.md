# Changelog

## 1.4.0 (2026-10-26)

**JoJo's Bizarre Adventure: All-Star Battle R, second pass.** The `jjasbr` set is rebuilt from scratch: **4,008
names** (5.4% of 73,928 functions), all high confidence, up from 596 (0.8%) in 1.3.0. It replaces the 5-row set
committed after 1.3.1, which used the contribution format (no `methods` column, so the apply scripts could not
read it) and named a task-registration function `ccBattleDirectionManager::Ctrl`.

- **139 self-named** from the exe's own strings: 42 functions it registers as tasks by name
  (`register-task`) and 97 UI constructors that receive their own `"<Class>::Ctrl"`/`"<Class>::Draw"` task names
  (`register-ctor`). The "function references only its own name string" rule is not used for this build: its hits
  were static initializers of type-info globals and a function that creates the named task, not the task itself.
- **3,869 matched** against eight Storm builds (adds Storm 2, Storm 3 HD, Storm 4 release and Revolution to the
  1.3.0 references) by shared strings, BSim and call graph, then voted. Methods read `<ref>-<how>` (e.g.
  `conn-bsim`, `s3hd-string`) plus `bsim-consensus`; `sources` counts the agreeing builds.
- Kept only where the evidence measured well in a blind review: a shared string, BSim consensus across 2+ builds,
  or a BSim match on a function of 64+ bytes whose class is shared engine/library code or exists in JJASBR's own
  source-file list. Dropped: Naruto-only classes, constructors/destructors without a string anchor, names from
  libraries JJASBR does not link (Bullet, tolua++) and libc++ spellings.
- Accuracy: 72 of 80 held-out names judged correct (90%) in a blind review; 2 of the 8 misses are wrong names in
  the reference build, not wrong matches.
- 465 of the 1.3.0 set's 596 names are kept unchanged and 1 is renamed. Of the rest, 40 fall below the published
  bar, 35 are constructors/destructors without a string anchor, and 55 are Naruto-only classes, names that
  collide with another function, or names the reference builds disagree on.
- README: JJASBR's compiler (VS2017 15.9), source tree and linked libraries are now recorded.

Totals: **140,038 names across the 10 Storm builds**, 144,046 with JoJo All-Star Battle R. Storm data is unchanged.

## 1.3.1 (2026-10-20)

**Struct coverage.** The 88 common engine groups in `structs/` now cover every class that has at least 3 named methods
in a build: 248 classes that had no struct in 1.3.0 were added across 123 group/build pairs, either as a struct with
asserted offsets or as a documented `<Class>: no instance state` note (methods checked, where the state lives). Struct
types: Storm 1 617 -> 684, Storm 2 940 -> 983, Storm 3 HD 1045 -> 1085, Storm 4 744 -> 783 (3,346 -> 3,535). Header
count is unchanged (562); existing headers grew.

Only `cmnNN` headers changed; no header lost a field or an offset assert. Also removed: one orphan field table without
a header (`s3Boss06-batch1-fields.tsv`). Function names are unchanged from 1.3.0.

## 1.3.0 (2026-10-19)

**Own-game names.** Storm 1, 2 and 3 now carry only names that exist in that game's own symbols (the Switch Trilogy
keeps each game's code in its own namespace, `ns1::` / `ns2::` / `ns3::`), its mobile build (Storm 1) or its own
strings. 1,552 names that existed only in another game's namespace were removed: Storm 1 227, Storm 1 debug 275,
Storm 2 532, Storm 3 HD 364, Storm 3 Full Burst 154. Library and string-derived names are unaffected.

**Storm 1 boss names.** 62 Storm 1 (35) and Storm 1 debug (27) functions that carried Storm 3 boss names (Deidara,
Sasori, Pein, Itachi, Killer Bee, ...) were renamed from what the Storm 1 code does (e.g.
`ccBossCompositeActionState::~ccBossCompositeActionState`, `ccBossBreakObj::ccBossBreakObj`); confidence `low`,
method `manual`.

**Struct layouts (new).** `structs/` adds 562 C11 headers with field tables for Storm 1, Storm 2, Storm 3 HD and Storm 4
release: core engine classes, 88 common engine groups, boss battles, network, sound, story cutscenes, rollback-relevant
state and statically linked libraries. All inferred from static analysis (`structs/README.md`); every header compiles
and asserts each field offset (`tests/test_structs.py`).

Totals: **140,038 names across the 10 Storm builds**, 140,634 with JoJo All-Star Battle R.

### JoJo's Bizarre Adventure: All-Star Battle R

New build: **JoJo's Bizarre Adventure: All-Star Battle R** (`jjasbr/4d6044d50cc4…`), a different
CyberConnect2 title that reuses the NUCC engine and `ccFramework` classes. It is included as a
partial, engine-only set:

- 73,928 functions in the exe; **596 named** (591 high, 5 medium), 0.8%.
- All 73,928 functions were matched with Ghidra BSim against four reference builds — Storm 4 debug,
  Storm 1, Connections and the Switch Trilogy (Connections signatures were generated and added to the
  BSim database for this run). Storm 4 release is not a reference here.
- Names were kept only where **2 or more independent reference builds agree on the same name**
  with similarity ≥ 0.9, no competing name above the same bar, and a target body of at least
  32 bytes. Single-build matches, conflicting matches and tiny wrappers were discarded; an
  independent decompilation sample showed even similarity-1.0 matches can be wrong.
- A further 57 names were dropped because they landed on more than one function (one such name
  matched 20 addresses 0x30 apart, all 48 bytes). This is the same one-address rule the package
  already applies to vtable-derived names.
- Methods read `nxc-bsim` / `nx4-bsim` / `nx1-bsim` / `switch-bsim`, naming the reference build the
  name came from. The 5 medium rows are the exe's own method-name strings (assert text)
  cross-referenced to their functions.
- No JoJo-specific game code is named; the set is engine, middleware (CRIWARE, Lua, Bullet,
  Recast/Detour, NintendoWare, curl — PhysX is not present) and `cc*` framework names only.
- 596 names (see the totals above).

## 1.2.0 (2026-10-16)

Accuracy release: 141,590 names in total, **down** from 144,211. The count fell because about 5,800 wrong names
were removed (mostly bad vtable names and names from libraries a build doesn't link), more than the new sources
added. Same 10 builds, same file format; scripts and tools are unchanged.

- **Vtable accuracy fix.** Vtable alignment had pasted one class's method names onto many other classes' vtables
  when base-class anchors tied. For example, `ccCameraOperatorDirection::*` landed on about 145 camera classes in
  Connections. Now a vtable whose top vote is a tie is skipped, and a vtable-derived name that lands on more than
  one function is dropped. Checked against Connections' own RTTI: its vtable-backed names went from 864 consistent
  and 1,612 contradicting to 665 consistent and 79 contradicting.
- **Stable passes.** RTTI anchors no longer count names backed only by rtti/vtable/fill, which ends the flip-flop
  between merge passes. Two identical passes now differ by 6 names net on Connections, down from about ±500. The
  RTTI pass names about 3,000 Connections functions, up from about 2,500.
- **Library presence fix.** Each build is now matched only against the open-source libraries it links (table in the
  README). Any name that came only from a library the build doesn't link is dropped, whatever it's called (for
  example `parsedate`, `math_randomseed`, `MD5_Update`, `copy_string_into_buffer`). None remain in any build.
- **curl and zlib rebuilt to match the games.** curl 7.29.0 is now built with every protocol the games ship; 1.1.0
  used an HTTP-only build. Connections is matched against its real versions, curl 7.50.1 and zlib 1.2.11. Rows
  citing a curl build: Storm 4 release 411 → 586, Storm 4 debug 413 → 591, Connections 336 → 735.
- **Switch bridges (new methods `nx4-…` / `nxc-…`).** The stripped Switch builds of Storm 4 (EUR update v196608)
  and Connections (update v65536) were built with the same compiler as the Trilogy. Trilogy symbols were matched
  onto them, then carried to the PC builds of the same game, and from there to the other builds. Rows citing a
  bridge: Storm 4 release 4,277, Storm 4 debug 4,151, Connections 4,499.
- **Generations:** transfers and the counterpart map (the same 12,926 functions) were rerun after the bridges.
- **Platform calls:** new `steam.csv` per build with Steamworks interface calls (1,113 across 9 builds), plus
  `steam_calls` / `steam_interfaces` in each `meta.json`. The Ghidra and IDA annotation scripts now also tag these
  calls `[steam]`. The D3D `calls.csv` and `imports.csv` files are unchanged.
- **Changes against 1.1.0, all builds:**
  - 4,658 names added and 7,279 removed;
  - 3,108 addresses renamed;
  - 4,754 names raised in confidence and 3,672 lowered.
- **Coverage now:**
  - Storm 1: 53%; Storm 1 debug: 45%;
  - Storm 2: 51%; Storm 3 HD: 52%; Storm 3 Full Burst: 36%; Revolution: 29%;
  - Storm 4 debug: 21%; Storm 4 release: 24%;
  - Connections: 18%; Generations 360: 15%.

Verification: the tool tests, PDBs for the 5 locally available exes (valid, matched, exact public-symbol counts),
the x64dbg database and symbol map (checked row by row against every build) and the repository audit were all rerun.
For those 5 exes every address is inside an executable section.

## 1.1.0 (2026-10-16)

Data update: 144,211 names in total, up from 139,450. Same 10 builds, same file format; scripts and tools are
unchanged.

- **New name source:** the Storm 1 mobile (Android arm64) build's exported C++ symbols, carried to the PC builds by
  strings, BSim and call-graph position (method `mobile-symbol`). 28,863 rows now cite it.
- **Changes against 1.0.0, all builds:**
  - 7,164 names added and 2,403 removed;
  - 3,582 addresses renamed;
  - 8,130 names raised in confidence.
- **Coverage now:**
  - Storm 1: 54%; Storm 1 debug: 46%;
  - Storm 2: 52%; Storm 3 HD: 54%; Storm 3 Full Burst: 36%; Revolution: 29%;
  - Storm 4 debug: 22%; Storm 4 release: 24%;
  - Connections: 18%; Generations 360: 15%.

  The README table has the per-confidence counts.
- **Removed 4 bad Connections rows that were also in 1.0.0:**
  - an `rtti-vtable` entry with a negative address (`ccAdvBaseDraw::GetCoordPos`);
  - three data exports that were listed as functions: `NvOptimusEnablement`, `AmdPowerXpressRequestHighPerformance`
    and Scaleform's `HasMemoryLeaks` flag. Applying them would have created functions on data.
- Generations `counterparts.csv`: the same 12,926 functions; the `name`/`confidence` columns follow the new names
  (227 rows changed).
- README: `fill-similar` documented, with its hold-out accuracy (95% medium, 89% low). Standard MIT licence text, so
  GitHub detects it.

Verification: the tool tests, the exporters (PDB, x64dbg database, symbol map) and the validator were rerun against
the new data, and the repository audit was repeated. A new check found the 4 bad rows above. Every remaining row
has a valid address, and for the 5 PC builds available locally every address is inside an executable section.

## 1.0.0 (2026-10-16)

First public release.

- **Names** for 10 builds, 139,450 in total: Storm 1 (release and debug), Storm 2, Storm 3 HD, Storm 3 Full Burst
  (2014), Storm Revolution, Storm 4 (debug 1.06 and release), Storm Connections, and Storm Generations (Xbox 360
  retail, USA).
- **Apply scripts** for Ghidra (`ApplyNuccNames.java`) and IDA (`apply_nucc_names_ida.py`), plus a Binary Ninja
  script (`binja_apply_nucc_names.py`) that has not been tested in Binary Ninja itself. All three check the exe's
  SHA-256.
- **Exporters** in `tools/`:
  - `make_pdb.py` writes a PDB of public symbols with the exe's own GUID and age, for x64dbg, WinDbg and Visual
    Studio.
  - `export_x64dbg.py` writes an x64dbg database (labels and comments).
  - `export_symbol_map.py` writes a plain `address name` map.
- **Contribution validator**: `tools/validate_contribution.py`.
- **Platform-call annotations** (`platform-calls/`): DLL import calls and proven Direct3D COM calls for the PC
  builds.
- The `mangled` column of the internal data set (Itanium-mangled names copied from the Switch build) is **not**
  included. `name`, `namespace` and `short_name` carry the same information in readable form.

Verification for this release:

- PDBs were generated for Storm 1, 2, 3 HD, 4 debug and 4 release. They pass `llvm-pdbutil dump -all` and are accepted
  as matched by LLDB (which rejects a PDB whose GUID was altered). The Storm 1 (age 1) and Storm 4 release (age 21)
  PDBs resolve names both ways in Microsoft DbgHelp: 8 of 8 sampled addresses and 4 of 4 names each.
- The x64dbg database, symbol map and validator outputs were checked row by row against `functions.csv`.
