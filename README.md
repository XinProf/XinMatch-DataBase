# XinMatch-DataBase

> [Chinese (Simplified)](README-CN.md) · English

**Build code, format specification, and release artifacts for the XinMatch reference phase database (`.xinDB` v3 format).**

This repository holds **code and documentation only**; the database itself is distributed as compressed archives in [Releases](../../releases)
(a single database is 68 MB / 87 MB / 1.3 GB, not kept in git history). The data source is
[Crystallography Open Database (COD)](http://www.crystallography.net/cod/) and its predicted/theoretical sub-libraries
PCOD / TCOD. This repository performs no structure refinement and does not rewrite the original CIFs; it only computes reflections and packs the indices.

> The sister repository [PolyXRD-DataBase](https://github.com/PolyXRD/PolyXRD-DataBase) publishes data from the same source in
> **polyDB** format (used by PolyXRD). **The two formats follow different conventions and are not interchangeable** (see §2).

---

## 0. Database Build Location Convention (effective 2026-10-24, required reading)

| Item | Convention |
|---|---|
| **Where the database is built** | **This repository, `XinMatch-DataBase`** (no longer scattered across other checkouts). |
| **Where the CIFs required for the build live** | **`sync-COD`** in the parent directory: `D:\Project\XRD\sync-COD` (can be overridden with `XRD_DATA_ROOT`). This repository does not carry the original CIFs. |
| **Default artifact location** | `<repo>/data/<Kind>-<YYYYMMDD>.xinDB`; `--out` takes highest priority, `XINMATCH_XINDB_DIR` overrides only when explicitly set. |
| **Supported scope** | **6 databases**: `COD` / `PCOD` / `TCOD` × `Inorg` (inorganic subset) / `Full` (full database). |
| **Full operating manual** | [Build and acceptance manual](docs/构建与验收手册.md) (build matrix, acceptance commands, release checklist, troubleshooting). |

**One chain**: `sync-COD (CIF) → ①scan_inorg (Inorg only) → ②build (this repo's data/) → ③validate → ④accept/dedicated re-check`.

---

## 1. Release Artifacts (Release `20261020`)

> This repository now supports **6 databases** (`COD/PCOD/TCOD × Inorg/Full`). The `20261020` Release actually publishes
> **three Inorg databases** (COD/PCOD/TCOD-Inorg in the table below); the three `*-Full` databases can already be built
> with the same set of commands (see §3.3 and the build manual), but their size and build time are on the order of hours, so they are not yet included in this Release.

| Database | entry_id range | Phases | Reflections | Blocks | Elements | After decompression | zip |
|---|---|---|---|---|---|---|---|
| **COD-Inorg** | `96-100-*` | 78,036 | 9,394,774 | 20 | 96 kinds | 118,710,272 B (113.2 MiB) | **87.2 MiB** (91,484,834 B) |
| **PCOD-Inorg** | `96-210-*` | 1,061,953 | 159,274,265 | 260 | 19 kinds | 1,776,353,280 B (1694.1 MiB) | **1.35 GiB** (1,447,251,707 B) |
| **TCOD-Inorg** | `96-110-*` | 46,846 | 5,874,790 | 12 | 91 kinds | 71,352,320 B (68.0 MiB) | **54.1 MiB** (56,694,762 B) |
| **Total** | — | **1,186,835** | **174,543,829** | 292 | — | 1.83 GiB | ≈1.52 GiB |

- The three databases **do not contain each other and are installed on demand** (XinMatch supports mounting multiple databases). The COD experimental database is the workhorse for routine interactive searches;
  PCOD (GRINSP predictions) and TCOD (theoretical) serve as the "fallback when the experimental database finds nothing".
- The `20261020` in the name is the **source data snapshot date** (`meta.data_date`), not the build date;
  see `meta.build_time` for the build time (COD 2026-10-21 17:25, TCOD 2026-10-22 23:29, PCOD 2026-10-22 23:46).
- All distributed builds use `journal_mode=DELETE`, with no `-wal`/`-shm` sidecars, so they can be placed directly on read-only media
  (NAS read-only shares, optical disc images) without worrying about "even reading requires write permission".
- Checksums are in `SHA256SUMS.txt` attached to the Release.

```bash
unzip COD-Inorg-20261020.xinDB.zip                    # produces COD-Inorg-20261020.xinDB
python -m tools.validate_xindb COD-Inorg-20261020.xinDB   # optional: 25-item self-check
```

> The three databases have already been placed under the repository's `data/` on this machine (`data/COD-Inorg-20261020.xinDB`, etc., not in git),
> and XinMatch's library registry `~/.xinmatch/libraries.json` already points to that directory.

> ⚠️ The filename must be `<library_kind>-<data_date>.xinDB`, otherwise `validate_xindb`'s
> `M3.identity` will report FAIL (it reconciles the filename against `meta`). Just rebuild the database after renaming; there is no need to fix `meta`.

### 1.1 2026-11-01 Rebuild (after the peak-engine fix, local artifacts in `data/`)

2026-10 fixed two kinds of "fake-peak phases": reflections at the 2θ=180° boundary (`d=λ/2`) were put into the peak table by floating-point rounding + LP divergence caused the whole pattern to collapse;
when gemmi cannot read a unit cell it used a default placeholder cell and computed a `d=1.0` fake peak. The three Inorg databases were rebuilt with the fixed engine,
**the source snapshot remains `20261020`** (`meta.data_date` unchanged). Before the rebuild, the old databases were placed in `data/_prev/` for entry-by-entry comparison,
**and were deleted on 2026-11-04 after acceptance was completed** (comparison results are kept in the acceptance report):

| Database | Phases | Reflections | Size (bytes) | SHA-256 (first 16) | Acceptance |
|---|---|---|---|---|---|
| **COD-Inorg** (rebuild) | **78,035** | **9,395,681** | **118,640,640** (113.1 MiB) | `7C08B34DCACA9BAC…` | validate 0F/1W (30 items); accept 0F/1W; 7 bad phases fixed, 1 phase without a unit cell removed |
| **TCOD-Inorg** (rebuild) | **46,846** | **5,874,790** | **71,352,320** (68.0 MiB) | `8F2A25D583B577A7…` | validate 0F/1W (29 items); accept 0F/1W; entry-by-entry `(d,i)` **exactly identical** to the old database |
| **PCOD-Inorg** (rebuild) | **1,061,953** | **159,275,318** | **1,777,357,376** (1694.1 MiB) | `ABB7EC8C1E68CD6F…` | validate 0F/1W (29 items); accept 0F/1W; 7 bad phases fixed; 17/1,061,953 payload changes |
| **TCOD-Full** (new database) | **361,701** | **53,099,238** | **623,173,632** (594.3 MiB) | `36838FBE1F18EDBF…` | validate 0F/2W (29 items); accept 0F/1W; 46,846/46,846 common phases with TCOD-Inorg byte-identical; self-recall 10/10 |
| **COD-Full** (new database) | **532,038** | **77,458,154** | **922,120,192** (879.4 MiB) | `E217F28FA03755A1…` | validate 0F/2W (30 items); accept 0F/2W (20 items); 78,035/78,035 common phases with COD-Inorg byte-identical; self-recall 10/10 |
| **MP** (external database) | **155,201** | **20,900,193** | **251,633,664** (240.0 MiB) | `26D7A9BED83A57C0…` | validate 0F/1W (29 items); accept 0F/1W (19 items); id set exactly equals the 155,201 archived `mp-id`s; self-recall Top-1/Top-5 20/20 |

> **PCOD-Full is not built**: all 1,061,962 CIFs in the PCOD archive **are inorganic** (manifest `is_inorg=1` 100%),
> so it is essentially equivalent to `PCOD-Inorg` (the original archive differs by only 2 CIFs that were not included); it was skipped by decision.

- **MP external database**: `data/MP-20261031.xinDB` (Materials Project, 155,201 phases, ids are native `mp-<n>`,
  `tier=2` computed structures); XinMatch mounts it via "Add external database". For the build/acceptance see [MP database build and acceptance](docs/MP-库构建与验收.md),
  released as [`20261031-mp`](https://github.com/PolyXRD/XinMatch-DataBase/releases/tag/20261031-mp) (zip + `SHA256SUMS.txt`).

- Acceptance details, the per-phase bad-phase list, and before/after comparison are in [`docs/ACCEPTANCE-20261101-rebuild.md`](docs/ACCEPTANCE-20261101-rebuild.md).
- **Release published**: [`20261020-rebuild`](https://github.com/PolyXRD/XinMatch-DataBase/releases/tag/20261020-rebuild) (2026-11-04) contains zips for 5 databases + `SHA256SUMS.txt`; release notes [`docs/release-notes-20261104.md`](docs/release-notes-20261104.md). Packaging command: `python -m tools.package_release --out _release\<date>` (manual §5).
- The pre-rebuild old databases in `data/_prev/` (COD 113 MB + TCOD 68 MB + PCOD 1.7 GB) **were deleted on 2026-11-04**; the old `20261020` Release (containing three Inorg zips with bad phases) has also been deleted, leaving only this rebuilt Release.

---

## 2. What `.xinDB` Is

**Single-file SQLite**, storing the "pre-screen matrix" and the "full reflection table" separately, with reflection data stored only once:

```
COD-Inorg-20261020.xinDB
├── meta          self-describing key/value (peak convention, database build parameters, traceability)
├── entries       one row per phase (chemical formula / space group / unit cell / density / elements + CSR offsets)
├── peak_chunks   columnar reflection blocks (one block per 4096 phases)
└── element_inv   element inverted index (element → 0-based phase index, uint32 array)
```

| Segment | Shape | Type | Purpose |
|---|---|---|---|
| `top_d`/`top_i` | `n × 32` | `<f4` | Pre-screen: NumPy overlap with the measured peak window, **pruning only, no sorting** |
| `all_d`/`all_i` | `Σpeak_n` (CSR) | `<f4` | Scoring: all reflections of candidate phases |
| `element_inv.eids` | variable length | `<u4` | Ascending, deduplicated, 0-based phase indices |

**Addressing rule**: `entries.rowid` is explicitly written starting from 0 = 0-based phase index; `element_inv.eids` and
`peak_chunks` block numbers are both based on it; they **cannot** be derived from `entry_id`.
For the complete field-level specification see [xinDB v3 format specification](docs/xinDB格式规范-v3.md).

### Peak Convention (hard-coded into `meta.peak_convention`)

```
CuKa1-1.5406-d0.7703-30.0-merge-mult-DW-norm1000
```

λ=1.5406 Å (Cu Kα1) | d 0.7703–30.0 Å | equivalent reflections merged by 2θ (including multiplicity) |
**including Debye-Waller** (`B = 8π²U_iso`, per site) | intensity normalized to 1000 | at most 150 reflections per phase | pre-screen width 32

| | xinDB (this repository) | polyDB (PolyXRD-DataBase) |
|---|---|---|
| Container | single-file SQLite | directory (8 segment files + mmap) |
| d range | 0.7703–30 Å | 1.0–26 Å (2θ 5–100.76°) |
| Temperature factor | **Yes** (Debye-Waller) | No |
| Peaks per phase | 150 | 100 (top K by intensity) |
| Positioning | integrated search + refinement | GUI interactive search |

> The absolute intensity systems computed from the same CIF differ between the two databases. **Mixing them introduces systematic bias**;
> do not use xinDB intensities to validate polyDB, or vice versa.

### Measured Peak Table: `peak_origin = observed`

The convention above describes databases whose peaks are **computed from CIFs** (COD / PCOD / TCOD / MP), whose
`meta.peak_origin` defaults to `computed`. **Measured card databases** such as ICDD PDF-2 are not computed;
when stored they deliberately do not follow this convention:

| meta | Computed database | Measured database (PDF-2) |
|---|---|---|
| `peak_origin` | `computed` (default) | `observed` |
| `peak_convention` | `CuKa1-1.5406-…-norm1000` | `observed-ICDD-PDF2-2004-norm1000` |
| Intensity scale | norm-1000 | **also norm-1000** (otherwise the scoring weights of the two database types are not comparable) |
| NULL unit cell / peaks >150 | build defect | data fact |

No second reader is needed on the read side: `reader_v3` only accepts `schema_version == 3`. The build tool is
`tools/build_pdf2_xindb.py`, and **ICDD commercial data does not enter this repository or Releases**;
see [PDF2 database build and acceptance](docs/PDF2-库构建与验收.md) for details.

---

## 3. Building from Scratch

### 3.1 Environment

| Dependency | Requirement | Notes |
|---|---|---|
| Python | 3.10+ | tested with 3.10.11 |
| numpy | required | reflection computation and columnar packing |
| gemmi | **required** | CIF parsing (hard dependency of `peak_compute`) |
| Disk | ≈ 2× database size | PCOD database 1.7 GB, peak build usage about 4 GB |
| Memory | 8 GB class | chunked writes, never holds the full reflection table in memory |

```bash
pip install numpy gemmi
```

### 3.2 CIF Archive and Inorganic Subset

The code does not download data. The chain is: **original archive → inorganic subset → `.xinDB`**.

**The source data is not in this repository**; since 2026-10-23 it lives in `D:/Project/XRD/sync-COD` (shared with
[PolyXRD-DataBase](https://github.com/PolyXRD/PolyXRD-DataBase),
for data-side tools and layout see [PolyXRD/sync-COD](https://github.com/PolyXRD/sync-COD)):

```
D:/Project/XRD/sync-COD/       ← data root, can be overridden with the XRD_DATA_ROOT environment variable
├── COD/        535,166 CIFs  # full original archive (level 3)
├── PCOD/     1,061,964 CIFs  # GRINSP predictions (level 2)
├── TCOD/       362,294 CIFs  # theoretical (level 3, 8-digit ids)
├── inorg_cif/COD|PCOD|TCOD/    # ← output of step ①: CIF copies judged inorganic (79,198 / 1,061,962 / 46,943)
└── inorg_manifest.sqlite       # ← output of step ①: one-row-per-file manifest (1,959,421 rows, id_clash=0)
```

Step ① (inorganic determination = "no C, or contains C but no H" + copy + manifest):

```bash
python tools/scan_inorg.py --workers 6          # reads and writes the data root above in place by default
XRD_DATA_ROOT=/other/root python tools/scan_inorg.py --workers 6   # override when switching drives
```

- The criterion, id offsets (PCOD +2e7 / TCOD +1e8) are **entry-by-entry identical** to the three databases this repository has already published (the script is reused verbatim).
- The `inorg_cif/` used by the published databases currently lives in `D:\Project\XRD\sync-COD\inorg_cif\` (1,188,103 CIFs,
  shared with polyDB). This repository does not carry this batch of data, and does not need to — read-only references are enough.
- At build time `--cif-root` defaults to `DEFAULT_INORG_*_ROOT` in `xinmatch/core/db/cif_scan.py`,
  which is derived from `XRD_DATA_ROOT` (defaulting to `D:\Project\XRD\sync-COD\inorg_cif\*`).
  On another machine **point to it with the environment variable or `--cif-root`**; no code changes needed.

### 3.3 Database Build (six databases, unified entry point)

`python -m tools.build_cod_xindb` is the only build entry point: `--source {COD,PCOD,TCOD}` × `--set {inorg,full}`,
the CIF root / `library_kind` / id offsets are derived automatically; `tools/build_cod_inorg_xindb.py` is a thin compatibility wrapper around it.

```bash
# Three inorganic databases (default databases for interactive search, small and fast to build)
python -m tools.build_cod_xindb --source COD  --set inorg --data-date 20261020 --workers 14 -v
python -m tools.build_cod_xindb --source PCOD --set inorg --data-date 20261020 --workers 14 -v
python -m tools.build_cod_xindb --source TCOD --set inorg --data-date 20261020 --workers 14 -v

# Three full databases (exhaustive reference; build time on the order of hours, see manual §3.6)
python -m tools.build_cod_xindb --source COD  --set full --data-date 20261020 --workers 14 -v
python -m tools.build_cod_xindb --source PCOD --set full --data-date 20261020 --workers 14 -v
python -m tools.build_cod_xindb --source TCOD --set full --data-date 20261020 --workers 14 -v
```

- Default output is `<repo>/data/<kind>-<data_date>.xinDB`; `--out` specifies any path, `XINMATCH_XINDB_DIR` overrides only when explicitly set.
- `--limit N` for a trial build (**always run a slice first and only build the full set once validate is all green**); `--resume` resumes from a checkpoint (== one complete run, verified);
  `--start` starts from a specified id range; `--workers` is unrelated to the output (1 vs 2 byte-identical, regression-tested).
- **Use 14 for `--workers` (measured optimum for the real full databases on this machine, 8C/16T)**: TCOD 46,846 phases in 15.2 min (16.9 min with the old implementation at 14 workers);
  COD with 8 workers took 61.6 min, slightly slower than the historical 58 min at 14 workers — the 500-phase slice benchmark underestimates the benefit of more workers,
  so do not drop workers to 8 on that basis.
- The build hot path uses `build_pipeline.compute_phase()`, **parsing each CIF only once** (the old implementation: 3 parses + 2 structure builds);
  structure-factor chunking adapts to the atom count (256–8192 rows); both optimizations leave the **output bit-identical**, increasing whole-database throughput by about +10–15% (see manual §3.7).
- Source defaults are `max_peaks=150`, `dmin=0.7703` (= λ/2); if you change the peak convention, also change
  `peak_compute.PEAK_CONVENTION`, otherwise `meta` will not match reality and the validator will block it.
- `--kind full|inorg` is only a backward-compatibility alias for COD (giving it together with `--set` is an error).

### 3.4 Measured Scale (after the 2026-11-01 rebuild)

| Database | Phases | Reflections | Avg peaks/phase | File | Build time (14-worker baseline) |
|---|---|---|---|---|---|
| COD-Inorg | **78,035** | **9,395,681** | 120.4 (min 2 / max 150) | **113.1 MiB** | 61.6 min (`--workers 8`; 14 workers should be faster) |
| PCOD-Inorg | **1,061,953** | **159,275,318** | 150.0 (min 20 / max 150) | **1694.1 MiB** | **13.3 h** (12.7 h with the old implementation; an external job contended during the run) |
| TCOD-Inorg | **46,846** | **5,874,790** | 125.4 (min 3 / max 150) | **68.0 MiB** | **15.2 min** (16.9 min with the old implementation) |
| TCOD-Full | **361,701** | **53,099,238** | 146.8 (min 3 / max 150) | **594.3 MiB** | **12.1 h** (499.3 phases/min, 14 workers) |
| COD-Full | **532,038** | **77,458,154** | 145.6 (min 1 / max 150) | **879.4 MiB** | **36.4 h** (243.7 phases/min, 14 workers) |
| MP (external database) | **155,201** | **20,900,193** | 134.7 (min 8 / max 150) | **240.0 MiB** | **9.0 min** (17,318 phases/min, 14 workers) |

> The old implementation ran at about 1,590 phases/min in a single process (COD slice); after the single-parse + adaptive-chunking optimization in 2026-10, whole-database throughput rose by about +10–15%;
> see [Build and acceptance manual](docs/构建与验收手册.md) §3.7.

---

## 4. Validation

```bash
python -m tools.validate_xindb data/COD-Inorg-20261020.xinDB --json tmp/validate.json
python -m tools.validate_xindb data/PCOD-Inorg-20261020.xinDB --json tmp/validate-pcod.json
python -m tools.validate_xindb data/COD-Inorg-20261020.xinDB --all-chunks   # full-chunk decode before release
```

Read-only, using only the public schema. Check groups: `S` tables/indices complete, `M` meta complete and consistent with the filename,
`C` chunk accounting (`Σpeak_n` = `Σn_peaks` = `entry_count`, block count = `ceil(N/4096)`, `peak_off` contiguous),
`E` unit-cell physical quantities / elements non-empty / `n_peaks ≤ 150` / **`E5` single-peak bad phases (180° boundary, placeholder cell)** / **`E6` empty phases**,
`P` sampled or full-chunk decode + norm-1000 + zero padding + `top⊆all`, `X` element inverted-index value range / ascending order / self-consistency.

Before release also run **`tools/accept_xindb.py`** (source/payload/bad-phase acceptance common to all 6 databases) plus two dedicated re-checks:

```bash
python -m tools.accept_xindb data/COD-Inorg-20261020.xinDB \
    --manifest D:/Project/XRD/sync-COD/inorg_manifest.sqlite --json tmp/accept.json
python -m tools._compare_xindb_peaks.py <old-db> <new-db>          # only metadata changed, physics untouched
python -m tools._verify_resume_equivalence                     # --resume is equivalent to one complete run
```

> ✅ **The three Inorg databases were rebuilt with the fixed peak engine on 2026-11-01/02**:
> All 8 bad phases in COD were cleared (7 restored with real peaks, 1 phase without a unit cell removed; phases 78,036→78,035, 9,395,681 reflections);
> all 7 bad phases in PCOD were cleared (159,275,318 reflections, only 17/1,061,953 payload changes);
> TCOD has no bad phases and its peak payload is byte-identical. All three databases pass `validate_xindb --all-chunks` and `accept_xindb` with **0 FAIL**.
> **TCOD-Full and COD-Full have both been built and accepted**: TCOD-Full 361,701 phases / 53,099,238 reflections;
> COD-Full 532,038 phases / 77,458,154 reflections (36.4 h); their common phases with the corresponding Inorg databases are **byte-identical**.
> The earlier `20261020` Release (COD 8 / PCOD 7 bad phases) **was deleted on 2026-11-04**, leaving only the rebuilt `20261020-rebuild`.
> If you still have old Inorg zips, do not use them; switch to the rebuilt version.
> See [`docs/ACCEPTANCE-20261101-rebuild.md`](docs/ACCEPTANCE-20261101-rebuild.md).

---

## 5. Directory Structure

```
.
├── _atomic_scattering_params.json Cromer-Mann coefficients (fastxrd looks for it under tools/../)
├── data/                          **local artifacts directory** (not committed): 5 databases after the rebuild (3 Inorg + 2 Full) + build logs
├── pytest.ini  tests/             toolchain regression tests (synthetic CIFs/archives, no dependency on the 1.8 GiB real databases)
├── tmp/                           local trial builds/bench/acceptance JSON (not committed, .gitignore)
├── xinmatch/                      **database build subset** (full program at PolyXRD/XinMatch)
│   ├── __init__.py                empty shell, to keep scipy/GUI out of the build chain
│   └── core/db/
│       ├── cif_meta.py            CIF metadata extraction (chemical formula / space group / unit cell / density / elements)
│       ├── cif_scan.py            CIF archive scan + six-database source registry (SOURCE/SUBSET/id offsets)
│       ├── peak_compute.py        reflection computation + peak convention constants (incl. Debye-Waller / boundary and placeholder-cell fixes)
│       ├── build_pipeline.py      build hot path: compute_phase() single parse (meta+peak table)
│       ├── schema_v3.py           v3 DDL / required tables / required meta keys
│       ├── writer_v3.py           chunked columnar writer + element_inv rebuild
│       └── data/atomic_scattering_params.json   Cromer-Mann coefficients (99 elements)
├── tools/                         four-step chain, index at tools/README.md
│   ├── scan_inorg.py              ① inorganic classification + copies + manifest (XRD_DATA_ROOT decides the root)
│   ├── cif_common.py              ① criterion implementation (parse_elements / is_inorganic / formula_mass)
│   ├── fastcif.py  fastxrd.py     ① CIF parsing / reflection computation dependencies
│   ├── build_cod_xindb.py         ② **only build entry point**: --source × --set (6 databases) + resume
│   ├── build_cod_inorg_xindb.py   ② thin compatibility wrapper (defaults to inorg)
│   ├── _bench_build.py            ② build throughput benchmark (old two-step vs single parse, workers/chunksize)
│   ├── validate_xindb.py          ③ read-only structural validator (S/M/C/E/P/X, incl. E5/E6)
│   ├── accept_xindb.py            ④ acceptance common to all 6 databases (id ranges / sources / elements / payload / bad phases)
│   ├── _compare_xindb_peaks.py    ④ peak payload block-by-block comparison (proves only metadata changed, physics untouched)
│   └── _verify_resume_equivalence.py ④ proves --resume is equivalent to one complete run
└── docs/
    ├── 构建与验收手册.md           **build/acceptance/release checklist (authoritative entry)**
    └── xinDB格式规范-v3.md        field-level specification (container / addressing / segments / meta / validation)
```

`xinmatch/` is a **trimmed subset**: only the modules needed for the database build are kept, and `__init__.py` is replaced with an empty shell
(the full version would `import scipy` etc., which the database build does not need). The remaining search/refinement/GUI modules are in the full repository.

Not committed (`.gitignore`): `data/ *.xinDB *.db *.sqlite tmp/ _release/ __pycache__/`.
This machine's `data/` currently holds the 5 rebuilt databases (COD-Inorg 113.1 MB / TCOD-Inorg 68.0 MB / PCOD-Inorg 1694.1 MB /
TCOD-Full 594.3 MB / COD-Full 879.4 MB) and `data/_build-*.log`; the old databases in `data/_prev/` were deleted on 2026-11-04.

---

## 6. Known Gaps

| Item | Current status | Impact |
|---|---|---|
| **Bad phases in the pre-rebuild old databases** | Old `20261020` Release: COD 8, PCOD 7, TCOD 0 | Built with the old peak engine; **after the 2026-11-01/02 rebuild all three Inorg databases have 0**. The old Release and its Inorg zips were deleted on 2026-11-04; only the rebuilt version remains |
| Half-collapsed boundary phases (WARN) | COD 2,251 / PCOD 5,598 (after rebuild) / TCOD 1,670: the strongest line falls at d≈λ/2 (2θ→180°), the remaining peaks exist (intensity sum ≥5/1000) | A **peak-convention edge phenomenon** caused by the LP divergence at 180° acting on real reflections, not fake peaks; **a rebuild will not remove it either**; a real fix requires changing the peak convention (raising `DEFAULT_DMIN` to 0.78–0.79 Å or dropping 2θ>179°, a v4 decision). Currently only a WARN, does not block release |
| Degenerate unit cell | No longer present in the rebuilt COD database (source CIF of `96-210-4629` had no unit cell → removed at build time) | — |
| No space group | 126 in the COD database (127 before the rebuild, `96-210-4629` removed; `E3` only warns) | Does not affect peak positions; such phases are excluded when filtering by space group |
| Full-database release | **TCOD-Full and COD-Full have both been built and accepted** (TCOD 361,701 phases / 53,099,238 reflections / 594.3 MiB; COD 532,038 phases / 77,458,154 reflections / 879.4 MiB; both validate/accept 0 FAIL, byte-identical common phases with the corresponding Inorg databases); `PCOD-Full` is equivalent to `PCOD-Inorg` (PCOD is all inorganic), so it is not built | Once built, XinMatch auto-discovers them via the built-in `xindb_tcod_full` / `xindb_cod_full` rows; 48,732 phases in TCOD-Full and 81,284 phases in COD-Full exceed the 63-bit `elem_mask` (`element_inv` is still complete) |
| Full-database release | **Build now supports** 6 databases (`--source × --set`); the `20261020` release only published the three Inorg databases | Organic/full-database search requires building the `*-Full` databases first (hours of build time, see build manual §3.6) |
| Narrow PCOD element coverage | Only 19 elements (O/Si/P/Al/V/S/B/F…) | An objective property of the GRINSP prediction set, not data lost during the build |
| `top_i` as float32 | Intensities use `<f4` instead of quantized integers | Size traded for precision; under norm-1000 the effective bits are sufficient |

---

## 7. Data Sources and License

- **Code**: follows XinMatch's dual licensing (academic non-commercial / commercial by separate agreement), see `LICENSE`, `LICENSE-CN`.
- **Data**: CIF content copyright belongs to COD and the authors of the original literature. When using artifacts from this repository, comply with
  [COD Terms of Use](http://www.crystallography.net/cod/) and cite COD and the corresponding original literature;
  PCOD (GRINSP predictions) and TCOD (theoretical) entries have their own original sources. This repository distributes only **derived indices**
  (reflection positions/intensities/metadata), and neither rewrites nor replaces the original CIFs.
- Intensities are **computed values**, including Debye-Waller but without instrumental/sample broadening, so they still deviate from experimental cards (ICDD PDF);
  sufficient for phase identification, but must be calibrated separately for quantification.

## 8. Version History

| Date | Event |
|---|---|
| 2026-10-21 / 22 | `COD-Inorg`, `TCOD-Inorg`, `PCOD-Inorg` three databases built (xinDB v3) |
| 2026-10-23 | This repository created and `20261020` release published (code + three zips); COD database `journal_mode` normalized from WAL to DELETE, `integrity_check` re-verified and passed |
| 2026-10-23 | The three databases moved from `PolyXRD-DataBase/xindb/` to this repository's `data/`; added inorganic classification (①) and two dedicated re-check (④) tools, making the chain self-contained |
| 2026-10-23 | **Source data moved out** → `D:/Project/XRD/sync-COD` (shared by both databases), `XRD_DATA_ROOT` default changed to that path; `build_dbs.py` stripped into the pure-function module `cif_common.py` (the old 3COD sqlite packager deleted) |
| 2026-10-23 | Data-side tools spun off into the public repository [**PolyXRD/sync-COD**](https://github.com/PolyXRD/sync-COD) (data still not in the repository) |
| 2026-10-24 | **Database build/acceptance tools rebuilt**: unified six-database build entry point (`--source × --set`, incl. PCOD/TCOD Full); added `build_pipeline.compute_phase()` single parse; added `accept_xindb.py` generic acceptance and `E5/E6` bad-phase checks; set up `tests/` regressions (synthetic CIFs, no dependency on real databases); added [Build and acceptance manual](docs/构建与验收手册.md) and recorded the convention "the database is built in this repository, CIF sources live in `sync-COD`" |
| 2026-11-01 | **The three Inorg databases rebuilt with the fixed peak engine** (source snapshot still `20261020`): COD 78,036→**78,035** phases / 9,395,681 reflections (8 bad phases cleared); TCOD 46,846 phases, peak payload byte-identical to the old database; PCOD 1,061,953 phases / 159,275,318 reflections (7 bad phases cleared); all three validate/accept **0 FAIL**. See [`docs/ACCEPTANCE-20261101-rebuild.md`](docs/ACCEPTANCE-20261101-rebuild.md) |
| 2026-11-02 | Started building **TCOD-Full** (362,294 source CIFs) and **COD-Full** (535,166 source CIFs); `PCOD-Full` skipped because PCOD is entirely inorganic and equivalent to `PCOD-Inorg` |
| 2026-11-02 | **TCOD-Full built and accepted**: 361,701 phases / 53,099,238 reflections / 623,173,632 B / 12.1 h; validate 0F/2W, accept 0F/1W; 46,846/46,846 common phases with TCOD-Inorg byte-identical; self-recall 10/10. COD-Full build in progress |
| 2026-11-04 | **COD-Full built and accepted**: 532,038 phases / 77,458,154 reflections / 922,120,192 B / **36.4 h**; validate 0F/2W, accept 0F/2W (`bad_phase_ids=0`); 78,035/78,035 common phases with COD-Inorg byte-identical; self-recall 10/10. All 5 databases (3 Inorg + 2 Full) complete, PCOD-Full skipped as equivalent |
| 2026-11-04 | **MP-20261031 external database built and accepted** (Materials Project, 155,204 CIFs): 155,201 phases / 20,900,193 reflections / 251,633,664 B / 9.0 min; `mp-<n>` ids, validate 0F/1W, accept 0F/1W (id set exactly matches the archive); self-recall 20/20. See [MP database build and acceptance](docs/MP-库构建与验收.md) |
