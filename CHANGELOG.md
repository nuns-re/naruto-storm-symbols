# Changelog

## 1.4.5 (2026-11-19)

**JoJo's Bizarre Adventure: All-Star Battle R: more OpenSSL, and the open OpenSSL questions settled.**
The `jjasbr` set goes from 7,180 to **8,095 names** (7,087 high, 1,008 medium), 11% of the exe's 73,928 functions.
Storm data is unchanged. Totals: 140,039 names across the 10 Storm builds, **148,134** with JoJo All-Star Battle R.

**OpenSSL (921 new names, medium).**
- **Fuller reference:** the 1.4.4 reference DLL was linked with `/OPT:REF /OPT:ICF`. That dropped unused functions
  and folded identical ones under one name. It is now relinked with `/OPT:NOREF /OPT:NOICF` and every object file
  kept, so every OpenSSL function has its own address and name.
- **Ties settled by the call graph** (`lib-openssl310;callgraph`). 1.4.4 left out 1,848 candidates that tied with
  another function or were claimed more strongly by one. Three rules now settle them, repeated until nothing changes:
  - BSim names that became clear once competitors were named (`lib-openssl310`);
  - a BSim candidate chosen because its callers and callees agree with already named neighbours;
  - a function placed by its call position under a caller that is already matched.
- **Check on hidden names:** 20% of the 1.4.4 OpenSSL names were hidden and re-derived. 565 of 570 came back right.
- **Blind review**, 15 sampled per group:
  - BSim: 13 of 15 correct.
  - Call-graph choice: 13 of 15.
  - Call position, functions of 64 bytes or more: 13 of 15.
  - These groups ship as medium.
  - Call position under 64 bytes scored 11 of 15 and is not published (594 names).
- Two candidates were dropped because a function holding a non-OpenSSL name scored as well for the same reference.

**Storm-matched names inside the OpenSSL code.**
- The 1.4.4 open table listed 7 such names that disagreed with the OpenSSL match. A wider search found 13.
- Each was checked by hand: the strings, callers, callees and source file named in the exe's `__FILE__` paths.
- Each OpenSSL name was then blind reviewed.

Replaced (`manual;lib-openssl310`, high):

| Address | Was | Now | Evidence |
|---|---|---|---|
| `0x858ae0` | `ccCommandHelp::Draw` | `X509V3_extensions_print` | prints the "critical" extension strings |
| `0x87cdf0` | `curl_strnequal` | `openssl_strerror_r` | `o_str.c`; called by `ossl_err_string_int` |
| `0x8be8c0` | `Curl_sspi_strerror` | `i2v_AUTHORITY_INFO_ACCESS` | uses the AIA literal |
| `0x8c7110` | `UnDecorator::getTemplateConstant` | `asn1_primitive_print` | `tasn_prn.c` |
| `0x949da0` | `ccUiCommandList::Command::SetAllAlpha` | `HMAC_CTX_set_flags` | called by `hmac_set_ctx_params` |
| `0x9555c0` | `nuccIpcMessage_DestroyQueueAll` | `ossl_store_destroy_loaders_int` | calls `OPENSSL_LH_free` on the loader table |

Removed: the code is OpenSSL, but no single OpenSSL name is certain:

| Address | Was | Why |
|---|---|---|
| `0x8b4ec0` | `Curl_protocol_getsock` | `X509_CRL_verify` ties with `X509_CRL_get0_by_serial`; the review judged it wrong |
| `0x982a70` | `emCmd_World` | `provider_conf.c` strings |
| `0x930450` | `CriMvEasyPlayer::SetFile` | `asn1_gen.c`; calls `CONF_parse_list` |
| `0xa06000` | `mmScriptNodeXML::IsComment` | calls `CRYPTO_gcm128_aad` |
| `0x847220` | `BattleResultBonus::Group::Penalty::Retry` | `bio_lib.c` |
| `0x89a7f0` | `ccStageObjectList::AddClump` | calls `0x847220` |

Kept: `0x86f210` `criAtomConfig_Finalize`. BSim also scores it 1.000 against `ossl_init_no_config_ossl_`, but so does
another function, so the evidence is not decisive.

Still open: the five library names from 1.4.4 that the package already uses at another address (`lua_toboolean`,
`currentline`, `luaL_loadbuffer`, `read_string`, `conn_free`).

## 1.4.4 (2026-11-19)

**JoJo's Bizarre Adventure: All-Star Battle R: OpenSSL, more Lua, and source-file matches.**
The `jjasbr` set goes from 4,156 to **7,180 names** (7,093 high, 87 medium), 9.7% of the exe's 73,928 functions.
Storm data is unchanged. Totals: 140,039 names across the 10 Storm builds, **147,219** with JoJo All-Star Battle R.

**OpenSSL (2,918 names, high, `lib-openssl310`).** The exe links OpenSSL statically.
- **Reference build:** we built OpenSSL 3.1.0 from the official source with MSVC, with no assembly and the exe's own
  compiler line (`cl /Zi /MT /Zl /Gs0 /GF /Gy /W3 /wd4090 /O2 -DL_ENDIAN -DOPENSSL_PIC`, kept in its strings). The
  result is a DLL with a PDB, 7,896 functions.
- **Version:** the exe also carries a `3.1.0-dev` version string, so its copy predates the 3.1.0 release slightly.
  Functions that changed before release simply don't match.
- **Matching:** BSim against the reference's PDB names.
  - Only names that OpenSSL's own object files define are used, so C runtime names are excluded.
  - OpenSSL names are placed only inside the exe's OpenSSL code, so matches that landed on curl, CRIWARE and game
    functions were rejected.
  - A JoJo function whose best reference function was claimed more strongly by another JoJo function, or tied with
    one, was left unnamed (1,848 candidates).
  - Existing package names were not replaced. 7 of them sit inside the OpenSSL code and disagree with the OpenSSL match;
    they are listed below as open questions.
- **Review:** 90 of 90 sampled matches were judged correct in a blind review, across similarity from ≥ 0.95 down to
  below 0.7, plus functions under 64 bytes.
- **Coverage:** 2,918 of the roughly 6,900 functions in the exe's OpenSSL code.

**Lua 5.1.4 (33 names, high, `lib-lua`).** Lua 5.1.4 built from source with MSVC and matched the same way. The blind
review judged 14 of 15 correct; the miss was `fprintf`, a C runtime inline that every object file carries.

**Source-file matches (73 names, medium, `srcfile`).**
- **Method:** BSim matches to a Storm build, accepted only where the reference name's class matches the source file the
  exe places the function in. The source-file map is ArcherOfLegend's ASBR map, built from the `__FILE__` paths the
  exe kept. Thanks again.
- **Review:** the group scored 25 of 30, so it ships as medium. Four of its five misses were under 64 bytes, and such
  functions are left out.
- **Constructors and destructors:** the package normally drops these without a string anchor. Here the source-file
  check plays that role, and 12 of 13 sampled were correct, so they are kept.
- **Not published:** three related groups scored 53–73%: existing medium/low names this confirmed, names supported
  only by the file, and new medium matches.

zlib 1.2.11 was built too; it added no new names.

Open: inside the OpenSSL code, 7 existing package names disagree with the OpenSSL match. The package names, from Storm
matching, stay until checked by hand:

| Address | Package name | OpenSSL match |
|---|---|---|
| `0x858ae0` | `ccCommandHelp::Draw` | `X509V3_extensions_print` (BSim 0.737) |
| `0x87cdf0` | `curl_strnequal` | `openssl_strerror_r` (BSim 1.000) |
| `0x8b4ec0` | `Curl_protocol_getsock` | `X509_CRL_verify` (BSim 1.000) |
| `0x8be8c0` | `Curl_sspi_strerror` | `i2v_AUTHORITY_INFO_ACCESS` (BSim 0.657) |
| `0x8c7110` | `UnDecorator::getTemplateConstant` | `asn1_primitive_print` (BSim 0.709) |
| `0x949da0` | `ccUiCommandList::Command::SetAllAlpha` | `HMAC_CTX_set_flags` (BSim 0.724) |
| `0x9555c0` | `nuccIpcMessage_DestroyQueueAll` | `ossl_store_destroy_loaders_int` (BSim 1.000) |


Also open: in five places a library match disagrees with where the package already uses that name. The package name stays
for now:

| Name | In the package at | Library match at |
|---|---|---|
| `lua_toboolean` | `0x675e80` | `0x675d10` |
| `currentline` | `0x67e720` | `0x67e6c0` |
| `luaL_loadbuffer` | `0x676f50` | `0x692064` |
| `read_string` (Lua and OpenSSL both have one) | `0x67c310` | `0x8740b0` |
| `conn_free` | `0x806c90` | `0x8f58a0` |

## 1.4.3 (2026-11-19)

**JoJo's Bizarre Adventure: All-Star Battle R: a community contribution, corrected allocator names and 20 new names.**
The `jjasbr` set goes from 4,117 to **4,156 names** (4,142 high, 14 medium). Storm data is unchanged.

**Contribution by ArcherOfLegend** (`contrib/jjasbr/4d6044d50cc4…/ArcherOfLegend.csv`, 34 rows). Each row is a function
the exe registers as a task under its own name. All 34 registration strings were checked against the exe.
- **20 new names, 10 high and 10 medium.** 19 of the rows ship at their own address. The 20th name is `0x41a51c`, the
  body behind one row's jump stub. The `evidence` column credits ArcherOfLegend.
- 8 of these are medium because a second blind review rated them lower. For 3 of them, the bodies are a few bytes long
  and their BSim matches point at unrelated Storm 4 functions. The other 5 were only rated low. Two more are medium from
  the start: an 8-byte body (`0x562888`) and the stub target `0x41a51c`.
- 12 rows are not published:
  - 10 name a `jmp [vtable+slot]` thunk (9 `ccPlayer::*`, plus `nuccAnm::Draw`). These name a vtable slot, not a body.
  - 2 name a 5-byte jump stub. The two targets are named instead: `0x2374cc` by its own row, `0x41a51c` as above.
- 3 rows were not taken. Their targets are generic vcall thunks shared by many unrelated places (`0x259af8`, `0x259b04`,
  `0x277950`).
- The contribution also found these wrong names in 1.4.2, now removed:

| Address | Was | Why |
|---|---|---|
| `0x5894c`, `0x1706c0`, `0x1708b0`, `0x1b6a40` | four `cc*` names | static initializers: called only from the CRT initializer table, they build global strings and register `atexit` handlers. Nothing in 0x1000–0x1e0000 is named any more |
| `0x6bf5e0` | `malloc` | the engine's own allocator entry (size, kind, file, line), not the CRT `malloc` |
| `0x467ef0`, `0x5682fc`, `0x6620c4` | `ccSceneDemoState::ScreenCapture`, `ccSceneNetMatchingLeave::CtrlDialogDispCount`, `ccSignoutNotifyTask::Ctrl` | these register the task; the task bodies are `0x4686e0`, `0x568208` and `0x661cb0`, which now carry the names |
| `0x797a20` | `ccUiManager::Ctrl` | a second function registered under the same string; the Storm lineage's `ccUiManager::Ctrl` is `0x7977d0`, which now carries it. What `0x797a20` is stays open |

**dlmalloc, corrected by hand** (method `manual`). Most 1.4.2 names in this range sat one function away from
the right body. They are now identified from the bodies and the call graph (dlmalloc 2.8: mparams magic `0x58585858`, 0x390-byte mstate):

| Address | Was | Now |
|---|---|---|
| `0x75dd20` | `mspace_free` | `init_mparams` |
| `0x75e350` | `mspace_malloc` | `internal_realloc` |
| `0x75f160` | `dlmalloc` | `tmalloc_large` (medium) |
| `0x75f6b0` | `dlfree` | `tmalloc_small` (medium) |
| `0x7601f0` | `mspace_memalign` | `dlmalloc` |
| `0x7605a0` | `nsts_AlignedAlloc_NX` | `mspace_calloc` |
| `0x75df40` | `Nucc::Memory::GetExclusion` | `internal_malloc_stats` (prints dlmalloc's "max system bytes" lines) |
| `0x760fd0` | `Nucc::Memory::SystemMemoryFree` | `mspace_malloc_stats` (medium) |

New: `mspace_malloc` `0x760c90`, `mspace_free` `0x760620`, `dlfree` `0x75faf0`, `mspace_memalign` `0x761010`,
`internal_memalign` `0x75e0e0`, `mspace_realloc` `0x761020`, `create_mspace_with_base` `0x75f980`,
`init_user_mstate` `0x75de30` (medium). The Storm builds' dlmalloc names probably carry the same shift: this
release does not change them.

**20 new matched names.** The match was re-run with BSim queries limited to one reference build at a time, plus slot-by-slot
vtable alignment. That gave 33 new candidates, and all 33 went through a blind review:
- 28 were judged correct (85%) and 5 wrong; only the 28 were kept.
- 8 of those 28 are Bullet, tolua++ or libc++ names. JJASBR links none of these, so they were dropped.
- 10 of the 20 come from vtable slots (new method `<ref>-vtable`, mostly destructors); the other 10 from the usual
  vote (UI, pad, layer, curl).

Still open: whether the Storm packages have the same task-registrar and dlmalloc errors.

Totals: **140,039 names across the 10 Storm builds**, 144,195 with JoJo All-Star Battle R.

## 1.4.2 (2026-11-16)

**Storm 1 PC: seven Framework and nuccSys names corrected.** Hand-verified against the exe and the Switch Trilogy
build. `Framework::Initialize` calls all of these in the same order as on Switch, and their bodies match the Switch
functions.

| Address | Was | Now |
|---|---|---|
| `0x53c4b0` | (no name; was `AddContentEventId` before 1.3.0) | `Framework::CommonSystemCallback` |
| `0x53c490` | `ccFramework::Internal::NetThreadUtil::PiaDispatchThreaded` (high) | `Framework::AddResidentTaskPlatformDependent` |
| `0x53a410` | a libc++ `std::__tree<…CreateSceneState*…>::__emplace_unique_key_args` (high) | `ccSceneManager::EntryCreateSceneCommand` |
| `0x48d9b0` | `btAxisSweep3Internal<unsigned_short>::Handle::SetNextFree` (high) | `nuccSys::SetInputActiveUser` |
| `0x48d9a0` | `spirv_cross::Compiler::get_remapped_declared_block_name` (high) | `nuccSys::SetFrameRate` |
| `0x48d9d0` | `nuccSys::SetCaptureButton` (medium) | `nuccSys::SetPauseButton` |
| `0x48d910` | `nuccSys::SetCaptureButton` (low) | `nuccSys::SetCaptureButton` (now high) |

- All seven are now high confidence with method `manual`; `conflict_with` keeps the old name.
- Most of the wrong names came from BSim and library matching on very short functions (two stores, or two calls),
  where many unrelated functions look alike. `0x53a410` took its name from the call graph, one step off.
- Storm 1: 13,586 names (6,790 high). Totals: **140,039 names across the 10 Storm builds**, 144,156 with JoJo
  All-Star Battle R. Other builds are unchanged.

## 1.4.1 (2026-10-31)

**JoJo's Bizarre Adventure: All-Star Battle R, 109 more names.** The `jjasbr` set grows from 4,008 to **4,117 names**
(5.6% of 73,928 functions), all high confidence. No existing name changes.

- The new names are matches the 1.4.0 pass rated medium and left out. A second route reached each one with the same
  name: Switch Trilogy to JoJo All-Star Battle R on Switch to JJASBR.exe. They carry the method `jojo-switch-bridge`.
- Accuracy: 15 of 16 sampled names were judged correct in a blind review. The sample is small and both routes use the
  Switch Trilogy's names, so the two are not fully independent.
- Not published: confirmed low-confidence names (6 of 11 correct) and names only this route found (14 of 19 correct).
- The JoJo Switch build has no game symbols of its own: it only served as a stepping stone between the two.

Totals: **140,038 names across the 10 Storm builds**, 144,155 with JoJo All-Star Battle R. Storm data is unchanged.

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
