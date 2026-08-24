---
name: diagram-generator
description: "Generate professional architecture diagrams using Draw.io XML (via MCP) or Mermaid. Covers AWS service icons, color contrast rules, edge visibility, label styling, and container nesting. Use for: architecture diagram, system diagram, Draw.io, Mermaid, flowchart, AWS diagram, service map, data flow diagram."
---

# Diagram Generator Skill

Generate production-quality architecture diagrams in **Draw.io XML** (opened via MCP) or **Mermaid** (.mmd files). This skill encodes battle-tested conventions, color rules, and pitfalls learned from iterative diagram creation across multi-service AWS architectures.

## When to Use

- Creating or updating architecture diagrams for AWS-based systems
- Generating Draw.io XML diagrams with AWS4 icons via MCP
- Creating Mermaid flowcharts with subgraph styling
- Fixing visual issues in existing diagrams (contrast, arrows, labels)
- Documenting multi-service data flows

## Procedure

### 1. Choose Format

| Format | Best For | Tool |
|--------|----------|------|
| **Draw.io XML** | Professional diagrams with AWS icons, presentations, detailed layouts | `mcp_drawio_open_drawio_xml` MCP tool |
| **Mermaid (.mmd)** | Quick diagrams in Markdown, version-controlled, CI-rendered | File output, rendered via Mermaid.js |

### 2. Draw.io XML: Structure and Rules

#### 2.1 Basic XML Structure
```xml
<mxGraphModel dx="2074" dy="1156" grid="1" gridSize="10" guides="1" tooltips="1" connect="1" arrows="1" fold="1" page="0" pageScale="1" pageWidth="3300" pageHeight="2400" math="0" shadow="0">
  <root>
    <mxCell id="0"/>
    <mxCell id="1" parent="0"/>
    <!-- All elements here -->
  </root>
</mxGraphModel>
```

#### 2.2 Container Groups (Service Boundaries)
Use rounded rectangles as containers with `container=1;collapsible=0`:
```xml
<mxCell id="group_id" value="Group Label" style="rounded=1;whiteSpace=wrap;html=1;fillColor=#1565C0;fontColor=#FFFFFF;strokeColor=#0D47A1;fontSize=14;fontStyle=1;verticalAlign=top;spacingTop=5;container=1;collapsible=0;arcSize=8;" vertex="1" parent="1">
  <mxGeometry x="20" y="270" width="900" height="580" as="geometry"/>
</mxCell>
```

Nest elements inside containers using `parent="group_id"`.

Sub-containers (layers within a service) use slightly lighter fill:
```xml
<mxCell id="sub_id" value="Sub Layer" style="rounded=1;...fillColor=#1E88E5;strokeColor=#1565C0;...container=1;collapsible=0;arcSize=6;" vertex="1" parent="group_id">
```

#### 2.3 AWS4 Icon Shapes
Use `shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.<icon_name>` for AWS service icons.

**Common AWS4 icon mappings:**
| Service | Icon Shape | Fill Color |
|---------|-----------|------------|
| API Gateway | `mxgraph.aws4.api_gateway` | `#E7157B` |
| Lambda | `mxgraph.aws4.lambda` | `#D05C17` |
| S3 | `mxgraph.aws4.s3` | `#3F8624` |
| SQS | `mxgraph.aws4.sqs` | `#E7157B` |
| SNS | `mxgraph.aws4.sns` | `#E7157B` |
| DynamoDB | `mxgraph.aws4.dynamodb` | `#C925D1` |
| Redshift | `mxgraph.aws4.redshift` | `#8C4FFF` |
| Glue | `mxgraph.aws4.glue` | `#8C4FFF` |
| EventBridge | `mxgraph.aws4.eventbridge` | `#E7157B` |
| ECS/Fargate | `mxgraph.aws4.ecs` | `#D05C17` |
| ALB | `mxgraph.aws4.application_load_balancer` | `#8C4FFF` |
| OpenSearch | `mxgraph.aws4.opensearch_service` | `#E7157B` |
| SageMaker/Bedrock | `mxgraph.aws4.sagemaker` | `#01A88D` |

**Non-AWS service fallback icons:**
| Service Type | Icon Shape |
|-------------|-----------|
| External apps (Contentful, Snowflake, etc.) | `mxgraph.aws4.traditional_server` |
| Tools/IDEs (VS Code, Claude, consoles) | `mxgraph.aws4.management_console` |
| People/Customers | `mxgraph.aws4.users` |

Icon element style pattern:
```xml
style="shape=mxgraph.aws4.resourceIcon;resIcon=mxgraph.aws4.lambda;labelBackgroundColor=none;sketch=0;fillColor=#D05C17;strokeColor=none;fontColor=#FFFFFF;fontSize=10;align=center;verticalLabelPosition=bottom;verticalAlign=top;whiteSpace=wrap;html=1;"
```

#### 2.4 Icon Labels
- Primary label: `<b>Service Name</b>` in white (`fontColor=#FFFFFF`)
- Secondary details: Use tinted color matching the parent group
  ```html
  <font color="#BBDEFB">Details here</font>  <!-- Blue group tint -->
  <font color="#E1BEE7">Details here</font>  <!-- Purple group tint -->
  <font color="#C8E6C9">Details here</font>  <!-- Green group tint -->
  <font color="#FFF3E0">Details here</font>  <!-- Orange group tint -->
  <font color="#B2EBF2">Details here</font>  <!-- Teal group tint -->
  <font color="#B0BEC5">Details here</font>  <!-- Gray group tint -->
  ```

### 3. CRITICAL: Color Contrast Rules

These rules prevent the most common visual defects. **Violation causes invisible elements.**

#### Rule 1: Edge stroke color MUST contrast with container fill
**NEVER** set `strokeColor` on an edge to the same value as the `fillColor` of any container it passes through.

| Container Fill | WRONG Edge Stroke | CORRECT Edge Stroke |
|---------------|-------------------|---------------------|
| `#66BB6A` (green) | `#66BB6A` | `#1B5E20` (dark green) |
| `#00838F` (teal) | `#00838F` | `#004D40` (dark teal) |
| `#1E88E5` (blue) | `#1E88E5` | `#0D47A1` (dark blue) |
| `#8E24AA` (purple) | `#8E24AA` | `#4A148C` (dark purple) |

**General pattern:** For internal edges within a container, use a color **2-3 shades darker** than the container fill from the Material Design palette.

#### Rule 2: Edge labels use black text with no background
```
fontSize=9;fontColor=#000000;fontStyle=1;labelBackgroundColor=none;
```
Do NOT use `labelBackgroundColor=#FFFFFF` — it creates visual clutter.

#### Rule 3: Icon labels on dark containers must be white
Always set `fontColor=#FFFFFF` on icons inside colored containers.

#### Rule 4: Cross-service edges use the source service color
Edges leaving a service group should use that group's primary/dark color for `strokeColor`.

### 4. Edge (Arrow) Rules

#### 4.1 Every edge mxCell MUST have geometry child
```xml
<mxCell id="e1" value="Label" style="..." edge="1" source="src_id" target="tgt_id" parent="1">
  <mxGeometry relative="1" as="geometry"/>
</mxCell>
```
**Omitting `<mxGeometry>` causes rendering errors.**

#### 4.2 Edge style pattern
```
edgeStyle=orthogonalEdgeStyle;rounded=1;orthogonalLoop=1;jettySize=auto;html=1;strokeColor=#COLOR;strokeWidth=2;
```
Use `orthogonalEdgeStyle` for clean right-angle connectors.

#### 4.3 Dashed edges for secondary/reverse flows
```
dashed=1;strokeWidth=1;strokeColor=#B0BEC5;
```

#### 4.4 Edge crossing visibility
Add `jumpStyle=arc;jumpSize=10;` to **every** edge style string. This renders a small arc wherever two edges cross, making the diagram readable even with overlapping routes.

#### 4.5 Cross-group edge routing
Edges that span multiple service groups must **never** pass directly over icons or labels in intermediate containers. Use explicit waypoints to route them through gaps between groups or along the diagram's outer margin:

```xml
<mxCell id="e_cross" style="edgeStyle=orthogonalEdgeStyle;...jumpStyle=arc;jumpSize=10;" edge="1" source="src" target="tgt" parent="1">
  <mxGeometry relative="1" as="geometry">
    <Array as="points">
      <mxPoint x="940" y="200"/>   <!-- route through gap between groups -->
      <mxPoint x="940" y="700"/>
    </Array>
  </mxGeometry>
</mxCell>
```

**Routing strategies:**
- **Gap corridor**: Route through the gap between adjacent groups (e.g., x between CIS right edge and xd-jes left edge)
- **Outer margin**: Route along the right or left margin of the diagram, outside all containers
- **Stagger x-coordinates**: When multiple edges share a corridor, offset each by 5-10 px to prevent overlap (e.g., x=935, x=940, x=945)

### 5. Color Palette: Service Group Reference

Consistent color scheme for multi-service architectures:

| Group | Fill | Stroke | Tint (secondary text) | Edge Color |
|-------|------|--------|----------------------|------------|
| External Sources | `#455A64` | `#37474F` | `#B0BEC5` | `#78909C` |
| Service A (primary) | `#1565C0` | `#0D47A1` | `#BBDEFB` | `#0D47A1` |
| Service B (AI/ML) | `#6A1B9A` | `#4A148C` | `#E1BEE7` | `#4A148C` |
| Service C (data) | `#2E7D32` | `#1B5E20` | `#C8E6C9` | `#1B5E20` |
| Storage (S3) | `#FF8F00` | `#E65100` | `#FFF3E0` | `#E65100` |
| Consumers | `#00838F` | `#006064` | `#B2EBF2` | `#004D40` |

### 6. Mermaid Flowchart Rules

#### 6.1 Use `<br/>` not `\n` for line breaks
Mermaid flowcharts do NOT support `\n`. Always use `<br/>`:
```
A["Line 1<br/>Line 2"]
```

#### 6.2 NEVER use `style SubgraphID`
Applying `style` to a subgraph ID creates phantom/blank boxes. Instead:
```mermaid
classDef blueStyle fill:#1565C0,color:#fff
class NodeA,NodeB blueStyle
```
Apply `classDef` + `class` to **individual node IDs**, never to subgraph IDs.

#### 6.3 Use dark `primaryTextColor` in theme
```
%%{init: {'theme': 'base', 'themeVariables': {'primaryTextColor': '#1a1a1a'}}}%%
```
Light text colors (e.g., `#fff`) become invisible on light backgrounds.

#### 6.4 Subgraph direction
Use `direction TB` or `direction LR` inside subgraphs for layout control.

### 7. Workflow Checklist

Before finalizing any diagram:

- [ ] Every edge `strokeColor` contrasts with the fill of containers it passes through
- [ ] Every edge `<mxCell>` has `<mxGeometry relative="1" as="geometry"/>` child
- [ ] Icon labels use `fontColor=#FFFFFF` on dark containers
- [ ] Edge labels use `fontColor=#000000;fontStyle=1;labelBackgroundColor=none`
- [ ] Cross-service edges go through `parent="1"` (root), not nested parents
- [ ] Container elements have `container=1;collapsible=0`
- [ ] Secondary text uses tinted color matching the parent group palette
- [ ] Non-AWS services use appropriate fallback icons (traditional_server, management_console, users)
- [ ] Every application or resource has an icon defined; if no exact match exists, assign a reasonable fallback icon
- [ ] All arrows connecting resources are logically correct and clearly visible against their background containers
- [ ] Cross-group edges are routed around containers using waypoints — never directly over icons or labels
- [ ] All edges include `jumpStyle=arc;jumpSize=10` for crossing visibility

### 8. Opening via MCP

Load the `mcp_drawio_open_drawio_xml` tool first, then call:
```
mcp_drawio_open_drawio_xml(content="<mxGraphModel>...</mxGraphModel>")
```
The diagram opens in the browser for interactive editing and export.

### 9. Saving Diagrams

- **Draw.io**: Save the XML content to a `.drawio` file for version control
- **Mermaid**: Save to `.mmd` file, render via Mermaid.js or GitHub native rendering
- **Export**: Draw.io supports PNG, SVG, PDF export from the browser UI
