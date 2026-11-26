# Qwen3.8-27B on QEngine 1.5.0 — three checkpoints × two KV precisions, at the club 8-pack with `--repeat 3`

*2026-11-27 · a bench report, not a product. Companion to `BENCH.md` and to `BENCH-2026-11-26-qwen38-27b.md`; every
cell below names its checkpoint revision, its pack versions and its draw count.*

---

## 0. The three questions, and their answers in one line each

1. **Which Qwen3.8-27B checkpoint should we point at?** The engine's own README names one for each format — the
   RedHatAI W4A16 checkpoint (plus its MTP head) as the INT4 default, and `bpx03/Qwen3.8-27B-INT8-QEngine` as the
   INT8 opt-in. We ran both, plus a third-party mixed NVFP4/FP8 pack.
2. **Does the 4-bit KV cache (`QE_KV_BITS=4`) cost quality?** **No measurable cost** — on all three checkpoints, at
   three times the context, the cross-arm difference is smaller than the same leg's own repeat-to-repeat churn.
3. **What does thinking cost or buy?** It is the only lever in the table that resolves: **+14 to +21 items of 150**,
   consistently, on every checkpoint and every KV precision.

**What the three checkpoints themselves are, side by side:** one band. Their pairwise differences (14–23 scenario
flips of 150) are all smaller than either leg's own disagreement (21–27), so on this instrument they are
indistinguishable — including the INT8 pack, whose author asks in its own card whether its 22× smaller KL buys
anything in real tasks.

---

## 1. Rig, engine, harness, protocol

| | |
|---|---|
| cards | **4× RTX 5080 16 GB** (sm_120), PCIe Gen5 ×8 through a PEX89048 switch, peer access granted |
| host | 125 GiB RAM, swap recycled to zero before every boot |
| engine | **upstream 1.5.0**, image by digest, build `d091eaee`; the two third-party checkpoints additionally need the sharded-set / NVFP4 / FP8 reader described in `BENCH-2026-11-26-qwen38-27b.md` |
| shape | 8 seats, `max-ctx` 262,144, KV in VRAM, DFlash2 draft enabled, `tp` across all four cards |
| harness | the club's `scripts/quality-test.sh` (pinned checkout) → **benchlocal-cli 0.11.0** |
| pack versions | `tc1.0.1 · if2.0.0 · so2.1.0 · de1.2.0 · rm1.1.0 · bf1.0.1 · hm1.0.0 · cli1.0.2` |
| protocol | `--full` (all eight packs, 150 scenarios), **server sampler**, **`--repeat 3`**, one boot per leg, **loop guards off**, `--strict-thinking` |
| floor | each leg's own **`split`** — the scenarios whose three repeats disagreed — measured from the same JSON (21–27 of 150 here) |

Everything in this report is `equivalent_score_150` from the harness, i.e. the 150 scenarios scored at pass@1 over
three repeats. Every leg finished with `validity=valid` and exit code 0 under `--strict-thinking`.

**The floor is the split, and it is not small.** A cross-arm flip total at or below the legs' own repeat-to-repeat
disagreement cannot be read as a difference; at `--repeat 3` the split (21–27) is a stricter floor than the 20-flip
one-draw count an earlier report used.

---

## 2. The three checkpoints

| pack | revision | what it is |
|---|---|---|
| `RedHatAI/Qwen3.8-27B-INT4` | `71dface3` | the checkpoint the engine's README names as this model's INT4 default — compressed-tensors W4A16, group 128, plus QEngine's own MTP head and two tuned files fetched at first start |
| `bpx03/Qwen3.8-27B-INT8-QEngine` | `5eae0ab6` | the engine author's own INT8 checkpoint, symmetric group 128, FP8 KV scales taken from his INT4 checkpoint. **Labelled experimental by its author and named in no release body** — a published artifact, not a released default |
| `primitive-ai/Qwen3.8-27B-mixed-NVFP4-FP8` | `8809128f` | 5 shards, 276 NVFP4 and 124 FP8 Linears. **The stock 1.5.0 engine refuses it by name** (it expects one `model.safetensors`, INT4 or symmetric INT8, group 128); it loads only with the sharded-set/foreign-layout reader. It also ships **no** `k_scale`/`v_scale` — 0 of 2,151 tensors |

**That last row is a reach cell, not a quality verdict**, and the numbers on it should be read that way: they say
what the extended loader can read at all. For its KV arms in particular, both are non-default — the FP8 arm exists
only by lending it the 32 BF16 `k_scale`/`v_scale` scalars from the INT4 checkpoint, so those cells have two authors
(the pack's weights, the INT4 checkpoint's calibration).

---

## 3. The headline — six arms, both thinking legs

| # | checkpoint | KV arm | shape | OFF (`/150`) | ON (`/150`) | thinking Δ |
|---|---|---|---:|---:|---:|---:|
| 1 | RedHatAI INT4 | FP8 | 1,022,976 tok | **113** | **128** | +15 |
| 2 | RedHatAI INT4 | int4 | 1,712,128 tok | **114** | **128** | +14 |
| 3 | bpx03 INT8 | FP8 | 672,768 tok | **107** | **127** | +20 |
| 4 | bpx03 INT8 | int4 | 1,116,160 tok | **110** | **128** | +18 |
| 5 | primitive-ai NVFP4/FP8 | FP8 (lent) | 1,107,968 tok | **108** | **122** | +14 |
| 6 | primitive-ai NVFP4/FP8 | int4 (lent) | 1,857,536 tok | **103** | **124** | +21 |

Twelve legs, one fresh boot each. pass@3 alongside: 126 · 128 · 121 · 122 · 120 · 122 · 127 · 128 · 122 · 124 · 113.

---

## 4. The KV axis — three independent pairs, both modes

Same checkpoint, same library, same seats and window, **one environment variable** (`QE_KV_BITS=4`), its own boot:

| pair | FP8 | int4 | flips of 150 | that pair's splits | verdict |
|---|---:|---:|---|---|---|
| RedHatAI INT4, OFF | 113 | 114 | **14** (6 lost, 8 gained) | 26 / 27 | indistinguishable |
| bpx03 INT8, OFF | 107 | 110 | **21** (9 lost, 12 gained) | 25 / 26 | indistinguishable |
| primitive-ai, OFF | 108 | 103 | **18** (10 lost, 8 gained) | 25 / 21 | indistinguishable |

The ON legs agree: Δ0, Δ+1, Δ+2.

**Every cross-arm flip count is at or below the legs' own disagreement**, so the 4-bit cache shows no quality cost on
any of the three checkpoints at this resolution — and it buys **1.66× to 1.68×** the KV pool
(1,022,976 → 1,712,128 · 672,768 → 1,116,160 · 1,107,968 → 1,857,536 tokens; the engine's own line reads
`KV cache: 4 bits … 144 bytes per row`). The 4-bit cache's gates are worth quoting because they decide whether an arm
is the arm you think: it needs the FP8 KV scales (the checkpoint's, or a `QE_KV_SCALES` side file), `QE_KV_FP8=1`, and
the `attn_dec8d` decode kernel — otherwise it silently falls back to FP8 or bf16 and prints why.

For anyone comparing against a **bf16** cache: on that state the 4-bit option is unavailable, and the batched-attention
copy fix described in the 11-26 report is live. On the FP8 and int4 arms it is inert by construction, so this walk says
nothing about it — that statement needs a bf16 arm, which this bench did not run.

---

## 5. The three checkpoints are one band

| pair (all OFF, `--repeat 3`) | flips of 150 | direction |
|---|---:|---|
| RedHatAI INT4 ↔ bpx03 INT8 | **14** (9 lost, 5 gained) | −6 |
| RedHatAI INT4 ↔ primitive-ai | **23** (14 lost, 9 gained) | −5 |
| bpx03 INT8 ↔ primitive-ai | **21** (11 lost, 10 gained) | +1 |

Totals span 103–114 and every pair's flip count is below either leg's own split. That is the only comparison this
instrument supports: **a cross-checkpoint row is format plus recipe plus, on the third row, a loader**, and the pack
suite issues one request at a time.

---

## 6. Thinking is the lever

+15 / +14 / +20 / +18 / +14 / +21 items across the six arms, always in the same direction, and always concentrated in
`instructfollow-15` and `cli-40` — the two packs that reward prose discipline. `reasonmath-15` and `bugfind-15` are
where thinking is worth least; on the third checkpoint's FP8 arm they were the only two packs that **lost** items on
the thinking leg (−1.4 and −1.0), both inside their pack's split.

---

## 7. Structural apparatus, every arm

Each arm was driven end to end on its own boot (the five phases below share it; the thinking-ON legs carry the 8-pack
only, which is a declared deviation, not an omission):

| arm | `bench.sh` narr / code (tok/s) | prefill 10K / 90K (tok/s) | `bench-agentic` TTFT growth | `concurrency-probe` N=8 @16K | `verify-stress` | `soak --continuous` |
|---|---|---|---:|---|---|---|
| INT4 FP8 | 386.9 / 753.6 | 7,674 / 6,932 | 25.1× ctx → **11.9×** | PASS · 82.8/stream · retention 105 % · +0 MB | 6/6 to 240,662 tok · 5/5 · 3/3 | PASS · 0 MB · 100 % |
| INT4 int4 | 379.6 / 710.6 | 7,793 / 7,057 | 25.1× → **11.7×** | PASS · 88.9/stream · 107.6 % · +0 MB | 6/6 to 240,663 · 5/5 · 3/3 | PASS · 0 MB · 100 % |
| INT8 FP8 | 262.1 / 511.4 | 7,355 / 6,989 | 24.9× → **9.3×** | PASS · 66.6/stream · 101.9 % · +0 MB | 6/6 to 240,663 · 5/5 · 3/3 | PASS · 0 MB · 100 % |
| INT8 int4 | 261.9 / 507.7 | 7,477 / 6,919 | 24.9× → **9.5×** | PASS · 71.1/stream · 102.1 % · +0 MB | 6/6 to 240,659 · 5/5 · 3/3 | PASS · 0 MB · 100 % |
| NVFP4 FP8 | 346.5 / 482.8 | 7,819 / 7,111 | 24.5× → **10.0×** | PASS · 77.5/stream · 99.3 % · +0 MB | 6/6 to 240,661 · 5/5 · 3/3 | PASS · 0 MB · 100 % |
| NVFP4 int4 | 349.5 / 509.4 | 7,904 / 7,036 | 24.5× → **8.4×** | PASS · 80.1/stream · 101.7 % · **+8 MB** | 6/6 to 240,657 · 5/5 · 3/3 | PASS · 0 MB · 100 % |

- **The context ceiling ladder filled to ~240,660 tokens on every arm** — 91 % of the 262,144 served context — with
  each rung recalling its own needle, on all four cards, including the pack the stock loader cannot open.
- **`concurrency-probe` passed at eight concurrent 16K-token prompts on all six arms**, 66.6–88.9 tok/s per stream,
  aggregate 88–99 tok/s, retention 99.3–107.6 %, post-warm VRAM growth 0 MB on five arms and **8 MB** on one (4 % of
  the harness's 200 MB threshold — stated rather than smoothed).
- `bench-agentic` reports no client-side decode figure on this engine because **every reply arrives in one output
  chunk**, so its TTFT includes the reply's decode. That is why the growth numbers above are TTFT growth, not
  prompt-read cost alone.

### Rates, and what they may be used for

The INT8 checkpoint decodes at **0.68×** the INT4 checkpoint here (262.1 vs 386.9 narrative, 511.4 vs 753.6 code).
Its author's own card says 0.69 / 0.71 on 3× RTX 3090 — the ratio reproduces on different cards, which is the useful
check on this arm: the pack costs what its author says it costs.

**The KV arms are flat at a short prompt and are NOT flat at depth — the depth cells are the ones that matter.** Both
are in the same `bench.sh` run per boot:

| arm (per checkpoint) | decode @1k, FP8 → int4 | prefill 10K, FP8 → int4 | TTFT @90K, FP8 → int4 | **decode @90K, FP8 → int4** |
|---|---|---|---|---|
| RedHatAI INT4 | 386.9 → 379.6 (−1.9 %) | 7,674 → 7,793 (+1.6 %) | 13,464 → 13,243 ms (−1.6 %) | **284.9 → 224.6 (−21 %)** |
| bpx03 INT8 | 262.1 → 261.9 (−0.1 %) | 7,355 → 7,477 (+1.7 %) | 12,877 → 13,007 ms (+1.0 %) | **178.9 → 166.8 (−7 %)** |
| primitive-ai NVFP4 | 346.5 → 349.5 (+0.9 %) | 7,819 → 7,904 (+1.1 %) | 12,205 → 12,792 ms (+4.8 %) | **301.1 → 217.5 (−28 %)** |

So the trade is real but it lives at depth, not at the short prompt a TTFT/rate check usually samples: at 1k tokens
the two arms are inside this box's churn, while **at 90K of context the 4-bit cache costs 7–28 % of decode
throughput** on every checkpoint measured — the group-of-32 rotate plus dequantise being read at depth is the
mechanism the direction implies, and the INT8 checkpoint, already weight-bound, pays least.

Two limits stated with it: **the 90K depth cell is one run per arm** (the 10K depth cells carry a 14–19 % CV), so
quote the direction and the magnitude range, not a precise percentage; and an **interleaved ABAB on one boot** is the
cell that would turn this into a priced cost. This report does not contain that pair.

Prefill is 1–2 % **cheaper** on int4 in all three checkpoints — consistent in sign across six boots, small enough to
be a shape rather than a number.

---

## 8. What this report does not say

- **Nothing about `step_batch`.** The pack suite issues one request at a time and this engine picks the batched entry
  only for two or more decoders, so a change that lives only there is invisible here by construction.
- **Nothing about the checkpoints as formats.** They are three quantised recipes from two authors plus one
  third-party mixed pack; cross-checkpoint rows are shape records.
- **Nothing about bf16 KV.** Not run. It is the one KV state where the 4-bit option is unavailable and where the
  batched-copy fix is live; a bf16 leg is the experiment this bench cannot answer.
- **No per-pack ranking.** The packs disagree with themselves on 21–27 scenarios per leg; a ranking inside that is a
  draw, not a result.

---

## 9. Reproduction

```bash
# one boot per leg; the endpoint must be reachable from a container for the three sandboxed packs
URL=http://172.17.0.1:8197 MODEL=<served id> CONTAINER=none SAMPLING_FROM_SERVER=1 \
  bash scripts/quality-test.sh --full --no-thinking --repeat 3 --strict-thinking \
  --sandbox-log-dir <dir> --save-json <leg>.json

# the thinking-ON leg is its own boot with the effort level set at the server, then:
URL=... bash scripts/quality-test.sh --full --enable-thinking --repeat 3 --strict-thinking ...

# the KV arm is one environment variable on an otherwise identical boot
QE_KV_BITS=4        # int4 KV; unset = the checkpoint's own FP8 cache
# loop guards off for measurement: QE_LOOP_GUARD_THINK=0 QE_LOOP_GUARD_NOTE=0
# thinking off at the server for the OFF leg: QE_REASONING_EFFORT=none
# thinking on with the published row's level: QE_REASONING_EFFORT=low
```

Structural phases: `bench.sh`, `bench-agentic.sh`, `concurrency-probe.sh`, `verify-stress.sh`,
`soak-test.sh --continuous`, each with `URL=` / `CONTAINER=none` (and `ENDPOINT=` for soak).

Verify a leg is the leg you meant by reading its JSON: `thinking_mode`, `repeat`, `sampling_source`,
`equivalent_score_150`, `totals`, `runaway`, and `thinking_validity` per pack. Note that `thinking_validity` has **no
entry for `hermesagent-20`**, so a `validity=valid` on a leg including the sandboxed packs is a partial claim — the
count covers seven of the eight.

---

*Licence: this document is part of a fork of QEngine and carries the same MIT terms as the repository. Numbers are
reported with their instruments; where a cell is a bound or a shape, it says so.*
