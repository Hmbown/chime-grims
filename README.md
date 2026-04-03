# bown-instruments

**Measure your signal path before you trust your result.**

Unified Python package (v0.2.0) for two astrophysical measurement instruments and a small **core** library of reusable patterns. Both instruments inherit an engineering discipline from Ralph Bown (1891–1966), Bell Telephone Laboratories: treat degradation and silent failure as normal operating conditions, and build the diagnostic into the measurement. See [CLAUDE.md](CLAUDE.md) for the project’s stated engineering questions and patent context.

| | **CHIME** | **GRIM-S** | **core** |
|---|-----------|------------|----------|
| **Role** | Channel Health & Instrument Metrology for Exoplanets | Gravitational Intermodulation Spectrometer | Shared measurement primitives |
| **Domain** | JWST transit spectroscopy | LIGO/Virgo/KAGRA ringdowns | — |
| **What it measures** | Per-wavelength noise quality (grades A–D) | Nonlinear mode-coupling coefficient κ (kappa) | Injection recovery; quality-weighted combination; health checks |
| **Bown anchor** | Diversity-style channel grading (US 1,747,221) | Self-test before trust (US 1,573,801) | Both patterns exposed as APIs |

**Repository:** [github.com/Hmbown/chime-grims](https://github.com/Hmbown/chime-grims) · **Tracking:** [Linear — bown-instruments](https://linear.app/shannon-labs/project/bown-instruments-9a003df614d9)

---

## Get started

```bash
git clone https://github.com/Hmbown/chime-grims.git && cd chime-grims
pip install -e ".[dev]"          # all instruments + tests + ruff

bown chime WASP-39               # JWST channel diagnostics (MAST)
bown grims --events 32           # GRIM-S stack (subset of catalog)
bown selftest                    # injection-recovery checks
bown --version                   # 0.2.0
```

Optional installs: `pip install -e ".[chime]"` or `".[grims]"` only. On PyPI (when published): `bown-instruments[chime]`, `bown-instruments[grims]`, or `[all]`.

---

## What each instrument does

### CHIME — JWST channel and segment quality

CHIME asks whether reduced archive products are empirically consistent with photon noise and white-noise averaging, per wavelength and per segment. It turns scatter, systematic excess, and Allan-style binning tests into **A/B/C/D grades** and **diversity weights** so bad channels or segments do not dominate combined spectra.

**Example result (WASP-39b, NIRSpec G395H, ERS Program 1366):** segment quality varies about **4×** within one observation: one segment near photon-limited (17/18 A-grade bins), another systematic-dominated (0/18 A-grade, **4.89×** photon noise). Concatenating without this step smears the transmission spectrum.

<p align="center">
  <img src="results/chime/wasp39_fit/chime_wasp_39_1.png" width="600" alt="CHIME diagnostic: WASP-39b segment quality varies within one JWST observation">
</p>

**Survey status:** the full archive survey covers 10 distinct targets, 61 observations, and 199 NIRSpec segments; the committed subset in `results/chime/channel_survey/` contains 10 segments across 3 observations and 2 targets. Median segment quality in the full survey is **1.97×** photon noise. **Ephemerides** for CLI/API use: **34** packaged lookup keys (aliases and multi-planet entries); `bown chime --targets` lists them.

### GRIM-S — ringdown coupling (κ)

After a BBH merger, the remnant ringdown is described by quasinormal modes. Second-order physics introduces a daughter contribution scaled by **κ**. GRIM-S **phase-locks** a κ-parameterized template to the parent mode and **stacks** events (with **weight caps** so no single event dominates—same diversity idea as CHIME, different domain).

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="results/grims_plots/grims_pipeline_dark.svg">
    <img alt="GRIM-S pipeline" src="results/grims_plots/grims_pipeline_light.svg" width="700">
  </picture>
</p>

| Phase | Events | κ | Significance | Change |
|---:|---:|---|---:|---|
| 1 | 32 | −0.047 ± 0.043 | 1.1σ | H1-only baseline |
| 2 | 134 | +0.028 ± 0.019 | 1.5σ | Expanded catalog |
| 2.5 | 122 | +0.016 ± 0.030 | 0.5σ | Colored-noise likelihood |
| **3** | **128** | **+0.015 ± 0.007** | **2.2σ local** | Multi-detector + capped weights; jackknife stability caveat currently prevents treating this as robust |

---

## Design principles (encoded, not decorative)

1. **Self-test** — Inject and recover through the same pipeline you use for science (`bown selftest`; `bown_instruments.core` and instrument `self_test` modules).
2. **Diversity weighting** — Measure quality per channel or per event, then combine with weights that down-rank poor contributors (`core.diversity_weight`; CHIME bin grades; GRIM-S weight caps).
3. **Observable diagnostics** — Every claim should point to a plot, metric, or table you can re-run.

`bown_instruments.core` supplies **`SelfTest`**, **`diversity_weight`**, **`check_instrument_health`** for reuse outside these two instruments.

---

## Python imports

```python
from bown_instruments.chime import compute_channel_map, compute_diversity
from bown_instruments.grims.qnm_modes import KerrQNMCatalog
from bown_instruments.core import SelfTest, diversity_weight
```

CHIME-specific workflow pieces: `find_x1dints`, `download_product`, `extract_transit_data`, `get_ephemeris` (see `docs/chime/README.md`).

---

## Repository layout

| Path | Contents |
|------|----------|
| `src/bown_instruments/core/` | `self_test`, `diversity`, `diagnostics` |
| `src/bown_instruments/chime/` | Channel map, diversity combine, ephemeris, MAST I/O, extract, transit fit, plot, CLI |
| `src/bown_instruments/grims/` | QNM catalog, templates, GWTC/GWOSC I/O, whitening, phase-locked search, Bayesian stack, jackknife, mass orchestration, + supporting modules |
| `tests/chime/` · `tests/grims/` | 50 + 19 tests (69 total); GRIM-S uses committed ~1 MB GW150914 where needed |
| `examples/chime/` | Seven worked examples |
| `scripts/grims/` | Catalog refresh, downloads, batch analyses |
| `results/` | Committed outputs (`chime/`, `grims/`, `grims_plots/`) |
| `patents/` | Thirteen Ralph Bown patent PDFs (historical context) |

Full filename-level tree is available under `src/bown_instruments/` in the source; rules for imports: **public** code uses the `bown_instruments.` prefix; GRIM-S **internal** modules may use relative imports.

---

## Data and tests

- **CHIME:** Results and manifests in `results/chime/`; FITS from [MAST](https://mast.stsci.edu) on demand. Details: [DATA.md](DATA.md).
- **GRIM-S:** Reference strain small enough to commit; full GWTC downloads via scripts (~8.5 GB for O4-scale work). See [DATA.md](DATA.md) and `scripts/grims/`.

```bash
python -m pytest tests/ -q       # expect 69 passed
```

---

## Author and license

**Hunter Bown** — hunter@shannonlabs.dev · Great-grandson of Ralph Bown.

MIT
