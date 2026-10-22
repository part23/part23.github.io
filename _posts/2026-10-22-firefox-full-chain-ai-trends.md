---
title: "The Evolution of Firefox Full Chain Exploits and Browser Security in the AI Era"
date: 2026-10-22 16:00:00 +0900
categories: [Exploit, Firefox]
tags: [firefox, spidermonkey, full-chain, ai, wasm-gc, sandbox-escape, ipc, patch-gap]
description: Structural changes in Firefox full chains and the impact of AI-assisted exploitation, based on 306 CVEs, 8 exploit PoCs, 516 GitHub repos, and MFSA 2026-57 through 2026-111
---

This post analyzes the structural changes in Firefox full chain exploits and the impact of AI-assisted exploitation. It is based on 306 public CVEs from 2019 to 2026, code analysis of 8 exploit PoCs, 516 GitHub repositories, and Mozilla security advisories from MFSA 2026-57 (FF152) through 2026-111 (FF158.0.1).

## 1. AI-Assisted Exploitation: A Game Changer

The clearest trend in Firefox security in 2026 is how quickly AI is being adopted on both offense and defense.

### Notable Cases

| Case | AI Model | Result |
|---|---|---|
| Anthropic Mythos | Claude Mythos Preview | Found **271** security bugs in Firefox 150 (180 sec-high) |
| Anthropic Opus 4.6 | Claude Opus 4.6 | Found **22 CVEs**; attempted an exploit for CVE-2026-2796 (2 successes out of 350 attempts) |
| OpenAI Codex Security | GPT-5.6-Sol | **20 CVEs** credited to Amy Burnett (5 releases in a row), **9 High** in FF158 |
| NebuSec VEGA | In-house AI agent | CVE-2026-10702 IonStack full chain (found a JIT bug that Mythos missed) |
| Tencent CodeBuddy | In-house model v2.0 | CVE-2026-100796 Wasm UAF (shared by KeenLab + YUNDING LAB) |

### Key Observation: AI Is the "Bug Finder," Humans Are the "Exploit Builders"

Today, AI's role is concentrated on **finding** vulnerabilities, while **building** exploit chains remains the domain of human researchers.

- **Anthropic**: Claude found 22 CVEs, but only 2 out of 350 exploit attempts succeeded
- **NebuSec**: VEGA AI found CVE-2026-10702, but the IonStack full chain (near-100% reliability) was built by hand
- **SneakyNachos**: Every exploit across 9 Firefox repositories was written by hand (5-stage RCE chain, IPC forgery, GC UAF)

The same pattern appears in XBOW's $250K Chrome full chain — the report that Claude did not independently discover the TerminationException-faking technique supports this.

### Attack Surfaces AI Is Actually Exhausting

```text
High AI collision:
  - JIT miscompilation (Amy Burnett: 2-9 per release)
  - GC UAF (a large share of Mythos's 180 sec-high)
  - Wasm type confusion (three-way race: KeenLab + Amy Burnett + NebuSec)
  - Memory safety in general (Mythos 271 + 6,450+ historical fuzzing bugs)

Not yet reached by AI:
  - IPC semantic bugs (logic bugs with no crash)
  - BrowsingContext state synchronization (pure logic)
  - Networking IPC (new surface in FF158)
  - Gaps between the JS spec and its implementation
```

## 2. Firefox Patch Gap: Where an N-day Becomes a 0-day

The patch gap is a real threat in the Firefox ecosystem too.

| Product / Situation | Gap | Impact |
|---|---|---|
| ESR 115 vs. Stable | 2-4 weeks | **60 of FF158's 77 CVEs are unpatched in ESR 115** |
| ESR 140 EOL | **Permanent** (from FF158) | No more security patches |
| Tor Browser | Days to weeks | **Deanonymization from a single URL visit** demonstrated with CVE-2026-10702 |
| Firefox-based browsers | Weeks | Delayed patching in LibreWolf, Waterfox, and others |
| Bugzilla embargo | Weeks to months | Staggered disclosure of restricted bugs |

**Tor Browser** is a special case: because it is based on Firefox ESR, the delay between a Firefox patch and a Tor Browser patch directly threatens user anonymity. NebuSec's IonStack chain (CVE-2026-10702) demonstrated exactly this.

## 3. Structural Evolution of Full Chains

### 2019-2022: The Classic 3-Stage Chain

```text
Renderer RCE → Sandbox Escape → OS LPE
```

Representative cases:

- CVE-2019-17026 + CVE-2020-0674 (Qihoo 360, the "Double Star" campaign)
- CVE-2024-9680 + CVE-2024-49039 (RomCom APT, Firefox Animation + Windows Task Scheduler)

### 2024-2025: Diversified Attack Surfaces

Driven by Pwn2Own, JIT miscompilation became the workhorse of Stage 1, but achieving a sandbox escape remained hard.

- CVE-2024-29943 + CVE-2024-29944 (Manfred Paul, Pwn2Own 2024, $100K)
- CVE-2025-4918 / CVE-2025-4919 (Pwn2Own Berlin 2025, no sandbox escape)

### 2025-2026: AI + Logic-only + Multi-vector

The Firefox full chains published in 2026 are a structurally new generation.

**Chain A — "PhiNaNigans" (str8outtaheap)**

```text
Wasm GC scalar replacement escape analysis bypass
  → addrof/fakeobj → fake Uint8Array → fcall primitive
  → NaN-boxing IPC (sandbox escape with ZERO memory errors)
  → shared memory vtable hijack → system()
```

The key point is that Stage 2 is a **pure logic bug** (a NaN semantics error).

**Chain B — SneakyNachos**

```text
Wasm call.bind type confusion → funcref forgery → RCE
  + IndexedDB IPC OOB write → parent process heap corruption
  + BrowsingContext IPC forgery (ZERO memory errors, CVSS 10)
```

It has three independent sandbox escape paths, two of which are logic bugs.

**Chain C — NebuSec IonStack**

```text
IonMonkey JIT miscompilation → RCE
  + GhostLock kernel UAF (CVE-2026-43499, 15-year-old Linux futex bug) → root
```

This chain combined an AI-discovered (VEGA) JIT bug with a 15-year-old kernel bug to achieve root on Android 17.

### Key Shifts

**1. More sandbox escapes from pure logic bugs, with no memory corruption**

PhiNaNigans' NaN-boxing IPC and SneakyNachos' BrowsingContext forgery achieve sandbox escapes without any memory errors. This is the same principle by which NSO's FORCEDENTRY used nothing but JBIG2 logic to fully sidestep PAC/MTE.

**Takeaway: pure logic bugs defeat every memory safety mitigation.**

**2. WasmGC dominates Stage 1 of full chains**

Three of the four chains published in 2026 use a WasmGC/Wasm-based Stage 1:

- `array.copy` UAF (kiddo-pwn)
- `call.bind` import type confusion (SneakyNachos)
- Scalar replacement escape analysis bypass (str8outtaheap)
- Subtype validation bypass (CVE-2024-8385)

WasmGC is the new core attack surface, replacing JIT.

**3. IPC is the new core of Stage 2**

Sandbox escapes are shifting from OS kernel LPE to **attacks on the IPC layer**:

- NaN-boxing IPC (PhiNaNigans)
- IndexedDB IPC OOB write (SneakyNachos)
- BrowsingContext IPC forgery (SneakyNachos)
- Networking IPC accounts for 50% of sandbox escapes in FF158

## 4. Standardization of Firefox Exploit Primitives

It is notable that four independent researchers/groups use the same chain of exploit primitives.

| Primitive | Used By | Technique |
|---|---|---|
| addrof/fakeobj | kiddo-pwn, SneakyNachos, str8outtaheap, id2 | Cross-reading i64↔anyref via WasmGC type confusion |
| Fake Uint8Array | kiddo-pwn, str8outtaheap, id2 | Hijack Shape* → replace DATA_SLOT → arbitrary R/W |
| fcall | SneakyNachos, str8outtaheap | Overwrite the FuncImportInstanceData pointer → native call |
| JIT shellcode | kiddo-pwn | IEEE754 float constants → shellcode embedded in JIT code |

**De facto standard chain**: addrof (WasmGC) → fakeobj → fake Uint8Array (DATA_SLOT) → AAR/AAW → fcall (FuncImportInstanceData)

This standardization means that once a new renderer bug is found, the existing primitive infrastructure can be reused to build a full exploit quickly.

## 5. New Attack Surfaces (2025-2026)

| Attack Surface | Severity | Status |
|---|---|---|
| **WebGPU IPC** | Critical | 15 months since being enabled by default in FF141 (2025-07). 7 CVEs in FF157. Unbuffered IPC, weak GPU process sandbox |
| **Networking IPC** | Critical | 4 sandbox escapes (50%) in FF158. PSocketProcessBridge = direct Content→Socket IPC |
| **WasmGC type system** | High | 9+ CVEs in 2 years. The spec keeps expanding |
| **NaN-boxing IPC variants** | Critical | PhiNaNigans demonstrated a full SBX with Bug 2022034. The patch is a point fix |
| **Promise combinator spec↔implementation** | High | CVE-2025-4918 (Pwn2Own $50K). Shared `alreadyCalled` state error |
| **JPEG XL parser** | High | Enabled by default in FF158. See the WebP/FreeType precedents |

## 6. The State of Mitigation Bypasses

Limits of Firefox's mitigations, as demonstrated by the collected chains:

| Mitigation | Bypass Cases | Takeaway |
|---|---|---|
| **Content Sandbox** | NaN-boxing IPC (logic only), IndexedDB OOB write, BrowsingContext forgery | Three independent bypass paths |
| **Fission (Site Isolation)** | CVE-2026-109705 (Networking + site isolation bypass) | Bypassed via Networking |
| **JIT Hardening** | CVE-2026-109721 (guard/fold mismatch) | A systemic weakness |
| **GC Safety** | CVE-2026-84118 (`MightBeForwarded<Symbol>` hardcoded to false) | Asserts are disabled in release builds |
| **Fuzzing** | Bug 2024918 (passed years of fuzzing → found by Mythos) | Logic bugs can't be found by fuzzers |

Just as FORCEDENTRY used JBIG2 logic alone to make PAC/MTE completely irrelevant, in Firefox too **logic bugs defeat every memory safety mitigation**.

## 7. How AI Is Changing the Future of Browser Security

### Offense

- **Compressed exploit development**: AI discovery → manual chain building cuts months down to weeks
- **Finding bugs that decades of fuzzing missed**: Bug 2024918 passed years of fuzzing, but Mythos found it
- **Faster variant analysis**: Amy Burnett's 9 High CVEs across 6 components in FF158 = systematic scanning
- **Growing AI contribution with every release**: FF153 (3) → FF154 (5) → FF157 (2) → FF158 (**9**)

### Defense

- Mythos drove a batch fix of 423 bugs in FF150, the most ever in a single release
- Mozilla OneCRL responded to the eMudhra CA certificate compromise within 3 hours
- From FF156, the switch to issuing individual CVEs increased transparency

### Predictions

1. By 2027, AI-assisted full chains will be more common than manually developed ones
2. As AI exhausts memory safety bugs, **logic bugs will rise to become the highest-value targets**
3. The WasmGC type system will grow into the core Stage 1 attack surface, replacing JIT
4. Networking IPC will replace DOM IPC as the next major sandbox escape vector

## 8. Firefox Attack Patterns by the Numbers

### Most Attacked Components (FF155-158)

| Component | Public CVEs | Representative Exploits |
|---|---|---|
| JS Engine: JIT | ~20+ | CVE-2026-109721 (16GB OOB), 9 from Amy Burnett |
| JS: WebAssembly | ~15+ | CVE-2026-2796 (full RCE), IonStack (Android root) |
| DOM: Core & HTML | ~15+ | Aldurayhim: ~27 CVEs / 12 sandbox escapes |
| Graphics: WebGPU | 7 (FF157) | CVE-2025-12380 SBX, 8 from 5up3rh3i |
| Networking | 8+ (FF158) | 4 SBX (50%) |

### Research Groups Publishing High-Quality Public Exploits

| Researcher | Key Contributions |
|---|---|
| SneakyNachos | 9 Firefox repos, 3 SBX paths, full chain code fully published |
| str8outtaheap | PhiNaNigans full sandbox escape chain + 4-part "What The Claude" writeup |
| NebuSec | IonStack full Android root chain + 3-part blog series |
| kiddo-pwn | Pwn2Own Berlin 2026 WasmGC renderer RCE + published code |

## Conclusion

The Firefox exploit ecosystem in 2026 is going through three structural shifts.

**First, AI has democratized vulnerability discovery.** Anthropic Mythos's 271 bugs and OpenAI's Amy Burnett with 9 per release show that a division of labor between human researchers and AI is the new norm. Building exploit chains, however, is still a human job.

**Second, pure logic bugs have emerged as the highest-value attack vector.** PhiNaNigans' NaN-boxing IPC sandbox escape and BrowsingContext forgery (CVSS 10) are achieved with no memory errors. As AI rapidly exhausts memory safety bugs, what remains are logic bugs that neither fuzzers nor AI can detect.

**Third, WasmGC and IPC are the new pillars of the full chain.** WasmGC is taking over from JIT as the Stage 1 attack surface, and IPC is replacing the OS kernel as the Stage 2 vector. In Firefox's multi-process architecture, the security of the IPC layer has become central to the security of the entire browser.
