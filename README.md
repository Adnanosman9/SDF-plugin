<div align="center">

# KiCad SDF Exporter

A KiCad plugin that converts schematics into AI-trainable structured design packages for training electronics design models.

<img src="https://img.shields.io/badge/Python-3.8+-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
<img src="https://img.shields.io/badge/KiCad-6.0+-314CB0?style=for-the-badge&logo=kicad&logoColor=white" alt="KiCad">
<img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" alt="MIT License">

</div>

---

## What Is This?

This plugin converts KiCad schematics into structured data packages for training AI models. It extracts **reusable circuit patterns** like I2C interfaces, power distribution, and sensor connections—teaching models design primitives that generalize across domains (drones, keyboards, sensors, motor controllers) rather than memorizing complete designs.

**Use Case:** Building AI datasets for PCB design and electronics reasoning models.

**The Goal:** Train AI models that understand electronics design patterns and can generate schematics from structured specifications.

---

## Quick Start

```bash
# Requires Python 3.8+
python --version

# Extract and run
unzip kicad_sdf_exporter_v0_2_0.zip
cd kicad_sdf_exporter

# Export your schematic
python -m kicad_sdf_exporter.cli path/to/schematic.kicad_sch -o output_folder
```

---

## Output Files

```
output_folder/
├── schematic.sdf           # Visual/electrical schematic format
├── components.json         # Parts, pins, values, footprints, positions
├── nets.json              # All connections including named labels
├── bom.json               # Bill of materials
├── blocks.json            # ⭐ Inferred functional blocks (key feature)
├── metadata.json          # Warnings, quality checks, statistics
└── design_package.json    # Combined training file
```

### Key Innovation: `blocks.json`

Infers **reusable circuit design patterns** for AI training:

- Power distribution blocks (3V3, 5V, GND rails)
- Interface blocks (I2C, SPI, UART, USB)
- MCU core blocks (microcontroller + decoupling + reset)
- Sensor interfaces (IMU, ADC sensors)
- Support components (pullup/pulldown networks)
- Connector blocks (headers, programming interfaces)

**Example:**
```json
{
  "id": "block_imu_i2c",
  "type": "i2c_interface",
  "components": ["U1", "R6", "R7"],
  "nets": ["IMU_SDA", "IMU_SCL", "3V3", "GND"],
  "rules": ["I2C lines require pullup resistors"]
}
```

This teaches models that I2C needs pullups, MCUs need decoupling, USB needs specific support—regardless of application domain.

---

## Usage

### Requirements
- Python 3.8+ (no dependencies, uses standard library only)
- KiCad 6.0+ schematic files (`.kicad_sch`)

### Basic Usage

```bash
# Export a schematic
python -m kicad_sdf_exporter.cli path/to/project.kicad_sch -o output_folder

# Check for quality warnings
type output_folder\metadata.json  # Windows
cat output_folder/metadata.json   # Linux/Mac
```

**Always check `metadata.json` for warnings.** Fix issues in KiCad before using exports as training data.

### Building a Dataset

```bash
# Export multiple designs
python -m kicad_sdf_exporter.cli drone_controller.kicad_sch -o dataset/drone_001
python -m kicad_sdf_exporter.cli keyboard.kicad_sch -o dataset/keyboard_001
python -m kicad_sdf_exporter.cli sensor_board.kicad_sch -o dataset/sensor_001

# Verify quality
# Use only exports with warning_count == 0 or acceptable threshold
```

---

## Troubleshooting

**"No module named 'kicad_sdf_exporter'"**
- Run from inside the `kicad_sdf_exporter` folder

**Net connection warnings**
- Open schematic in KiCad
- Verify labels are placed exactly on pins or wires
- Check label names match pin functions
- Fix any floating/disconnected labels

**"No components found"**
- File may be from older KiCad version
- Verify file opens correctly in KiCad

**Power symbols missing from BOM**
- Intentional: GND, +3V3, etc. are treated as net names, not components
- They appear in `nets.json` instead

---

## Engineering Notes

Key problems solved during development:

**1. Net Resolution:** KiCad uses named labels as invisible connections. Built a net resolver that merges all pins/wires/labels with the same name into unified nets.

**2. Power Symbol Handling:** Power symbols were polluting the BOM. Now detected by type and converted to net labels.

**3. Block Inference:** Raw schematics only show "U1 pin 5 → R3 pin 1" without context. Added pattern detection engine for I2C interfaces, power rails, MCU cores, and other functional blocks.

**4. Data Quality:** Filtered anonymous single-pin nets and added quality warnings for suspicious patterns.

---

## Research Context

Part of a high school research project exploring:

> **"Can structured schematic representations with inferred functional blocks improve AI generation of valid electronic circuits?"**

---

## Limitations

- **Rule-based inference:** Block detection uses pattern matching, not learned models
- **Error propagation:** Source schematic errors carry into exports
- **No validation:** No electrical rule checking or simulation
- **Single-file only:** Hierarchical multi-sheet designs not supported
- **Limited patterns:** Only common blocks (I2C, power, MCU) currently detected

---

## Roadmap

- [ ] Web-based schematic renderer
- [ ] AI model training pipeline
- [ ] ERC validation layer
- [ ] Hierarchical design support
- [ ] ML-based block detection
- [ ] SPICE export integration

---

## Credits

**Development:** Codex (AI implementation system) handled plugin implementation, net resolution, block inference, and debugging.

**Design & Direction:** I myself provided electronics domain expertise, project vision, schematic testing, and guided debugging through multiple iterations.

<div align="center">

**Built with:**

<img src="https://img.shields.io/badge/Codex-AI_Implementation-412991?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMjQiIGhlaWdodD0iMjQiIHZpZXdCb3g9IjAgMCAyNCAyNCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHBhdGggZD0iTTEyIDI0QzE4LjYyNzQgMjQgMjQgMTguNjI3NCAyNCAxMkMyNCA1LjM3MjU4IDE4LjYyNzQgMCAxMiAwQzUuMzcyNTggMCAwIDUuMzcyNTggMCAxMkMwIDE4LjYyNzQgNS4zNzI1OCAyNCAxMiAyNFoiIGZpbGw9IndoaXRlIi8+Cjwvc3ZnPg==" alt="Codex">

</div>

---

## License

MIT License

---

## Contributing

Contributions welcome:
- Test on diverse schematics
- Report edge cases or incorrect block inference
- Suggest additional block types
- Improve SDF format specification

---

<div align="center">

**Note:** This is research software. Always verify generated designs with proper ERC, simulation, and engineering review before fabrication.

</div>
