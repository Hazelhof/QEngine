# Data appendix — the 27B walk, 27 November 2026

Companion to `BENCH-2026-11-27-qwen38-27b-walk.md`. Everything here is read straight out of the harness's own
artefacts: the per-leg JSON (`equivalent_score_150`, `packs[].scenarios[]`), the `bench.sh` capture, the
`bench-agentic.sh` table, the `concurrency-probe.sh` round table, the `verify-stress.sh` log and the
`soak-test.sh` summary. Machine-readable form: `walk-scenarios.csv` (5,400 rows = 12 legs × 150 scenarios × 3
repeats) beside this file.

---

## RedHatAI INT4 — FP8 KV

**Engine: STOCK** — the image's own `/app/build/libqengine.so`, no mount; its boot log carries **no** `Q4bit` line.

**`w1-int4-fp8-off-r3`** — `equivalent_score_150` **113**, totals 339/450 (75.3 %), `thinking_mode` force-off, `repeat` 3, `sampling_source` server, duration 749 s, runaways 1, started 2026-11-26T10:20:57.426864Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 12.0/15 | 12 | 0 |
| instructfollow-15 | 10.0/15 | 12 | 3 |
| structoutput-15 | 11.0/15 | 11 | 0 |
| dataextract-15 | 10.3/15 | 12 | 3 |
| reasonmath-15 | 13.7/15 | 14 | 1 |
| bugfind-15 | 14.7/15 | 15 | 1 |
| hermesagent-20 | 13.0/20 | 16 | 6 |
| cli-40 | 28.3/40 | 34 | 12 |
| **total** | **113.0/150** | **126** | **26** |

`thinking_validity`: bugfind-15: ok/0 reasoned, cli-40: ok/0 reasoned, dataextract-15: ok/0 reasoned, instructfollow-15: ok/0 reasoned, reasonmath-15: ok/0 reasoned, structoutput-15: ok/0 reasoned, toolcall-15: ok/0 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

**`w1-int4-fp8-on-r3`** — `equivalent_score_150` **128**, totals 383/450 (85.1 %), `thinking_mode` force-on, `repeat` 3, `sampling_source` server, duration 1175 s, runaways 0, started 2026-11-26T10:58:50.689104Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.3/15 | 14 | 1 |
| instructfollow-15 | 14.3/15 | 15 | 2 |
| structoutput-15 | 13.0/15 | 14 | 2 |
| dataextract-15 | 13.3/15 | 14 | 1 |
| reasonmath-15 | 14.3/15 | 15 | 2 |
| bugfind-15 | 14.0/15 | 15 | 2 |
| hermesagent-20 | 14.3/20 | 17 | 5 |
| cli-40 | 31.0/40 | 36 | 9 |
| **total** | **127.7/150** | **140** | **24** |

`thinking_validity`: bugfind-15: ok/45 reasoned, cli-40: ok/120 reasoned, dataextract-15: ok/45 reasoned, instructfollow-15: ok/45 reasoned, reasonmath-15: ok/45 reasoned, structoutput-15: ok/45 reasoned, toolcall-15: ok/45 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

### `bench.sh` — RedHatAI INT4 — FP8 KV

| cell | n | mean | std | CV | min | max | TTFT / depth note |
|---|---:|---:|---:|---:|---:|---:|---|
| narrative | 5 | 386.88 tok/s | 27.08 | 7.0 % | 362.23 | 430.54 | TTFT 25-26 ms |
| code | 5 | 753.60 tok/s | 29.52 | 3.9 % | 712.21 | 784.70 | TTFT 24-25 ms |
| prefill-10k | 3 | 7673.94 tok/s | 90.14 | 1.2 % | 7569.91 | 7728.96 | TTFT 1257-1295 ms; decode at depth 279.5 tok/s |
| prefill-90k | 1 | 6931.77 tok/s | 0.00 | 0.0 % | 6931.77 | 6931.77 | TTFT 13464-13464 ms; decode at depth 284.9 tok/s |

<details><summary>per-run rows (every warm-up and measured run)</summary>

```
warm-1     wall=  2.81s  ttft=    29ms  toks=1000  wall_TPS=355.97  decode_TPS=359.64
warm-2     wall=  2.74s  ttft=    25ms  toks= 998  wall_TPS=364.41  decode_TPS=367.77
warm-3     wall=  2.71s  ttft=    24ms  toks=1000  wall_TPS=369.43  decode_TPS=372.72
run-1      wall=  2.55s  ttft=    25ms  toks= 996  wall_TPS=390.34  decode_TPS=394.19
run-2      wall=  2.72s  ttft=    25ms  toks=1000  wall_TPS=367.14  decode_TPS=370.52
run-3      wall=  2.35s  ttft=    26ms  toks=1000  wall_TPS=425.85  decode_TPS=430.54
run-4      wall=  2.74s  ttft=    24ms  toks= 983  wall_TPS=359.04  decode_TPS=362.23
run-5      wall=  2.68s  ttft=    24ms  toks=1000  wall_TPS=373.51  decode_TPS=376.95
warm-1     wall=  0.76s  ttft=    22ms  toks= 550  wall_TPS=725.34  decode_TPS=747.41
warm-2     wall=  0.84s  ttft=    26ms  toks= 600  wall_TPS=710.78  decode_TPS=733.64
warm-3     wall=  0.59s  ttft=    25ms  toks= 433  wall_TPS=737.02  decode_TPS=769.86
run-1      wall=  0.77s  ttft=    24ms  toks= 528  wall_TPS=689.87  decode_TPS=712.21
run-2      wall=  1.06s  ttft=    24ms  toks= 800  wall_TPS=756.01  decode_TPS=773.41
run-3      wall=  0.54s  ttft=    23ms  toks= 395  wall_TPS=729.60  decode_TPS=762.42
run-4      wall=  1.11s  ttft=    22ms  toks= 800  wall_TPS=720.67  decode_TPS=735.28
run-5      wall=  0.51s  ttft=    25ms  toks= 377  wall_TPS=746.21  decode_TPS=784.70
warm-1     wall=  1.27s  ttft=  1262ms  prompt_toks=  9919  PP_tok/s=7862.19
run-1      wall=  1.68s  ttft=  1295ms  prompt_toks= 10003  PP_tok/s=7722.95
run-2      wall=  1.74s  ttft=  1227ms  prompt_toks=  9289  PP_tok/s=7569.91
run-3      wall=  1.74s  ttft=  1248ms  prompt_toks=  9646  PP_tok/s=7728.96
warm-1     wall= 11.84s  ttft= 11825ms  prompt_toks= 85894  PP_tok/s=7263.90
run-1      wall= 13.91s  ttft= 13464ms  prompt_toks= 93331  PP_tok/s=6931.77
```
</details>

### `bench-agentic.sh` — RedHatAI INT4 — FP8 KV

```
SUMMARY — multi-turn prefill stress (2 session(s) × 12 turns)
  whole replies  24/24 turn(s) arrived in ONE output chunk (the engine
    Turn 1 (cold):            246 ms TTFT  — compile/warmup, excluded from growth
    Turn 2 (warm base):       93 ms TTFT @ 1,406 prompt tokens
    Turn 12:                 1099 ms TTFT @ 35,289 prompt tokens
    Context grew 25.1×,  TTFT grew 11.9× (warm baseline → last turn)
       (see 'whole replies' above), so this growth is not prompt-read cost alone.
```

### `concurrency-probe.sh` — RedHatAI INT4 — FP8 KV

```
round    done  silent  errors  vram_MB  agg_t/s  per-strm  ttft_ms  pf_t/s  ttft_p95  tps_p05   run/wait pfxhit
    1    8/8        0       0    60160     97.1      83.1    11191    1430     20499     53.4    -/-          -
    2    8/8        0       0    60160     99.7      80.3    10648    1503     19933     52.7    -/-          -
    3    8/8        0       0    60160     97.6      79.1    10686    1497     20006     50.0    -/-          -
    4    8/8        0       0    60160     99.1      84.6    10665    1500     20082     49.6    -/-          -
    5    8/8        0       0    60160     98.0      82.8    11023    1452     20302     48.8    -/-          -
=== verdict (N=8) ===
  VRAM: cold 59936 -> warm 60160 MB (pool fill 224 MB, expected) -> final 60160 MB (post-warm growth 0 MB / 200)  peak 60160 MB
  VRAM peak per GPU: GPU 0 15038 MB · GPU 1 15042 MB · GPU 2 15042 MB · GPU 3 15038 MB  (every GPU on the host (rig-wide: no container to scope to))
  per-stream decode: 82.8 tok/s (steady) · aggregate 98.0 tok/s (8 streams) · retention 105.0% (min 98%) · floor off
  concurrent prefill: steady TTFT 11023 ms (median of 8 streams) · ~1452 tok/s/stream uncached @ 16000-tok prompts · agg-prefill ~6305 tok/s
  tail: TTFT p95 20302 ms (vs p50 11023) · slowest-decile stream 48.8 tok/s (vs median 82.8)
  concurrency 8 @ ~16000 tok: PASS — sustained clean
RESULT N=8 ctx=16000 clean=1 pass=1 mps_tps=82.84 agg_tps=98.05 retention=1.050 leak=0 vram_peak=60160 floor_ok=1 ttft_ms=11023 pf_tps=1451.6 ttft_p95_ms=20302 tps_p05=48.76 running=- waiting=- prefix_hit=- cache=cold cached_toks=0 pf_agg=6304.7 vram_gpus=all vram_peak_gpus=0:15038,1:15042,2:15042,3:15038 running_max=- waiting_max=-
```

### `verify-stress.sh` — RedHatAI INT4 — FP8 KV

```
  [32m✓[0m all long-ctx depths recalled secret correctly
  [32m✓[0m all long-ctx depths recalled secret correctly
[8/8] Context ceiling ladder (staggered NIAH from ~95000 → ~0.92 × n_ctx) ...
  [32m✓[0m ceiling ladder: all 6 rungs passed — fillable to 240662 tok (91% of n_ctx=262144)
  [32mboundary checks[0m  5/5 passed  [2m(tool-prefill OOM, agent shapes, reasoning-heavy, VRAM ceiling)[0m
  [32mrecall ladder[0m    3/3 passed  [2m(uniform-haystack: ADDRESSABILITY, not retrieval[0m
[32mAll stress / boundary checks passed.[0m KV-cache and prefill paths are sound for the deployed config.
```

### `soak-test.sh --continuous` — RedHatAI INT4 — FP8 KV

```
[soak]   verdict              PASS
[soak]   boot_vram_mib        60160
[soak]   max_vram_mib         60160
[soak]   max_growth_mib       0 / 200
[soak]   errors               0
[soak]   silent_empty         0 / 25 (0.0%)
[soak]   p50_decode_tps       405.64
[soak]   decode_basis         4 client-timed / 21 no decode figure (reply arrived whole in one output chunk, no engine counter) of 25
[soak]   whole_replies        21 / 25  (one output chunk: no client decode window; TTFT includes their decode)
[soak]   p95_ttft_ms          1126
[soak]   tps_retention        100.0%
```

---

## RedHatAI INT4 — int4 KV

**Engine: STOCK** — the image's own `/app/build/libqengine.so`, no mount; its boot log carries **no** `Q4bit` line.

**`w1-int4-kv4-off-r3`** — `equivalent_score_150` **114**, totals 342/450 (76.0 %), `thinking_mode` force-off, `repeat` 3, `sampling_source` server, duration 751 s, runaways 1, started 2026-11-26T11:19:35.010928Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 12.7/15 | 14 | 2 |
| instructfollow-15 | 10.7/15 | 12 | 2 |
| structoutput-15 | 11.0/15 | 11 | 0 |
| dataextract-15 | 11.0/15 | 13 | 4 |
| reasonmath-15 | 13.3/15 | 14 | 1 |
| bugfind-15 | 14.3/15 | 15 | 2 |
| hermesagent-20 | 13.7/20 | 17 | 6 |
| cli-40 | 27.3/40 | 32 | 10 |
| **total** | **114.0/150** | **128** | **27** |

`thinking_validity`: bugfind-15: ok/0 reasoned, cli-40: ok/0 reasoned, dataextract-15: ok/0 reasoned, instructfollow-15: ok/0 reasoned, reasonmath-15: ok/0 reasoned, structoutput-15: ok/0 reasoned, toolcall-15: ok/0 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

**`w1-int4-kv4-on-r3`** — `equivalent_score_150` **128**, totals 384/450 (85.3 %), `thinking_mode` force-on, `repeat` 3, `sampling_source` server, duration 1219 s, runaways 0, started 2026-11-26T11:54:37.448264Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.0/15 | 13 | 0 |
| instructfollow-15 | 14.3/15 | 15 | 2 |
| structoutput-15 | 13.0/15 | 14 | 3 |
| dataextract-15 | 13.3/15 | 14 | 2 |
| reasonmath-15 | 15.0/15 | 15 | 0 |
| bugfind-15 | 14.7/15 | 15 | 1 |
| hermesagent-20 | 14.3/20 | 17 | 6 |
| cli-40 | 30.3/40 | 33 | 6 |
| **total** | **128.0/150** | **136** | **20** |

`thinking_validity`: bugfind-15: ok/45 reasoned, cli-40: ok/120 reasoned, dataextract-15: ok/45 reasoned, instructfollow-15: ok/45 reasoned, reasonmath-15: ok/45 reasoned, structoutput-15: ok/45 reasoned, toolcall-15: ok/45 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

### `bench.sh` — RedHatAI INT4 — int4 KV

| cell | n | mean | std | CV | min | max | TTFT / depth note |
|---|---:|---:|---:|---:|---:|---:|---|
| narrative | 5 | 379.60 tok/s | 19.93 | 5.3 % | 347.32 | 396.13 | TTFT 23-25 ms |
| code | 5 | 710.57 tok/s | 31.60 | 4.4 % | 673.44 | 748.50 | TTFT 24-25 ms |
| prefill-10k | 3 | 7792.95 tok/s | 99.33 | 1.3 % | 7695.68 | 7894.22 | TTFT 1346-1360 ms; decode at depth 273.6 tok/s |
| prefill-90k | 1 | 7056.78 tok/s | 0.00 | 0.0 % | 7056.78 | 7056.78 | TTFT 13243-13243 ms; decode at depth 224.6 tok/s |

<details><summary>per-run rows (every warm-up and measured run)</summary>

```
warm-1     wall=  2.64s  ttft=    85ms  toks= 937  wall_TPS=354.98  decode_TPS=366.79
warm-2     wall=  2.78s  ttft=    22ms  toks=1000  wall_TPS=359.50  decode_TPS=362.32
warm-3     wall=  2.55s  ttft=    22ms  toks= 971  wall_TPS=381.39  decode_TPS=384.73
run-1      wall=  2.27s  ttft=    25ms  toks= 889  wall_TPS=391.79  decode_TPS=396.13
run-2      wall=  2.64s  ttft=    22ms  toks= 908  wall_TPS=344.45  decode_TPS=347.32
run-3      wall=  2.51s  ttft=    24ms  toks= 970  wall_TPS=386.90  decode_TPS=390.60
run-4      wall=  2.58s  ttft=    21ms  toks= 957  wall_TPS=370.51  decode_TPS=373.59
run-5      wall=  2.37s  ttft=    22ms  toks= 918  wall_TPS=386.70  decode_TPS=390.38
warm-1     wall=  0.99s  ttft=    22ms  toks= 728  wall_TPS=733.62  decode_TPS=750.49
warm-2     wall=  1.04s  ttft=    23ms  toks= 724  wall_TPS=698.38  decode_TPS=714.05
warm-3     wall=  1.10s  ttft=    24ms  toks= 800  wall_TPS=728.05  decode_TPS=744.38
run-1      wall=  0.78s  ttft=    21ms  toks= 537  wall_TPS=689.32  decode_TPS=708.84
run-2      wall=  1.11s  ttft=    24ms  toks= 800  wall_TPS=719.22  decode_TPS=735.24
run-3      wall=  0.54s  ttft=    25ms  toks= 388  wall_TPS=714.22  decode_TPS=748.50
run-4      wall=  1.19s  ttft=    24ms  toks= 800  wall_TPS=672.71  decode_TPS=686.81
run-5      wall=  0.96s  ttft=    23ms  toks= 632  wall_TPS=657.55  decode_TPS=673.44
warm-1     wall=  1.23s  ttft=  1219ms  prompt_toks=  9565  PP_tok/s=7849.00
run-1      wall=  1.90s  ttft=  1347ms  prompt_toks= 10367  PP_tok/s=7695.68
run-2      wall=  1.75s  ttft=  1331ms  prompt_toks= 10367  PP_tok/s=7788.95
run-3      wall=  1.81s  ttft=  1360ms  prompt_toks= 10737  PP_tok/s=7894.22
warm-1     wall= 11.52s  ttft= 11503ms  prompt_toks= 82713  PP_tok/s=7190.71
run-1      wall= 13.81s  ttft= 13243ms  prompt_toks= 93454  PP_tok/s=7056.78
```
</details>

### `bench-agentic.sh` — RedHatAI INT4 — int4 KV

```
SUMMARY — multi-turn prefill stress (2 session(s) × 12 turns)
  whole replies  24/24 turn(s) arrived in ONE output chunk (the engine
    Turn 1 (cold):            215 ms TTFT  — compile/warmup, excluded from growth
    Turn 2 (warm base):       95 ms TTFT @ 1,406 prompt tokens
    Turn 12:                 1119 ms TTFT @ 35,277 prompt tokens
    Context grew 25.1×,  TTFT grew 11.7× (warm baseline → last turn)
       (see 'whole replies' above), so this growth is not prompt-read cost alone.
```

### `concurrency-probe.sh` — RedHatAI INT4 — int4 KV

```
round    done  silent  errors  vram_MB  agg_t/s  per-strm  ttft_ms  pf_t/s  ttft_p95  tps_p05   run/wait pfxhit
    1    8/8        0       0    60008    100.4      85.7    10515    1522     19811     65.9    -/-          -
    2    8/8        0       0    60176     98.2      81.8    10602    1509     19998     53.8    -/-          -
    3    8/8        0       0    60176     97.9      79.5    10713    1494     20115     53.0    -/-          -
    4    8/8        0       0    60176     99.6      84.7    10614    1507     19934     53.4    -/-          -
    5    8/8        0       0    60176     99.3      88.9    10660    1501     19795     52.6    -/-          -
=== verdict (N=8) ===
  VRAM: cold 59856 -> warm 60176 MB (pool fill 320 MB, expected) -> final 60176 MB (post-warm growth 0 MB / 200)  peak 60176 MB
  VRAM peak per GPU: GPU 0 15042 MB · GPU 1 15046 MB · GPU 2 15046 MB · GPU 3 15042 MB  (every GPU on the host (rig-wide: no container to scope to))
  per-stream decode: 88.9 tok/s (steady) · aggregate 99.3 tok/s (8 streams) · retention 107.6% (min 98%) · floor off
  concurrent prefill: steady TTFT 10660 ms (median of 8 streams) · ~1501 tok/s/stream uncached @ 16000-tok prompts · agg-prefill ~6466 tok/s
  tail: TTFT p95 19795 ms (vs p50 10660) · slowest-decile stream 52.6 tok/s (vs median 88.9)
  concurrency 8 @ ~16000 tok: PASS — sustained clean
RESULT N=8 ctx=16000 clean=1 pass=1 mps_tps=88.86 agg_tps=99.26 retention=1.076 leak=0 vram_peak=60176 floor_ok=1 ttft_ms=10660 pf_tps=1501.0 ttft_p95_ms=19795 tps_p05=52.55 running=- waiting=- prefix_hit=- cache=cold cached_toks=0 pf_agg=6466.2 vram_gpus=all vram_peak_gpus=0:15042,1:15046,2:15046,3:15042 running_max=- waiting_max=-
```

### `verify-stress.sh` — RedHatAI INT4 — int4 KV

```
  [32m✓[0m all long-ctx depths recalled secret correctly
  [32m✓[0m all long-ctx depths recalled secret correctly
[8/8] Context ceiling ladder (staggered NIAH from ~95000 → ~0.92 × n_ctx) ...
  [32m✓[0m ceiling ladder: all 6 rungs passed — fillable to 240663 tok (91% of n_ctx=262144)
  [32mboundary checks[0m  5/5 passed  [2m(tool-prefill OOM, agent shapes, reasoning-heavy, VRAM ceiling)[0m
  [32mrecall ladder[0m    3/3 passed  [2m(uniform-haystack: ADDRESSABILITY, not retrieval[0m
[32mAll stress / boundary checks passed.[0m KV-cache and prefill paths are sound for the deployed config.
```

### `soak-test.sh --continuous` — RedHatAI INT4 — int4 KV

```
[soak]   verdict              PASS
[soak]   boot_vram_mib        60176
[soak]   max_vram_mib         60176
[soak]   max_growth_mib       0 / 200
[soak]   errors               0
[soak]   silent_empty         0 / 25 (0.0%)
[soak]   p50_decode_tps       399.00
[soak]   decode_basis         5 client-timed / 20 no decode figure (reply arrived whole in one output chunk, no engine counter) of 25
[soak]   whole_replies        20 / 25  (one output chunk: no client decode window; TTFT includes their decode)
[soak]   p95_ttft_ms          1084
[soak]   tps_retention        100.0%
```

---

## bpx03 INT8 — FP8 KV

**Engine: STOCK** — the image's own `/app/build/libqengine.so`, no mount; its boot log carries **no** `Q4bit` line.

**`w2-int8-fp8-off-r3`** — `equivalent_score_150` **107**, totals 322/450 (71.6 %), `thinking_mode` force-off, `repeat` 3, `sampling_source` server, duration 802 s, runaways 1, started 2026-11-26T12:21:11.116126Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.0/15 | 13 | 0 |
| instructfollow-15 | 9.7/15 | 11 | 2 |
| structoutput-15 | 10.3/15 | 11 | 1 |
| dataextract-15 | 10.7/15 | 11 | 1 |
| reasonmath-15 | 13.3/15 | 14 | 2 |
| bugfind-15 | 14.0/15 | 15 | 2 |
| hermesagent-20 | 13.7/20 | 17 | 6 |
| cli-40 | 22.7/40 | 29 | 11 |
| **total** | **107.3/150** | **121** | **25** |

`thinking_validity`: bugfind-15: ok/0 reasoned, cli-40: ok/0 reasoned, dataextract-15: ok/0 reasoned, instructfollow-15: ok/0 reasoned, reasonmath-15: ok/0 reasoned, structoutput-15: ok/0 reasoned, toolcall-15: ok/0 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

**`w2-int8-fp8-on-r3`** — `equivalent_score_150` **127**, totals 382/450 (84.9 %), `thinking_mode` force-on, `repeat` 3, `sampling_source` server, duration 1379 s, runaways 0, started 2026-11-26T13:01:49.362185Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.0/15 | 13 | 0 |
| instructfollow-15 | 14.0/15 | 14 | 0 |
| structoutput-15 | 12.0/15 | 13 | 2 |
| dataextract-15 | 13.7/15 | 14 | 1 |
| reasonmath-15 | 15.0/15 | 15 | 0 |
| bugfind-15 | 14.7/15 | 15 | 1 |
| hermesagent-20 | 14.0/20 | 16 | 5 |
| cli-40 | 31.0/40 | 34 | 7 |
| **total** | **127.3/150** | **134** | **16** |

`thinking_validity`: bugfind-15: ok/45 reasoned, cli-40: ok/120 reasoned, dataextract-15: ok/45 reasoned, instructfollow-15: ok/45 reasoned, reasonmath-15: ok/45 reasoned, structoutput-15: ok/45 reasoned, toolcall-15: ok/45 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

### `bench.sh` — bpx03 INT8 — FP8 KV

| cell | n | mean | std | CV | min | max | TTFT / depth note |
|---|---:|---:|---:|---:|---:|---:|---|
| narrative | 5 | 262.09 tok/s | 6.36 | 2.4 % | 255.33 | 269.04 | TTFT 27-28 ms |
| code | 5 | 511.35 tok/s | 7.24 | 1.4 % | 502.89 | 518.11 | TTFT 27-28 ms |
| prefill-10k | 3 | 7354.53 tok/s | 433.52 | 5.9 % | 6856.04 | 7643.45 | TTFT 1346-1407 ms; decode at depth 212.3 tok/s |
| prefill-90k | 1 | 6989.04 tok/s | 0.00 | 0.0 % | 6989.04 | 6989.04 | TTFT 12877-12877 ms; decode at depth 178.9 tok/s |

<details><summary>per-run rows (every warm-up and measured run)</summary>

```
warm-1     wall=  3.84s  ttft=    95ms  toks= 948  wall_TPS=247.02  decode_TPS=253.32
warm-2     wall=  3.56s  ttft=    28ms  toks=1000  wall_TPS=280.83  decode_TPS=283.07
warm-3     wall=  3.93s  ttft=    24ms  toks=1000  wall_TPS=254.69  decode_TPS=256.28
run-1      wall=  3.75s  ttft=    28ms  toks=1000  wall_TPS=267.00  decode_TPS=269.04
run-2      wall=  3.94s  ttft=    28ms  toks=1000  wall_TPS=253.52  decode_TPS=255.33
run-3      wall=  3.78s  ttft=    25ms  toks=1000  wall_TPS=264.72  decode_TPS=266.50
run-4      wall=  3.81s  ttft=    28ms  toks=1000  wall_TPS=262.23  decode_TPS=264.14
run-5      wall=  3.94s  ttft=    28ms  toks=1000  wall_TPS=253.65  decode_TPS=255.45
warm-1     wall=  1.51s  ttft=    26ms  toks= 800  wall_TPS=528.13  decode_TPS=537.41
warm-2     wall=  1.23s  ttft=    27ms  toks= 607  wall_TPS=493.05  decode_TPS=504.27
warm-3     wall=  1.02s  ttft=    27ms  toks= 546  wall_TPS=534.04  decode_TPS=548.30
run-1      wall=  1.01s  ttft=    27ms  toks= 510  wall_TPS=504.08  decode_TPS=518.03
run-2      wall=  1.28s  ttft=    27ms  toks= 645  wall_TPS=502.12  decode_TPS=513.07
run-3      wall=  1.30s  ttft=    28ms  toks= 640  wall_TPS=493.84  decode_TPS=504.67
run-4      wall=  0.86s  ttft=    28ms  toks= 431  wall_TPS=501.38  decode_TPS=518.11
run-5      wall=  0.81s  ttft=    27ms  toks= 396  wall_TPS=486.08  decode_TPS=502.89
warm-1     wall=  1.32s  ttft=  1308ms  prompt_toks=  9919  PP_tok/s=7583.49
run-1      wall=  2.11s  ttft=  1407ms  prompt_toks=  9646  PP_tok/s=6856.04
run-2      wall=  1.91s  ttft=  1322ms  prompt_toks= 10003  PP_tok/s=7564.10
run-3      wall=  1.84s  ttft=  1309ms  prompt_toks= 10003  PP_tok/s=7643.45
warm-1     wall= 12.55s  ttft= 12531ms  prompt_toks= 89075  PP_tok/s=7108.47
run-1      wall= 13.59s  ttft= 12877ms  prompt_toks= 89999  PP_tok/s=6989.04
```
</details>

### `bench-agentic.sh` — bpx03 INT8 — FP8 KV

```
SUMMARY — multi-turn prefill stress (2 session(s) × 12 turns)
  whole replies  24/24 turn(s) arrived in ONE output chunk (the engine
    Turn 1 (cold):            273 ms TTFT  — compile/warmup, excluded from growth
    Turn 2 (warm base):      121 ms TTFT @ 1,420 prompt tokens
    Turn 12:                 1128 ms TTFT @ 35,306 prompt tokens
    Context grew 24.9×,  TTFT grew 9.3× (warm baseline → last turn)
       (see 'whole replies' above), so this growth is not prompt-read cost alone.
```

### `concurrency-probe.sh` — bpx03 INT8 — FP8 KV

```
round    done  silent  errors  vram_MB  agg_t/s  per-strm  ttft_ms  pf_t/s  ttft_p95  tps_p05   run/wait pfxhit
    1    8/8        0       0    60168     88.0      65.7    10925    1465     20960     43.3    -/-          -
    2    8/8        0       0    60168     89.5      65.3    11092    1442     21327     42.5    -/-          -
    3    8/8        0       0    60168     88.3      68.8    11213    1427     21246     42.0    -/-          -
    4    8/8        0       0    60168     89.8      70.0    11171    1432     21218     41.5    -/-          -
    5    8/8        0       0    60168     89.9      66.6    11171    1432     21414     39.9    -/-          -
=== verdict (N=8) ===
  VRAM: cold 60136 -> warm 60168 MB (pool fill 32 MB, expected) -> final 60168 MB (post-warm growth 0 MB / 200)  peak 60168 MB
  VRAM peak per GPU: GPU 0 15042 MB · GPU 1 15042 MB · GPU 2 15042 MB · GPU 3 15042 MB  (every GPU on the host (rig-wide: no container to scope to))
  per-stream decode: 66.6 tok/s (steady) · aggregate 89.9 tok/s (8 streams) · retention 101.9% (min 98%) · floor off
  concurrent prefill: steady TTFT 11171 ms (median of 8 streams) · ~1432 tok/s/stream uncached @ 16000-tok prompts · agg-prefill ~5977 tok/s
  tail: TTFT p95 21414 ms (vs p50 11171) · slowest-decile stream 39.9 tok/s (vs median 66.6)
  concurrency 8 @ ~16000 tok: PASS — sustained clean
RESULT N=8 ctx=16000 clean=1 pass=1 mps_tps=66.64 agg_tps=89.93 retention=1.019 leak=0 vram_peak=60168 floor_ok=1 ttft_ms=11171 pf_tps=1432.2 ttft_p95_ms=21414 tps_p05=39.89 running=- waiting=- prefix_hit=- cache=cold cached_toks=0 pf_agg=5977.5 vram_gpus=all vram_peak_gpus=0:15042,1:15042,2:15042,3:15042 running_max=- waiting_max=-
```

### `verify-stress.sh` — bpx03 INT8 — FP8 KV

```
  [32m✓[0m all long-ctx depths recalled secret correctly
  [32m✓[0m all long-ctx depths recalled secret correctly
[8/8] Context ceiling ladder (staggered NIAH from ~95000 → ~0.92 × n_ctx) ...
  [32m✓[0m ceiling ladder: all 6 rungs passed — fillable to 240663 tok (91% of n_ctx=262144)
  [32mboundary checks[0m  5/5 passed  [2m(tool-prefill OOM, agent shapes, reasoning-heavy, VRAM ceiling)[0m
  [32mrecall ladder[0m    3/3 passed  [2m(uniform-haystack: ADDRESSABILITY, not retrieval[0m
[32mAll stress / boundary checks passed.[0m KV-cache and prefill paths are sound for the deployed config.
```

### `soak-test.sh --continuous` — bpx03 INT8 — FP8 KV

```
[soak]   verdict              PASS
[soak]   boot_vram_mib        60168
[soak]   max_vram_mib         60168
[soak]   max_growth_mib       0 / 200
[soak]   errors               0
[soak]   silent_empty         0 / 25 (0.0%)
[soak]   p50_decode_tps       289.94
[soak]   decode_basis         5 client-timed / 20 no decode figure (reply arrived whole in one output chunk, no engine counter) of 25
[soak]   whole_replies        20 / 25  (one output chunk: no client decode window; TTFT includes their decode)
[soak]   p95_ttft_ms          1144
[soak]   tps_retention        100.0%
```

---

## bpx03 INT8 — int4 KV

**Engine: STOCK** — the image's own `/app/build/libqengine.so`, no mount; its boot log carries **no** `Q4bit` line.

**`w2-int8-kv4-off-r3`** — `equivalent_score_150` **110**, totals 331/450 (73.6 %), `thinking_mode` force-off, `repeat` 3, `sampling_source` server, duration 823 s, runaways 0, started 2026-11-26T13:27:46.398630Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.3/15 | 14 | 2 |
| instructfollow-15 | 10.3/15 | 11 | 2 |
| structoutput-15 | 11.3/15 | 12 | 1 |
| dataextract-15 | 10.0/15 | 12 | 4 |
| reasonmath-15 | 13.3/15 | 14 | 1 |
| bugfind-15 | 14.0/15 | 15 | 3 |
| hermesagent-20 | 15.7/20 | 17 | 3 |
| cli-40 | 22.3/40 | 27 | 10 |
| **total** | **110.3/150** | **122** | **26** |

`thinking_validity`: bugfind-15: ok/0 reasoned, cli-40: ok/0 reasoned, dataextract-15: ok/0 reasoned, instructfollow-15: ok/0 reasoned, reasonmath-15: ok/0 reasoned, structoutput-15: ok/0 reasoned, toolcall-15: ok/0 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

**`w2-int8-kv4-on-r3`** — `equivalent_score_150` **128**, totals 383/450 (85.1 %), `thinking_mode` force-on, `repeat` 3, `sampling_source` server, duration 1387 s, runaways 0, started 2026-11-26T14:12:01.108500Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.0/15 | 13 | 0 |
| instructfollow-15 | 13.7/15 | 14 | 1 |
| structoutput-15 | 12.7/15 | 14 | 2 |
| dataextract-15 | 13.3/15 | 14 | 2 |
| reasonmath-15 | 15.0/15 | 15 | 0 |
| bugfind-15 | 15.0/15 | 15 | 0 |
| hermesagent-20 | 15.0/20 | 17 | 3 |
| cli-40 | 30.0/40 | 35 | 9 |
| **total** | **127.7/150** | **137** | **17** |

`thinking_validity`: bugfind-15: ok/45 reasoned, cli-40: ok/120 reasoned, dataextract-15: ok/45 reasoned, instructfollow-15: ok/45 reasoned, reasonmath-15: ok/45 reasoned, structoutput-15: ok/45 reasoned, toolcall-15: ok/45 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

### `bench.sh` — bpx03 INT8 — int4 KV

| cell | n | mean | std | CV | min | max | TTFT / depth note |
|---|---:|---:|---:|---:|---:|---:|---|
| narrative | 5 | 261.88 tok/s | 12.14 | 4.6 % | 254.17 | 283.02 | TTFT 26-29 ms |
| code | 5 | 507.68 tok/s | 45.83 | 9.0 % | 475.82 | 586.88 | TTFT 28-28 ms |
| prefill-10k | 3 | 7476.72 tok/s | 146.59 | 2.0 % | 7310.74 | 7588.47 | TTFT 1339-1417 ms; decode at depth 203.1 tok/s |
| prefill-90k | 1 | 6919.26 tok/s | 0.00 | 0.0 % | 6919.26 | 6919.26 | TTFT 13007-13007 ms; decode at depth 166.8 tok/s |

<details><summary>per-run rows (every warm-up and measured run)</summary>

```
warm-1     wall=  3.57s  ttft=    75ms  toks= 934  wall_TPS=261.37  decode_TPS=266.99
warm-2     wall=  3.67s  ttft=    28ms  toks= 982  wall_TPS=267.91  decode_TPS=269.96
warm-3     wall=  4.08s  ttft=    27ms  toks=1000  wall_TPS=244.98  decode_TPS=246.63
run-1      wall=  3.96s  ttft=    25ms  toks=1000  wall_TPS=252.57  decode_TPS=254.17
run-2      wall=  3.53s  ttft=    24ms  toks= 993  wall_TPS=281.07  decode_TPS=283.02
run-3      wall=  3.72s  ttft=    26ms  toks= 965  wall_TPS=259.47  decode_TPS=261.31
run-4      wall=  3.94s  ttft=    27ms  toks=1000  wall_TPS=254.09  decode_TPS=255.87
run-5      wall=  3.95s  ttft=    29ms  toks=1000  wall_TPS=253.18  decode_TPS=255.03
warm-1     wall=  1.27s  ttft=    28ms  toks= 625  wall_TPS=493.37  decode_TPS=504.55
warm-2     wall=  1.33s  ttft=    28ms  toks= 641  wall_TPS=481.72  decode_TPS=491.89
warm-3     wall=  0.90s  ttft=    27ms  toks= 441  wall_TPS=488.32  decode_TPS=503.55
run-1      wall=  1.65s  ttft=    27ms  toks= 800  wall_TPS=485.81  decode_TPS=494.03
run-2      wall=  0.91s  ttft=    28ms  toks= 422  wall_TPS=461.27  decode_TPS=475.82
run-3      wall=  0.68s  ttft=    27ms  toks= 386  wall_TPS=563.58  decode_TPS=586.88
run-4      wall=  1.70s  ttft=    28ms  toks= 800  wall_TPS=469.62  decode_TPS=477.46
run-5      wall=  1.61s  ttft=    28ms  toks= 800  wall_TPS=495.56  decode_TPS=504.20
warm-1     wall=  1.31s  ttft=  1290ms  prompt_toks=  9919  PP_tok/s=7688.44
run-1      wall=  1.94s  ttft=  1318ms  prompt_toks= 10003  PP_tok/s=7588.47
run-2      wall=  1.92s  ttft=  1281ms  prompt_toks=  9646  PP_tok/s=7530.94
run-3      wall=  2.05s  ttft=  1417ms  prompt_toks= 10360  PP_tok/s=7310.74
warm-1     wall= 12.76s  ttft= 12739ms  prompt_toks= 89075  PP_tok/s=6992.45
run-1      wall= 13.77s  ttft= 13007ms  prompt_toks= 89999  PP_tok/s=6919.26
```
</details>

### `bench-agentic.sh` — bpx03 INT8 — int4 KV

```
SUMMARY — multi-turn prefill stress (2 session(s) × 12 turns)
  whole replies  24/24 turn(s) arrived in ONE output chunk (the engine
    Turn 1 (cold):            314 ms TTFT  — compile/warmup, excluded from growth
    Turn 2 (warm base):      121 ms TTFT @ 1,420 prompt tokens
    Turn 12:                 1152 ms TTFT @ 35,297 prompt tokens
    Context grew 24.9×,  TTFT grew 9.5× (warm baseline → last turn)
       (see 'whole replies' above), so this growth is not prompt-read cost alone.
```

### `concurrency-probe.sh` — bpx03 INT8 — int4 KV

```
round    done  silent  errors  vram_MB  agg_t/s  per-strm  ttft_ms  pf_t/s  ttft_p95  tps_p05   run/wait pfxhit
    1    8/8        0       0    60392     89.7      66.4    10966    1459     21169     47.9    -/-          -
    2    8/8        0       0    60392     87.8      68.2    11126    1438     20971     42.3    -/-          -
    3    8/8        0       0    60392     87.2      61.1    11156    1434     21226     44.5    -/-          -
    4    8/8        0       0    60392     84.4      61.0    11152    1435     21220     38.7    -/-          -
    5    8/8        0       0    60392     88.3      71.1    11073    1445     21127     41.9    -/-          -
=== verdict (N=8) ===
  VRAM: cold 60136 -> warm 60392 MB (pool fill 256 MB, expected) -> final 60392 MB (post-warm growth 0 MB / 200)  peak 60392 MB
  VRAM peak per GPU: GPU 0 15098 MB · GPU 1 15098 MB · GPU 2 15098 MB · GPU 3 15098 MB  (every GPU on the host (rig-wide: no container to scope to))
  per-stream decode: 71.1 tok/s (steady) · aggregate 88.3 tok/s (8 streams) · retention 102.1% (min 98%) · floor off
  concurrent prefill: steady TTFT 11073 ms (median of 8 streams) · ~1445 tok/s/stream uncached @ 16000-tok prompts · agg-prefill ~6059 tok/s
  tail: TTFT p95 21127 ms (vs p50 11073) · slowest-decile stream 41.9 tok/s (vs median 71.1)
  concurrency 8 @ ~16000 tok: PASS — sustained clean
RESULT N=8 ctx=16000 clean=1 pass=1 mps_tps=71.06 agg_tps=88.34 retention=1.021 leak=0 vram_peak=60392 floor_ok=1 ttft_ms=11073 pf_tps=1444.9 ttft_p95_ms=21127 tps_p05=41.88 running=- waiting=- prefix_hit=- cache=cold cached_toks=0 pf_agg=6058.5 vram_gpus=all vram_peak_gpus=0:15098,1:15098,2:15098,3:15098 running_max=- waiting_max=-
```

### `verify-stress.sh` — bpx03 INT8 — int4 KV

```
  [32m✓[0m all long-ctx depths recalled secret correctly
  [32m✓[0m all long-ctx depths recalled secret correctly
[8/8] Context ceiling ladder (staggered NIAH from ~95000 → ~0.92 × n_ctx) ...
  [32m✓[0m ceiling ladder: all 6 rungs passed — fillable to 240659 tok (91% of n_ctx=262144)
  [32mboundary checks[0m  5/5 passed  [2m(tool-prefill OOM, agent shapes, reasoning-heavy, VRAM ceiling)[0m
  [32mrecall ladder[0m    3/3 passed  [2m(uniform-haystack: ADDRESSABILITY, not retrieval[0m
[32mAll stress / boundary checks passed.[0m KV-cache and prefill paths are sound for the deployed config.
```

### `soak-test.sh --continuous` — bpx03 INT8 — int4 KV

```
[soak]   verdict              PASS
[soak]   boot_vram_mib        60392
[soak]   max_vram_mib         60392
[soak]   max_growth_mib       0 / 200
[soak]   errors               0
[soak]   silent_empty         0 / 25 (0.0%)
[soak]   p50_decode_tps       287.49
[soak]   decode_basis         5 client-timed / 20 no decode figure (reply arrived whole in one output chunk, no engine counter) of 25
[soak]   whole_replies        20 / 25  (one output chunk: no client decode window; TTFT includes their decode)
[soak]   p95_ttft_ms          1162
[soak]   tps_retention        100.0%
```

---

## primitive-ai NVFP4/FP8 — FP8 KV

**Engine: OURS** — `libqengine.so` sha256 `313ec400971cc3b0…` mounted over the image's, plus the tree's `server/assets.py`; its boot log carries `Q4bit P5b: sharded pack: 5 shard(s) from model.safetensors.index.json`.

**`w3-nvfp4-fp8-off-r3`** — `equivalent_score_150` **108**, totals 323/450 (71.8 %), `thinking_mode` force-off, `repeat` 3, `sampling_source` server, duration 810 s, runaways 0, started 2026-11-26T14:38:36.306898Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.0/15 | 14 | 2 |
| instructfollow-15 | 9.0/15 | 10 | 2 |
| structoutput-15 | 12.0/15 | 13 | 2 |
| dataextract-15 | 12.0/15 | 14 | 4 |
| reasonmath-15 | 13.7/15 | 14 | 1 |
| bugfind-15 | 14.0/15 | 15 | 2 |
| hermesagent-20 | 13.3/20 | 15 | 3 |
| cli-40 | 20.7/40 | 25 | 9 |
| **total** | **107.7/150** | **120** | **25** |

`thinking_validity`: bugfind-15: ok/0 reasoned, cli-40: ok/0 reasoned, dataextract-15: ok/0 reasoned, instructfollow-15: ok/0 reasoned, reasonmath-15: ok/0 reasoned, structoutput-15: ok/0 reasoned, toolcall-15: ok/0 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

**`w3-nvfp4-fp8-on-r3`** — `equivalent_score_150` **122**, totals 366/450 (81.3 %), `thinking_mode` force-on, `repeat` 3, `sampling_source` server, duration 1474 s, runaways 0, started 2026-11-26T15:22:23.588107Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.0/15 | 13 | 0 |
| instructfollow-15 | 14.7/15 | 15 | 1 |
| structoutput-15 | 11.3/15 | 13 | 3 |
| dataextract-15 | 13.7/15 | 14 | 1 |
| reasonmath-15 | 12.3/15 | 14 | 4 |
| bugfind-15 | 13.0/15 | 14 | 3 |
| hermesagent-20 | 13.7/20 | 16 | 5 |
| cli-40 | 30.3/40 | 33 | 6 |
| **total** | **122.0/150** | **132** | **23** |

`thinking_validity`: bugfind-15: ok/45 reasoned, cli-40: ok/120 reasoned, dataextract-15: ok/45 reasoned, instructfollow-15: ok/45 reasoned, reasonmath-15: ok/45 reasoned, structoutput-15: ok/45 reasoned, toolcall-15: ok/45 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

### `bench.sh` — primitive-ai NVFP4/FP8 — FP8 KV

| cell | n | mean | std | CV | min | max | TTFT / depth note |
|---|---:|---:|---:|---:|---:|---:|---|
| narrative | 5 | 346.46 tok/s | 9.39 | 2.7 % | 332.27 | 355.98 | TTFT 22-25 ms |
| code | 5 | 482.78 tok/s | 16.62 | 3.4 % | 458.53 | 499.30 | TTFT 23-25 ms |
| prefill-10k | 3 | 7818.56 tok/s | 47.00 | 0.6 % | 7764.30 | 7846.74 | TTFT 1279-1288 ms; decode at depth 372.3 tok/s |
| prefill-90k | 1 | 7110.65 tok/s | 0.00 | 0.0 % | 7110.65 | 7110.65 | TTFT 12205-12205 ms; decode at depth 301.1 tok/s |

<details><summary>per-run rows (every warm-up and measured run)</summary>

```
warm-1     wall=  3.04s  ttft=   114ms  toks=1000  wall_TPS=328.52  decode_TPS=341.31
warm-2     wall=  2.89s  ttft=    21ms  toks=1000  wall_TPS=345.65  decode_TPS=348.22
warm-3     wall=  2.88s  ttft=    25ms  toks=1000  wall_TPS=346.86  decode_TPS=349.85
run-1      wall=  3.03s  ttft=    22ms  toks=1000  wall_TPS=329.82  decode_TPS=332.27
run-2      wall=  2.89s  ttft=    25ms  toks=1000  wall_TPS=345.53  decode_TPS=348.53
run-3      wall=  2.94s  ttft=    22ms  toks=1000  wall_TPS=340.04  decode_TPS=342.62
run-4      wall=  2.83s  ttft=    21ms  toks=1000  wall_TPS=353.36  decode_TPS=355.98
run-5      wall=  2.85s  ttft=    21ms  toks=1000  wall_TPS=350.28  decode_TPS=352.91
warm-1     wall=  1.06s  ttft=    26ms  toks= 518  wall_TPS=486.53  decode_TPS=498.60
warm-2     wall=  0.90s  ttft=    25ms  toks= 440  wall_TPS=487.65  decode_TPS=501.64
warm-3     wall=  0.85s  ttft=    25ms  toks= 417  wall_TPS=492.71  decode_TPS=507.87
run-1      wall=  1.16s  ttft=    25ms  toks= 566  wall_TPS=488.66  decode_TPS=499.30
run-2      wall=  0.91s  ttft=    22ms  toks= 420  wall_TPS=461.64  decode_TPS=473.14
run-3      wall=  1.71s  ttft=    24ms  toks= 774  wall_TPS=452.20  decode_TPS=458.53
run-4      wall=  0.88s  ttft=    24ms  toks= 419  wall_TPS=477.71  decode_TPS=490.99
run-5      wall=  0.92s  ttft=    23ms  toks= 442  wall_TPS=479.60  decode_TPS=491.92
warm-1     wall=  1.29s  ttft=  1270ms  prompt_toks=  9565  PP_tok/s=7529.41
run-1      wall=  1.58s  ttft=  1274ms  prompt_toks=  9997  PP_tok/s=7844.64
run-2      wall=  1.69s  ttft=  1288ms  prompt_toks=  9997  PP_tok/s=7764.30
run-3      wall=  1.62s  ttft=  1274ms  prompt_toks=  9997  PP_tok/s=7846.74
warm-1     wall= 12.32s  ttft= 12307ms  prompt_toks= 89075  PP_tok/s=7237.90
run-1      wall= 12.63s  ttft= 12205ms  prompt_toks= 86785  PP_tok/s=7110.65
```
</details>

### `bench-agentic.sh` — primitive-ai NVFP4/FP8 — FP8 KV

```
SUMMARY — multi-turn prefill stress (2 session(s) × 12 turns)
  whole replies  24/24 turn(s) arrived in ONE output chunk (the engine
    Turn 1 (cold):            314 ms TTFT  — compile/warmup, excluded from growth
    Turn 2 (warm base):      110 ms TTFT @ 1,445 prompt tokens
    Turn 12:                 1099 ms TTFT @ 35,382 prompt tokens
    Context grew 24.5×,  TTFT grew 10.0× (warm baseline → last turn)
       (see 'whole replies' above), so this growth is not prompt-read cost alone.
```

### `concurrency-probe.sh` — primitive-ai NVFP4/FP8 — FP8 KV

```
round    done  silent  errors  vram_MB  agg_t/s  per-strm  ttft_ms  pf_t/s  ttft_p95  tps_p05   run/wait pfxhit
    1    8/8        0       0    60104     98.2      79.7    10703    1495     20163     49.4    -/-          -
    2    8/8        0       0    60168     98.0      77.5    10679    1498     20158     49.7    -/-          -
    3    8/8        0       0    60168     98.6      78.7    10755    1488     20148     48.5    -/-          -
    4    8/8        0       0    60168     98.6      77.7    10572    1513     20073     50.0    -/-          -
    5    8/8        0       0    60168     97.4      77.4    10898    1468     20359     48.3    -/-          -
=== verdict (N=8) ===
  VRAM: cold 59944 -> warm 60168 MB (pool fill 224 MB, expected) -> final 60168 MB (post-warm growth 0 MB / 200)  peak 60168 MB
  VRAM peak per GPU: GPU 0 15042 MB · GPU 1 15042 MB · GPU 2 15042 MB · GPU 3 15042 MB  (every GPU on the host (rig-wide: no container to scope to))
  per-stream decode: 77.4 tok/s (steady) · aggregate 97.4 tok/s (8 streams) · retention 99.3% (min 98%) · floor off
  concurrent prefill: steady TTFT 10898 ms (median of 8 streams) · ~1468 tok/s/stream uncached @ 16000-tok prompts · agg-prefill ~6287 tok/s
  tail: TTFT p95 20359 ms (vs p50 10898) · slowest-decile stream 48.3 tok/s (vs median 77.4)
  concurrency 8 @ ~16000 tok: PASS — sustained clean
RESULT N=8 ctx=16000 clean=1 pass=1 mps_tps=77.45 agg_tps=97.39 retention=0.993 leak=0 vram_peak=60168 floor_ok=1 ttft_ms=10898 pf_tps=1468.2 ttft_p95_ms=20359 tps_p05=48.30 running=- waiting=- prefix_hit=- cache=cold cached_toks=0 pf_agg=6287.3 vram_gpus=all vram_peak_gpus=0:15042,1:15042,2:15042,3:15042 running_max=- waiting_max=-
```

### `verify-stress.sh` — primitive-ai NVFP4/FP8 — FP8 KV

```
  [32m✓[0m all long-ctx depths recalled secret correctly
  [32m✓[0m all long-ctx depths recalled secret correctly
[8/8] Context ceiling ladder (staggered NIAH from ~95000 → ~0.92 × n_ctx) ...
  [32m✓[0m ceiling ladder: all 6 rungs passed — fillable to 240661 tok (91% of n_ctx=262144)
  [32mboundary checks[0m  5/5 passed  [2m(tool-prefill OOM, agent shapes, reasoning-heavy, VRAM ceiling)[0m
  [32mrecall ladder[0m    3/3 passed  [2m(uniform-haystack: ADDRESSABILITY, not retrieval[0m
[32mAll stress / boundary checks passed.[0m KV-cache and prefill paths are sound for the deployed config.
```

### `soak-test.sh --continuous` — primitive-ai NVFP4/FP8 — FP8 KV

```
[soak]   verdict              PASS
[soak]   boot_vram_mib        60168
[soak]   max_vram_mib         60168
[soak]   max_growth_mib       0 / 200
[soak]   errors               0
[soak]   silent_empty         0 / 25 (0.0%)
[soak]   p50_decode_tps       367.96
[soak]   decode_basis         4 client-timed / 21 no decode figure (reply arrived whole in one output chunk, no engine counter) of 25
[soak]   whole_replies        21 / 25  (one output chunk: no client decode window; TTFT includes their decode)
[soak]   p95_ttft_ms          1314
[soak]   tps_retention        100.0%
```

---

## primitive-ai NVFP4/FP8 — int4 KV

**Engine: OURS** — `libqengine.so` sha256 `313ec400971cc3b0…` mounted over the image's, plus the tree's `server/assets.py`; its boot log carries `Q4bit P5b: sharded pack: 5 shard(s) from model.safetensors.index.json`.

**`w3-nvfp4-kv4-off-r3`** — `equivalent_score_150` **103**, totals 310/450 (68.9 %), `thinking_mode` force-off, `repeat` 3, `sampling_source` server, duration 798 s, runaways 3, started 2026-11-26T15:49:42.663694Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 12.7/15 | 13 | 1 |
| instructfollow-15 | 9.0/15 | 10 | 2 |
| structoutput-15 | 11.7/15 | 13 | 2 |
| dataextract-15 | 11.7/15 | 13 | 3 |
| reasonmath-15 | 13.0/15 | 13 | 0 |
| bugfind-15 | 12.0/15 | 13 | 3 |
| hermesagent-20 | 13.0/20 | 14 | 2 |
| cli-40 | 20.3/40 | 24 | 8 |
| **total** | **103.3/150** | **113** | **21** |

`thinking_validity`: bugfind-15: ok/0 reasoned, cli-40: ok/0 reasoned, dataextract-15: ok/0 reasoned, instructfollow-15: ok/0 reasoned, reasonmath-15: ok/0 reasoned, structoutput-15: ok/0 reasoned, toolcall-15: ok/0 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

**`w3-nvfp4-kv4-on-r3`** — `equivalent_score_150` **124**, totals 372/450 (82.7 %), `thinking_mode` force-on, `repeat` 3, `sampling_source` server, duration 1459 s, runaways 0, started 2026-11-26T16:26:48.382355Z

| pack | pass@1 (mean of 3) | pass@3 | repeats disagreed (`split`) |
|---|---:|---:|---:|
| toolcall-15 | 13.0/15 | 13 | 0 |
| instructfollow-15 | 14.0/15 | 15 | 2 |
| structoutput-15 | 12.0/15 | 13 | 3 |
| dataextract-15 | 13.7/15 | 14 | 1 |
| reasonmath-15 | 14.0/15 | 15 | 3 |
| bugfind-15 | 13.3/15 | 15 | 3 |
| hermesagent-20 | 13.3/20 | 16 | 5 |
| cli-40 | 30.7/40 | 34 | 7 |
| **total** | **124.0/150** | **135** | **24** |

`thinking_validity`: bugfind-15: ok/45 reasoned, cli-40: ok/120 reasoned, dataextract-15: ok/45 reasoned, instructfollow-15: ok/45 reasoned, reasonmath-15: ok/45 reasoned, structoutput-15: ok/45 reasoned, toolcall-15: ok/45 reasoned — and `hermesagent-20` is absent from the map, so the count covers seven of the eight packs.

### `bench.sh` — primitive-ai NVFP4/FP8 — int4 KV

| cell | n | mean | std | CV | min | max | TTFT / depth note |
|---|---:|---:|---:|---:|---:|---:|---|
| narrative | 5 | 349.52 tok/s | 5.87 | 1.7 % | 340.88 | 357.45 | TTFT 25-26 ms |
| code | 5 | 509.41 tok/s | 36.53 | 7.2 % | 456.96 | 552.03 | TTFT 24-25 ms |
| prefill-10k | 3 | 7904.48 tok/s | 98.37 | 1.2 % | 7790.94 | 7964.08 | TTFT 1236-1284 ms; decode at depth 312.4 tok/s |
| prefill-90k | 1 | 7035.70 tok/s | 0.00 | 0.0 % | 7035.70 | 7035.70 | TTFT 12792-12792 ms; decode at depth 217.5 tok/s |

<details><summary>per-run rows (every warm-up and measured run)</summary>

```
warm-1     wall=  3.12s  ttft=   164ms  toks=1000  wall_TPS=320.33  decode_TPS=338.12
warm-2     wall=  3.07s  ttft=    21ms  toks=1000  wall_TPS=325.22  decode_TPS=327.42
warm-3     wall=  2.98s  ttft=    21ms  toks=1000  wall_TPS=335.12  decode_TPS=337.54
run-1      wall=  2.88s  ttft=    22ms  toks=1000  wall_TPS=347.50  decode_TPS=350.16
run-2      wall=  2.89s  ttft=    25ms  toks=1000  wall_TPS=346.54  decode_TPS=349.53
run-3      wall=  2.89s  ttft=    26ms  toks=1000  wall_TPS=346.50  decode_TPS=349.61
run-4      wall=  2.82s  ttft=    25ms  toks=1000  wall_TPS=354.27  decode_TPS=357.45
run-5      wall=  2.96s  ttft=    26ms  toks=1000  wall_TPS=337.91  decode_TPS=340.88
warm-1     wall=  0.90s  ttft=    21ms  toks= 446  wall_TPS=494.27  decode_TPS=506.29
warm-2     wall=  1.74s  ttft=    25ms  toks= 800  wall_TPS=460.76  decode_TPS=467.50
warm-3     wall=  0.69s  ttft=    24ms  toks= 357  wall_TPS=518.52  decode_TPS=537.51
run-1      wall=  0.62s  ttft=    23ms  toks= 332  wall_TPS=531.34  decode_TPS=552.03
run-2      wall=  0.78s  ttft=    25ms  toks= 399  wall_TPS=513.76  decode_TPS=530.61
run-3      wall=  0.84s  ttft=    24ms  toks= 420  wall_TPS=500.53  decode_TPS=515.03
run-4      wall=  0.99s  ttft=    23ms  toks= 474  wall_TPS=480.69  decode_TPS=492.42
run-5      wall=  1.78s  ttft=    25ms  toks= 800  wall_TPS=450.54  decode_TPS=456.96
warm-1     wall=  1.33s  ttft=  1320ms  prompt_toks=  9919  PP_tok/s=7516.44
run-1      wall=  1.76s  ttft=  1284ms  prompt_toks= 10003  PP_tok/s=7790.94
run-2      wall=  1.59s  ttft=  1256ms  prompt_toks= 10003  PP_tok/s=7964.08
run-3      wall=  1.61s  ttft=  1167ms  prompt_toks=  9289  PP_tok/s=7958.42
warm-1     wall= 12.09s  ttft= 12078ms  prompt_toks= 85894  PP_tok/s=7111.55
run-1      wall= 13.38s  ttft= 12792ms  prompt_toks= 89998  PP_tok/s=7035.70
```
</details>

### `bench-agentic.sh` — primitive-ai NVFP4/FP8 — int4 KV

```
SUMMARY — multi-turn prefill stress (2 session(s) × 12 turns)
  whole replies  24/24 turn(s) arrived in ONE output chunk (the engine
    Turn 1 (cold):            333 ms TTFT  — compile/warmup, excluded from growth
    Turn 2 (warm base):      132 ms TTFT @ 1,445 prompt tokens
    Turn 12:                 1118 ms TTFT @ 35,394 prompt tokens
    Context grew 24.5×,  TTFT grew 8.4× (warm baseline → last turn)
       (see 'whole replies' above), so this growth is not prompt-read cost alone.
```

### `concurrency-probe.sh` — primitive-ai NVFP4/FP8 — int4 KV

```
round    done  silent  errors  vram_MB  agg_t/s  per-strm  ttft_ms  pf_t/s  ttft_p95  tps_p05   run/wait pfxhit
    1    8/8        0       0    59968     98.8      76.1    10551    1516     20033     50.4    -/-          -
    2    8/8        0       0    60176     97.7      76.5    10798    1482     20288     45.6    -/-          -
    3    8/8        0       0    60176     98.3      77.3    10777    1485     20179     45.9    -/-          -
    4    8/8        0       0    60184     97.2      76.3    10835    1477     20337     45.7    -/-          -
    5    8/8        0       0    60184     98.8      80.1    10641    1504     20070     49.6    -/-          -
=== verdict (N=8) ===
  VRAM: cold 59904 -> warm 60176 MB (pool fill 272 MB, expected) -> final 60184 MB (post-warm growth 8 MB / 200)  peak 60184 MB
  VRAM peak per GPU: GPU 0 15046 MB · GPU 1 15046 MB · GPU 2 15046 MB · GPU 3 15046 MB  (every GPU on the host (rig-wide: no container to scope to))
  per-stream decode: 80.1 tok/s (steady) · aggregate 98.8 tok/s (8 streams) · retention 101.7% (min 98%) · floor off
  concurrent prefill: steady TTFT 10641 ms (median of 8 streams) · ~1504 tok/s/stream uncached @ 16000-tok prompts · agg-prefill ~6378 tok/s
  tail: TTFT p95 20070 ms (vs p50 10641) · slowest-decile stream 49.6 tok/s (vs median 80.1)
  concurrency 8 @ ~16000 tok: PASS — sustained clean
RESULT N=8 ctx=16000 clean=1 pass=1 mps_tps=80.11 agg_tps=98.84 retention=1.017 leak=8 vram_peak=60184 floor_ok=1 ttft_ms=10641 pf_tps=1503.6 ttft_p95_ms=20070 tps_p05=49.62 running=- waiting=- prefix_hit=- cache=cold cached_toks=0 pf_agg=6377.8 vram_gpus=all vram_peak_gpus=0:15046,1:15046,2:15046,3:15046 running_max=- waiting_max=-
```

### `verify-stress.sh` — primitive-ai NVFP4/FP8 — int4 KV

```
  [32m✓[0m all long-ctx depths recalled secret correctly
  [32m✓[0m all long-ctx depths recalled secret correctly
[8/8] Context ceiling ladder (staggered NIAH from ~95000 → ~0.92 × n_ctx) ...
  [32m✓[0m ceiling ladder: all 6 rungs passed — fillable to 240657 tok (91% of n_ctx=262144)
  [32mboundary checks[0m  5/5 passed  [2m(tool-prefill OOM, agent shapes, reasoning-heavy, VRAM ceiling)[0m
  [32mrecall ladder[0m    3/3 passed  [2m(uniform-haystack: ADDRESSABILITY, not retrieval[0m
[32mAll stress / boundary checks passed.[0m KV-cache and prefill paths are sound for the deployed config.
```

### `soak-test.sh --continuous` — primitive-ai NVFP4/FP8 — int4 KV

```
[soak]   verdict              PASS
[soak]   boot_vram_mib        60184
[soak]   max_vram_mib         60184
[soak]   max_growth_mib       0 / 200
[soak]   errors               0
[soak]   silent_empty         0 / 25 (0.0%)
[soak]   p50_decode_tps       359.25
[soak]   decode_basis         5 client-timed / 20 no decode figure (reply arrived whole in one output chunk, no engine counter) of 25
[soak]   whole_replies        20 / 25  (one output chunk: no client decode window; TTFT includes their decode)
[soak]   p95_ttft_ms          1314
[soak]   tps_retention        100.0%
```

---

## Failure inventory

Counts are **scored rows** (a scenario contributes three, one per repeat); a leg has 450.

| leg | wrong_answer | verifier_fail | timeout | token_limit | server_error | other |
|---|---:|---:|---:|---:|---:|---:|
| `w1-int4-fp8-off-r3` | 6 | 97 | 0 | 1 | 1 | 6 |
| `w1-int4-fp8-on-r3` | 2 | 62 | 0 | 0 | 0 | 3 |
| `w1-int4-kv4-off-r3` | 6 | 94 | 0 | 1 | 0 | 7 |
| `w1-int4-kv4-on-r3` | 0 | 64 | 0 | 0 | 0 | 2 |
| `w2-int8-fp8-off-r3` | 8 | 111 | 0 | 1 | 0 | 8 |
| `w2-int8-fp8-on-r3` | 0 | 64 | 0 | 0 | 0 | 4 |
| `w2-int8-kv4-off-r3` | 9 | 103 | 0 | 0 | 0 | 7 |
| `w2-int8-kv4-on-r3` | 0 | 63 | 0 | 0 | 0 | 4 |
| `w3-nvfp4-fp8-off-r3` | 7 | 114 | 0 | 0 | 1 | 5 |
| `w3-nvfp4-fp8-on-r3` | 8 | 71 | 0 | 0 | 0 | 5 |
| `w3-nvfp4-kv4-off-r3` | 4 | 127 | 0 | 3 | 1 | 5 |
| `w3-nvfp4-kv4-on-r3` | 3 | 71 | 0 | 0 | 0 | 4 |

### The truncations, named

- `w1-int4-fp8-off-r3` · **reasonmath-15 / RM-04** · token_limit · output truncated at token limit (finish_reason=length); underlying verdict was wrong_answer: Answer axis 0/2, trace axis 1/2 (15%). Missing final "ANSWER: " line. Matched 1/4 check
- `w1-int4-kv4-off-r3` · **reasonmath-15 / RM-04** · token_limit · output truncated at token limit (finish_reason=length); underlying verdict was wrong_answer: Answer axis 0/2, trace axis 0/2 (0%). Missing final "ANSWER: " line. No published check
- `w2-int8-fp8-off-r3` · **reasonmath-15 / RM-04** · token_limit · output truncated at token limit (finish_reason=length); underlying verdict was wrong_answer: Answer axis 0/2, trace axis 1/2 (15%). Missing final "ANSWER: " line. Matched 1/4 check
- `w3-nvfp4-kv4-off-r3` · **reasonmath-15 / RM-04** · token_limit · output truncated at token limit (finish_reason=length); underlying verdict was wrong_answer: Answer axis 0/2, trace axis 1/2 (15%). Missing final "ANSWER: " line. Matched 2/4 check

Every truncation above is a scenario whose *underlying* verdict was already a failure, so no pass was lost to a
cap; the detail strings are the verifier's own words, unedited.

