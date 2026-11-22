# `sweet16/` — bench reports from one 4 × RTX 5080 box

This directory holds the reports, drafts and diffs from our own bench work on `bpx03/QEngine`. **None of it has been
sent to the maintainer or to a club.** They are here so the measurement and the diff sit in public next to each other,
and so anyone can check a number against the body that claims it.

| file | what it is |
|---|---|
| `dsv4-sm120.md` + `dsv4-sm120.patch` | the DSV4 sm_120 build arm against an identical sm_86 control |
| `report-row-cost-and-pfb-2026-11-22.md` | the drafter's row-cost counter in the serving server (with the patch), and `FLASH_PFB_BIG` at `FLASH_KV=ram` |
| `report-club-hardware-note-2026-11-22.md` | the same box described as hardware rather than as a benchmark row |
| `comments-qe-41-40-2026-11-22.md` | draft comments for two reproduced dsv4 defects |
| `comments-qe-31-28-2026-11-22.md` | draft notes on two closed reports |

**What is not here:** our own workload's numbers (the row-cost series, the card-count verdict) and anything about the
hosts we run — those stay with us. Every file here is scrubbed of paths, hostnames and identifiers before it is
committed.
