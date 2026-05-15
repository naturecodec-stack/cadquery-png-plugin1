# cadquery-png-plugin

This plugin converts CadQuery scripts (`.py`) into high-quality PNG images with accurate edge rendering and consistent shading, closely matching the original GenCAD dataset style.

---

## Installation

This plugin is not available on PyPI yet. You can install it directly from GitHub:

```bash
pip install git+https://github.com/naturecodec-stack/cadquery-png-plugin1.git
```

This installation will also install other required packages such as CadQuery if the environment does not already include them.

## Usage

Below is an example script for generating 8-view dataset images from CadQuery scripts.

```python
import os
import runpy
import cadquery as cq
import cadquery_png_plugin.plugin

# =========================
# CONFIG
# =========================

INPUT_ROOT = r"C:\Users\Desktop\inputfolder"

# OUTPUT image folder
OUTPUT_ROOT = r"C:\Users\Desktop\image_8views"

os.makedirs(OUTPUT_ROOT, exist_ok=True)

render_options = {
    "width": 400,
    "height": 400,
    "background_color": (1, 1, 1),
    "color_theme": "default",
    "view": "front-top-right",
    "zoom": 0.9,
    "show_edges": True,
    "edge_color": (0, 0, 0),
    "edge_width": 1.5
}

# ✅ Multiple angles (fix from your bug)
# ✅ Correct angles
angles = [0, 90, 180, 270]


def render_model(solid, parts, base_name, output_dir):

    for i, angle in enumerate(angles):

        # Rotate each part individually
        rotated_parts = [p.rotate((0, 0, 0), (0, 0, 1), angle) for p in parts]

        # Build assembly from separate parts (no union — edges preserved)
        assy_front = cq.Assembly()
        for j, part in enumerate(rotated_parts):
            assy_front.add(part, color=cq.Color(0.95, 0.95, 0.95), name=f"part_{j}")

        front_file = os.path.join(output_dir, f"{base_name}_front_{i}.png")
        assy_front.exportPNG(
            options={**render_options, "view": "front-top-right"},
            file_path=front_file
        )

        # Back views
        rotated_parts_back = [p.rotate((0, 0, 0), (0, 1, 0), 180) for p in rotated_parts]

        assy_back = cq.Assembly()
        for j, part in enumerate(rotated_parts_back):
            assy_back.add(part, color=cq.Color(0.95, 0.95, 0.95), name=f"part_{j}")

        back_file = os.path.join(output_dir, f"{base_name}_back_{i}.png")
        assy_back.exportPNG(
            options={**render_options, "view": "front-top-right"},
            file_path=back_file
        )

# =========================
# MAIN LOOP
# =========================

for root, dirs, files in os.walk(INPUT_ROOT):

    for file in files:

        if file.endswith(".py"):

            py_path = os.path.join(root, file)
            print("Processing:", py_path)
            try:
                # Run CadQuery script
                namespace = runpy.run_path(py_path)

                if "solid" in namespace:
                    solid = namespace["solid"]
                    # Use parts list if available, otherwise fall back to single solid
                    parts = namespace.get("parts", [solid])

                    base_name = os.path.splitext(file)[0]
                    rel_path = os.path.relpath(root, INPUT_ROOT)
                    output_dir = os.path.join(OUTPUT_ROOT, rel_path)
                    os.makedirs(output_dir, exist_ok=True)

                    render_model(solid, parts, base_name, output_dir)

                else:
                    print("❌ No 'solid' variable in:", file)

            except Exception as e:
                print("❌ Error in:", file)
                print(e)
```
## Running Tests

## Running Tests

1. Install the plugin and its development dependencies.
```
pip install -e .
pip install -e .[dev]
```

2. Run the tests
```
pytest 
```

## Acknowledgement

## Acknowledgement

This project is based on the original repository:
https://github.com/jmwright/cadquery-png-plugin

We acknowledge and appreciate the original authors for their contribution.

### Modifications
- Improved edge rendering using VTK feature edges
- Enhanced shading and lighting consistency
- Dataset-quality rendering output for machine learning workflows