# Crystal Lab — X-ray diffraction & reciprocal-space simulator

**Version 1.02**  ·  VR viewer **v0.4.7**

An interactive, zero-install browser tool for X-ray crystallography. It ties together the
real-space crystal, the reciprocal lattice + Ewald sphere, and a live simulated detector, all
driven by real structure-factor intensities — so you can *see* how a structure produces its
diffraction pattern, how the Ewald construction and Bragg's law work, and how rotating the
crystal sweeps reflections onto the detector.

Built as a single self-contained HTML file (Three.js from a CDN). No build step, no server, no
install — just open it in a browser.

## Files

- **crystal_lab.html** — the main tool (this is v1.02).
- **vr_viewer.html** — standalone WebXR viewer for Meta Quest 3 (v0.4.7): stand inside the
  experiment with real space in front, reciprocal space behind, a control panel on your left, and
  a whole-molecule viewer on your right. Served over HTTPS via GitHub Pages. See the *VR viewer*
  section below.
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
  spots); **spot-contrast** gamma slider (1.0 = linear/true intensity, < 1 boosts weak spots,
  **> 1 shows only strong reflections** — applies to both spots and powder rings, via the same
  2% visibility gate); hkl labels; **Accumulate** to build a rotation dataset.
- **Detector distance**: log-scaled slider **40 mm → 10 m** (number box takes exact mm), so the
  same tool covers macromolecular crystallography and the **SAXS** regime. Resolution rings
  auto-scale to the detector at long camera lengths. Note: at long distances only large
  d-spacings reach the detector (> ~29 Å at 5 m, > ~57 Å at 10 m), so use large-cell structures
  for SAXS-range spots.
- **Beam**: wavelength **0.01–2.0 Å** with a unit selector — **λ (Å)**, **X-ray energy (keV)**
  (E = hc/λ), or **electron energy (keV)** (relativistic de Broglie). Short wavelengths reach the
  electron-diffraction regime (0.0251 Å ≈ 200 keV), where the Ewald sphere is nearly flat.
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

## VR viewer (`vr_viewer.html`, v0.4.7)

A standalone WebXR build for Meta Quest 3 (also runs on desktop as a mouse-driven preview),
served over HTTPS from GitHub Pages. When you enter VR you stand inside the experiment:

- **Front** — real space: the incident beam (left→right), the crystal on the goniometer, red
  diffracted rays, and a detector ⟂ the beam at the downstream end.
- **Behind you** — reciprocal space: the reciprocal lattice and the Ewald sphere (which scales
  with λ), sharing the same beam direction as the front.
- **Left** — a floating control panel (plane ⟂ the beam), driven with the controller laser:
  point + trigger for buttons/toggles, hold the trigger and sweep to drag a slider.
- **Right** — a whole-molecule viewer showing the intact, un-chopped molecule (the crystal view
  wraps atoms into the cell, cutting molecules at boundaries; this shows the complete one).

Built-in structures: NaCl, diamond, CsCl, Ice Ih, **crambin (1CRN)**, and **DNA d(CGCGCG)₂
(6AQT)**. Small molecules render as instanced spheres; macromolecules as element-coloured bond
lines (split at each bond's midpoint, same colour code as the ball model).

Panel controls: structure picker · supercell (1×1×1 / 2×2×2 / 3×3×3) · beam wavelength with an
**Å / X-ray keV / electron keV** selector · detector distance · resolution d_min · spot contrast ·
goniometer spin speed + on/off · and show/hide toggles for atoms, diffracted rays, reciprocal
points, detector, the molecule viewer, **Miller hkl labels**, and **powder rings**. The right
thumbstick (up/down) zooms the crystal and molecule; immersive-VR and AR passthrough are both
supported.

Parity with `crystal_lab.html`: identical structure factors and Ewald construction, and the
excitation band is matched (0.008) so a VR *still* overlays a crystal_lab *still* spot-for-spot at
the same structure, resolution, and orientation (the VR detector is rotated 90° because its beam
runs along +x rather than +z). Reflections are capped at 50k with resolution-based truncation, so
the reciprocal lattice stays a complete sphere (to a slightly coarser radius) rather than chopped.

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
5. **VR (Meta Quest 3)** — *shipped as `vr_viewer.html` v0.4.7* (see the VR viewer section above):
   full real/reciprocal/detector scene with an in-headset control panel, a whole-molecule viewer,
   crambin + DNA, Miller-index labels, and powder rings. Next: drive it live from the desktop
   controls; larger structures in-headset; an optional oscillation (Δφ wedge) model.

## Getting started

Open `crystal_lab.html` in any modern browser (needs internet the first time to load Three.js
from the CDN; cached afterwards). Pick a structure, hit **Rotate**, and switch between the
Real-space and Reciprocal tabs. Try: turn on **Powder rings** with Ice loaded; compare crambin
vs 6AQT; type a PDB code to fetch your own.

## Tech

Single-file HTML + vanilla JavaScript + Three.js (r128) from cdnjs. No dependencies to install.
