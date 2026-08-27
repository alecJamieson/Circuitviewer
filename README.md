# Circuit Viewer

An interactive 3D/2D viewer for DCM/PEB effective-connectivity results, rendered on a real cortical surface mesh. Ships as a single self-contained HTML file — no install, no server, no build step. Open it in a browser and it runs.

The bundled demo dataset is a 5-node dynamic causal model (DCM) fit per subject and taken to group level with Parametric Empirical Bayes (PEB), from the CNBT depression & anxiety cohort (N = 143, PEB search, BMA model 1). The viewer also accepts any other connectivity matrix via JSON or CSV import (see [Loading your own data](#loading-your-own-data)).

## Quick start

Double-click `frontolimbic_circuit_viewer.html`, or open it from your browser's File → Open menu. Everything — the Three.js library, the mesh, the demo data, the app — is embedded in that one file.

## Layout

- **3D panel** — drag to rotate, scroll to zoom, `Reset view` to return to the default angle.
- **Top / Side / Posterior panels** — the same connectivity drawn as flat 2D projections, useful when the 3D view is hard to read at a given angle.
- **Connections table** — every edge that clears the current evidence threshold, sortable by column, with From/To, Ep, SD, and Posterior probability.
- **Reading this viewer** — an in-app methods panel explaining Ep/SD/Pp and the color convention, kept in sync with whichever dataset (demo or custom) is currently loaded.

## Controls

| Control | What it does |
|---|---|
| **Connection type** (Intrinsic · A / Modulatory · B) | Switches between task-independent (A) and task/condition-dependent (B) connectivity. |
| **Covariate** | Selects a column of the PEB design matrix: Commonalities (group mean), Dep / Anx / Stress (mean-centred symptom severity), Age / Gender (nuisance covariates). |
| **Posterior prob. method** | **BMA search** (default) — evidence for models with the connection switched on vs. off, from the reduced-model search; accounts for covariance among parameters. **Marginal P>0** — each parameter's own Gaussian posterior area above zero, computed in isolation; ignores covariance and can disagree with BMA search near the threshold. Matches the two methods in SPM's `spm_dcm_peb_review.m`. No effect on custom-imported data, which only supplies one evidence measure. |
| **Evidence threshold · Pp ≥** | Hides any connection whose posterior probability (by the method above) falls below this value. |
| **Render · Solid ↔ Glass** | Continuously blends the brain surface from opaque to translucent, so connections buried inside the tissue (e.g. the Amygdala) become visible. |
| **Self-connections** | Show/hide diagonal (self-inhibition) terms, drawn as a ring around each node rather than a line. |
| **Node labels** | Show/hide the floating name labels in the 3D view. |
| **Auto-rotate** | Continuous slow spin of the 3D view. |

### Color convention

Red = excitatory (positive Ep), blue = inhibitory (negative Ep). Line weight and opacity scale with effect size and posterior probability.

**Diagonal (self) terms are the one exception**, and only for the built-in DCM dataset: DCM fixes every region's self-connection as inhibitory and estimates only a log-scaling parameter on it (self-connection = −exp(Ep) Hz), so a *positive* Ep means *stronger* self-inhibition and a *negative* Ep means *weaker* self-inhibition (a more disinhibited, excitable region) — the opposite sense from an off-diagonal Ep. To keep red/blue meaning the same thing everywhere, self-connection rings are colored by the *opposite* of their raw sign. This flip is specific to DCM's parameterization, so it's **not** applied to custom-imported matrices, where a diagonal entry is whatever your own data means and is colored by its raw sign like any other cell.

### Bilateral regions

A region estimated as a single bilateral parameter (e.g. the Amygdala) is plotted at *both* of its physical MNI coordinates (left and right), each showing the identical connectivity value — there's only one underlying estimated parameter either way. The connections table lists it once, since that's the actual estimated quantity.

## Exporting

### Export figure (PNG)

Combines the current 3D view and the three 2D panels into one composed, publication-style image, captured at whatever camera angle, matrix, covariate, threshold, and tissue setting is currently on screen.

- **Background** — Dark (matches the app), White, or Transparent.
- **Resolution** — Standard / High / Print (roughly 1×, 2×, 3× the on-screen size).
- **Panel labels** — adds A/B/C/D letter chips to each panel, journal-figure style.
- **Legend** — the color/size legend, as an overlay on the 3D panel.
- **Parameters caption** — a text line recording matrix, covariate, Pp method, threshold, and N, so the figure is self-documenting for a methods section.

### Export rotation (GIF)

Spins the 3D view one full turn from its current angle (pitch, zoom, and all data/color settings stay as they are) and saves it as a looping GIF.

- **Background** — Dark / White / Transparent.
- **Size** — Small (320px) / Medium (480px) / Large (640px) wide.
- **Smoothness** — Draft (24 frames) / Standard (36) / Smooth (60).

The GIF encoder uses a fixed 216-color palette with light dithering rather than per-image adaptive quantization — fast and reliable, at the cost of some color fidelity versus the PNG export. It's meant as a quick shareable preview loop, not the publication asset.

**Note:** both export buttons trigger a browser download, which only works when the file is opened directly (as downloaded, or from wherever you've saved it). If you're viewing this tool through a hosted web link rather than the file itself, the export buttons won't produce a download there.

## Loading your own data

Use **Load a different connectivity matrix** at the bottom of the page. Two formats are accepted:

**JSON** — a file with `nodes` (name + MNI x/y/z) and a square `matrix` of connection strengths, optionally a matching `pvalues` matrix used as the evidence measure:

```json
{
  "nodes": [{"name": "Amygdala", "x": -26, "y": 2, "z": -20}, "..."],
  "matrix": [[0, 0.42, "..."], "..."],
  "pvalues": [[1, 0.98, "..."], "..."]
}
```

`matrix[row][col]` is the connection *from* the column node *to* the row node. If `pvalues` is omitted, every nonzero matrix entry is treated as fully confident. See `connectivity_matrix_template.json` for a complete, working, documented example.

**CSV** — a plain N×N matrix. Nodes are placed in a generic ring layout and labeled Node 1…N (no anatomical coordinates, so no cortical-surface placement).

**Back to CNBT dataset** restores the original demo data and its DCM-specific methods text and self-connection coloring.

## What Ep, SD, and Pp mean

- **Ep** — expected value of the parameter (BMA posterior mean), in Hz for the A matrix, unitless scaling for B.
- **SD** — posterior standard deviation of Ep (√diag(Cp), the BMA posterior covariance SPM returns alongside Ep) — each parameter's own marginal uncertainty, ignoring covariance between parameters.
- **Pp** — posterior probability the parameter is non-zero, by whichever of the two methods above is selected.

## Technical notes

- Single HTML file: Three.js r128, the demo dataset, and a FreeSurfer fsaverage5 pial cortical surface mesh (MNI-aligned) are all embedded inline — nothing is fetched at runtime except the IBM Plex Sans/Mono web fonts (falls back to system fonts if offline).
- Requires a browser with WebGL support (any current version of Chrome, Firefox, Safari, or Edge).
- No dependencies to install, no server to run.

## Known limitations

- The GIF export's fixed color palette can show mild banding on smooth gradients (most visible on the brain surface's shading), and its export button — like the PNG export's — only works when the file is opened directly rather than through a hosted link.
- CSV import has no way to specify anatomical coordinates, so custom CSV networks use a generic ring layout rather than a real cortical placement.
- The Posterior prob. method toggle only has an effect on the built-in CNBT dataset, which supplies both evidence measures; custom imports have only one.
