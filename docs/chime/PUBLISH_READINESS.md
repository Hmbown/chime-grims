# Publish Readiness Assessment

Date: 2026-04-03

## Current status

CHIME ships inside the unified **`bown-instruments`** package in this monorepo ([`Hmbown/chime-grims`](https://github.com/Hmbown/chime-grims)). PyPI publication, if desired, is for **`bown-instruments`** with optional extras **`[chime]`** and **`[grims]`**, not a separate `chime-jwst` distribution.

## Ready now

- Single top-level [`pyproject.toml`](../../pyproject.toml) defines `bown-instruments` and the `bown` CLI.
- CHIME implementation lives under [`src/bown_instruments/chime/`](../../src/bown_instruments/chime/).
- Tests under [`tests/chime/`](../../tests/chime/); examples under [`examples/chime/`](../../examples/chime/).
- Committed CHIME result artifacts under [`results/chime/`](../../results/chime/).

## Recommended before a PyPI release

- Confirm the **`bown-instruments`** name on PyPI (test upload to TestPyPI first if unsure).
- Tag releases from this repo (e.g. `v0.2.0`) so `repository-code` in citation metadata matches a reproducible revision.

## Verified checks (monorepo)

- `python -m pytest tests/ -q`: run from repository root after `pip install -e ".[dev]"`.
- `bown --version` and `bown chime --help` / `bown chime --targets`: CLI smoke tests.

## Historical note

Older drafts referred to a standalone `chime/` tree and the name `chime-jwst`; those paths are superseded by the unified package layout above.
