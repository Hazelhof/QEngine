# A hardware note: four RTX 5080 16 GB behind a Gen5 switch, measured

**Bench note, 2026-11-22.** Published on this fork for the record; **nothing here has been sent to a maintainer or a
club.** Upstream `bpx03/QEngine` is the product; this fork is a bench, and the notes are not a support offer.
Upstream's licence and headers are untouched.

Measurement host: 4 × RTX 5080 16 GB, PCIe Gen5 ×8 behind a Broadcom/LSI PEX890xx Gen 5 switch, peer access OK on
all pairs, driver 610.57.04, CUDA 13.1, 125 GiB host RAM, power caps at the 252/250 W vBIOS floor. Weights on local
NVMe, never a network mount.

---

*(Written for `noonghunna/club-3090` as a hardware note rather than a benchmark row; not sent.)*

**Proposed title:** `4× RTX 5080 16 GB: building QEngine's dsv4 engine for sm_120 halves start-up and does not move decode`

**Body (short):**

QEngine ships its DeepSeek-V4.1-Flash engine compiled for `sm_86` only (`cuobjdump` on the shipped `libqdsv4.so`:
4× `sm_86` cubin + `sm_86` PTX, no `sm_120`), so on 50-series cards it runs JITed PTX. Built the same source at
`sm_120` with a one-variable patch and ran both beside each other on four 5080s, same image, same weights, same cache,
`QE_DSV4_LIB` swapping only the library:

| boot | narrative / code (tok/s) | CV | boot → health |
|---|---|---|---|
| sm_86 control (2 boots) | 27.56 / 39.00 · 28.37 / 39.94 | 5.6 / 5.5 · 2.0 / 4.8 | 48 s · 51 s |
| sm_120 (2 boots) | 28.25 / 39.83 · 26.41 / 36.85 | 2.6 / 1.5 · 3.4 / 3.9 | **24 s · 22 s** |
| shipped library, same night | 28.04 / 39.30 | 2.7 / 4.8 | 47 s |

No material decode delta (means −2.3 % / −2.9 %, sign flips between pairings, later-boot effect 2–8 %). **Start-up
halves (47–51 s → 22–24 s)**, and a cold-probe shows the JIT is paid at load, not on the first request (first and second
12 942-token requests both 15.20 s). Shape: 262 144 served, 8 slots, KV `fp8_e4m3`, three cards (engine ignores the
fourth), caps at 252/250/250 W.

**Rig:** 4 × RTX 5080 16 GB, each link PCIe Gen5 ×8 behind a PEX890xx Gen 5 switch (×8 is the switch's lane budget), peer access OK on all pairs, driver
610.57.04, CUDA 13.1, 125 GiB host, caps at the 252/250 W vBIOS floor. Weights on local NVMe.

**Two caveats worth more than the numbers.** (i) **The same shipped library read 23.15 / 34.97 earlier the same day and
28.04 / 39.30 that night — ~19 % on identical bytes.** If you are comparing a cell against a filed one, re-run the
reference in the same session or the delta is the host. (ii) **Greedy text is not reproducible within one boot on this
engine** (2 of 6 payloads varied across 4 draws; `DSV4_NOCPUPF=1` did not fix it) — any output-identity claim needs more
care than "temperature 0".

*(Bench build from a fork branch, not the released image; upstream is the product.)*

---
