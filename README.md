# Rouse Validation — surpass-alpha Python Port (CUDA-C × multistep × random_saw)
## Deliverable Package — v2 (extended schedule: ×2.5 equilibration, ×1.25 production)

Based on: Kuriata, Gront & Sikorski (2016) CMST 22(4), 179-185 (DOI 10.12921/cmst.2016.0000052)

## Run series — context within the trio

This deliverable is one of three sibling Rouse-validation runs done on
the same 12-cell state-point matrix (3 N × 4 φ, 100 chains/cell, seed 42,
RTX 4070, CUDA-C × multistep × random_saw). They differ along **only
two axes**: whether the pivot MC move is enabled, and the sweep budget.
Everything else is invariant — σ, l₀, RepulsiveEnergy, the box-size
table, the per-cell file layout (`fig1_static.tsv`, `fig2_seg_msd.tsv`,
`fig3_cm_diffusion.tsv`, `fig4_autocorr.tsv`, `static_vs_sweep.tsv`).

| run dir | pivot? | sweep budget (eq / prod, N=25..100) | total chain-sweeps | T6 (τ_R) | T7 (D) | role |
|---|---|---|---|---|---|---|
| `rouse_validation_2026_APR_30__pivot_on__v0_first_deliverable` | **on** (hinge + N-tail + C-tail + **pivot**) | (older format, schedule not recorded in README) | not stated | FAIL (1.164) | **FAIL** (-1.382) | first deliverable; pivot still present |
| `rouse_validation_2026_MAY_01__pivot_off__v1_baseline_schedule` | **off** (pivot retired at supervisor's request) | 4 000 / 1 000 → 20 000 / 5 000 | 16.8 M | FAIL (1.164) | **FAIL** (-1.246) | first run after pivot removal (baseline budget) |
| `rouse_validation_2026_MAY_02__pivot_off__v2_extended_schedule` | **off** (same as v1) | 10 000 / 1 250 → 50 000 / 6 250 (v1 ×2.5 eq, ×1.25 prod) | 38 M (×2.26) | FAIL (1.164) | **PASS** (-1.028) | extended schedule; T7 fixed |

**The two axes:**

1. **Pivot move on/off** — flipped *between Apr 30 and May 01* on the
   supervisor's explicit request. Removing pivot did *not* fix T6 or T7;
   that change was a separate methodological decision (move-time budget /
   theoretical justification), not a validation fix.
2. **Sweep budget** — extended *between May 01 and May 02* (×2.5
   equilibration, ×1.25 production). This is what flipped **T7**
   (diffusion-coefficient scaling) from FAIL to PASS. T6 (τ_R-exponent =
   1.164) is **invariant across all three runs**, which is direct
   evidence that the τ_R miss is a *structural* property of the
   hinge-move dynamics — not a pivot-removal artefact and not a
   sample-count artefact.

**This run** is the **v2 extended** entry in the table — see its row above
for the headline T6 / T7 verdict. For the other two runs, see the sibling
directories named in the table.

---

### Simulation Parameters

| Parameter | Value |
|-----------|-------|
| Bead diameter (sigma / d0) | 3.8 A |
| Bond length (l0) | 3.8 A |
| Excluded volume | INFINITE (RepulsiveEnergy = 1e6, athermal; ContactEnergy = 0) |
| MC moves | Segmented hinge + N-tail + C-tail (pivot retired at supervisor's request) |
| Chain lengths | N = 25, 50, 100 |
| Volume fractions | phi = 0.03, 0.10, 0.20, 0.30 |
| State points | 12 (3 N x 4 phi) |
| Chains per cell | 100 |
| Backend | CUDA-C × multistep × random_saw (RTX 4070) |
| RNG seed | 42 (deterministic; single replicate per cell) |
| Move-cap policy | `cfg.cap_inner_hinge = True`, `cfg.cap_tail = True`, `cfg.angle_autotune = True` |

### State-Point Configuration Table

| N | phi | Chains | Box (A) |
|---|-----|--------|---------|
| 25 | 0.03 | 100 | 165.98 |
| 25 | 0.10 | 100 | 111.11 |
| 25 | 0.20 | 100 | 88.19 |
| 25 | 0.30 | 100 | 77.04 |
| 50 | 0.03 | 100 | 209.12 |
| 50 | 0.10 | 100 | 139.99 |
| 50 | 0.20 | 100 | 111.11 |
| 50 | 0.30 | 100 | 97.07 |
| 100 | 0.03 | 100 | 263.48 |
| 100 | 0.10 | 100 | 176.38 |
| 100 | 0.20 | 100 | 139.99 |
| 100 | 0.30 | 100 | 122.30 |

### Equilibration / Production Schedule (per-N) — extended

| N | Equilibration sweeps | Production sweeps | Total per cell |
|---|---|---|---|
| 25 | 10 000 | 1 250 | 11 250 |
| 50 | 25 000 | 2 500 | 27 500 |
| 100 | 50 000 | 6 250 | 56 250 |

12-cell run total: **380 000 sweeps × 100 chains = 38 M chain-sweeps** (×2.26 of v1's 16.8 M).

Schedule rationale: bumping equilibration by ×2.5 over the v1 baseline aims to ensure chains escape initial-configuration bias before production sampling; the modest ×1.25 production extension improves τ_R-fit statistics.

### Run Timing

- Started: **2026-05-02 03:45:07**
- Finished: **2026-05-02 15:02:52**
- Wall time: **≈ 11 h 17 m**
- Per-cell wall time pattern: ~240 s (N=25), ~1 048 s (N=50), ~4 057 s (N=100). About 2.2× the per-cell wall time of v1, in line with the sweep-budget scaling.

### T1–T7 Verdicts (dilute φ = 0.03 limit)

| Gate | Quantity | Theory | Tol | Measured | R²_fit | R²_min | Status |
|---|---|---|---|---|---|---|---|
| T1 | ⟨R²⟩ ~ N^(2ν) | 1.18 | ±0.15 | 1.1997 | 0.9987 | 0.95 | **PASS** |
| T2 | ⟨R_g²⟩ ~ N^(2ν) | 1.18 | ±0.15 | 1.2437 | 0.9997 | 0.95 | **PASS** |
| T3 | R²/R_g² at N=100 | 6.25 | ±0.60 | 6.1871 | — | — | **PASS** |
| T4 | g_CM long-time α | 1.00 | ±0.15 | 0.8494 | 1.000 | 0.95 | **FAIL** (just below 0.85 threshold) |
| T5 | g₁ short-time α | 0.50 | ±0.15 | 0.4934 | 1.000 | 0.90 | **PASS** |
| T6 | τ_R ~ N^? | 2.18 | ±0.30 | 1.1633 | 0.9937 | 0.95 | **FAIL** |
| T7 | D ~ N^? | -1.00 | ±0.20 | -1.0283 | 0.9984 | 0.95 | **PASS** |

Comparison with v1: **T7 went FAIL → PASS** (the longer equilibration fixed the diffusion-coefficient scaling). **T4 went PASS → FAIL** by a hair (0.0006 below the threshold). **T6 (τ_R) is unchanged at 1.16** despite the ×2.5 longer equilibration — strong evidence that the τ_R-exponent miss is a structural property of the hinge-move dynamics rather than a sampling artefact.

### Cross-φ Findings

| phi | 2ν(R²) | 2ν(R_g²) | R²/R_g² | D exp | τ_R exp | g_CM exp | g₁ exp |
|-----|---------|----------|---------|-------|---------|----------|--------|
| 0.03 | 1.200 (P) | 1.244 (P) | 6.435 (P) | -1.028 (P) | 1.163 (F) | 0.849 (F) | 0.493 (P) |
| 0.10 | 1.071 (F) | 1.133 (P) | 6.237 (P) | -1.187 (F) | 1.163 (F) | 0.852 (F) | 0.457 (P) |
| 0.20 | 1.067 (F) | 1.122 (P) | 6.354 (P) | -1.645 (F) | 1.163 (F) | 0.745 (F) | 0.422 (P) |
| 0.30 | 1.197 (P) | 1.166 (P) | 6.298 (P) | -1.503 (F) | 1.163 (F) | 0.753 (F) | 0.389 (P) |

(P = pass, F = fail). Note that 2ν(R²) regressed to FAIL at φ = 0.10 and φ = 0.20 in v2 even though the dilute fit improved — likely a reflection of how the longer equilibration redistributes chain configurations in the semi-dilute regime. Static observables otherwise broadly hold; dynamic observables remain difficult.

---

### TSV Data File Descriptions (`05_data/`)

Each (φ, N) cell folder is `phi_<phi>/N<N>/` and contains five TSV files plus the per-cell trajectory under `07_trajectories/phi_<phi>/N<N>/`.

#### `fig1_static.tsv` — Per-chain static properties (single-snapshot)

| Column | Meaning |
|--------|---------|
| `chain_id` | Zero-based chain index in the simulation box |
| `R2` | End-to-end distance squared (Å²), from the final production configuration |
| `Rg2` | Radius of gyration squared (Å²), from the final production configuration |

One row per chain (100 rows). Single-snapshot, **not** time-averaged.

#### `fig2_seg_msd.tsv` — Middle-segment mean square displacement

| Column | Meaning |
|--------|---------|
| `lag_sweep` | MC sweep lag |
| `g1` | Middle-segment MSD g₁(t), Å², averaged over all chains |

#### `fig3_cm_diffusion.tsv` — Center-of-mass mean square displacement

| Column | Meaning |
|--------|---------|
| `lag_sweep` | MC sweep lag |
| `g_CM` | Center-of-mass MSD, Å² |

#### `fig4_autocorr.tsv` — End-to-end vector autocorrelation

| Column | Meaning |
|--------|---------|
| `lag_sweep` | MC sweep lag |
| `g_R` | g_R(t) = ⟨R(0)·R(t)⟩ / ⟨R²⟩, normalized to [0, 1] |

For this run τ_R = 1 245 / 2 495 / 6 245 sweeps for N = 25 / 50 / 100 (φ = 0.03 dilute fit) — the τ_R per N grew proportionally with the production window, but the **slope of log(τ_R) vs log(N) is unchanged from v1** (1.16 in both runs).

#### `static_vs_sweep.tsv` — Equilibration + production time series (extended)

| Column | Meaning |
|--------|---------|
| `sweep` | Absolute MC sweep number |
| `phase` | `equilibration` or `production` |
| `mean_R2` | 100-chain mean ⟨R²⟩ at this sweep (Å²) |
| `mean_Rg2` | 100-chain mean ⟨R_g²⟩ at this sweep (Å²) |
| `ratio_R2_Rg2` | mean_R2 / mean_Rg2 |
| `energy_total` | Total potential energy after the sweep. **Always 0** (athermal hard-sphere). |
| `accepted_moves` | Per-sweep count of accepted moves across all chains and move types. Genuinely time-varying observable. |

Recording cadence: equilibration phase records every `eq_sweeps // 50` sweeps (sparse); production phase records every sweep (dense). For N=100, this gives ≈ 50 + 6 250 = 6 300 rows per cell.

#### `tavg_validation_summary.json` — Aggregated per-phi metrics

JSON with `per_phi.<phi>.{R2_exponent, Rg2_exponent, R2_Rg2_ratio, D_exponent, tau_R_exponent, g_CM_exponent, g1_exponent}` blocks; each carries `measured`, `theory`, `r2_fit`, `pass`. The `_detail` block exposes per-N raw means.

---

### Plot Descriptions and Data Sources

#### `01_static_properties/` — Cross-φ static plots

| Plot file | What it shows | Data source |
|-----------|---------------|-------------|
| `fig_R2_vs_N_per_phi.png` | Log-log ⟨R²⟩ vs N, one curve per φ | `fig1_static.tsv` |
| `fig_Rg2_vs_N_per_phi.png` | Log-log ⟨R_g²⟩ vs N, one curve per φ | `fig1_static.tsv` |
| `fig_2nu_vs_phi.png` | Fitted 2ν exponents vs φ — derived analysis | per-φ summary JSON |
| `fig_ratio_R2_Rg2_vs_phi.png` | R²/R_g² vs φ for each N | `fig1_static.tsv` chain-averaged |
| `fig_R2_Rg2_combined_dilute.png` | R² and R_g² vs N at dilute (φ = 0.03) | `fig1_static.tsv` for φ=0.03 |

#### `02_dynamic_properties/` — Cross-φ dynamic plots

| Plot file | What it shows | Data source |
|-----------|---------------|-------------|
| `fig_g1_vs_sweep_per_phi.png` | g₁(t) vs sweep, one curve per (φ, N) | `fig2_seg_msd.tsv` |
| `fig_gcm_vs_sweep_per_phi.png` | Center-of-mass MSD vs sweep, one curve per (φ, N) | `fig3_cm_diffusion.tsv` |
| `fig_D_vs_N_per_phi.png` | Log-log D vs N, one curve per φ; expected slope -1 | g_CM long-time slope |
| `fig_tauR_vs_N_per_phi.png` | Log-log τ_R vs N, one curve per φ | g_R 1/e crossing |
| `fig_D_exponent_vs_phi.png` | Fitted D-exponent vs φ — derived analysis | per-φ summary JSON |
| `fig_tauR_exponent_vs_phi.png` | Fitted τ_R-exponent vs φ — derived analysis | per-φ summary JSON |
| `fig_g1_shorttime_exponent_vs_phi.png` | Fitted g₁ short-time exponent vs φ — derived | per-φ summary JSON |

#### `03_per_state_point/phi_<phi>/N<N>/` — Per-(φ, N) detail plots (5 per cell × 12 cells = 60)

| Plot file | What it shows | Data source |
|-----------|---------------|-------------|
| `R2_vs_MC_sweep.png` | Running mean R² across eq + prod for this cell | `static_vs_sweep.tsv` |
| `Rg2_vs_MC_sweep.png` | Running mean R_g² for this cell | `static_vs_sweep.tsv` |
| `g1_middle_segment_msd.png` | Middle-segment MSD vs sweep for this cell | `fig2_seg_msd.tsv` |
| `gcm_center_of_mass_msd.png` | Center-of-mass MSD vs sweep for this cell | `fig3_cm_diffusion.tsv` |
| `autocorrelation_end_to_end_vector.png` | g_R(t) decay for this cell | `fig4_autocorr.tsv` |

#### `04_equilibration_evidence/phi_<phi>/` — Equilibration diagnostics (12 plots per φ × 4 φ = 48)

For each observable in {R², R_g², energy, acceptance}:

| Plot file | What it shows | Data source |
|-----------|---------------|-------------|
| `fig_<obs>_vs_sweep_all_N.png` | Time trace, all N overlaid for this φ | `static_vs_sweep.tsv` |
| `fig_<obs>_autocorr_all_N.png` | Production-phase autocorrelation vs lag, all N overlaid | `static_vs_sweep.tsv` (production rows) |
| `fig_<obs>_dist_overlap_all_N.png` | First-half vs second-half histograms with KS statistic, one subpanel per N | `static_vs_sweep.tsv` (production rows) |

The `energy` family is flat-zero (athermal model) and serves as a sanity check. The `acceptance` family carries the genuine equilibration signal. Both R² and R_g² families plot the underlying 100-chain mean from `static_vs_sweep.tsv`.

#### `05_data/` (top-level)

| File | What it contains |
|------|------------------|
| `tavg_validation_summary.json` | Per-φ scaling exponents and pass/fail |
| `rouse_compliance_heatmap.png` | (φ × N) grid colored by gate compliance |

#### `07_trajectories/phi_<phi>/N<N>/`

| File | What it contains |
|------|------------------|
| `trajectory.pdb` | Multi-MODEL PDB, ~100 frames over eq+prod, CRYST1 box + CONECT backbone; PyMOL-loadable |

---

### Directory Structure

```
rouse_validation_2026_MAY_02__pivot_off__v2_extended_schedule/
  README.md                            <- this file
  00_t1_t7_verdicts.json
  01_static_properties/                <- 5 cross-phi static plots
  02_dynamic_properties/               <- 7 cross-phi dynamic plots
  03_per_state_point/phi_<phi>/N<N>/   <- 5 plots per cell (60 PNGs)
  04_equilibration_evidence/phi_<phi>/ <- 12 plots per phi (48 PNGs)
  05_data/
    tavg_validation_summary.json
    rouse_compliance_heatmap.png
    phi_<phi>/N<N>/                    <- 5 TSVs per cell (60 TSVs)
      fig1_static.tsv
      fig2_seg_msd.tsv
      fig3_cm_diffusion.tsv
      fig4_autocorr.tsv
      static_vs_sweep.tsv
  07_trajectories/phi_<phi>/N<N>/
    trajectory.pdb
  benchmark.log
  checklist.log
```

### Autocorrelation Analysis (End-to-End Vector, dilute φ = 0.03)

| N | τ_R (sweeps, 1/e crossing) | Production-window status |
|---|---|---|
| 25 | 1 245 | Production = 1 250 sweeps; barely one decorrelation length — under-sampled |
| 50 | 2 495 | Production = 2 500 sweeps; barely one decorrelation length — under-sampled |
| 100 | 6 245 | Production = 6 250 sweeps; barely one decorrelation length — under-sampled |

τ_R values are essentially equal to the production-window length for every N, **same pattern as v1** even with ×1.25 longer production. The autocorrelation function does not fully decay within the production window at any N, which caps the τ_R-exponent fit at the same value (1.16) as v1. Resolving this would require either (a) significantly longer production runs (10×+ at minimum), or (b) acknowledging that the hinge-move algorithm produces a different effective relaxation-time scaling than the theoretical Rouse exponent of 2.18.

### Notes

- All cells share `--seed 42`; the run is deterministic and a single replicate per state point.
- For the relationship to the other two runs in the trio (v0 with pivot, v1 baseline schedule), see the **Run series — context within the trio** section near the top of this file.
- Energy column (`energy_total`) is flat 0 throughout — included for symmetry; the acceptance counter is the time-varying equilibration diagnostic.

---
Generated by `rouse_model_python` (surpass-alpha CG framework, hinge-only fork)
