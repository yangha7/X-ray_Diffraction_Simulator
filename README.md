# Crystal Lab — diffraction & reciprocal-space teaching/exploration tool

Interactive, zero-install browser tools for X-ray crystallography: the Ewald
construction, Bragg's law, reciprocal space, and structure factors, coupled to a
live simulated detector. Built as single self-contained HTML files (Three.js via
CDN) — just double-click to open in any modern browser.

## Files

- **crystal_lab.html** — main tool. Real-space crystal ⇄ reciprocal/Ewald ⇄ detector,
  with real structure-factor intensities. Now generalized to **molecular crystals**:
  general (triclinic) unit cells, full space-group symmetry, CIF upload, built-in examples.
- **diffraction_simulator.html** — the original lattice/Ewald simulator (adjustable cell,
  wavelength, detector distance, oscillation; rotation image build-up). Kept as a simpler
  reference / teaching view.

## crystal_lab.html — features

- Two synchronized 3D views (tabs): real-space crystal, and reciprocal lattice + Ewald sphere.
- Live detector panel; spot size/brightness ∝ |F| (so strong/weak/absent reflections show).
- Real structure factors F(hkl)=Σ f·exp(2πi(hx+ky+lz))·exp(−B s²); absences are physical.
- General triclinic cells via the reciprocal metric (verified vs analytic d-spacings).
- Full space-group symmetry: operators expand the asymmetric unit to the whole cell.
- Structure picker is two-level: a **category** dropdown (Inorganic crystals / Small molecules /
  Macromolecules — coming soon / Uploaded CIF) then the species within it.
- Inorganic set (NaCl, CsCl, diamond, Si, Cu fcc, Fe bcc, Po, ZnS, CaF₂) retained, now driven
  through the symmetry engine (F/I/P centering operators).
- Structure input: built-in examples + **CIF upload** (reads cell, symmetry, atoms; U→B).
- Every left-panel parameter (λ, d_min, distance, φ, oscillation, B-factor, occupancy) is a
  slider **plus a number box** — drag or type an exact value. Oscillation steps down to 0.05.
- Edit the asymmetric unit (element / x,y,z / B-factor / occupancy); symmetry regenerates live.
- Crystal orientation: free 3-D reorientation by dragging (beam fixed, like the goniometer)
  + goniometer φ on top; "rotate whole system" checkbox; reset button.
- Live Bragg readout + λ = 2 d sinθ equation with numbers substituted in real time.
- (hkl) Bragg planes highlighted in the rotating crystal (correct orientation, spacing,
  registered to lattice nodes); diffracted rays at 2θ; resolution rings; hkl labels on spots.
- Supercell view (1/2/3 cells), atoms-vs-dots display, bonds.

## Verified (numerically)

- fcc/bcc/diamond/CsCl/NaCl systematic absences correct; NaCl (111) weak vs (200) strong.
- Triclinic/hexagonal reciprocal metric matches analytic d-spacing.
- Symmetry expansion multiplicity correct (special-position dedup).
- CIF parser: cell, symop strings, atom_site, U_iso→B conversion.

## Honest limitations (for research use)

- Scattering factors are an approximate Z·Gaussian + B-factor (absences exact; fine
  relative intensities are not). Cromer–Mann form factors are a drop-in upgrade.
- No Lorentz–polarization, partiality, mosaicity convolution, beam divergence/bandwidth,
  background, or noise — i.e. NOT a quantitative image simulator (see nanoBragg note below).
- Built-in molecular example (urea) uses idealized geometry; upload a CIF for real data.
- Proteins not yet supported (large atom counts + space groups + cached |F| — next tier).

## Parked to-do / backlog

1. **Desktop polish**: FPS/reflection meter, fullscreen, keyboard arrows to step φ,
   click a detector spot / reciprocal point to read its (hkl), declutter labels,
   fix drag-release-outside-window, plane contrast.
2. **Protein tier**: PDB loading, large space groups, compute-|F|-once-and-cache.
3. **nanoBragg backend (research)**: couple UI to simtbx.nanoBragg via a Python
   service for photon-count images (lattice shape transform, mosaicity, divergence,
   spectrum, noise, background). A JS lattice shape-transform (sincg) is a partial
   client-side bridge.
4. **VR (long-term, Meta Quest 3)**: WebXR build of the reciprocal-space scene with
   grab-to-rotate crystal; 2D panels become floating in-world surfaces; MR passthrough.
5. Cromer–Mann form factors + Lorentz–polarization toggle for closer-to-experiment intensities.

## Tech

Single-file HTML + vanilla JS + Three.js (r128) from cdnjs. No build step, no server,
no dependencies to install. Works offline once loaded (except the CDN fetch of Three.js).
