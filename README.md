# 2D Laser Scanning Heat Transfer Simulation - FreeFEM++

<p align="center">
  <img src="https://img.shields.io/badge/FreeFEM++-Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/Laser%20Scanning-Heat%20Transfer-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Melt%20Pool-Tracking-red?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/ParaView-VTK%20Export-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/DOI-10.5281%2Fzenodo.19739207-blue?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey?style=for-the-badge"/>
</p>

<p align="center">
  A 2D finite element simulation of <b>laser scanning heat transfer and melt pool dynamics</b> using FreeFEM++.
  Models a Gaussian laser beam scanning across a steel substrate surface at 5 mm/s with
  transient heat conduction, convective boundary conditions, and real-time melt pool detection.
</p>
<img width="1008" height="772" alt="laser heat" src="https://github.com/user-attachments/assets/de967f66-a483-406c-b490-5c09445a6cef" />

---

## Citation

If you use this code in your research, please cite:

```bibtex
@software{mishra_2026_laser,
  author    = {Mishra, A.},
  title     = {2D Laser Scanning Heat Transfer Simulation - FreeFEM++},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.19739207},
  url       = {https://doi.org/10.5281/zenodo.19739207}
}
```

Plain text citation:

> Mishra, A. (2026). *2D Laser Scanning Heat Transfer Simulation - FreeFEM++*. Zenodo. https://doi.org/10.5281/zenodo.19739207

---

## Physics

Laser surface processing is a widely used manufacturing technique where a focused laser beam scans across a metallic substrate, inducing rapid heating, melting, and resolidification. The resulting melt pool geometry and thermal history determine microstructure, residual stress, and bonding quality in applications such as laser welding, cladding, and selective laser melting.

This simulation models the following coupled phenomena:

- Transient 2D heat conduction with implicit Euler time integration
- Moving Gaussian laser beam as a surface flux (Neumann boundary condition)
- Convective heat loss on all boundaries (Newton cooling)
- Real-time melt pool detection and cumulative heat-affected zone tracking
- Temperature, heat flux, and thermal gradient fields exported per timestep

---

## Geometry

```
  Laser beam (Gaussian, moves left to right)
       |
       v
  |-------------------------------------------|  <- TopEdge (laser flux here)
  |                                           |
  |           [Steel Substrate]               |  Ly = 5 mm
  |                                           |
  |___________________________________________|
  0                                          Lx
                   Lx = 10 mm
```

The laser beam starts at `x = 0` (left edge) and scans to `x = Lx` (right edge) along the top surface. The beam intensity follows a Gaussian profile centred at the current beam position.

---

## Material Parameters

### Steel Substrate

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Density | rho | 8000 | kg/m3 |
| Specific heat | cp | 500 | J/kg/K |
| Thermal conductivity | kk | 50 | W/m/K |
| Convection coefficient | hc | 20 | W/m2/K |
| Initial temperature | T0 | 300 | K |
| Melting temperature | Tmelt | 1700 | K |

---

## Laser Parameters

| Parameter | Symbol | Value | Unit |
|-----------|--------|-------|------|
| Laser power | P | 500 | W |
| Beam radius (1/e2) | r0 | 0.5 | mm |
| Surface absorptivity | absorb | 0.3 | - |
| Scan speed | vbeam | 5 | mm/s |
| Peak flux | Qpeak | absorb * 2P / (pi * r0^2) | W/m2 |

---

## Governing Equation

### Transient Heat Conduction

```
rho * cp * dT/dt - div(k * grad(T)) = 0        in Omega
```

### Boundary Conditions

```
-k * dT/dn = Q_laser(x, t)                     on TopEdge (laser flux)
-k * dT/dn = hc * (T - T0)                     on all edges (convection)
```

### Gaussian Laser Flux

```
Q(x, t) = absorb * (2P / (pi * r0^2)) * exp(-2 * (x - xc(t))^2 / r0^2)
```

Where `xc(t) = vbeam * t` is the moving beam centre.

### Melt Pool Detection

```
meltpool(x, t) = 1    if T(x,t) >= Tmelt
meltpool(x, t) = 0    if T(x,t) <  Tmelt

meltever(x)    = 1    if T(x,t') >= Tmelt for any t' <= t   [latching]
meltdepth(x,t) = T(x,t) - Tmelt           [positive inside pool]
```

---

## Numerical Method

| Aspect | Choice |
|--------|--------|
| Spatial discretisation | Finite Element Method (FEM) |
| Element type | P1 (linear) for all fields |
| Time integration | Implicit Euler (unconditionally stable) |
| Matrix assembly | varf + A^-1 * b (LHS assembled once) |
| Linear solver | Conjugate Gradient (CG) |
| Mesh refinement | adaptmesh once before loop, refined at beam start |
| Laser source | Inline Gaussian in int1d on TopEdge |

The system matrix `A` is assembled **once** before the time loop since material properties and geometry are fixed. Only the RHS vector `b` is rebuilt each timestep to account for the moving beam position and updated temperature history. This makes the simulation efficient even for large step counts.

---

## Output Fields

Each `.vtu` file contains the following fields:

| Field | Description | Typical range |
|-------|-------------|---------------|
| `Temperature` | Nodal temperature | 300 to 1700 K |
| `MeltPool` | Instantaneous melt indicator | 0 (solid) or 1 (molten) |
| `MeltEver` | Cumulative melt / HAZ indicator | 0 or 1 (latched) |
| `MeltDepth` | T - Tmelt (signed distance to liquidus) | negative to 0 |
| `gradTx` | Temperature gradient in x | W/m |
| `gradTy` | Temperature gradient in y | W/m |
| `HeatFlux` | Magnitude of heat flux vector | W/m2 |

---

## Repository Structure

```
laser-scanning-freefem/
|
|-- laser.edp                    # Main FreeFEM++ simulation script
|-- laser_results/
|   |-- laser.pvd                # ParaView collection file
|   |-- result_0004.vtu          # Time step 4
|   |-- result_0008.vtu          # Time step 8
|   |-- ...                      # Every 4 steps
|-- README.md
```

---

## How to Run

### Requirements

- FreeFEM++ v4.10 or later: https://freefem.org
- ParaView v5.x or later: https://www.paraview.org

### Step 1 - Run the simulation

```bash
FreeFem++ laser.edp
```

The script will:
1. Build the 10 mm x 5 mm substrate mesh
2. Refine the mesh near the top surface
3. Assemble the system matrix once
4. Run timesteps until beam reaches the right edge
5. Save VTU files every 4 steps to `laser_results/`
6. Write `laser.pvd` linking all files with time values

Console output per saved step:
```
Step 4/4000    t=0.002 s   xbeam=0.01 mm   Tmax=1412.3 K   MeltNodes=0
Step 8/4000    t=0.004 s   xbeam=0.02 mm   Tmax=1698.7 K   MeltNodes=14
Step 12/4000   t=0.006 s   xbeam=0.03 mm   Tmax=1700.0 K   MeltNodes=31
```

### Step 2 - Open in ParaView

1. `File > Open` > navigate to `laser_results/`
2. Change Files of type to `All Files (*.*)`
3. Select `laser.pvd` > OK
4. Choose PVD Reader when prompted > OK
5. Click `Apply`
6. Set colour field to `Temperature` or `MeltPool`
7. Click `Rescale to Data Range Over All Timesteps`

### Step 3 - Visualize the melt pool

**Option A — Color by MeltPool**
```
Set colour field to:   MeltPool
Colour map:            Blue-Red  (0=solid, 1=molten)
Press Play
```

**Option B — Contour at solidus boundary (recommended)**
```
Filters > Contour
  Contour By:    MeltDepth
  Value:         0.0
  Apply
  Colour:        White
Press Play
```

**Option C — Heat affected zone track**
```
Set colour field to:   MeltEver
This shows the full HAZ track left behind as the beam scans across
```

Press `Play` to watch the laser scan animation.

---

## What to Look for in Results

### Temperature Field

The thermal plume follows the beam, heating the surface above the melting point near the beam centre. A steep gradient is visible ahead of the beam (cold solid) and a slower decay behind (cooling weld track).

### Melt Pool (MeltPool field)

A small elliptical region at the beam centre where `T >= Tmelt`. Pool length increases as the beam slows down relative to the thermal diffusion length. Pool disappears rapidly behind the beam as the material resolidifies.

### Melt Ever (MeltEver field)

The permanent record of every node that reached melting temperature. This is the simulated weld bead or remelted track. Its width and depth are key outputs for process parameter optimisation.

### Heat Flux (HeatFlux field)

Largest at the beam edges where the temperature gradient is steepest. Use `Filters > Glyph` with the `gradTx / gradTy` vectors to show the heat flow direction field.

---

## Common Errors and Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `End of String` lex error | Em dash or non-ASCII in string literal | Replace with plain hyphen or ASCII only |
| `Compile error: )` line ~99 | `problem` or `func` defined inside loop | Use `varf` + `A^-1 * b` outside loop |
| `Compile error: )` in `int1d` | `^` operator inside boundary integral | Replace `r0^2` with `r0*r0`, `(x-xc)^2` with `(x-xc)*(x-xc)` |
| Multiple labels in `int1d` | `int1d(Th, label1, label2)` not supported | Use one `int1d` per label |
| `k` or `h` variable name | Reserved keywords in FreeFEM++ | Rename to `kk` and `hc` |
| Matrix blow-up | Tmelt clamping applied after Told update | Always clamp before copying to Told |
| Melt pool never appears | Power too low or r0 too large | Increase P or decrease r0 |
| Zero temperature change | Mesh too coarse at top edge | Reduce hmin in adaptmesh |

---

## Extending the Model

| Extension | What to change |
|-----------|----------------|
| Different material | Update rho, cp, kk, Tmelt |
| Raster scan pattern | Add Y-direction scan after X completes |
| 3D model | Replace mesh with mesh3, use int3d and int2d for surface flux |
| Latent heat of fusion | Add enthalpy source term near Tmelt |
| Temperature-dependent k | Replace kk with kk(T) as a func |
| Solidification tracking | Add a resolidification field where dT/dt < 0 and T was molten |
| Keyhole / vapour pressure | Add recoil pressure term on top surface BC |
| Multi-layer deposition | Repeat scan at y = Ly + layer_thickness |

---

## Author

**akshansh11**  
GitHub: https://github.com/akshansh11

---

## License

<p>
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
<img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png"/>
</a>
<br/>
This work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.
</p>

You are free to:

- **Share** - copy and redistribute the material in any medium or format
- **Adapt** - remix, transform, and build upon the material

Under the following terms:

- **Attribution** - You must give appropriate credit to akshansh11 and provide a link to this repository
- **NonCommercial** - You may not use the material for commercial purposes

Copyright 2026 akshansh11. All rights reserved for commercial use.
