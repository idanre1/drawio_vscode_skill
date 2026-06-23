---
name: drawio-vscode-skill
version: 0.1.0
description: "Use when creating or editing Draw.io diagrams in an active VS Code workspace with the Draw.io VS Code extension. Builds on the bundled drawio-skill submodule for diagram XML, shapes, styles, and validation guidance, but edits .drawio files directly and uses VS Code for rendering instead of the draw.io desktop CLI."
compatibility: "Requires an active VS Code workspace with a Draw.io-compatible extension installed, such as hediet.vscode-drawio. Does not require the draw.io desktop CLI."
platforms: [macos, linux, windows]
metadata: {"related_skills":["drawio-skill"],"requires_vscode":true,"requires_desktop_cli":false}
---

# Draw.io for VS Code

## Purpose

This skill creates and edits `.drawio` diagrams as XML files that the VS Code Draw.io extension can render directly.

It is intentionally close to the bundled `drawio-skill`, but the runtime assumption is different:

- Use `./drawio-skill/skills/drawio-skill/SKILL.md` as the canonical reference for Draw.io XML structure, styles, shape resources, diagram planning, validation scripts, and troubleshooting.
- Do not require, install, or invoke the draw.io desktop CLI.
- Do not perform PNG/SVG/PDF export steps unless the user explicitly asks for export and an available non-CLI route exists.
- Prefer source `.drawio` files as the deliverable because VS Code can open and preview them.

## When to Use

Use this skill when the user asks to create or edit:

- `.drawio` or `.dio` files in VS Code
- Flowcharts, architecture diagrams, ER diagrams, UML diagrams, sequence diagrams, network diagrams, mind maps, or other Draw.io diagrams
- Existing Draw.io XML where the user wants changes visible in the VS Code Draw.io extension
- Diagrams that should avoid the draw.io desktop CLI

Use the bundled `drawio-skill` directly when the user explicitly wants CLI exports to PNG, SVG, PDF, or JPG and the draw.io desktop CLI is available.

## Required Context

Before editing or creating a diagram:

1. Read `./drawio-skill/skills/drawio-skill/SKILL.md` for the current Draw.io XML guidance.
2. If the task needs specific stock shapes, read or use the bundled Draw.io skill resources under `./drawio-skill/skills/drawio-skill/references/` and `./drawio-skill/skills/drawio-skill/scripts/`.
3. Inspect the target `.drawio` file before editing. Preserve its existing XML encoding style when practical.
4. If the user named a file that does not exist, search nearby before creating a new one; small typos in diagram filenames are common.

## Photo or Whiteboard Conversion

When converting a photo, screenshot, or whiteboard sketch into `.drawio`, first choose the conversion mode.

- **Reproduction mode**: Use for clean screenshots, existing diagrams, UI captures, block diagrams, or cases where the user expects a close visual match.
- **Interpretation mode**: Use for whiteboards, rough sketches, photos with glare, crossed-out marks, ambiguous handwriting, informal brainstorming, or cases where a professional figure is more useful than a literal trace.

In reproduction mode:

1. Preserve the visible layout, relative spacing, labels, colors, and connector directions as closely as practical.
2. Use direct coordinates and simple Draw.io primitives to mirror the source.
3. Keep labels exact when they are legible.

In interpretation mode:

1. Extract the likely semantic components and relationships before drawing.
2. Preserve important labels, but normalize unclear handwriting into concise engineering terms.
3. Omit incidental marker strokes, glare, duplicated arrows, erased fragments, and low-confidence details.
4. Use a professional palette instead of copying whiteboard marker colors exactly.
5. Group related elements into containers if the sketch implies a boundary.
6. Prefer clean orthogonal routing over hand-drawn curved arrows unless curvature carries meaning.
7. Mention normalized or uncertain labels in the final response.

If a handwritten label is unclear:

- Use the nearest likely engineering term only when context is strong.
- Preserve exact spelling only when it is legible.
- Do not invent fine protocol details from ambiguous marks.
- In the final response, list any meaningful label normalization.

Before writing the diagram, state the inferred components and flows in one short working note.

## VS Code Extension Workflow

### Create or Edit XML

For simple diagrams or targeted edits, modify the `.drawio` XML directly:

- Keep required root cells `id="0"` and `id="1"`.
- Add shapes as `mxCell` vertices with `parent="1"`, `vertex="1"`, and an `mxGeometry` child.
- Add connectors as `mxCell` edges with `edge="1"`, `parent="1"`, `source`, `target`, and relative `mxGeometry`.
- Use `html=1` in shape and edge styles.
- Escape XML attribute values: `&amp;`, `&lt;`, `&gt;`, and `&quot;`.
- Use `&#xa;` for line breaks inside labels.
- Prefer stable ids that do not collide with existing cells.
- Never create or edit the same `.drawio` file from parallel tool calls. Parallel reads are fine; writes to a target diagram must be serial.
- If replacing a malformed generated diagram, delete it first, then recreate it in a separate step.

Example vertex:

```xml
<mxCell id="3" value="example" style="whiteSpace=wrap;html=1;" vertex="1" parent="1">
  <mxGeometry x="430" y="280" width="120" height="60" as="geometry"/>
</mxCell>
```

Example connector:

```xml
<mxCell id="4" value="" style="edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;endArrow=classic;endFill=1;" edge="1" parent="1" source="2" target="3">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>
```

### Validate Without Desktop CLI

After editing:

1. Check the file contains exactly one XML declaration, one `<mxfile`, and one `</mxfile>`.
2. Parse the XML with a local parser. In this environment, use `uv run python -c 'import xml.etree.ElementTree as ET; ET.parse("path/to/file.drawio")'` when Python is needed. If you can't find `uv`, try `/home/pure_scratch/ovl2/iregev/apps/uv/uv`.
3. Verify there are no duplicate `mxCell` ids.
4. Verify the expected labels are present.
5. If the bundled validator is applicable, run it from `./drawio-skill/skills/drawio-skill/scripts/validate.py` without invoking draw.io desktop.
6. Treat validator overlap warnings as acceptable only when they are intentional, such as labels inside containers or labels placed on top of arrows.
7. Ask the user to open or preview the `.drawio` file in the VS Code Draw.io extension for visual review when a rendered image is required.

Reusable validation pattern:

```bash
uv run python - <<'PY'
from pathlib import Path
import xml.etree.ElementTree as ET

path = Path("diagram.drawio")
text = path.read_text()
assert text.count("<?xml") == 1
assert text.count("<mxfile") == 1
assert text.count("</mxfile>") == 1
root = ET.parse(path).getroot()
ids = [cell.attrib["id"] for cell in root.iter("mxCell") if "id" in cell.attrib]
assert len(ids) == len(set(ids))
PY
```

### Preview and Review

The source `.drawio` file is the review artifact.

- Do not export a preview PNG through the desktop CLI.
- If an active VS Code session can open the file, rely on the Draw.io extension preview.
- For visual corrections requested after review, make minimal XML edits in place.
- Preserve layout changes the user may have made manually in the extension.

## Relationship to the Bundled Draw.io Skill

This repository includes `drawio-skill` as a git submodule so the full upstream Draw.io skill remains available beside this VS Code-focused wrapper.

Load and follow the upstream guidance for:

- Draw.io XML skeletons
- Shape syntax and style presets
- Diagram-type references
- Auto-layout helpers
- Structural validation
- Troubleshooting malformed diagrams

Override the upstream workflow for:

- Dependency checks for `drawio` or `draw.io`
- Desktop CLI preview/export steps
- CLI-generated PNG/SVG/PDF/JPG review loops
- Desktop app launch instructions

In this skill, active VS Code plus its Draw.io extension is the rendering environment.