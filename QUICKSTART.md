# Quick Start Guide

## What Is This?

KiCad SDF Exporter converts KiCad schematics into structured data packages for training AI models on electronics design. It extracts reusable circuit patterns (I2C interfaces, power distribution, sensor connections) that teach models design primitives rather than memorizing complete designs.

**Use Case:** Building AI datasets for PCB design and electronics reasoning models.

---

## Installation

```bash
# Requires Python 3.8+
python --version

# Extract the plugin
unzip kicad_sdf_exporter_v0_2_0.zip
cd kicad_sdf_exporter
```

---

## Usage

```bash
# Export a schematic
python -m kicad_sdf_exporter.cli path/to/your_schematic.kicad_sch -o output_folder

# Example
python -m kicad_sdf_exporter.cli ~/projects/drone.kicad_sch -o ~/exports/drone_v1
```

---

## Output Files

```
output_folder/
├── schematic.sdf          # Schematic description format
├── components.json        # Parts, pins, values, footprints
├── nets.json             # All electrical connections
├── bom.json              # Bill of materials
├── blocks.json           # ⭐ Inferred functional blocks
├── metadata.json         # Quality checks and warnings
└── design_package.json   # Combined training file
```

**Key file:** `blocks.json` contains inferred circuit patterns (I2C interfaces, power rails, MCU cores) for AI training.

**Always check** `metadata.json` for quality warnings before using as training data.

---

## Troubleshooting

**"No module named 'kicad_sdf_exporter'"**
- Make sure you're inside the `kicad_sdf_exporter` folder

**Net connection warnings in metadata.json**
- Open schematic in KiCad and verify labels are placed on pins/wires
- Fix any floating or disconnected labels

**"No components found"**
- Verify the file opens correctly in KiCad 6.0+

**Power symbols (GND, +3V3) missing from BOM**
- Intentional: they're treated as net names, not components
- Find them in `nets.json` instead

---

## Next Steps

- Review `metadata.json` for quality warnings
- Fix any issues in your KiCad schematic
- Export multiple diverse designs to build a dataset
- Use `blocks.json` for AI model training

📘 **For detailed documentation, see the main [README.md](README.md)**
