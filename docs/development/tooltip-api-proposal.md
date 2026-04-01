# Tooltip API Proposal

## Purpose

This proposal defines a two-tier tooltip API for Slint:

- a simple API for native-looking text tooltips
- a complex API for fully custom tooltip content

The goal is to keep common usage easy while supporting rich and interactive tooltip UI when needed.

## Why This Shape

Common patterns across modern UI frameworks converge on two APIs:

- **simple** text tooltip API for most use-cases
- **rich/custom** tooltip API for advanced content and interaction

This proposal follows that expectation to keep Slint intuitive for users.

## API Overview

### 1) Simple text tooltip (default path)

Public API:

```slint
Button {
    text: "Save";
    tooltip: "Save this document";
}
```

Behavior:

- `tooltip` is shown after hover delay.
- Slint core requests native text tooltip display through `WindowAdapterInternal` (platform integration layer, including winit).
- If native display is unavailable, Slint shows a built-in fallback tooltip UI.

### 2) Complex/custom tooltip content (rich path)

Public API shape:

```slint
TooltipArea {
    trigger: hover;

    Button {
        text: "Network";
    }

    Tooltip {
        VerticalLayout {
            Text { text: "Connection details"; }
            Button { text: "Reconnect"; }
        }
    }
}
```

Definitions:

- `trigger-content`: the hovered/focused UI element the tooltip attaches to
- `Tooltip { ... }`: arbitrary custom tooltip content

Capabilities:

- custom components and layouts
- optional interactive controls
- trigger modes (`hover`, `focus`, `manual`)
- show/hide delays
- placement options

## Final Decisions

### Tooltip vs PopupWindow

- `Tooltip` is a semantic tooltip surface, not just a visual alias for `PopupWindow`.
- `PopupWindow` remains the generic popup primitive.
- Tooltips provide tooltip-specific behavior defaults (trigger lifecycle, delays, close behavior, placement).

### Positioning

- Auto-position is the default for both simple and complex APIs.
- Complex API also supports optional manual placement controls.

### Native and fallback behavior

- Native integration applies to the simple text tooltip path.
- Fallback to Slint-rendered tooltip UI is guaranteed where native tooltip display is not available.

## Runtime Integration

The implementation integrates through `WindowAdapterInternal`:

- `set_tooltip_text(text: &str, position: LogicalPosition, options: TooltipOptions)`
- `clear_tooltip()`

winit backend implements platform behavior where possible; unsupported cases use Slint fallback rendering.

## Rollout Plan

### Phase 1 (MVP)

- `tooltip: "..."` on items
- hover-delay lifecycle in core
- native-preferred display request via `WindowAdapterInternal`
- Slint fallback text tooltip

### Phase 2

- `TooltipArea` + `Tooltip` rich/custom API
- trigger modes and interaction controls
- placement refinements

### Phase 3

- platform-specific native improvements in winit path
- additional advanced tuning knobs as needed

## Interaction and Accessibility Defaults

- Interactive tooltips are opt-in for focusability.
- Non-interactive tooltips auto-close on leave/focus change.
- Escape key closes visible custom tooltips unless explicitly configured otherwise.

