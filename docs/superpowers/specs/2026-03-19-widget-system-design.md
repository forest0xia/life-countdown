# Widget Management System — Design Spec

## Overview
Turn all dashboard sections into configurable widgets with a slide-out settings panel. Users can show/hide widgets, drag-to-reorder, and persist configuration in the URL.

## Architecture

### Always Visible (not configurable)
- Hero age display
- Life progress bar

### Configurable Widgets

| ID | Name | Default | Element |
|---|---|---|---|
| `matrix` | 生命矩阵 | ON | `#life-dots` |
| `countdown` | 剩余时间 | ON | `#cdGrid` |
| `allocation` | 时间分配 | ON | `#allocation` |
| `companion` | 陪伴时间 | ON | `#companion` |
| `progress` | 时间进度 | ON | `#progress` |
| `quotes` | 每日一言 | ON | `#quote` |
| `holidays` | 节日倒数 | OFF | `#holidays` (new) |

### URL Persistence
Parameter `w` contains ordered, comma-separated list of visible widget IDs:
```
&w=matrix,countdown,allocation,companion,progress,quotes
```
Hidden widgets are absent from the list. Order = display order.
Default (when `w` param missing): all ON except `holidays`, in the order above.

## Settings Panel

### Trigger
Gear icon button in the top-bar, right side (alongside clock and share buttons).

### Layout
Slide-out panel from the right edge, ~360px wide, full height, z-index above content.
Semi-transparent backdrop behind it.

### Content (top to bottom)
1. **Panel title**: "设置"
2. **Widget list**: Sortable rows via Sortable.js CDN
   - Each row: `☰ drag-handle` | `widget name` | `toggle switch`
   - Drag to reorder, toggle to show/hide
3. **Divider**
4. **预期寿命**: Current value display + edit button (opens existing modal)
5. **桌面时钟**: Button to open desk clock mode
6. **时钟字体**: Font selector (reuse existing font options from desk clock)
7. **Divider**
8. **恢复默认**: Reset button

### Behavior
- Changes apply immediately (live preview)
- On close: update URL with new `w` param via `stateToURL()`
- Backdrop click or close button dismisses panel

## PC Dashboard Layout
- Hero + life bar: fixed rows 1-2 (unchanged)
- Widgets render in configured order
- CSS Grid `grid-auto-flow: dense`, 3 columns, `1fr 1fr 1fr`
- Hidden widgets get `display: none`
- Grid auto-fills, so removing a widget causes others to reflow

## Mobile Layout
- Same widget order as configured
- Vertical stack (unchanged flow)
- Hidden widgets get `display: none`
- Separators between widgets remain

## 节日倒数 Widget (New)

### Content
Compact list showing:
- 🎂 你的第 X 个生日 — 还有 X 天
- 🧧 你的第 X 个春节 — 还有 X 天
- 🥮 你的第 X 个中秋 — 还有 X 天
- 🐉 你的第 X 个端午 — 还有 X 天
- 🌿 你的第 X 个清明 — 还有 X 天
- 🇨🇳 你的第 X 个国庆 — 还有 X 天

### Lunar Calendar
For 春节/中秋/端午/清明: use a lookup table of dates for 2024-2040.
This avoids complex lunar calculation. If user's lifespan extends beyond 2040,
use approximate formula (acceptable accuracy).

### Structure
Same pattern as other widgets: `glass-strong` card with section title.
Compact rows similar to time progress widget.

## Dependencies
- **Sortable.js**: `https://cdn.jsdelivr.net/npm/sortablejs@1.15.6/Sortable.min.js` (~2KB gzipped)

## Separator Handling
- Mobile: existing `.sep` dividers between widgets — hide separators for hidden widgets
- PC: separators already hidden in dashboard mode (no change)

## Implementation Notes
- Widget DOM elements stay in place; visibility controlled by CSS class `.widget-hidden`
- On PC, `order` CSS property controls display order based on user config
- `initDashboard()` reads widget config and applies order + visibility
- `renderAll()` skips rendering for hidden widgets (performance)
- Settings panel is a new HTML element outside `.app`, similar to modals
