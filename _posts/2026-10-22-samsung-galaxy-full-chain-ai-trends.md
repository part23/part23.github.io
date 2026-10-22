---
title: "Samsung Galaxy Full Chain Exploits: Trends and Security Threats in the AI Era"
date: 2026-10-22 15:30:00 +0900
categories: [Exploit, Samsung]
tags: [samsung, galaxy, full-chain, ai, pwn2own, ghostlock, dirtyfrag, patch-gap, knox]
description: Cross-analysis of 1,448 CVEs, 10 years of Pwn2Own data, Project Zero's codec campaign, and 20+ public root repositories targeting Samsung Galaxy
---

This post summarizes the Samsung Galaxy threat landscape as of October 2026, based on a cross-analysis of 1,448 CVEs, 10 years of Pwn2Own data, Google Project Zero's 14-month codec campaign, and more than 20 public root repositories.

## 1. AI Is Changing Samsung Attack Research

2026 is the first year AI directly shaped Samsung security research.

| Case | Role of AI | Result |
|---|---|---|
| **Team ZyGoat** (Pwn2Own Ireland 2026) | djini.ai — AI mobile SAST | Galaxy S26 0-day, $12,500. **Zero memory corruption**, a pure logic flaw |
| **Nebula Security VEGA** | AI kernel scanner | Found CVE-2026-43499 (GhostLock) — a futex UAF dormant for 15 years. The origin of the Samsung root ecosystem |
| **Calif.io OEMpocalypse** | Credited to "Claude and Anthropic Research" | CVE-2026-21102 (DualDAR) — **the first time AI collaboration appeared as an official credit** in a Samsung SMR |

**Key points:**

- ZyGoat used AI-assisted tooling to find a 0-day in the latest flagship and submitted the **only unique (non-colliding) bug** at Pwn2Own.
- The VEGA AI scanner found a kernel bug humans had missed for 15 years, which exploded into a root ecosystem with 1,451 GitHub stars in four months.
- AI is strong at finding logic flaws, but kernel memory corruption exploit development is still the domain of human experts.

## 2. Samsung's Patch Gap Problem

### Measured Patch Gaps

| Component | Patch Gap | Real-World Impact |
|---|---|---|
| Samsung Internet V8 | ~6 months | V8 patch gap PoC demonstrated at DEF CON 34 |
| GhostLock (CVE-2026-43499) | ~3 months | 1,451 GitHub stars, ported to 20+ devices |
| DirtyFrag (CVE-2026-43284) | ~3 months | Reported 100% success rate over 50 consecutive runs |
| Samsung in-house library (libsavscmn.so) | ~6 months | Reported in 2025 → patched 2026-05 |
| Samsung Continuity Service (PC) | ~12 months | Separate update cycle outside the SMR |

### Samsung CNA vs. NVD — A Systematic Scoring Gap

There is a consistent gap between the CVSS scores Samsung assigns as a CNA and the scores from NVD's analysis.

| CVE | Samsung CNA | NVD | Gap |
|---|---|---|---|
| CVE-2025-21074 (Quram OOB read) | 4.3 | **7.5** | +3.2 |
| CVE-2025-21075 (Quram OOB write) | 4.3 | **7.5** | +3.2 |
| CVE-2025-21055 (Quram R/W, linked to TAG) | 4.0 | **7.8** | +3.8 |
| CVE-2025-21042 (LANDFALL, ITW) | 8.8 | **9.8** | +1.0 |

**Pattern:** Samsung underrates remote media parser bugs by +3.0 on average. Samsung sets `UI:R` (user interaction required), but an automatic media scan path (the `MEDIA_SCANNER_SCAN_FILE` broadcast) exists, so NVD rates them `UI:N` (no interaction required).

Enterprises that prioritize patches using Samsung CNA scores alone may be left exposed to remote media parser attacks.

## 3. How Samsung Full Chains Have Evolved

### By Era

```text
2015-2019: The baseband era
  Shannon Stack BOF → Kernel
  ★ Requires a rogue BTS, proximity attack

2020-2022: The app logic era
  Galaxy Store deeplink → silent install
  ★ No memory corruption, remote 1-click

2023-2024: Media parsers in the wild
  Quram DNG opcode → JOP → system() (LANDFALL)
  Samsung Account → Bixby IPC → system (Pwn2Own 2025)
  ★ 0-click path proven, commercial spyware confirmed

2025-2026: AI + logic-only + hardware bypasses
  AI-discovered UAF → weaponized by the community (GhostLock)
  Logic Bug × 1 → System (Pwn2Own 2026 S26)
  Page Cache Write → bypasses every layer: KDP/BTI/PAC/SELinux
  ★ From PoC to automated platform in 4 months
```

### Catalog of Real-World Full Chains

| Chain | Year | Memory Corruption | Attack Vector | Impact |
|---|---|---|---|---|
| **LANDFALL** | 2024-25 | Heap (DNG opcode) | WhatsApp 0/1-click | ITW commercial spyware |
| **Bixby Chain** | 2025 | All logic (5 bugs) | 1-click URL | $50K at Pwn2Own |
| **IonStack** | 2026 | JIT + Kernel UAF | 1-click URL | Full root in 10 seconds |
| **ZyGoat S26** | 2026 | None (1 logic bug) | Remote | $12.5K at Pwn2Own |
| **DirtyFrag-Galaxy** | 2026 | Page Cache | App only (no PC needed) | Root with 100% success rate |
| **AT Command Chain** | 2026 | All logic (3-stage) | Physical | File write as system_server |
| **DualDAR** | 2026 | Kernel UAF | Local | Kernel root, CVSS 9.3 |
| **ERISED** (NSO) | 2019-20 | Unknown | WhatsApp remote | Surveillance of 725 devices |

### The Most Important Finding: 100% Logic-Only Chains in the Pwn2Own App Layer

| Period | Memory Corruption | Logic-Only | Ratio |
|---|---|---|---|
| 2015-2019 (Baseband) | 5 | 1 | 83:17 |
| 2020-2024 (App layer) | 0 | 6 | **0:100** |
| 2025-2026 | 0 | 2 | **0:100** |

**Since 2020, not a single successful Pwn2Own attack on the Samsung app layer has relied on memory corruption.** No matter how much MTE, CFI, and PAC are hardened, that reality does not change.

## 4. The Exploit Supply Chain

Tracing how Samsung exploits flow from discovery to final use reveals a clear pattern.

```text
[Discovery]              [Weaponization]          [Distribution]       [End Use]

NSO ERISED ────────────→ WhatsApp delivery ─────→ 725 devices surveilled ← State actors
P0 Quram audit ─────────→ (defensive disclosure) → LANDFALL patched       ← ITW stopped
VEGA AI → GhostLock ──→ Root-My-Galaxy ──→ KSuRoot → 1,451★              ← Community
          ↓                   ↓              ↓
    diabl0w DFRoot ──→ GhostSam ──→ DirtyFrag-Galaxy → Coolapk            ← Chinese community
          ↓                                  ↓
    GalaxyCrash (stability) ──────→ debian-on-s23 (server) → Homelab      ← Non-experts
```

**Notable points:**

- Commercial spyware vendors (NSO, Cellebrite) explicitly target Samsung Galaxy.
- An AI-discovered bug (GhostLock) evolved into an automated root platform in four months.
- The Chinese community is the main driver in removing the PC dependency and expanding across vendors.
- Even when the original author deletes their account, the fork network preserves the code (the coey0814 case).

## 5. Eight New Attack Surfaces

Samsung-specific attack surfaces identified in the second half of 2026:

| Attack Surface | Representative CVE | Severity | Notes |
|---|---|---|---|
| **HEIF decoder** (`libsimba.heifdec`) | CVE-2026-21124/21125 | High (8.6) | Found independently by P0 and an independent researcher (RTV) at the same time. Parsed automatically by Gallery |
| **MKV extractor** (`libsmkvextractor.so`) | CVE-2026-21114 | High (7.0) | First appearance. Found by KAIST Hacking Lab |
| **TTS engine** (`libsamsungtts.so`) | CVE-2026-21122 | High (7.0) | First appearance. Reachable via Accessibility / Bixby Voice |
| **AI sound separation** (`lib_sag_ai_sound_sep`) | CVE-2026-21119 | Medium | AI feature exclusive to Android 16/17 |
| **Android XR** (CmcCore) | CVE-2026-21121 | Medium | First XR-specific vulnerability |
| **WSM service** | CVE-2026-21120 | High (8.9) | Possible Knox Workspace isolation bypass |
| **DualDAR kernel driver** | CVE-2026-21102 | Critical (9.3) | A pattern where Knox security features become attack surface |
| **IpSecManager API** | (No CVE assigned) | Critical | Kernel page cache tampering with ordinary app permissions |

### How the Media Parser Attack Surface Is Spreading

```text
2024:   Quram DNG/JPEG ━━━━━━━━━━━ LANDFALL ITW
          ↓
2025:   Quram continues (9 from P0) ━━━━━━ TIFF
          ↓
2026H1: VC1/MPEG4/FLAC/QCELP
          ↓
2026H2: ★HEIF ━━━ ★MKV ━━━ ★TTS ━━━ ★AI sound separation
          ↓
          [Audio codecs: P0 work in progress]
```

As Quram becomes saturated, researchers are spreading out to less-audited parsers.

## 6. Samsung's Hardware Security Layers — Public Bypasses Exist for All of Them

As of October 2026, public bypass research exists for every major security layer on Samsung Galaxy flagships.

| Mitigation | Bypass Project | Key Technique |
|---|---|---|
| **MTE** | MTEscape (KAIST, Hexacon 2026) | Neutralizes tags via general memory corruption under asynchronous MTE |
| **BTI/PAC** | DFRoot (Oct 15 commit) | Runtime check of `system_supports_address_auth()`, then conditionally emits RET |
| **RKP/KDP** | GhostSam | Masquerades as `call_usermodehelper_exec_work` — leaves `struct cred` untouched to evade KDP EL2 monitoring |
| **SELinux** | DFRoot (Oct 16 commit) | Patches `avc_has_perm()` directly — handles production kernels with `CONFIG_SELINUX_DEVELOP=n` |
| **DEFEX** | GhostSam | Bind-mounts `ksud` over a dormant system binary → bypasses the path whitelist |
| **Knox e-fuse** | Root-My-Galaxy | No partition changes, RAM only → restored on reboot → e-fuse never trips |

### Timeline of Bypass Techniques

```text
2026-06: GhostLock disclosed (AI-discovered)
    ↓
2026-07~08: Per-device ports (KASLR bypass)
    ↓
2026-09: DFRoot integration (DirtyFrag + GhostLock)
    ↓
2026-10-15: BTI/PAC bypass ←── ARM hardware CFI
2026-10-16: SELinux bypass on production kernels
2026-10-17: MTEscape presented at Hexacon ←── ARM MTE
    ↓
2026-10-19: Stability forks appear (code review stage)
    ↓
[Prediction] 2026 Q4 ~ 2027 Q1: A chain that bypasses all four layers together
```

And **none of these hardware mitigations matter against logic bug chains:**

| Mitigation | LANDFALL | Bixby Chain | ZyGoat S26 | AT Command Chain |
|---|---|---|---|---|
| MTE | Irrelevant | **Completely irrelevant** | **Completely irrelevant** | **Completely irrelevant** |
| PAC | Irrelevant | Completely irrelevant | Completely irrelevant | Completely irrelevant |
| CFI | Irrelevant | Completely irrelevant | Completely irrelevant | Completely irrelevant |

Against logic flaws in the Samsung app layer, MTE, PAC, and CFI provide zero protection. The only defense is getting input validation and the IPC permission model right.

## 7. The Root Ecosystem: From PoC to Automated Platform in Four Months

### Growth Timeline

```text
2026-06-24  NebuSec IonStack demo
2026-07-07  GhostLock disclosed
2026-07-17  Root-My-Galaxy v0.1
2026-09-25  DFRoot (DirtyFrag integration)
2026-10-09  DirtyFrag-Galaxy (no PC needed, on-device)
2026-10-15  DFRoot BTI/PAC bypass
2026-10-21  Root-My-Galaxy reaches 1,451★
```

### Current Size of the Ecosystem

- **20+ Samsung-specific repositories**
- **14+ device families covered**: S22U through S26U, Z Fold4/6/8, A17/A37/A56, M55, Tab S7, Note10
- **Kernel range**: 5.10 ~ 6.18 (Android 12 ~ 17)
- **Chinese community contributions**: GhostSam (55 stars, integrated manager), KSuRoot (127 payloads, cross-vendor), DirtyFrag-Galaxy (no PC needed)

### Five Levels of Technical Maturity

| Level | Tools | Characteristics |
|---|---|---|
| 1. Research PoC | NebuSec original | Single device, manual offsets |
| 2. Per-device ports | Various CVE-2026-43499 ports | Optimized for a single device |
| 3. Automated tools | Root-My-Galaxy (1,451 stars) | Multi-device, 1-click UI |
| 4. Integrated frameworks | GhostSam, DFRoot (457 stars) | Dual exploits, open source |
| **5. Fully automated platforms** | **KSuRoot (127 payloads), DirtyFrag-Galaxy** | **Cross-vendor, automatic firmware matching, no PC needed** |

In four months, it evolved **from a research PoC into a cross-vendor automated platform with 127 payloads**.

## 8. How AI Will Shape Samsung Security Going Forward

### Offense

| Area | Today (2026-10) | Prediction (2027) |
|---|---|---|
| Logic bug discovery | Proven by ZyGoat at Pwn2Own | AI automatically explores IPC in Samsung preloaded apps |
| Kernel bug discovery | VEGA found GhostLock | AI fuzzers automatically audit DualDAR, PROCA, and GPU drivers |
| Variant analysis | P0 found 18 Quram DNG bugs manually | AI generates variants automatically; small teams reach P0-level coverage |
| Root tool automation | KSuRoot parses boot.img | AI handles firmware release → offset extraction → payload distribution with no human involvement |

### Defense

| Area | Today | Needed |
|---|---|---|
| Message Guard | DNG/image sandboxing | Extend to HEIF, MKV, and audio parsers |
| Knox Attestation | KernelSU module detection | Add IpSecManager SA, workqueue UMH, and page cache integrity checks |
| SMR delivery | Monthly, delayed by carriers | Emergency OTAs driven by AI threat prediction |
| IPC auditing | Manual code review | AI automatically audits exported components across all preloaded apps |

## 9. The Triple Threat Facing Samsung

Samsung Galaxy is exposed to three very different threats **at the same time**.

**1. Commercial spyware** (NSO, LANDFALL): Remote attacks on Samsung media parsers via WhatsApp and similar apps. A 0-click path has been proven.

**2. Logic flaw chains** (Pwn2Own researchers): Abuse trust relationships in the IPC of Samsung preloaded apps. **MTE/PAC/CFI are completely irrelevant**; the only defense is code quality.

**3. An automated root ecosystem** (GhostLock/DirtyFrag): Grew into a 1,451-star platform in four months. Even after patching, DFRoot's BTI/PAC/SELinux bypass techniques can be reused immediately with the next kernel bug.

## Most Attacked Samsung Technology Stacks

| Rank | Technology Stack | Key Researchers |
|---|---|---|
| 1 | **Quram image codecs** (DNG/JPEG/TIFF) | Google P0, TAG, ZDI, KAIST |
| 2 | **Preloaded app IPC** (Members/Account/Bixby/Smart Switch) | Valsamaras/Gannon, Oversecured (176 bugs), LucidBit |
| 3 | **Linux kernel** (futex/xfrm/page cache) | NebuSec/VEGA, diabl0w, snothin |
| 4 | **Exynos GPU/NPU** | P0, STAR Labs, halal-beef |
| 5 | **HEIF/MKV/TTS parsers** (new) | P0, RTV, KAIST |
| 6 | **Knox kernel defenses** (PROCA/DualDAR) | LucidBit, Calif, dayzerosec |
| 7 | **Shannon baseband** | TASZK, P0, STAR Labs |

## Top 8 Samsung Research Groups

| Rank | Group | Highlights |
|---|---|---|
| 1 | **Google Project Zero** | 14-month systematic codec campaign, the gold standard |
| 2 | **Ken Gannon / djini.ai** | Pwn2Own S24-S25-S26 three years in a row, builds AI tooling |
| 3 | **KAIST Hacking Lab** | MKV parser + Pwn2Own S26 + MTEscape, academic and practical |
| 4 | **Oversecured** | Bulk disclosure of 176 bugs in Samsung preloaded apps |
| 5 | **LucidBit Labs** | Found the "defense becomes attack surface" pattern (PROCA, Smart Switch) |
| 6 | **snothin** | GhostSam KDP bypass, production-grade exploit code |
| 7 | **diabl0w** | DFRoot BTI/PAC + SELinux production bypass |
| 8 | **STAR Labs SG** | Exynos coprocessor specialists |
