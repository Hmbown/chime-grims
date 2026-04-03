# Data Acquisition and Status

How to find, fetch, and verify external data for both instruments.

## Quick Status Check

```bash
# GRIM-S: what strain data do we have? (from repository root)
python scripts/grims/check_data.py --ringdown-only

# CHIME: list known targets and ephemeris
python -c "from bown_instruments.chime.ephemeris import list_targets; print(list_targets())"
```

## GRIM-S (Gravitational Wave Ringdowns)

### Data Source
GWOSC (Gravitational Wave Open Science Center): https://gwosc.org

Strain data in HDF5 format, 4096 Hz sampling.
- O1-O3 events: 32-second files (~250 KB each)
- O4 events (GWTC-4.0): 4096-second files (~64 MB each)

### Files
| File | Purpose |
|------|---------|
| `results/grims/gwtc_full_catalog.json` | Full BBH catalog (150 events, O1-O4) |
| `results/grims/download_queue.json` | Pre-resolved strain file URLs (448 entries) |
| `scripts/data/*.hdf5` | Local strain cache (gitignored); this is the directory `scripts/grims/check_data.py` and `download_multidet.py` scan |

Some older runners expect a copy of `gwtc_full_catalog.json` under `scripts/data/`; the committed source of truth is `results/grims/gwtc_full_catalog.json`. Only GW150914 (~1 MB) is committed when present, per repository policy.

### Commands

```bash
# From repository root
# Check what data is locally available
python scripts/grims/check_data.py --ringdown-only

# Download missing L1/V1 strain for events that have H1
python scripts/grims/download_multidet.py

# Refresh catalog from GWOSC (if new events released)
python scripts/grims/refresh_catalog.py --write

# Rebuild download queue with resolved URLs
python scripts/grims/refresh_download_queue.py --write
```

### Storage Budget
- O1-O3 (66 events): ~32 MB total
- O4 (68 events): ~8.5 GB total
- Only GW150914 H1 (1 MB) is committed to git

### O4 Compatibility Notes
- GWOSC API works identically for O4 events
- O4 files use naming: `H-H1_GWOSC_O4a_4KHZ_R1-{GPS}-4096.hdf5`
- O4a has H1 + L1 only (no V1 data)
- The `download_multidet.py` and `load_gwosc_strain_hdf5()` handle O4 files without modification

## CHIME (JWST Channel Diagnostics)

### Data Source
MAST (Mikulski Archive for Space Telescopes): https://mast.stsci.edu

JWST NIRSpec x1dints FITS files (per-integration extracted spectra).

### Files
| File | Purpose |
|------|---------|
| `results/chime/data_manifest.json` | Pinned observation IDs, filenames, MAST URIs |
| `results/chime/channel_survey/phase2_survey.json` | Per-segment quality measurements |
| `results/chime/channel_survey/segment_file_inventory.json` | MAST product inventory |
| `src/bown_instruments/chime/ephemeris.py` | Transit ephemerides (34 lookup keys in `EPHEMERIDES`) |

### Commands

```bash
# Search for JWST observations of a target
python -c "from bown_instruments.chime.mast import find_x1dints; print(find_x1dints('WASP-39'))"

# Run multi-target survey
python examples/chime/multi_target_survey.py

# Check ephemeris for a target
python -c "from bown_instruments.chime.ephemeris import get_ephemeris; print(get_ephemeris('WASP-39'))"
```

### Cache Locations
- Downloads go to `tempfile.mkdtemp(prefix='chime_')` by default (not persistent)
- To persist: pass `download_dir='chime/data/'` to `download_product()`
- FITS files are NOT committed to git (too large)
- Results JSONs in `results/chime/` ARE committed

### Completed Survey Data
Already analyzed (pinned in `data_manifest.json`):
- WASP-39: Program 1366, Obs 3 (G395H, 3 segments) + Obs 4 (PRISM, 4 segments)
- WASP-107: Program 1224, Obs 3 (G395H, 3 segments) + Program 1201, Obs 502 (G395M)

### Next Campaign Targets (SHA-4152)
- HD-189733, TRAPPIST-1, WASP-121, GJ-3470
- Search criteria: NIRSpec G395H/G395M/PRISM, public, calibration level >= 2

## What's Missing and How to Get It

### GRIM-S
Run `python scripts/grims/check_data.py --ringdown-only` for a current report.
As of 2026-04-03: 133 of 134 ringdown events need strain data downloaded.
Estimated total download: ~8.5 GB.

### CHIME
All results from the completed survey are committed.
For the next campaign, use `bown_instruments.chime.mast.find_x1dints(target)` to discover
available observations, then `bown_instruments.chime.mast.download_product()` to fetch.
The `data_manifest.json` pins exactly which files were used in prior analysis.
