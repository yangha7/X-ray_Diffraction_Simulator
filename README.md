# Crystal Lab — X-ray diffraction & reciprocal-space simulator

**Version 1.0**

An interactive, zero-install browser tool for X-ray crystallography. It ties together the
real-space crystal, the reciprocal lattice + Ewald sphere, and a live simulated detector, all
driven by real structure-factor intensities — so you can *see* how a structure produces its
diffraction pattern, how the Ewald construction and Bragg's law work, and how rotating the
crystal sweeps reflections onto the detector.

Built as a single self-contained HTML file (Three.js from a CDN). No build step, no server, no
install — just open it in a browser.

## Files

- **crystal_lab.html** — the main tool (this is v1.0).
- **diffraction_simulator.html** — the original, simpler lattice/Ewald simulator (schematic
  intensities). Kept as a stripped-down teaching view of just the Ewald construction.
- **sample_1CRN.pdb** — crambin coordinates, handy for testing the upload path.

## What it does

Three synchronized panels:

- **Real space** — the unit cell and atoms; a fixed horizontal incident beam; the crystal
  rotates on the goniometer; the (hkl) planes in Bragg condition and the diffracted rays (at 2θ)
  are drawn as it turns.
- **Reciprocal / Ewald** — the reciprocal lattice, the Ewald sphere (r = 1/λ), and which points
  are on the sphere right now (brightness ∝ |F|).
- **Detector** — the simulated diffraction image, with resolution rings and a live Bragg readout
  (hkl, d, θ, 2θ) plus the λ = 2 d sinθ equation filled in with live numbers.

Physics engine:

- Real structure factors F(hkl) = Σ f·exp(2πi(hx+ky+lz))·exp(−B·s²) over the symmetry-expanded
  unit cell; systematic absences are physically correct.
- General (triclinic) unit cells via the reciprocal metric.
- Full space-group symmetry: operators expand the asymmetric unit to the whole cell.
- Reflections generated to d_min, capped cleanly by resolution (a complete sphere, cap 250k).

## Structures

Two-level picker — **category** then species:

- **Inorganic crystals**: NaCl, CsCl, diamond, silicon, Cu (fcc), Fe (bcc), Po (simple cubic),
  ZnS (zinc blende), CaF₂ (fluorite), and a hexagonal P1 demo — all driven through the symmetry
  engine (F/I/P centering).
- **Small molecules**: Ice Ih (hexagonal, proton-disordered water; default) and a urea demo.
- **Macromolecules**: crambin (1CRN, protein) and 6AQT (DNA duplex d(CGCGCG)₂, chains A/B).

Load your own:

- **Upload** a PDB, mmCIF, or CIF file (parsed entirely in the browser, no size limit).
  - PDB: CRYST1 (cell) + REMARK 290 (symmetry) + ATOM/HETATM (Cartesian → fractional via the
    reciprocal metric, validated against the file's own SCALE records). ANISOU lines skipped.
  - mmCIF: dotted tags (`_cell.length_a`, `_atom_site.Cartn_x`, `_space_group_symop.operation_xyz`).
  - CIF: fractional coordinates + `x,y,z` symmetry-operator strings; U→B conversion.
- **Fetch by PDB code** — type a 4-character code (e.g. `4HHB`) and it downloads that entry from
  RCSB and loads it. (Needs internet; uses RCSB's CORS-enabled file server.)

## Controls & display

- **Detector**: resolution rings; **Powder rings** toggle (orientation-averaged Debye–Scherrer
  rings, e.g. ice rings); **Log-scale intensity** toggle (5-decade log, reveals weak/high-res
  spots); **spot-contrast** gamma slider (1.0 = linear/true intensity, < 1 boosts weak spots);
  hkl labels; **Accumulate** to build a rotation dataset. Detector reaches ~1 Å at the default
  distance.
- **Rotation**: beam is horizontal (left→right); goniometer spindle = x axis (horizontal, ⟂ beam).
  Auto-rotation advances one oscillation-width Δφ per step (contiguous wedges, like real data
  collection). Drag to reorient the crystal freely (beam fixed); Shift/right-drag orbits the view.
- **Editing**: edit the asymmetric unit (element / x,y,z / B-factor / occupancy) and symmetry
  regenerates the cell and pattern live.
- Every left-panel parameter (λ, d_min, distance, φ, oscillation, contrast, B, occupancy) is a
  slider **plus a number box** — drag or type an exact value.
- Supercell view (1/2/3 cells), atoms-vs-dots display, bonds, per-structure atom-colour legend.

## Defaults

- Spot contrast = 1.0 (no boost, true intensity); Bragg planes off; log-scale off; powder off.
- Resolution: preloaded structures (inorganic, small molecules, crambin, 6AQT) and small custom
  uploads open at **1 Å**. Large custom/fetched structures (> 1000 atoms in the cell, i.e.
  proteins/DNA) open at **3 Å** to stay responsive — raise or lower d_min manually anytime.

## Verified (numerically)

- fcc/bcc/diamond/CsCl/NaCl systematic absences; NaCl (111) weak vs (200) strong.
- Triclinic/hexagonal reciprocal metric matches analytic d-spacings.
- Ice Ih: O–O H-bonds 2.74–2.75 Å; 6₃ screw absence exact (|F(001)|² = 0, |F(002)|² > 0);
  powder rings reproduce the known ice-Ih ring positions (3.9, 3.66, 3.44, 2.67, 2.25, 2.07 Å).
- PDB/mmCIF/CIF parsers vs real crambin + 6AQT + synthetic files; Cartesian→fractional
  reproduces each file's SCALE records exactly.
- Beam-axis rotation is degenerate (0 reflections sweep); x-axis rotation sweeps normally.

## Honest limitations (for research use)

- Scattering factors are an approximate Z·Gaussian + B-factor: absences and B-factor falloff are
  correct, but *fine relative intensities are not*. Cromer–Mann form factors are a drop-in upgrade.
- No Lorentz–polarization, partiality, mosaicity, divergence/bandwidth, background, or noise — so
  this is NOT a quantitative image simulator (that's the nanoBragg-style path below).
- Structure factors are computed by direct summation, so large custom proteins at fine resolution
  are slow (seconds-to-minutes of one-time compute) — hence the 3 Å default for them.
- Built-in 6AQT is the chains-A/B duplex only (the full PDB exceeded the offline fetch limit used
  when embedding it); for the exact full structure, fetch/upload it.

## Roadmap

1. **Performance**: cache the mounting-rotated reciprocal vectors so dragging doesn't recompute
   them; FFT-based structure factors for large proteins.
2. **nanoBragg backend (research)**: couple the UI to `simtbx.nanoBragg` via a Python service for
   photon-count images (lattice shape transform, mosaicity, divergence, spectrum, noise,
   background). A JS lattice shape-transform (sincg) is a partial client-side bridge.
3. **Cromer–Mann form factors + Lorentz–polarization toggle** for closer-to-experiment intensities.
4. **Desktop polish**: FPS/reflection meter, fullscreen, keyboard φ stepping, click-a-spot-to-read-hkl.
5. **VR (Meta Quest 3)**: WebXR build of the reciprocal-space scene with grab-to-rotate crystal;
   2D panels as floating in-world surfaces; mixed-reality passthrough.

## Getting started

Open `crystal_lab.html` in any modern browser (needs internet the first time to load Three.js
from the CDN; cached afterwards). Pick a structure, hit **Rotate**, and switch between the
Real-space and Reciprocal tabs. Try: turn on **Powder rings** with Ice loaded; compare crambin
vs 6AQT; type a PDB code to fetch your own.

## Tech

Single-file HTML + vanilla JavaScript + Three.js (r128) from cdnjs. No dependencies to install.
