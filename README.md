# Crystal Lab — diffraction & reciprocal-space teaching/exploration tool

Interactive, zero-install browser tools for X-ray crystallography: the Ewald
construction, Bragg's law, reciprocal space, and structure factors, coupled to a
live simulated detector. Built as single self-contained HTML files (Three.js via
CDN) — just double-click to open in any modern browser.

## Files

- **crystal_lab.html** — main tool. Real-space crystal ⇄ reciprocal/Ewald ⇄ detector,
  with real structure-factor intensities. Handles inorganic crystals, small molecules,
  and macromolecules (proteins / DNA): general (triclinic) cells, full space-group
  symmetry, PDB/mmCIF/CIF upload, built-in examples.
- **diffraction_simulator.html** — the original lattice/Ewald simulator (adjustable cell,
  wavelength, detector distance, oscillation; rotation-image build-up). Kept as a simpler
  reference / teaching view.
- **sample_1CRN.pdb** — crambin PDB, handy for testing the upload path.

## crystal_lab.html — features

- Two synchronized 3D views (tabs): real-space crystal, and reciprocal lattice + Ewald sphere.
- Live detector panel; spot size/brightness ∝ |F|. Systematic absences are physically correct.
- Real structure factors F(hkl)=Σ f·exp(2πi(hx+ky+lz))·exp(−B s²) over the symmetry-expanded cell.
- General triclinic cells via the reciprocal metric (verified vs analytic d-spacings).
- Full space-group symmetry: operators expand the asymmetric unit to the whole cell.
- Two-level structure picker: **category** (Inorganic / Small molecules / Macromolecules /
  Uploaded) then the species.
- **Macromolecule support**: loads PDB, mmCIF, and CIF.
  - PDB: CRYST1 (cell) + REMARK 290 (symmetry) + ATOM/HETATM (Cartesian → fractional via the
    reciprocal metric, validated against the file's SCALE records). ANISOU lines skipped.
  - mmCIF: dotted tags (`_cell.length_a`, `_atom_site.Cartn_x`, `_space_group_symop.operation_xyz`).
  - Built-in demos: **crambin (1CRN)** protein, and **6AQT** DNA duplex d(CGCGCG)₂ (chains A/B).
  - Large structures render as points, compute |F| once, and boost weak spots by default.
- Inorganic set (NaCl, CsCl, diamond, Si, Cu fcc, Fe bcc, Po, ZnS, CaF₂) driven through the
  symmetry engine (F/I/P centering operators).
- Beam is horizontal (left→right); **goniometer spindle = x axis** (horizontal, ⟂ beam).
  Auto-rotation advances one oscillation-width Δφ per step (contiguous wedges, like real
  data collection).
- Detector: resolution rings, **Log-scale intensity** toggle (5-decade, reveals weak/high-res
  spots) + a **spot-contrast** gamma slider, hkl labels, detector reaches ~1 Å at default distance.
- Reflection generation caps cleanly by resolution (a complete sphere), not a lopsided hkl cut;
  cap 250k so proteins/DNA reach 1 Å.
- Every left-panel parameter (λ, d_min, distance, φ, oscillation, contrast, B, occupancy) is a
  slider **plus a number box** — drag or type. Oscillation steps down to 0.05.
- Edit the asymmetric unit (element / x,y,z / B-factor / occupancy); symmetry regenerates live.
- Crystal orientation: free 3-D reorientation by dragging (beam fixed), goniometer φ on top;
  "rotate whole system" checkbox; reset button.
- Live Bragg readout + λ = 2 d sinθ equation with numbers substituted in real time.
- (hkl) Bragg planes highlighted in the rotating crystal (on-scene toggle); diffracted rays at 2θ.
- Supercell view (1/2/3 cells), atoms-vs-dots display, bonds, per-structure atom-colour legend.

## Verified (numerically)

- fcc/bcc/diamond/CsCl/NaCl systematic absences; NaCl (111) weak vs (200) strong.
- Triclinic/hexagonal reciprocal metric matches analytic d-spacing.
- PDB/mmCIF/CIF parsers vs real crambin + 6AQT + synthetic files; Cartesian→fractional
  reproduces each file's SCALE records exactly.
- Symmetry expansion multiplicity (special-position dedup).
- Beam-axis rotation is degenerate (0 reflections sweep); x-axis rotation sweeps normally.

## Honest limitations (for research use)

- Scattering factors are an approximate Z·Gaussian + B-factor (absences exact; fine relative
  intensities are not). Cromer–Mann form factors are a drop-in upgrade.
- No Lorentz–polarization, partiality, mosaicity, divergence/bandwidth, background, or noise —
  NOT a quantitative image simulator (see nanoBragg note).
- Direct-summation |F|²; big proteins at 1 Å are a few seconds to load and heavier to rotate.
- Built-in 6AQT is the chains-A/B duplex only (full PDB exceeded the fetch tool); for the exact
  full structure, upload the PDB.

## Parked to-do / backlog

1. **Desktop polish**: FPS/reflection meter, fullscreen, keyboard arrows to step φ,
   click a detector spot / reciprocal point to read its (hkl), drag-release-outside-window.
2. **Performance**: cache the mounting-rotated reciprocal vectors so dragging doesn't recompute
   them (smoother 1 Å rotation); consider FFT structure factors for large proteins.
3. **nanoBragg backend (research)**: couple UI to simtbx.nanoBragg via a Python service for
   photon-count images (lattice shape transform, mosaicity, divergence, spectrum, noise,
   background). A JS lattice shape-transform (sincg) is a partial client-side bridge.
4. **VR (long-term, Meta Quest 3)**: WebXR build of the reciprocal-space scene with
   grab-to-rotate crystal; 2D panels become floating in-world surfaces; MR passthrough.
5. Cromer–Mann form factors + Lorentz–polarization toggle for closer-to-experiment intensities.

## Tech

Single-file HTML + vanilla JS + Three.js (r128) from cdnjs. No build step, no server,
no install. Works offline once loaded (except the CDN fetch of Three.js).
