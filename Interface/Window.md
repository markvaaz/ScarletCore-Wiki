# Window

A `Window` is the top-level container for all your UI. You construct it, add elements to `Children`, and call `.Send()` to deliver it to the player.

## Creating a Window

```csharp
// Target a specific player
new Window(player, "myplugin", "my-window") { ... }.Send();

// Broadcast to all connected players
new Window("myplugin", "my-window") { ... }.Send();
```

The three constructor arguments are:
- **player** — the `PlayerData` to target (omit for broadcast)
- **plugin** — a unique identifier for your plugin (e.g. `"myplugin"`)
- **id** — a unique identifier for this window within your plugin (e.g. `"shop"`, `"hud"`)

The combination of `plugin + id` identifies a window on the client. Sending the same `plugin + id` again replaces the existing window.

## Properties

### Dimensions

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Width` | `Dimension` | auto | Window width — pixels (`400`) or percentage (`"80%"`) |
| `Height` | `Dimension` | auto | Window height — pixels or percentage |

### Background

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Background` | `UIBackground?` | none | Solid color, gradient, image, or sprite background |

```csharp
Background = UIBackground.FromColor(UIColor.Hex("#1a1a2e"))
Background = UIBackground.FromGradient(UIGradient.Linear(90, UIColor.Hex("#1a1a2e"), UIColor.Hex("#2a2a5e")))
Background = UIBackground.FromImage("https://example.com/bg.png")
```

### Border

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Border` | `Border?` | none | Border color, thickness, and corner radius |

```csharp
Border = new Border(UIColor.Hex("#555"), width: 1, radius: 8)
```

### Spacing & Layout

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Padding` | `Spacing?` | none | Inner spacing between the border and children |
| `Gap` | `float` | `0` | Gap in pixels between child rows |
| `Overflow` | `OverflowMode` | `Visible` | How overflowing content is handled |

```csharp
Padding = Spacing.All(12)
Padding = new Spacing(top: 8, right: 12, bottom: 8, left: 12)
Gap = 8
Overflow = OverflowMode.ScrollY
```

### Scrollbar

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `ScrollbarColor` | `UIColor?` | system | Scrollbar thumb color |
| `ScrollbarBackgroundColor` | `UIColor?` | system | Scrollbar track color |
| `ScrollbarWidth` | `float` | `8` | Scrollbar width in pixels |

### Positioning

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Anchor` | `Anchor` | `MiddleCenter` | Screen anchor point for the window |
| `Position` | `Position?` | none | X/Y offset from the anchor point, optional Z-index |
| `Pivot` | `Pivot?` | `TopLeft` | Which point of the window is placed at the anchor |
| `Rotation` | `float` | `0` | Rotation in degrees |

```csharp
Anchor = Anchor.TopRight
Position = new Position(x: -20, y: -20)  // 20px from the top-right corner
```

Available `Anchor` values: `TopLeft`, `TopCenter`, `TopRight`, `MiddleLeft`, `MiddleCenter`, `MiddleRight`, `BottomLeft`, `BottomCenter`, `BottomRight`.

### Shadow

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `BoxShadow` | `BoxShadow?` | none | Shadow rendered behind the window |

```csharp
BoxShadow = new BoxShadow(UIColor.RGBA(0, 0, 0, 0.5f), offsetX: 0, offsetY: 4, blur: 12)
```

### Behaviour

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Draggable` | `bool` | `true` | Whether the player can drag the window |
| `Transparent` | `bool` | `false` | Makes the window click-through — mouse clicks pass through it and are not captured by the window |
| `HideOnMenuOpen` | `bool` | `true` | Hide when any in-game menu (inventory, map, etc.) opens |
| `NativeParent` | `string` | none | Attach to an existing game UI element path |

### Animations

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `OpenAnimation` | `WindowAnimation` | `None` | Animation when the window appears |
| `CloseAnimation` | `WindowAnimation` | `None` | Animation when the window closes |
| `AnimationDuration` | `float` | `0.2` | Duration in seconds |

Available animations: `None`, `Fade`, `Scale`, `SlideDown`, `SlideUp`.

```csharp
OpenAnimation = WindowAnimation.Scale,
CloseAnimation = WindowAnimation.Fade,
AnimationDuration = 0.3f
```

### Custom Texture Frame (9-slice)

Use `SetCustomTexture(...)` to apply a 9-piece tiled image frame (4 corners + 4 borders + background) to the window:

```csharp
new Window(player, "myplugin", "framed") {
  Width = 400, Height = 300
}
.SetCustomTexture(
  topLeftCorner:     "https://example.com/tl.png",
  topRightCorner:    "https://example.com/tr.png",
  bottomLeftCorner:  "https://example.com/bl.png",
  bottomRightCorner: "https://example.com/br.png",
  topBorder:         "https://example.com/top.png",
  bottomBorder:      "https://example.com/bottom.png",
  leftBorder:        "https://example.com/left.png",
  rightBorder:       "https://example.com/right.png",
  background:        "https://example.com/bg.png",
  cornerSize: 32, frameExpand: 0
)
.Send();
```

## Children

Add elements inside `Children` using the collection initializer or `Add()`:

```csharp
new Window(player, "myplugin", "demo") {
  Children = {
    new Row { ... },
    new Text { ... }
  }
}.Send();

// Or with Add():
var w = new Window(player, "myplugin", "demo");
w.Add(new Text { Content = "Hello" });
w.Send();
```

Direct children of a Window should be `Row` or `Accordion` elements. Standalone elements (Text, Button, etc.) can also be added directly and will each occupy their own implicit row.

## Sending

### Send — open or replace a window

```csharp
window.Send();                     // opens/replaces
window.Send(WindowAction.Close);   // closes
window.Send(WindowAction.Clear);   // removes all elements but keeps the window
```

### Close a window

```csharp
InterfaceManager.CloseWindow(player, "myplugin", "shop");
```

## Partial Updates

When a window is already open, you can update or remove individual elements without re-sending the entire window. The element must have an `ElemId` set when first sent.

### Update a single element

```csharp
// First send — assign an ElemId
new Window(player, "myplugin", "hud") {
  Children = {
    new Row {
      Children = {
        new Text { ElemId = "hp-label", Content = "HP: 100", FontSize = 14 }
      }
    }
  }
}.Send();

// Later — update just that element
var win = new Window(player, "myplugin", "hud");
win.SendUpdate("hp-label", new Text { Content = "HP: 87", FontSize = 14 });

// Static shorthand
Window.SendUpdate(player, "myplugin", "hud", "hp-label", new Text { Content = "HP: 87" });
```

### Remove a single element

```csharp
var win = new Window(player, "myplugin", "hud");
win.SendDelete("hp-label");

// Static shorthand
Window.SendDelete(player, "myplugin", "hud", "hp-label");
```

> Partial updates only work on player-targeted windows (not broadcast).
