---
name: drawio-diagrams
description: Use when asked to create, edit, or version a diagram as a .drawio / draw.io / diagrams.net file (mxGraph XML) — architecture/infra schemas, flowcharts, network maps, org charts, sequence/ER-style boxes-and-arrows — and you want a hand-authored, openable, validated file rather than an image.
---

# Authoring draw.io diagrams (mxGraph XML)

Generate `.drawio` files directly as mxGraph XML. The format is plain XML that opens in
diagrams.net / the VS Code draw.io extension and stays fully editable — far better than an
exported image. This skill is the generalist house style; for cloud/infra schemas the
`cloud-infra-mcp` skill adds provider color codes and the explore→document workflow on top.

## Core model

A diagram is a flat list of `<mxCell>`. Every shape and connector is one cell with:
- a unique `id`, `parent="1"` (the default layer), and a `<mxGeometry>`
- `vertex="1"` for a box, OR `edge="1" source="…" target="…"` for a connector
- a `style` string of `key=value;` pairs

**Two non-obvious rules that cause most mistakes:**
1. Geometry is **absolute** (`x`,`y` from page top-left) unless a cell's `parent` is another
   vertex. Keep everything `parent="1"` with absolute coords — it's the simplest mental model.
   Group visually with `swimlane` containers that *overlap* their children; you don't need to
   reparent children for them to look "inside".
2. Text containing `<`, `>`, `&`, or line breaks must be HTML-escaped **inside the XML
   attribute**: use `&lt; &gt; &amp;` and `&lt;br&gt;` for a newline (with `html=1` in style).

## Minimal skeleton

Copy `template.drawio` in this directory and add cells inside `<root>`. The required wrapper:

```xml
<mxfile host="app.diagrams.net" version="24.0.0" type="device">
  <diagram id="my-diagram" name="My Diagram">
    <mxGraphModel dx="1200" dy="800" grid="1" gridSize="10" guides="1" arrows="1"
                  page="1" pageScale="1" pageWidth="1700" pageHeight="1100" math="0" shadow="0">
      <root>
        <mxCell id="0" />
        <mxCell id="1" parent="0" />
        <!-- your cells go here, all with parent="1" -->
      </root>
    </mxGraphModel>
  </diagram>
</mxfile>
```

`id="0"` and `id="1"` are mandatory boilerplate (root + default layer). Never reuse those ids.

## Building blocks (copy & adapt)

Box (rounded), with an escaped line break:
```xml
<mxCell id="api" value="API Gateway&lt;br&gt;:443" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#dae8fc;strokeColor=#6c8ebf;fontSize=12;" vertex="1" parent="1">
  <mxGeometry x="200" y="120" width="180" height="56" as="geometry" />
</mxCell>
```

Group container (draw it first / behind; size it to cover its members):
```xml
<mxCell id="zone" value="Production zone" style="swimlane;rounded=1;html=1;startSize=26;verticalAlign=top;align=center;fillColor=#f5f5f5;strokeColor=#666666;fontStyle=1;" vertex="1" parent="1">
  <mxGeometry x="160" y="80" width="600" height="280" as="geometry" />
</mxCell>
```

Connector (arrow) — geometry is just `relative="1"`, never x/y:
```xml
<mxCell id="e1" value="HTTPS" style="edgeStyle=orthogonalEdgeStyle;rounded=1;html=1;endArrow=block;strokeColor=#333333;fontSize=10;" edge="1" parent="1" source="api" target="db">
  <mxGeometry relative="1" as="geometry" />
</mxCell>
```

Standalone label (title / credit box), no border:
```xml
<mxCell id="title" value="System architecture · v1" style="text;html=1;strokeColor=none;fillColor=none;align=left;fontSize=22;fontStyle=1;" vertex="1" parent="1">
  <mxGeometry x="40" y="12" width="800" height="32" as="geometry" />
</mxCell>
```

## Style quick reference

| Want | style keys |
|---|---|
| Rounded box | `rounded=1;whiteSpace=wrap;html=1` |
| Plain rectangle | `whiteSpace=wrap;html=1` |
| Ellipse / circle | `ellipse;whiteSpace=wrap;html=1` |
| Cylinder (DB) | `shape=cylinder;whiteSpace=wrap;html=1` |
| Diamond (decision) | `rhombus;whiteSpace=wrap;html=1` |
| Group / zone | `swimlane;startSize=26;verticalAlign=top` |
| Text only, no box | `text;html=1;strokeColor=none;fillColor=none` |
| Dashed | add `dashed=1` |
| Bold / size | `fontStyle=1` (1=bold,2=italic) · `fontSize=N` |
| Arrow ends | `startArrow=…;endArrow=block|open|none` |
| Orthogonal routing | `edgeStyle=orthogonalEdgeStyle;rounded=1` |
| Force a side | `exitX=0.5;exitY=1;entryX=0.5;entryY=0` (0–1 along the box) |

Common fill/stroke pairs (draw.io defaults — readable together): blue `#dae8fc`/`#6c8ebf`,
green `#d5e8d4`/`#82b366`, red `#f8cecc`/`#b85450`, purple `#e1d5e7`/`#9673a6`,
orange `#ffe6cc`/`#d79b00`, grey `#f5f5f5`/`#666666`. Pick one hue per logical domain.

## House style: infrastructure & cloud diagrams

Follow this so every infra diagram comes out with the same look & feel. The goal is a
**top-down traffic flow** read like a story: where requests enter → which host they land on →
which container/resource serves them → what they depend on.

### Layout (top → bottom)
1. **Ingress at the top.** A cloud shape (`ellipse;shape=cloud`) labelled `Internet` (add the
   public DNS/wildcard under it). Requests flow downward from here.
2. **A "Sources externes" swimlane** top-left for things that are not hosts but feed the system
   (Git repos, container registries, package sources). Boxes here connect to resources with
   **dashed** edges (`build`, `pull image`).
3. **One outer swimlane per host / trust boundary** (a server, a VM, a cluster). Its title bar
   carries the host facts: name, IP, OS/kernel, CPU/RAM/disk, access method.
4. **Nested swimlanes inside a host** for logical groups — one per project / namespace /
   compose-project. Use a **dashed** swimlane for a docker-compose project box.
5. **Resource boxes** (apps, DBs, services) live inside their group swimlane.
6. **Title** top-left (`text`, `fontSize=16;fontStyle=1`), **source/credit box** bottom
   (small italic, light fill) — see below.

### Color = role (not decoration)
Assign color by what a thing *is*, consistently across the whole diagram:

| Role | fill / stroke | shape |
|---|---|---|
| Host zone — per cloud provider | provider hue, light fill: Scaleway `#f7f2fc`/`#4f0599`, generic server `#f0f7ff`/`#6c8ebf` | outer `swimlane` |
| Ingress / reverse-proxy (Traefik, nginx, LB) | orange `#ffe6cc`/`#d79b00` | rounded box |
| Application / web service | white `#ffffff` + group's stroke, or green `#d5e8d4`/`#82b366` | rounded box |
| Database / storage / model volume | blue `#dae8fc`/`#6c8ebf` | `shape=cylinder3` |
| Managed service / admin tool | purple `#e1d5e7`/`#9673a6` | rounded box |
| Project / namespace group | one hue per project (yellow `#fff2cc`, green `#d5e8d4`, red `#f8cecc`…) | inner `swimlane` |
| Empty / placeholder / unused | grey `#f5f5f5`/`#999999` + `dashed=1`, body text `(aucune ressource)` | inner `swimlane` |
| Internet / network cloud | blue `#dae8fc`/`#6c8ebf` | `ellipse;shape=cloud` |

### Resource box content recipe
Left-aligned, `align=left;spacingLeft=8;fontSize=11`, white fill with the group's stroke color.
Stack these lines (escaped `&lt;br&gt;`):
```
<b>name</b>            ← bold first line
image / tech           ← e.g. postgres:16-alpine, repo@branch
https://public.fqdn    ← the routable URL if exposed
port :NNNN · status    ← exposed port + running:healthy / running:unknown
<i>caveat</i>          ← italic note (healthcheck off, no FQDN, …)
```
Use **real values pulled live** — never `app-1`, `db`, `<host>`. Mark unknowns explicitly.

### Edges carry meaning
- **Traffic** (proxy → app): solid `edgeStyle=orthogonalEdgeStyle;rounded=1;endArrow=block`,
  proxy-orange `#d79b00`, label with the subdomain (`webui.`, `api.`).
- **Internal dependency** (app → db, webui → ollama): `endArrow=open`, label the link
  (`prisma`, `OLLAMA_BASE_URL http://ollama:11434`).
- **External feed** (registry/git → app): `dashed=1;endArrow=open`, label `pull image` / `build`.
- An edge's `source`/`target` is a cell **id**, not its label.

### Emphasis & warnings
Flag risks inline inside the box with red bold text, not a separate shape:
`&lt;font color=#b85450&gt;&lt;b&gt;⚠ exposé public sans auth&lt;/b&gt;&lt;/font&gt;`.

### Credit / source box (always include, bottom of page)
A bordered light box (`fillColor=#fbfbfb;strokeColor=#cccccc`, italic, `fontSize=10`) stating
**how the data was obtained + date + known caveats**, e.g.
`Source: live via MCP coolify (v4.1.2) + SSH — généré le YYYY-MM-DD. Caveats: …`.

### Multi-host & crossing edges
When two hosts share the same public DNS but are different machines, give **each host its own
`Internet` entry node** instead of one shared cloud with long wires crossing the page. Short,
local edges read far better than a web of long diagonals.

### Sizing rules that prevent overlap
- `swimlane` title bar height = `startSize`. **Bump it when the title is long or wraps**:
  short title `startSize=24`; a host title with full specs that wraps to 2–3 lines needs
  `startSize=40–46`, else the first child cell sits under the title. (Set `spacing=4` too.)
- Inner project swimlanes `startSize=24`; dashed compose-project box `startSize=26`.
- Lay boxes on the `gridSize=10` grid with ~30–40px gutters; verify `x+width` of a child stays
  inside its swimlane and children don't overlap.

## Workflow

1. Sketch the layout on a grid: pick `pageWidth`/`pageHeight`, assign each box an `x,y,w,h`.
   Leave ~40px gutters; keep `gridSize=10` multiples so things align.
2. Emit containers (swimlanes) first, then boxes, then edges (order only affects z-stacking;
   later cells render on top).
3. **Validate the XML before handing off** — a single unescaped `&` breaks the whole file:
   ```bash
   python3 -c "import xml.etree.ElementTree as ET; ET.parse('diagram.drawio'); print('XML OK')"
   ```
4. When revising, save a new version (`-v2`, …), bump `<diagram name>` + the title cell, and
   **extend `pageHeight`/`pageWidth`** to add content rather than shuffling existing coords.

## Common mistakes

- **Unescaped `&`/`<` in `value`** → file won't parse. Escape every one; validate.
- **Putting x/y on an edge** → use `<mxGeometry relative="1" as="geometry"/>` only.
- **Reusing id `0`/`1`** or duplicate ids → cells silently vanish or merge. Keep ids unique.
- **Overlapping boxes** because coords were eyeballed → lay out on the grid; check w/h sums.
- **Embedding huge base64 SVG icons inline** → fine but bloats the file and is unreadable in
  diffs; only do it when the icon genuinely adds meaning, and keep it on its own cell.
- Source/target of an edge must be an existing cell `id`, not its `value`.
