# QEngine — SweetSixteen bench

A **bench fork** of [bpx03/QEngine](https://github.com/bpx03/QEngine) (MIT — upstream's licence and headers are kept
intact). It is not a product, not a supported build, and nothing here is served in anger: the estate that runs this
bench serves the **upstream** images.

This fork exists to answer one narrow question — **what does QEngine do, and what would it take to make it good, on a
4× RTX 5080 16 GB machine with a 122 GiB host** — a class upstream's author has said plainly is not his target.

### The hardware under test

| | |
|---|---|
| cards | 4× RTX 5080 16 GB (sm_120), Gen5 ×8 through a PEX89048 switch |
| host | 125 GiB RAM (≈122 GiB usable), AVX2 CPU (no AVX-512), NVMe local storage |
| engine | upstream releases only, pulled by digest; every result names its build |

### What is measured, and what is only a hypothesis

**Measured on this class** (numbers live in the community bench repo, [noonghunna/club-3090](https://github.com/noonghunna/club-3090)):

- **GLM-5.3-Flash does not fit.** Its routed experts are 158.2 GB; the engine pins the full set in host RAM, and the
  kernel OOM-kills the server at ~120.7 GiB before any host-RAM plan line prints. Not a tuning problem — the loader has
  no fallback path. *(club-3090 #2130 / this fork's notes.)*
- **DeepSeek-V4.1-Flash runs on sm_120** although 1.5's notes say the class is unsupported: it boots, warms cuBLAS and
  serves — at roughly a quarter of the 3× 3090 figures, on a local NVMe copy of the GGUF set. *(club-3090 #2130.)*
- **Why DSpark doesn't pay here.** Per-row draft acceptance on this box is **0.67–0.84** — the drafter is fine. A
  verified **row** costs ~**0.53** of a plain decode step here, against **0.29** upstream's own cost table measures on
  24 GB cards. A drafter pays when acceptance exceeds row cost; here they meet, and the result is a wash.
  *(club-3090 #2130, comment 7060505978.)*
- Qwen3.8-27B and Flash-Next at TP4 and TP3 on 1.5.0, with the full instrument chain. *(club-3090 #2001, #1966.)*

**Measured, and two of them settled the question they were opened for:**

- **`sm_120` builds buy LOAD TIME, not decode rate.** The `dsv4` engine ships compiled for `sm_86` only
  (`src/dsv4/build.sh` hard-codes `-arch=sm_86` on every `nvcc` line), so on these cards it runs 3090-targeted code.
  Rebuilt from the same commit at `sm_120` and booted four times: boot → `/health` **47–51 s → 24 s**, decode
  **−2.3 % / −2.9 %** — inside a 6.9 % spread, so no rate claim, and the build is kept for its load time.
  (`gpt-oss` has the same build shape and was not rebuilt.)
- **A fourth card works for `dsv4`.** Generalising `NC` from its hard-coded three: 3 → 4 cards boots clean, residency
  **2 576 → 3 880** expert slots (**1.51×**), row cost **−13…−22 %**, decode **+15.7…+20 %** — on a c=1,
  prefix-reused, long-reply shape. It is the last cheap step on this box, and a *shape* result, not a general one:
  a cold-document lane above ~19K prompt tokens would prefer pipelined prefill end-to-end instead.
- **The four-card decode step runs as one captured CUDA graph, and the served text does not move.** The graph was
  gated to exactly three cards (`E->graph = … && E->cards.size() == 3`) and the head split *inside* it was two-wide —
  two peer stores and a two-word reduce — so the gate hid the limitation: the assertion inside that path could never
  fire at four cards. Generalised to N−1 peers (and the gate opened to 2–4 cards), the graph captures at four cards
  with the drafter **off and on**, and serves the **same greedy text** as the eager path: the same six-prompt identity
  set on both arms, a 512-token reply byte-identical, zero CUDA faults across boots. Its rate is **+2.4 %** inside a
  3.9–5.2 % within-arm spread — a correctness result, not a speed lever. Branch: `ss16/nc4`.
- **Two engine facts, each reproducible in a minute from the shipped image.** `--hsplit` is parsed by the CLI
  (`src/dsv4/dsv4_main.cpp`) and **absent from the C API** (`src/dsv4/dsv4_capi.cpp`), so an arm that passes it
  through the server dies at `dsv4_create: unknown argument --hsplit` — unreachable through the only door the server
  has, rather than merely unset. And a greedy reply at four cards is a function of its **states**, not of the prompt
  alone: the drafter on against off, and the per-card expert tier at two different slot counts, each move the served
  text while everything else is held. A four-card number that does not name those states is not a cell.
- **Two cards serves now — and is servable, not reproducible, which is its own result.** Opening the gate to 2–4 cards
  put `--nc 2` inside it, and it crashed: `attn_share()` fills its peer list from the cards that exist and then read
  the *second* peer unconditionally, so with one peer an uninitialised stack index became a garbage segment descriptor
  and a wild source pointer — an unspecified launch failure on the first request, identically on the library built
  before this port. Guarding the two-peer launch and giving one peer the single-source pull is the whole fix (branch
  `ss16/nc2`): the first request now answers 200, zero CUDA faults across thirteen requests, and **three and four
  cards serve their filed identities unchanged**. At that width the tier is 1 240 slots over 40 layers and **four of
  six identity prompts take different text across two draws**, so two cards is a width that runs, not a serving shape:
  no rate, quality or identity cell is claimed for it.
- **Upstream 1.5.1 (2026-11-22) changes Flash-Next only, and it is the fix this fork had already reproduced.** Its two
  notes — `FLASH_CKPT=0` no longer takes SIGFPE at a divergence checkpoint, and the archive returns under its budget
  after a pinned prefix is dropped — are the two defects reproduced here (#40, #41); upstream credits the issues, and
  says the other models are unchanged. So every number above still describes the same code for the models it names.

### Branch policy

One candidate per branch, rebased on upstream's `main`, each carrying: the change, the measurement that justifies it,
and a **no-change-at-N=3** proof — byte-identical greedy replies on a frozen payload set from the pre- and post-patch
build at three cards. A change that cannot be expressed as a patch on upstream stays an experiment on its own branch
and is not merged into this fork's `main`. If a change lands upstream, this fork deletes the branch and points at his
commit.

**One branch is already in that state.** The two Flash-Next defects reproduced here are fixed in upstream **1.5.1**;
the defect branch keeps the reproduction and its probe scripts and points at his commit for the fix, rather than
competing with it.

### Read this before filing anything here

Upstream is the product; **issues and PRs belong on
[bpx03/QEngine](https://github.com/bpx03/QEngine)**, not here. This fork's issue tracker is not watched — it exists so
the diffs, the harness commands and the raw logs next to them stay public and citable.

*Bench build, MIT, upstream's licence retained. Nothing here is a support offer.*

---
