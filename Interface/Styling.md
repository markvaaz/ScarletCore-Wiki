# Styling

This page covers all visual styling types used by Window and Element properties: colors, backgrounds, gradients, borders, shadows, spacing, and dimensions.

## Table of Contents

- [UIColor](#uicolor)
- [UIBackground](#uibackground)
- [UIGradient](#uigradient)
- [Border](#border)
- [BoxShadow](#boxshadow)
- [Spacing](#spacing)
- [Dimension](#dimension)
- [Position](#position)
- [Anchor & Pivot](#anchor--pivot)
- [Text Styling](#text-styling)
  - [UITextGradient](#uitextgradient)
  - [UITextShadow](#uitextshadow)
  - [UITextOutline](#uitextoutline)
- [UIIcons](#uiicons)

---

## UIColor

Represents an RGBA color. Components are in the 0–1 range.

```csharp
// From a hex string
UIColor.Hex("#1a1a2e")       // 6-digit hex
UIColor.Hex("#fff")          // 3-digit shorthand
UIColor.Hex("#ffffff80")     // 8-digit hex with alpha

// From float components (0–1)
UIColor.RGB(0.1f, 0.1f, 0.2f)
UIColor.RGBA(0f, 0f, 0f, 0.5f)   // semi-transparent black

// Named presets
UIColor.White
UIColor.Black
UIColor.Red
UIColor.Green
UIColor.Blue
UIColor.Transparent
```

---

## UIBackground

A unified background descriptor that supports solid colors, gradients, remote images, and native game sprites — alone or in combination.

### Solid color

```csharp
Background = UIBackground.FromColor(UIColor.Hex("#1a1a2e"))
```

### Gradient

```csharp
Background = UIBackground.FromGradient(
  UIGradient.Linear(90, UIColor.Hex("#1a1a2e"), UIColor.Hex("#2a2a5e"))
)
```

### Remote image

```csharp
Background = UIBackground.FromImage("https://example.com/bg.png")
Background = UIBackground.FromImage("https://example.com/bg.png", ImageFit.Cover)
```

### Native game sprite

```csharp
Background = UIBackground.FromSprite("StatBG")
Background = UIBackground.FromSprite("ItemSlotBG", ImageFit.Stretch)
```

### Layering modifiers

Static backgrounds can be layered by chaining `With*` methods:

```csharp
// Image with a semi-transparent color overlay
Background = UIBackground.FromImage("https://example.com/bg.png")
  .WithColor(UIColor.RGBA(0f, 0f, 0f, 0.4f))

// Gradient on top of a sprite
Background = UIBackground.FromSprite("PanelBG")
  .WithGradient(UIGradient.Linear(180, UIColor.Transparent, UIColor.Hex("#000")))
```

### Animated backgrounds

```csharp
// Cycle through remote images, looping
Background = UIBackground.AnimatedFromUrls(new[] {
  "https://example.com/frame1.png",
  "https://example.com/frame2.png",
  "https://example.com/frame3.png"
}, duration: 0.5f)

// Cycle through native sprites on hover, bounce
Background = UIBackground.AnimatedFromSprites(new[] { "SprA", "SprB", "SprC" }, duration: 0.4f)
  .WithAnimTrigger(AnimationTrigger.Hover)
  .WithAnimLoopType(AnimationLoopType.Bounce)
  .WithColor(UIColor.Hex("#222"))  // shown while frames are loading
```

**Animation modifier methods:**

| Method | Description |
|--------|-------------|
| `.WithAnimTrigger(trigger)` | When the animation starts (`Always`, `Hover`, `Click`, etc.) |
| `.WithAnimLoopType(type)` | `Loop` (wrap) or `Bounce` (ping-pong) |
| `.WithAnimLoopCount(n)` | Number of cycles (0 = infinite) |
| `.WithAnimReleaseMode(mode)` | Behavior when the trigger is released |
| `.WithAnimPlaying(bool)` | Start playing immediately (for `Manual` trigger) |

**`ImageFit` values:** `Stretch`, `Contain`, `Cover`, `Fill`

---

## UIGradient

A gradient for use in `UIBackground.FromGradient()`.

```csharp
// Linear gradient — angle in degrees (0 = left→right, 90 = bottom→top)
UIGradient.Linear(0, UIColor.Hex("#ff0000"), UIColor.Hex("#0000ff"))
UIGradient.Linear(90, UIColor.Hex("#1a1a2e"), UIColor.Hex("#4a4a8e"), UIColor.Hex("#1a1a2e"))

// Radial gradient — cx/cy are center position in 0–1 coords
UIGradient.Radial(0.5f, 0.5f, UIColor.Hex("#4a4aff"), UIColor.Hex("#000"))
```

---

## Border

Defines a border with color, thickness, and optional corner radius.

```csharp
new Border(color, width, radius)

// Examples
new Border(UIColor.Hex("#555"), 1)          // 1px border, no rounding
new Border(UIColor.Hex("#7070ff"), 1, 8)    // 1px border, 8px corner radius
new Border(UIColor.White, 2, 4)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `color` | `UIColor` | Border color |
| `width` | `float` | Border thickness in pixels |
| `radius` | `float` | Corner radius in pixels (default `0`) |

---

## BoxShadow

A shadow rendered behind an element or window.

```csharp
new BoxShadow(color, offsetX, offsetY, blur, spread)

// Examples
new BoxShadow(UIColor.RGBA(0, 0, 0, 0.6f))                        // default offsets
new BoxShadow(UIColor.RGBA(0, 0, 0, 0.6f), offsetX: 0, offsetY: 4, blur: 12)
new BoxShadow(UIColor.Hex("#7070ff"), offsetX: 0, offsetY: 0, blur: 16, spread: 2)
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `color` | `UIColor` | — | Shadow color |
| `offsetX` | `float` | `0` | Horizontal offset (positive = right) |
| `offsetY` | `float` | `4` | Vertical offset (positive = down) |
| `blur` | `float` | `8` | Feather radius in pixels (0 = hard edge) |
| `spread` | `float` | `0` | Expand (+) or contract (-) the shadow size |

---

## Spacing

Per-side spacing for `Padding` and `Margin`.

```csharp
Spacing.All(12)              // 12px on all sides
Spacing.Horizontal(8)        // 8px left/right, 0 top/bottom
Spacing.Vertical(4)          // 4px top/bottom, 0 left/right
Spacing.XY(horizontal: 8, vertical: 4)

new Spacing(top, right, bottom, left)   // explicit per-side
new Spacing(vertical, horizontal)       // symmetric shorthand

// Implicit float conversion — equivalent to Spacing.All
Padding = 12f
```

---

## Dimension

Width and height values support pixels or percentage strings.

```csharp
Width = 300           // 300 pixels (int implicit)
Width = 300f          // 300 pixels (float implicit)
Width = "50%"         // 50% of parent width (string implicit)
Width = "100%"        // fill parent
```

---

## Position

X/Y offset from the anchor point, plus an optional Z-index for canvas sorting.

```csharp
Position = new Position(x: -20, y: -20)         // offset only
Position = new Position(x: 0, y: 0, zIndex: 10) // with Z-index
Position = new Position(zIndex: 5)               // Z-index only
```

X and Y accept `Dimension` values (pixels or `"50%"`).

---

## Anchor & Pivot

`Anchor` sets which point on the parent (or screen for windows) the element attaches to. `Pivot` sets which point of the element itself is placed at that anchor.

**`Anchor` values:**

```
TopLeft      TopCenter      TopRight
MiddleLeft   MiddleCenter   MiddleRight
BottomLeft   BottomCenter   BottomRight
```

**`Pivot` values:** same 9-point grid as `Anchor`.

```csharp
// Window pinned to the top-right corner, 20px inset
Anchor = Anchor.TopRight
Position = new Position(x: -20, y: -20)

// Element centered on its parent
Anchor = Anchor.MiddleCenter,
Pivot = Pivot.MiddleCenter
```

---

## Text Styling

Text elements (`Text`, `Button`, `Input`, `Dropdown`, `CloseButton`) all support `UITextGradient`, `UITextShadow`, and `UITextOutline`.

### UITextGradient

A vertex-color gradient applied to text characters.

```csharp
// Top-to-bottom
TextGradient = UITextGradient.Vertical(UIColor.White, UIColor.Hex("#aaaaff"))

// Left-to-right
TextGradient = UITextGradient.Horizontal(UIColor.Hex("#ff8800"), UIColor.Hex("#ffff00"))

// Four corners
TextGradient = UITextGradient.FourCorner(
  topLeft:     UIColor.White,
  topRight:    UIColor.Hex("#aaaaff"),
  bottomLeft:  UIColor.Hex("#8888cc"),
  bottomRight: UIColor.Hex("#6666aa")
)

// Multi-stop linear
TextGradient = UITextGradient.Linear(0, UIColor.Red, UIColor.Hex("#ff8800"), UIColor.Yellow)

// Multi-stop radial — cx/cy in 0–1 coords
TextGradient = UITextGradient.Radial(0.5f, 0.5f, UIColor.White, UIColor.Hex("#7070ff"))
```

### UITextShadow

A drop shadow behind text characters.

```csharp
TextShadow = new UITextShadow(UIColor.RGBA(0, 0, 0, 0.8f))
TextShadow = new UITextShadow(UIColor.Black, offsetX: 1f, offsetY: 2f)
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `color` | `UIColor` | — | Shadow color |
| `offsetX` | `float` | `2` | Horizontal offset in pixels |
| `offsetY` | `float` | `2` | Vertical offset in pixels |

### UITextOutline

An outline around each text glyph.

```csharp
TextOutline = new UITextOutline(UIColor.Black)
TextOutline = new UITextOutline(UIColor.Hex("#7070ff"), width: 2f)
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `color` | `UIColor` | — | Outline color |
| `width` | `float` | `1` | Outline thickness in pixels |

---

## UIIcons

Embeds an in-game item icon inline inside a text string. Use inside `Text.Content`, `Button.Label`, etc.

```csharp
// Basic icon token
UIIcons.Icon(-1234567890)

// Icon with explicit size
UIIcons.Icon(-1234567890, size: 24f)

// Icon with size and spacing
UIIcons.Icon(-1234567890, size: 24f, spacing: 4f)

// Example — inside a Text element
new Text { Content = $"Requires: 5x {UIIcons.Icon(-1234567890)} Iron Ingot" }
```

The `guidHash` is the raw `int` value of the item's `PrefabGUID`. You can find these in the [Prefabs Browser](Data/Prefabs.md).
