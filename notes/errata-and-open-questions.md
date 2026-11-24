# Errata & open questions

Corrections to the original syllabus and things still to verify on a pinned release.

## Corrections applied

1. **Boot lifecycle (Ch 1).** ONIE is the *install* path, not part of normal boot. Normal boot:
   BIOS/UEFI → GRUB (U-Boot on ARM) → SONiC kernel → systemd → containers → supervisord.
2. **Ordered delivery (Ch 6).** `ProducerStateTable` coalesces (latest value per key wins) — it
   does not guarantee ordering. sairedis → ASIC_DB uses `ProducerTable`/`ConsumerTable`, a
   strictly ordered queue (optionally ZMQ in newer releases).
3. **DB IDs (Ch 5).** Instances and IDs come from `database_config.json` via `SonicDBConfig`;
   `schema.h` holds table names. Some IDs are shared (PFC watchdog tables also in DB 5).
   Multi-ASIC runs one Redis per namespace.
4. **FLEX_COUNTER_DB writer (Ch 16).** Changed between releases — older: orchagent writes
   directly; newer: via sairedis → syncd.

## Structure notes

- SAI primer moved forward (end of Ch 3) because Ch 11 uses SAI objects before Ch 13.
- Ch 4 (build system) may move after Part II or into an appendix.
- SONiC-VPP cannot model buffers/PFC/WRED or most HW counters — Ch 12 and parts of Ch 16 need
  sonic-vs, `sairedis.rec` traces, or real hardware.

## Open questions

- [ ] Pin exact release branch for the book.
- [ ] Confirm current `portsyncd` vs. `portmgrd` responsibilities on the pinned branch.
- [ ] Confirm FLEX_COUNTER_DB writer on the pinned branch.
