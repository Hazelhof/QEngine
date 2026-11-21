# dsv4 at `sm_120`: no decode-rate change, half the start-up

**Bench note, 2026-11-21.** Branch `ss16/dsv4-arch` on this fork; one commit, one file (`src/dsv4/build.sh`, +11/−9).
The default build is unchanged (`DSV4_ARCH` unset ⇒ today's `sm_86`), so this branch is inert unless you ask for the arm.

**This is a bench build, not a product, and the notes are not a support offer.** Upstream `bpx03/QEngine` is the
product; nothing here has been sent upstream, and this branch exists so the measurement and the diff sit in public next
to each other. Upstream's licence and headers are untouched.

Measurement host: 4 × RTX 5080 16 GB, PCIe Gen5 ×8 behind a Broadcom/LSI PEX890xx Gen 5 switch, peer access OK on all
pairs, driver 610.57.04, CUDA 13.1, 125 GiB host RAM, power caps at the 252/250 W vBIOS floor. Weights on local NVMe.

---

I built the DSV4 engine for `sm_120` from your 1.5.0 tree, ran it beside an identical `sm_86` control, and measured both
on four RTX 5080 16 GB cards. Three findings, one of which is a build-matrix gap rather than a speed result.

**Rig, in full** (so a reader can place the numbers): 4 × RTX 5080 16 GB, compute 12.0, each link at **PCIe Gen5 ×8** behind a
**Broadcom/LSI PEX890xx Gen 5 switch** (`1000:c034`) — ×8 is the switch's lane budget, and the club harness flags it
against a ×16 slot maximum, which is why the card carries that warning; peer access granted on every pair (`topo -p2p r` = OK,
`topo -m` = PIX within each pair, PXB across); host = 24-thread Intel Core Ultra 7 270K, 125 GiB RAM; driver 610.57.04,
CUDA 13.1 runtime in the image; power caps **252/250/250/250 W**, which are the vBIOS floors (default ceilings 360 W), so
every number above is at the floor. Weights on local NVMe, never a network mount — the engine reads expert rows per
routing decision.

**1. `src/dsv4/build.sh` is the only thing keeping the previews off sm_86.** Every nvcc line in that 40-line script
hard-codes `-arch=sm_86` (nine of them), and `Dockerfile:4` says so — *"the DSV4 and gpt-oss engines sm_86"* — while
`ARG CUDA_ARCHS="86;89;120"` reaches only the CMake stage. `src/gptoss/build.sh:8` is the same on its single `NV=` line.
So of the five engines in the image, **three carry sm_86/89/120 (`libqengine.so`, `libqflash.so`, `libqglm.so`) and two
carry sm_86 only, and the difference is that `qengine`/`qflash`/`qglm` are CMake targets while `dsv4`/`gptoss` are not**
(`grep -c "dsv4\|gptoss" CMakeLists.txt` = 0). GLM arrived in the same release as the two previews and got the arch list
for free because it went into CMake; the two script-built engines did not. Worth noting for the docs: `cuobjdump` on the
shipped `libqdsv4.so` shows **four `.sm_86.cubin` ELFs and three `sm_86` PTX files** — so on a 50-series card the engine
is running **driver-JITed compute_86 PTX**, which is why it works at all on a class the README excludes.

**2. A one-variable change, and what it buys.** Patch (branch below, one file, +11/−9):

```bash
 OUT=${OUT:-../../build_dsv4}
+# GPU code target: DSV4_ARCH overrides it (e.g. DSV4_ARCH=sm_120); the default is today's shipped behaviour.
+ARCH=${DSV4_ARCH:-sm_86}
```
…with `-arch=sm_86` → `-arch=$ARCH` on the nine sites. With `DSV4_ARCH` unset the nvcc command lines are byte-identical
to the current build, so nothing changes for your toolchain.

Built both arms in `nvidia/cuda:13.1.1-devel-ubuntu24.04` (the Dockerfile's own builder; nvcc 13.1.115, GCC 13), same
tree, same flags, `OUT=` per arm. **Both compile with zero errors**; the libraries differ exactly where they should:

| library | size | `cuobjdump` | exported `dsv4_*`/`qe_*` |
|---|---:|---|---:|
| shipped `libqdsv4.so` (1.5.0) | 17 984 272 B | 4 ELF `sm_86` + 3 PTX `sm_86` | 36 |
| control, rebuilt at the default (`sm_86`) | **17 984 272 B** | 4 ELF `sm_86` + 3 PTX `sm_86` | 36 |
| arm, rebuilt at `sm_120` | **17 541 904 B** (−2.5 %) | 4 ELF `sm_120` + 3 PTX `sm_120` | 36 |

The control reproduces the shipped library's byte size and arch census exactly (different sha, as expected from a
different link/build id), and all three export the same 36 names — so the substitution is like-for-like and no optional
entry point is silently missing.

**3. Measured: no decode-rate change; start-up halves.** Both arms boot the **unmodified 1.5.0 image**
(`ghcr.io/bpx03/qengine@sha256:6c96a7c6…`) with the engine library substituted through `QE_DSV4_LIB` — the server, the
Python side, the weights and the cache are identical, and a stamped `qe_build.txt` beside each library makes the boot
banner name the arm. Shape on every boot: `served 262144`, 8 slots, KV `fp8_e4m3`, `host RAM plan: needed 99.23 GiB …
budget 99.24 GiB`, three cards with the fourth idle. `bench.sh CARD=snapshot`, 3 warm + 5 measured per shape,
`temperature=0.6 top_p=0.95`.

| boot | narrative tok/s (CV) | code tok/s (CV) | prefill 10K / 90K tok/s | boot → `/health` |
|---|---|---|---|---|
| control 1 | 27.56 (5.6 %) | 39.00 (5.5 %) | 1 024.6 / 2 842.4 | **48 s** |
| arm 1 | 28.25 (2.6 %) | 39.83 (1.5 %) | 1 028.0 / 2 806.5 | **24 s** |
| arm 2 | 26.41 (3.4 %) | 36.85 (3.9 %) | 883.3 / 2 693.3 | **22 s** |
| control 2 | 28.37 (2.0 %) | 39.94 (4.8 %) | 945.1 / 2 721.6 | **51 s** |
| shipped library (no `QE_DSV4_LIB`) | 28.04 (2.7 %) | 39.30 (4.8 %) | 1 034.9 / 2 760.7 | **47 s** |

- **Pairing 1** (control → arm): **+2.50 % narrative, +2.13 % code**. **Pairing 2** (arm → control): **−6.93 % /
  −7.74 %**, i.e. the sign flips. Means: control **27.97 / 39.47**, arm **27.33 / 38.34** → **−2.3 % / −2.9 %**.
- In **both** pairs the *later* boot was faster (~2 %, then ~8 %) regardless of which arm it was — the position effect
  is larger than the difference, and the within-arm spread (up to 6.9 %) is too. Prefill agrees: −3.0 % at 10K
  (CVs 8–22 %), −1.2 % at 90K. **I do not claim an effect in either direction; I claim none at or above ~8 %,** which is
  twice the CV of my filed cells.
- **Start-up is the real difference: 47–51 s → 22–24 s**, across three control/reference boots and two arm boots.
- **And the JIT is paid at load, not on the first request.** A cold probe (boot, then a 12 942-token request as the
  first traffic, then the same request with a fresh nonce so the prefix cache cannot answer it):
  **control first 15.20 s / second 15.42 s; arm first 15.20 s / second 15.57 s** — both 851 tok/s to three digits, ratio
  ≈ 1.0 on each arm. So prefill, TTFT and first-token latency are unchanged; the win is purely before the server answers.

**4. Two instrumentation gaps this cost me, offered as observations rather than asks.** (a) `src/flash/card_probe.cu`
already prints `[box] build …, CUDA archs …, module loading EAGER/LAZY` and a per-card verdict — the source comment is
*"native code of the build … or PTX JIT-compiled at start"*, with a `GPU%d sm_%d no code in the build` branch. `src/dsv4`
has no equivalent, so the question "is this card JIT-ing?" is unanswerable from a dsv4 boot and had to be settled with
`cuobjdump` on the library. (b) `QE_GIT_HASH`/`qe_build.txt` are CMake-only (`CMakeLists.txt:53–54, 80–82`), so a
script-built engine reports `build unknown` — I supplied a `qe_build.txt` beside each library to tell my two arms apart.

**5. A correctness observation, stated as an observation.** On this shape, **greedy output is not reproducible within a
single boot**: with the full sampler set (`temperature 0`, `top_p 1.0`, `top_k 0`, `min_p 0.0`, both penalties 0) and
4 draws per payload, **2 of 6 payloads produced 2–3 distinct completions**, and `DSV4_NOCPUPF=1` — the documented switch
whose purpose is exactly *"greedy text no longer depends on `--cpu-threads`, `--max-ctx` or prompt chunking"* — did not
stabilise them. The other 4 payloads were stable and identical across both builds. So the instability looks like the
decode-time MoE path rather than prefill; I could not separate it further, and it means an output-identity check cannot
be written naively on this engine.

**What I did NOT measure** — and it matters for how far to trust the above: I own **no 24 GB card**, so the control is a
rebuilt `sm_86`, not your 3090 shape, and nothing here says your hardware is unaffected (the default keeps `sm_86`, and
that is the argument, not this bench). No 8-pack, no quality leg, no ceiling ladder, no concurrency: this is a rate and
start-up comparison, not a row. n=2 boots per arm. Prefill CVs (8–22 %) are outside the club's ≤2 % norm on both arms.
One control boot is excluded because I ran another client alongside it (declared in my ledger, not silently dropped).
And the **shipped library read 28.04 / 39.30 on the night of the test against 23.15 / 34.97 filed earlier the same day**
— a ~19 % host-state swing on identical bytes, which is why the reference boot is in the table at all: any comparison
against an older filed cell on this box needs a same-session reference.

**Offer, not a promise:** if you want the arm as a shipped fatbin (`code=[sm_86,sm_120]`) rather than my single-arch
bench build, the same variable takes a list — that is a different build shape (double the SASS, different load cost) and
I would rather ask which you want than assume.

*(Bench build, not the released image; your licence and headers retained; nothing here is a support offer.
Branch and patch: `ss16/dsv4-arch` on the fork, one commit, one file.)*

---

## The diff

```diff
@@ -3,6 +3,8 @@
 set -e
 cd "$(dirname "$0")"
 OUT=${OUT:-../../build_dsv4}
+# GPU code target: DSV4_ARCH overrides it (e.g. DSV4_ARCH=sm_120); the default is today's shipped behaviour.
+ARCH=${DSV4_ARCH:-sm_86}
 mkdir -p $OUT
 CU=/usr/local/cuda
```
…and `-arch=sm_86` → `-arch=$ARCH` on all nine nvcc lines (22–26, 35–38 of a 40-line script). `dsv4-sm120.patch` in
this directory is the same change as a file.

## Reproducing

```bash
git clone --branch ss16/dsv4-arch https://github.com/Hazelhof/QEngine
docker pull nvidia/cuda:13.1.1-devel-ubuntu24.04        # the Dockerfile's own builder (nvcc 13.1.115, GCC 13)
docker run --rm -v "$PWD":/src -v "$PWD"/out:/out nvidia/cuda:13.1.1-devel-ubuntu24.04 \
  bash -lc 'cd /src/src/dsv4 && CXX=g++ OUT=/out DSV4_ARCH=sm_120 bash build.sh'
```

The arm boots the **unmodified upstream image** with only the library substituted — `-e QE_DSV4_LIB=/arm/libqdsv4.so`
(`server/app.py:4525` reads it for the dsv4 family, and the image's entrypoint never passes `--engine-lib`). Put a
`qe_build.txt` beside the library and the boot banner names it, because a script-built engine otherwise reports
`build unknown` (`QE_GIT_HASH`/`qe_build.txt` are CMake-only).
