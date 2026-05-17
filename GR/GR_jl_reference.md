# GR.jl High-Level Plot API Reference

Source: `src/jlgr.jl` in the GR.jl package. All plotting goes through the `GR.plot` (and related) functions in the `GR.jlgr` submodule, which is re-exported from the top-level `GR` module.

---

## Plot Functions

| Function | Kind | Notes |
|---|---|---|
| `plot(args...; kv...)` | `:line` | Main 2D line/marker plot |
| `oplot(args...; kv...)` | `:line` | Overlays onto existing plot (sets `ax=true`) |
| `stairs(args...; kv...)` | `:stairs` | Step/staircase plot |
| `scatter(args...; kv...)` | `:scatter` | Scatter plot |
| `stem(args...; kv...)` | `:stem` | Stem plot |
| `barplot(labels, heights; kv...)` | `:bar` | Bar chart |
| `histogram(x; kv...)` | `:hist` | Histogram |
| `polarhistogram(x; kv...)` | `:polarhist` | Polar histogram |
| `contour(x,y,z; kv...)` | `:contour` | Contour lines |
| `contourf(x,y,z; kv...)` | `:contourf` | Filled contours |
| `hexbin(x,y; kv...)` | `:hexbin` | Hexagonal bin density |
| `heatmap(z; kv...)` | `:heatmap` | 2D color grid |
| `wireframe(x,y,z; kv...)` | `:wireframe` | 3D wireframe |
| `surface(x,y,z; kv...)` | `:surface` | 3D surface |
| `volume(V; kv...)` | `:volume` | 3D volume rendering |
| `plot3(args...; kv...)` | `:plot3` | 3D line plot |
| `scatter3(args...; kv...)` | `:scatter3` | 3D scatter |
| `isosurface(V; kv...)` | `:isosurface` | 3D isosurface |
| `polar(θ, ρ; kv...)` | `:polar` | Polar line plot |
| `trisurf(x,y,z; kv...)` | `:trisurf` | Triangulated surface |
| `tricont(x,y,z; kv...)` | `:tricont` | Triangulated contour |
| `shade(args...; kv...)` | `:shade` | Density plot for large datasets |
| `imshow(img; kv...)` | `:imshow` | Image display |

### Argument Parsing for `plot`

- `plot(y)` — uses indices 1..n as x
- `plot(x, y)` — basic series
- `plot(x, y, fmt)` — with format string (see below)
- `plot(x, y, c)` — with color-data vector (last non-string arg, only when it's the sole remaining arg)
- `plot(x1,y1, x2,y2, ...)` — multiple series; each may have its own fmt string
- `plot(x, Y)` — matrix Y: each column is a series

---

## Format String (`fmt`)

Passed to the C-level `GR.uselinespec(spec)`. Combines color, line style, and marker type in one string. Same syntax as matplotlib.

### Colors (single char)
| Char | Color |
|---|---|
| `b` | blue |
| `r` | red |
| `g` | green |
| `k` | black |
| `c` | cyan |
| `m` | magenta |
| `y` | yellow |
| `w` | white |
| `1`–`8` | auto-cycle colors 1–8 (see below) |

Using a digit selects the same color that would be used for that series position in an unstyled multi-series plot. Combines freely with line style and marker: `"2--"`, `"5o"`, `"3-+"`, etc.

### Line Styles
| Char | Style |
|---|---|
| `-` | solid (default) |
| `--` | dashed |
| `:` | dotted |
| `-.` | dash-dot |
| ` ` (space) | no line |

### Markers
| Char | Marker |
|---|---|
| `+` | plus |
| `o` | circle |
| `*` | asterisk |
| `.` | dot |
| `x` | cross |
| `s` | square |
| `d` | diamond |
| `^` | triangle up |
| `v` | triangle down |
| `<` | triangle left |
| `>` | triangle right |
| `p` | pentagon |
| `h` | hexagon |

Example: `"r--o"` = red dashed line with circle markers.

---

## Color Specification

### Option 1: fmt string (8 colors only)
```julia
plot(x1,y1,"r", x2,y2,"b--", x3,y3,"gd")
```

### Option 2: `c` vector — per-point colormap coloring
```julia
plot(x, y, c_vals)   # c_vals is Float vector, same length as x/y
```
- Colors each point along the line via the current colormap
- Normalized as `(c .- min(c)) ./ (max(c) - min(c))`, then mapped to colormap indices 1000–1255
- Only triggered when the third arg is the **sole remaining argument** (non-string)
- Set colormap with `GR.setcolormap(GR.COLORMAP_VIRIDIS)` etc. (48 built-in colormaps)

### Option 3: Default auto-cycling
- Driven by C `gr_uselinespec`; uses `color = 980 + predef_colors[i]` cycling through 20 entries
- First 8 color indices: `(989, 982, 980, 981, 996, 983, 995, 988)`

| Step | Index | Hex | Color |
|---|---|---|---|
| 1 | 989 | `#00538a` | dark blue |
| 2 | 982 | `#ff6800` | orange |
| 3 | 980 | `#ffb300` | amber |
| 4 | 981 | `#803e75` | purple |
| 5 | 996 | `#93aa00` | olive green |
| 6 | 983 | `#5aa3c0` | steel blue |
| 7 | 995 | `#7f180d` | dark red |
| 8 | 988 | `#f6768e` | pink |

- Query actual RGB: `GR.inqcolor(idx)` returns packed `0x00RRGGBB`

### Option 4: Color schemes
```julia
usecolorscheme(n)  # n = 1, 2, 3, or 4
```
| n | Name | Palette |
|---|---|---|
| 1 | Classic | white, black, red, green, blue, cyan, yellow, magenta |
| 2 | Atom (dark) | muted dark-theme colors |
| 3 | Solarized light | solarized palette |
| 4 | Solarized dark | solarized dark palette |

### Option 5: Pre-load custom cycling colors (before `plot`)
```julia
GR.setcolorrep(989, r, g, b)  # 1st series color
GR.setcolorrep(982, r, g, b)  # 2nd series color
GR.setcolorrep(980, r, g, b)  # 3rd series color
# ... indices: (989, 982, 980, 981, 996, 983, 995, 988)
```
r, g, b are floats in [0.0, 1.0].

### isosurface-only: arbitrary RGB
```julia
isosurface(V, color=(0.8, 0.2, 0.1))
```

---

## All Keyword Arguments (`kw_args`)

### Figure / Layout

| Kwarg | Default | Applies to | Description |
|---|---|---|---|
| `size` | `(600, 450)` | all | Figure size in pixels `(width, height)` |
| `figsize` | — | all | Physical size in inches `(w, h)`; overrides `size` |
| `dpi` | screen DPI | all | DPI for size calculations; >200 scales by `dpi/100` |
| `subplot` | `[0,1,0,1]` | all | Manual viewport in normalized coords; prefer `subplot()` function |
| `keepaspect` | `false` | all | Lock aspect ratio (GRM mode) |
| `backgroundcolor` | — | all | GR color index for the plot background fill |

### Appearance

| Kwarg | Default | Applies to | Description |
|---|---|---|---|
| `alpha` | — | all | Transparency [0.0–1.0] for all series in the call |
| `font` | `"CMUSerif-Math"` | all | Font name (see `GR.jlgr.fonts` dict for valid names) |
| `fontscale` | `1.0` | all | Scale factor applied to the auto-computed font size; `1.0` = default, `2.0` = double. Plot margins adjust automatically. |
| `linewidth` | `1` | line, stairs, stem | Line width scale factor |
| `markersize` | `1` | line (+markers), scatter | Marker size scale factor |
| `borderwidth` | `1` | line (+markers) | Marker border width |
| `grid` | `true` | 2D/3D | Show/hide grid lines |
| `colormap` | `COLORMAP_VIRIDIS` | heatmap, contour, surface, scatter3... | Colormap index; use `GR.COLORMAP_*` constants |

### Axes

| Kwarg | Default | Applies to | Description |
|---|---|---|---|
| `xlabel` | — | all | X-axis label string (supports LaTeX via textext) |
| `ylabel` | — | all | Y-axis label string |
| `zlabel` | — | 3D | Z-axis label string |
| `title` | — | all | Plot title string |
| `xlim` | auto | all | `(xmin, xmax)` axis range; `nothing` in slot = use data |
| `ylim` | auto | all | `(ymin, ymax)` |
| `zlim` | auto | 3D | `(zmin, zmax)` |
| `clim` | auto | scatter3, heatmap... | `(cmin, cmax)` for color data range |
| `xlog` | `false` | all | Log₁₀ scale on x-axis |
| `ylog` | `false` | all | Log₁₀ scale on y-axis |
| `zlog` | `false` | 3D | Log₁₀ scale on z-axis |
| `xflip` | `false` | all | Reverse x-axis direction |
| `yflip` | `false` | all | Reverse y-axis direction |
| `zflip` | `false` | all | Reverse z-axis direction |
| `scale` | `0` | all | Raw bitmask for `GR.setscale`; OR of `GR.OPTION_*` constants. Use for log2/ln scaling not covered by convenience flags. |

**`GR.OPTION_*` constants for `:scale`:**
```
OPTION_X_LOG=1, OPTION_Y_LOG=2, OPTION_Z_LOG=4
OPTION_FLIP_X=8, OPTION_FLIP_Y=16, OPTION_FLIP_Z=32
OPTION_X_LOG2=64, OPTION_Y_LOG2=128, OPTION_Z_LOG2=256
OPTION_X_LN=512, OPTION_Y_LN=1024, OPTION_Z_LN=2048
```

### Legend

| Kwarg | Default | Description |
|---|---|---|
| `labels` | — | `["label1", "label2", ...]` — enables legend for line/stairs/scatter/stem |
| `location` | `1` | Legend position (1=upper right, 2=upper left, 3=lower left, 4=lower right, 11–13=outside right) |
| `nlabels` | `length(labels)` | Maximum number of series entries shown in the legend. Useful when plotting many series but only labelling the first few. |

### Contour / Heatmap

| Kwarg | Default | Applies to | Description |
|---|---|---|---|
| `levels` | 20 (contour), 256 (heatmap) | contour, contourf, heatmap | Int = number of levels; Vector = exact level values |
| `clabels` | `false` | contour, contourf | Draw value labels on contour lines |
| `clines` | `true` | contourf | Draw contour lines on top of filled regions |

### 3D Plots

| Kwarg | Default | Applies to | Description |
|---|---|---|---|
| `rotation` | `40` | 3D plots | Azimuthal rotation in degrees |
| `tilt` | `60` | 3D plots | Viewing elevation in degrees (0=side, 90=top) |
| `accelerate` | `true` | surface | Use OpenGL-accelerated gr3 renderer; `false` = software |

### Volume

| Kwarg | Default | Description |
|---|---|---|
| `algorithm` | `0` | Volume rendering: 0=emission, 1=absorption, 2=MIP |

### Isosurface

| Kwarg | Default | Description |
|---|---|---|
| `isovalue` | `0.5` | Threshold for isosurface extraction |
| `color` | `(0.0, 0.5, 0.8)` | RGB float tuple for isosurface color |

### Bar Plot

| Kwarg | Default | Description |
|---|---|---|
| `barwidth` | `0.8` | Bar width as fraction of inter-bar spacing |
| `baseline` | `0.0` | Y-value at bar base |

### Staircase

| Kwarg | Default | Description |
|---|---|---|
| `where` | `"mid"` | Step placement: `"pre"`, `"mid"`, or `"post"` |

### Shade (density plot)

| Kwarg | Default | Description |
|---|---|---|
| `xform` | `5` | Density transform: 0=boolean, 1=linear, 2=log, 3=loglog, 4=cubic, 5=equalized |

### Polar

| Kwarg | Default | Description |
|---|---|---|
| `theta_direction` | `1` | `+1`=CCW (math), `-1`=CW (compass) |
| `theta_zero_location` | `"E"` | Where θ=0 is: `"E"`, `"N"`, `"W"`, `"S"` |

### Histogram

| Kwarg | Default | Description |
|---|---|---|
| `nbins` | auto | Number of histogram bins |

---

## Kwargs NOT in `kw_args` (use setter functions)

These must be called as standalone functions — passing them to `plot(...)` will error.

```julia
xticks(minor, major=1)          # tick interval; major = minor ticks per major tick
yticks(minor, major=1)
zticks(minor, major=1)
xticklabels(f_or_array)         # f: value -> string, or array of strings at x=1,2,...
yticklabels(f_or_array)
drawgrid(flag)                  # same effect as grid= kwarg
```

---

## Font Size

Font size is computed automatically as `max(0.018 * diag, 0.012)` (2D) or `max(0.024 * diag, 0.012)` (3D) where `diag` is the viewport diagonal.

**Recommended: use `fontscale`** — a kwarg that multiplies the auto-computed size and adjusts margins automatically:

```julia
plot(x, y, fontscale=1.5)   # 50% larger text, margins expand to fit
plot(x, y, fontscale=0.8)   # slightly smaller
```

**Alternative: make the figure larger** (increases physical resolution without changing the relative font size):

```julia
plot(x, y, size=(1200, 900))
```

**Low-level override:** `charheight=h` sets the absolute char height directly (a fraction of viewport diagonal, e.g. `0.03`). Stacks with `fontscale` if both are provided. Can also be called as `GR.setcharheight(h)` after `plot` but that is overwritten on the next call.

---

## Multi-Panel Layout

### `figure([width, height, dpi]; kv...)`
Creates a fresh plot state, resetting all settings. Any kwargs passed persist in the figure for all subsequent plots.

### `subplot(nr, nc, p)`
Positions the next plot in a `nr×nc` grid at cell `p` (1-based, row-major, top-left).
- `p` can be a range `(start, end)` to span multiple cells
- Window is cleared only when `p == 1`
- Display is refreshed only when `p == nr*nc` (last panel)

### `hold(flag)`
- `hold(true)`: subsequent `plot` calls add to the current plot without clearing axes or rescaling
- `hold(false)`: restore normal behavior

### Call sequence for multiple panels

```julia
figure(1200, 800)           # fresh figure, sets size for all panels

subplot(2, 3, 1)
plot(x1, y1, title="A")    # clears window, draws panel 1

subplot(2, 3, 2)
plot(x2, y2, title="B")    # no clear; additive

subplot(2, 3, 3)
scatter(x3, y3, title="C")

subplot(2, 3, 4)
histogram(d1, title="D")

subplot(2, 3, 5)
contourf(x,y,z, title="E")

subplot(2, 3, 6)
surface(x,y,z, title="F")  # last panel → triggers screen refresh
```

### kwarg lifetime

- `figure(kv...)` kwargs → persist for all subsequent plots in that figure
- `subplot()` settings (`:subplot`, `:clear`, `:update`) → persist until next `subplot()` or `figure()`
- Standalone setters (`title()`, `xlabel()`, `xticks()`, etc.) → write to `plt[].kvs`, persist until next `figure()`
- `plot(...; kv...)` kwargs → **only for that one call** (context is saved/restored around each `plot`)

### `oplot` vs `hold`

```julia
# oplot: add a series, reuse existing axes/window
plot(x, y1)
oplot(x, y2)   # adds y2 on same axes; same as hold(true) + plot + hold(false)

# hold: more explicit control
plot(x, y1)
hold(true)
plot(x, y2, ylim=(0, 2))   # can override some settings while holding
hold(false)
```

### `redraw(; kv...)`
Re-renders the current figure's data with new kwargs applied. Useful for changing settings without re-specifying data.

---

## Misc Global Functions

```julia
usecolorscheme(n)           # n=1..4, sets auto-cycle color palette
GR.setcolorrep(idx, r, g, b)  # redefine a color index (floats 0.0-1.0)
GR.inqcolor(idx)            # query color at index, returns 0x00RRGGBB
savefig("file.png")         # save current figure (supports png, pdf, svg, etc.)
gcf()                       # get current figure (deep copy of PlotObject)
```

---

## Updates

Changes added to this fork of `jlgr.jl` beyond the upstream version.

### 1. `fontscale` — relative font size kwarg

```julia
plot(x, y, title="hello", xlabel="x", fontscale=1.5)
```

- **Kwarg:** `fontscale` (Float, default `1.0`)
- Multiplies the auto-computed char height at every rendering site (2D axes, 3D axes, polar axes, colorbar).
- Plot margins scale proportionally so tick labels, axis labels, and the title never clip:
  - Left/bottom base margins scale as `0.075 * fontscale` (tick label space).
  - Axis-label margins (`ylabel`, `xlabel`) and title margin stay fixed — the growing base already provides clearance.
- Stacks with the low-level `charheight` kwarg: if both are set, `charheight` is used as the base and `fontscale` multiplies it.

### 2. Digit colors `'1'`–`'8'` in fmt string

```julia
plot(x1, y1, "1",     # same blue as 1st auto-cycle series
     x2, y2, "2--",   # orange dashed
     x3, y3, "3o",    # amber with circle markers
     x4, y4, "4-+")   # purple with plus markers
```

Digits `'1'`–`'8'` select from the 8 auto-cycle palette colors in order, matching exactly what an unstyled multi-series plot would use. The digit is stripped before the rest of the spec is passed to C, so it combines freely with any line style or marker character.

| Digit | GR index | Hex | Color |
|---|---|---|---|
| `1` | 989 | `#00538a` | dark blue |
| `2` | 982 | `#ff6800` | orange |
| `3` | 980 | `#ffb300` | amber |
| `4` | 981 | `#803e75` | purple |
| `5` | 996 | `#93aa00` | olive green |
| `6` | 983 | `#5aa3c0` | steel blue |
| `7` | 995 | `#7f180d` | dark red |
| `8` | 988 | `#f6768e` | pink |

### 3. `nlabels` — limit legend entries

```julia
plot(x1,y1, x2,y2, x3,y3, x4,y4,
     labels=["A","B","C","D"],
     nlabels=2)   # legend shows only A and B
```

- **Kwarg:** `nlabels` (Int, default `length(labels)`)
- Caps the number of series entries rendered in the legend box. Series beyond `nlabels` are plotted normally but have no legend entry.
- The legend box is sized to fit exactly `nlabels` rows, so the outside-right viewport cutout (locations 11–13) is also correctly sized.
- If `nlabels ≥ length(labels)` the behaviour is identical to omitting it.

### Correction: auto-cycle color indices

The `distinct_cmap` constant (used by `usecolorscheme` to remap the auto-cycle palette) previously held incorrect indices `(0, 1, 984, 987, 989, 983, 994, 988)`. The correct values, derived directly from the C source (`predef_colors` array in `gr.c`, offset by 980) are:

```julia
const distinct_cmap = (989, 982, 980, 981, 996, 983, 995, 988)
```

This also fixes `usecolorscheme(n)` for schemes 2–4, which was remapping the wrong color slots.
