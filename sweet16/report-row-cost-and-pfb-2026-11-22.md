# Two engine notes: the drafter's row-cost counter in the serving server, and `FLASH_PFB_BIG` at host-RAM KV

**Bench note, 2026-11-22.** Published on this fork for the record; **nothing here has been sent to a maintainer or a
club.** Upstream `bpx03/QEngine` is the product; this fork is a bench, and the notes are not a support offer.
Upstream's licence and headers are untouched.

Measurement host: 4 × RTX 5080 16 GB, PCIe Gen5 ×8 behind a Broadcom/LSI PEX890xx Gen 5 switch, peer access OK on
all pairs, driver 610.57.04, CUDA 13.1, 125 GiB host RAM, power caps at the 252/250 W vBIOS floor. Weights on local
NVMe, never a network mount.

---

Kept separate from A/B so neither body has to be re-read: **A and B stand as written**, and these are additive drafts in
the same venue shape. Both are measured on the same rig as A; both are scrubbed of paths and identifiers; **neither has
been sent anywhere.**

### D1 — the row-cost counter in the serving server (a patch, off by default)

**What.** `server/app.py` never printed the drafter's `T_obs` block, while the legacy `server/dsv4_app.py` does: the legacy
entrypoint holds the ctypes wrapper (`dsv4_engine.CLib`) directly, and in `app.py` the same wrapper sits one level down as
`Dsv4Engine.c`, so the counter was never read. A ten-line change calls `CLib.drafter_stats()` / `drafter_block()` after a
reply **only when `DSV4_DSTATS=1`** and prints one line in the legacy server's own format. With the variable unset nothing
changes.

**Why it is worth your while.** With it, a row-cost series can be taken through the server people actually run, instead of
having to fall back to the legacy entrypoint — which is the only reason our own record had to keep two shapes apart. It is
also the smallest possible demonstration of the shape you might want in the engine: *one counter, one line, off by
default*.

**Measured with it** (four boots, interleaved 4·3·4·3, four RTX 5080 16 GB, 262 144, judge off, forced cold, one 760-token
greedy request a boot; `T_obs[n]` in ms is the mean time of a verify pass that tested `n` draft rows):

| `n` | four cards | three cards | change (mean of pairs) |
|---:|---:|---:|---:|
| 1 | 48.5 | 59.9 | −18.9 % |
| 2 | 62.6 | 73.6 | −14.9 % |
| 3 | 76.8 | 89.2 | −13.9 % |
| 4 | 108.0 | 170.5 | −36.7 % (wide) |
| 5 | 109.3 | 144.8 | −24.5 % (wide) |

The absolute values agree with the legacy entrypoint's to about 1 % at four cards, so the counter is a property of the
shape, not of the server.

**Two things the patch's own wiring taught us, in case they save you a boot:** mount the engine module as well as the
server file when running from source over an image (the counter's reader lives in the engine module, and mounting only
`app.py` silently runs the image's older one), and `app.py`'s `--host` defaults to `127.0.0.1`, so a published container
port accepts and then resets every request unless `--host 0.0.0.0` is passed.

**Status:** branch `ss16/serving-tobs` on our fork (public), one commit, nothing sent.

### D2 — `FLASH_PFB_BIG` at `FLASH_KV=ram`: a default, and the reading it invites

**What we measured** (four RTX 5080 16 GB, 262 144, one variable per boot, both boot orders, fresh-nonce 32K and 90K
prompts; the 32K prefill wall time):

| `FLASH_PFB_BIG` | 32K prefill | 90K prefill |
|---|---:|---:|
| `0` / `2048` (the floor) | 6.43 / 6.41 s | 13.27 / 13.18 s |
| `4096` (what the docs say is the value at `FLASH_KV=ram`) | 4.99 s | 10.44 s |
| **`8192`** | **4.34 s** | **8.98 s** |
| `16384` | 4.52 s | 9.41 s |
| `8192` + `FLASH_PFB_ALIAS=0` | 4.32 s | 8.97 s |

**The point is not the number, it is the sentence.** `docs/advanced.md` says the large-batch rows are mapped onto the KV
tail and that "the mapping needs the KV cache in VRAM; with `FLASH_KV=ram` the default stays `4096`". We read that as a
statement about the **flag** and spent months treating `FLASH_PFB_BIG=8192` as a recorded no-op at our shape. It is a
statement about the **default**: the flag works at `FLASH_KV=ram`, the alias trick does not (`ALIAS=0` changes nothing),
and the difference between the default and the optimum is **15 % of prefill at 32K and 16 % at 90K** (32 % from the
floor). The same reading reproduced at a 1 M context (−14.2 % / −14.5 %).

**Suggested framing if you want it upstream:** a one-sentence clarification in that section ("the `4096` default applies
when the variable is unset; setting it still sizes the batch at `FLASH_KV=ram`, where only the KV-tail alias is
unavailable"). We are happy to send it as a doc PR against a named commit; it is your call whether it is worth a diff.

### D3 — what we are NOT sending

The absolute `T_obs` values, the saturation arithmetic and the four-card verdict are our own workload's; they are not published here.
they are our workload's, not a general result.
