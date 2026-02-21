---
name: anywidget
language: python
description: Create custom anywidget components in marimo notebooks using vanilla JavaScript and CSS.
---

# anywidget in Marimo

Create custom anywidget components in marimo notebooks using vanilla JavaScript and CSS.

## JavaScript

Define a `render` function that takes `{ model, el }` as parameters and export it as default. Use:
- `model.get()` / `model.set()` / `model.save_changes()` for state management
- `model.on("change:traitname", callback)` for reactive updates

## CSS

Include `_css` styling that remains minimal. Accommodate light and dark modes:
```css
@media (prefers-color-scheme: dark) { ... }
```

## Display in Marimo

Wrap widgets with marimo's anywidget wrapper:
```python
widget = mo.ui.anywidget(OriginalAnywidget())
```

Access state values through `widget.value` (returns a dictionary).

## Philosophy

Dumber is better. Prefer obvious, direct code over clever abstractions. For complex widgets with substantial JS/CSS, reference external files via `pathlib` rather than embedding inline.

Keep examples minimal and focused on core functionality.

## Complete Widget Example: ICU Stay Timeline

A minimal widget that visualizes ICU stay segments as a horizontal timeline.

### Python Class

```python
import anywidget
import traitlets
import json

class ICUTimelineWidget(anywidget.AnyWidget):
    """Displays ICU stay segments as a horizontal timeline bar chart."""

    _esm = """
    function render({ model, el }) {
      const canvas = document.createElement("canvas");
      canvas.width = 600;
      canvas.height = 120;
      canvas.style.width = "100%";
      el.appendChild(canvas);

      const colors = {
        icu: "#e74c3c",
        ward: "#3498db",
        ed: "#f39c12",
        stepdown: "#9b59b6",
        other: "#95a5a6"
      };

      function draw() {
        const ctx = canvas.getContext("2d");
        const segments = JSON.parse(model.get("segments"));
        const totalHours = model.get("total_hours") || 168;
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Draw axis
        ctx.strokeStyle = "#666";
        ctx.beginPath();
        ctx.moveTo(40, 80);
        ctx.lineTo(580, 80);
        ctx.stroke();

        // Draw segments
        segments.forEach(seg => {
          const x = 40 + (seg.start_hour / totalHours) * 540;
          const w = Math.max(2, ((seg.end_hour - seg.start_hour) / totalHours) * 540);
          ctx.fillStyle = colors[seg.location] || colors.other;
          ctx.fillRect(x, 30, w, 45);
        });

        // Legend
        let lx = 40;
        Object.entries(colors).forEach(([label, color]) => {
          ctx.fillStyle = color;
          ctx.fillRect(lx, 95, 12, 12);
          ctx.fillStyle = "#333";
          ctx.font = "11px Arial";
          ctx.fillText(label, lx + 16, 106);
          lx += 80;
        });
      }

      draw();
      model.on("change:segments", draw);
      model.on("change:total_hours", draw);
    }
    export default { render };
    """;

    _css = """
    :host { display: block; padding: 8px; }
    canvas { border: 1px solid #ddd; border-radius: 4px; }
    @media (prefers-color-scheme: dark) {
      canvas { border-color: #444; }
    }
    """;

    segments = traitlets.Unicode("[]").tag(sync=True)
    total_hours = traitlets.Int(168).tag(sync=True)
```

### Usage in Marimo

```python
import marimo as mo
import json

# Create sample ICU stay data
stay_data = [
    {"location": "ed", "start_hour": 0, "end_hour": 4},
    {"location": "icu", "start_hour": 4, "end_hour": 72},
    {"location": "stepdown", "start_hour": 72, "end_hour": 96},
    {"location": "ward", "start_hour": 96, "end_hour": 144},
]

widget = mo.ui.anywidget(ICUTimelineWidget(
    segments=json.dumps(stay_data),
    total_hours=168
))
widget
```

```python
# Access state in downstream cells
selected = widget.value  # dict of all synced traitlets
mo.md(f"Timeline shows {widget.widget.total_hours} hours")
```
