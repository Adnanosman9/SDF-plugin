# KiCad SDF Exporter for AI Training

A powerful KiCad plugin and standalone CLI tool designed to convert `.kicad_sch` files into highly structured, block-aware datasets for training AI electronic design agents. 

Unlike standard netlists, this exporter doesn't just list wires and pins. It understands KiCad's geometry, resolves named labels (like `SDA` or `SCL`), and infers **functional design blocks** (like I2C interfaces, power distribution, and pullup networks) so AI models can learn reusable circuit patterns rather than memorizing flat data.

## Features
* **Native Parsing:** Reads `.kicad_sch` files directly without needing KiCad UI hooks.
* **Smart Net Resolution:** Accurately connects physical wires, resolves same-name KiCad labels, and maps them to physical component pins.
* **Functional Block Inference (v0.2.0+):** Automatically groups components into logical blocks (e.g., Power Rails, SPI/I2C interfaces, Pullup/Pulldown networks, Core MCUs).
* **AI-Safe Quality Flags:** Detects floating labels and unconnected pins, flagging suspicious blocks so your AI doesn't train on bad data.
* **Multi-Format Export:** Generates `.sdf` (Schematic Description Format) for rendering, alongside rich JSON metadata for model training.

## Installation

You can run this tool from anywhere on your computer. 

1. Download the latest release `.zip` from the Releases page or clone this repository.
2. Extract the folder to a location of your choice.
3. Ensure you have Python 3 installed. No external dependencies are required for the core parser.

## Usage

Run the exporter as a Python module from your terminal or command prompt.

1. Open your terminal.
2. Navigate (`cd`) into the folder where you extracted the tool.
3. Run the CLI, pointing it to your schematic and your desired output folder:

**Windows Command:**
```powershell
python -m kicad_sdf_exporter.cli "C:\path\to\your_schematic.kicad_sch" -o "C:\path\to\output_folder"
```

**Mac/Linux Command:**
```bash
python3 -m kicad_sdf_exporter.cli "/path/to/your_schematic.kicad_sch" -o "/path/to/output_folder"
```

*Note: If your paths contain spaces, ensure you wrap them in quotation marks as shown above.*

## Output Structure

Running the tool generates a complete "Design Package" in your output folder:

* `design_package.json`: The master file containing all data (ideal for AI ingestion).
* `blocks.json`: The inferred functional blocks and design intent.
* `schematic.sdf`: The structured format used for visual rendering.
* `components.json`: Raw component data, coordinates, and properties.
* `nets.json`: Fully resolved electrical connections.
* `bom.json`: Cleaned Bill of Materials (ignoring power symbols).
* `metadata.json`: Exporter warnings and schematic health checks.

---

## Data Guide: How to Read the Exported Package

### The Goal of the Data
When looking at the exported folder, remember the goal: **We are teaching an AI "Why" things connect, not just "How".** If you only feed an AI `nets.json`, it learns to draw lines. If you feed it `blocks.json`, it learns electronic engineering.

### How to Read the Files (In Order of Importance for AI)

* **`blocks.json` (The Brains):** Read this file to understand the *intent* of the circuit. You will see JSON objects categorizing chunks of the schematic. For example, you won't just see a random resistor; you will see an `interface_i2c` block containing `["R6", "R7", "U1"]` and the nets `["IMU_SDA", "IMU_SCL"]`. 
    * *AI Usage:* Use this to teach the model design patterns. "When prompted for an IMU, generate this I2C block."
    * *Quality Check:* Always look at the `"is_clean_for_training"` boolean. If it is `false`, the AI pipeline should ignore this specific block to avoid learning from a schematic error.

* **`nets.json` (The Nervous System):**
    Read this to verify electrical truth. It proves that the KiCad labels actually touch the pins. 
    * *What to look for:* Ensure signal nets (like `sensor_1`) list both the MCU pin (e.g., `U2.GPIO4`) and the destination pin (e.g., `R1.A`). If a net has only one pin, it's a floating wire.

* **`components.json` & `bom.json` (The Physical Parts):**
    These files separate the layout from the purchasing data. Power symbols (`+3V3`, `GND`) are deliberately stripped from the BOM because you can't buy "ground."
    * *AI Usage:* The model uses this to choose the right footprint and verify if a requested component matches the project constraints.

* **`schematic.sdf` (The Canvas):**
    This is the spatial map. It contains X/Y coordinates.
    * *AI Usage:* The model generates this to tell the web renderer exactly where to place symbols on the screen so the final output looks like a human drew it, rather than a tangled mess.

* **`metadata.json` (The Health Inspector):**
    Always read this file when evaluating a new KiCad project. It contains an array of warnings. If the schematic has labels that barely miss their target pins, this file will catch it.