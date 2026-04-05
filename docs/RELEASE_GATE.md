# Release Gate: Pre-Publication Evidence Checklist

This checklist defines the minimum criteria that must be met before `chime-grims` findings are shared with the external scientific community.

## 1. GRIM-S Claim Language (Evidence Bar)

*   **Detection**: Requires an empirical significance $\ge 3\sigma$ over a full, end-to-end null distribution (e.g., phase-scrambled or time-shifted injections), not just asymptotic approximations. Influence analysis must show the result survives the removal of any single event or detector.
*   **Hint / Evidence for**: Requires an empirical significance between $2\sigma$ and $3\sigma$, with Bayes factors $> 3$. The result must be robust against reasonable variations in preprocessing (e.g., whitening bounds, $t_{\rm start}$).
*   **Upper Limit**: If significance is $< 2\sigma$, we publish a bounded posterior on $\kappa$, clearly stating no detection was made, backed by full injection recovery tests demonstrating that the pipeline *would* have found a signal if it existed at realistic SNR.

## 2. Reproducibility Bar (CHIME & GRIM-S)

*   **Data Availability**: All strain files, lightcurves, and external catalogs required to reproduce the result must be pinned with exact versions/URLs (e.g., GWOSC O4 paths).
*   **Artifacts**: Result files (JSONs, ECSVs) must be version-controlled and reproducible from raw data via a single, documented top-level script.
*   **Scripts**: No manual or notebook-only steps in the critical path. Everything from raw data to final plots must be executable in a clean environment.
*   **Tests**: The test suite (`pytest`) must pass $100\%$.

## 3. Required Analyses Before Announcement

*   [x] **Phase 3 Null Distribution**: Verify the false-alarm rate empirically (completed).
*   [x] **Phase 3 Robustness**: Leave-k-out and influential event analysis (completed).
*   [ ] **Realistic-SNR Injection Recovery**: Verify signal recovery over all preprocessing choices.
*   [ ] **Sub-band Correlation (CHIME)**: Validate the diversity gain calibration against real data, not just synthetics.
*   [ ] **O4 Data Ingestion**: Ensure the kappa constraint includes the latest available robust events.
