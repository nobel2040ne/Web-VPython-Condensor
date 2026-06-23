# Web VPython Capacitor Simulator

An interactive 3D parallel-plate capacitor simulation built with Web VPython.

Adjust the plate gap, plate area, and dielectric constant with sliders and watch capacitance (C) and charge (Q) update live. Charges and field vectors are drawn on the plates, and a dielectric slab can be dragged in and out of the gap. Connect the capacitor to a battery to charge it, or to an LED to watch it discharge as the light fades.

## Controls

- **Sliders** — distance between plates (d), plate area (S), dielectric constant (ε)
- **Drag & drop** — move the dielectric into or out of the capacitor
- **Connect Battery** — charge the capacitor
- **Connect LED** — discharge through an LED that dims as charge drops to zero
- **Arrow keys** — pan the view

## Run

Paste `Implementation.py` into the [Web VPython](https://www.glowscript.org) editor and run, or run locally:

```bash
pip install vpython
python Implementation.py
```
