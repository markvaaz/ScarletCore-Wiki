# Elements

All visible content inside a Window is made up of elements. Every element inherits from `UIElement` and shares a common set of layout and visual properties.

## Table of Contents

- [Shared Properties (UIElement)](#shared-properties-uielement)
- [Row](#row)
- [Text](#text)
- [Button](#button)
- [CloseButton](#closebutton)
- [Image](#image)
- [Input](#input)
- [Dropdown](#dropdown)
- [ProgressBar](#progressbar)
- [Accordion](#accordion)
- [AnimatedSheet](#animatedsheet)
- [PortraitCamera](#portraitcamera)

---

## Shared Properties (UIElement)

Every element supports these properties:

| Property | Type | Description |
|----------|------|-------------|
| `Width` | `Dimension` | Width in pixels or percentage (`"50%"`) |
| `Height` | `Dimension` | Height in pixels or percentage |
| `Background` | `UIBackground?` | Background fill (color, gradient, image, sprite) |
| `Border` | `Border?` | Border color, thickness, and corner radius |
| `Padding` | `Spacing?` | Inner spacing |
| `Margin` | `Spacing?` | Outer spacing |
| `Rotation` | `float` | Rotation in degrees |
| `BoxShadow` | `BoxShadow?` | Shadow behind the element |
| `Anchor` | `Anchor?` | When set, positions the element absolutely instead of in flow |
| `Position` | `Position?` | X/Y offset and optional Z-index |
| `Pivot` | `Pivot?` | Which point of the element is placed at the anchor |
| `ElemId` | `string` | Stable ID for partial updates via `Window.SendUpdate` |
| `Tooltip` | `string` | Window ID to show as a tooltip on hover |

Elements that display text also implement `ITextElement`:

| Property | Type | Description |
|----------|------|-------------|
| `TextColor` | `UIColor?` | Text color |
| `FontSize` | `float` | Font size in pixels (0 = inherit) |
| `Font` | `string` | TMP font asset name (null = game default) |
| `TextGradient` | `UITextGradient?` | Vertex-color gradient on text characters |
| `TextShadow` | `UITextShadow?` | Drop shadow behind text |
| `TextOutline` | `UITextOutline?` | Outline around each glyph |

---

## Row

A horizontal container. Children are laid out left-to-right.

```csharp
new Row {
  Height = 40,
  Gap = 8,
  JustifyContent = JustifyContent.SpaceBetween,
  AlignItems = AlignItems.Center,
  Children = {
    new Text { Content = "Name", Width = "50%" },
    new Button { Label = "Action", Width = 80, Height = 30 }
  }
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Gap` | `float` | `0` | Gap between children in pixels |
| `JustifyContent` | `JustifyContent` | `Start` | Horizontal distribution of children |
| `AlignItems` | `AlignItems` | `Start` | Vertical alignment of children |
| `Overflow` | `OverflowMode` | `Visible` | How overflowing content is handled |
| `ScrollbarColor` | `UIColor?` | system | Scrollbar thumb color |
| `ScrollbarBackgroundColor` | `UIColor?` | system | Scrollbar track color |
| `ScrollbarWidth` | `float` | `8` | Scrollbar width in pixels |

**`JustifyContent` values:** `Start`, `Center`, `End`, `SpaceBetween`, `SpaceAround`, `SpaceEvenly`

**`AlignItems` values:** `Start`, `Center`, `End`

---

## Text

A text label.

```csharp
new Text {
  Content = "Hello, World!",
  FontSize = 16,
  TextColor = UIColor.White,
  TextAlign = TextAlignment.Center,
  Wrap = true,
  Width = "100%"
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Content` | `string` | — | The text to display. Supports inline icons via `UIIcons.Icon()` |
| `TextAlign` | `TextAlignment` | `Left` | Horizontal alignment: `Left`, `Center`, `Right` |
| `Wrap` | `bool` | `false` | Wrap text when it exceeds the element width |
| + all `ITextElement` and `UIElement` properties | | | |

**Inline icons:**
```csharp
new Text { Content = $"Requires: 5x {UIIcons.Icon(-1234567890)}" }
```

---

## Button

A clickable button that sends a chat command to the server when clicked.

```csharp
new Button {
  Label = "Buy Sword",
  Command = ".shop buy sword",
  Width = "100%", Height = 36,
  Background = UIBackground.FromColor(UIColor.Hex("#2a5c2a")),
  HoverBackground = UIBackground.FromColor(UIColor.Hex("#3a7c3a")),
  HoverScale = true
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Label` | `string` | — | Button label text |
| `Command` | `string` | — | Chat command sent when clicked |
| `HoverBackground` | `UIBackground?` | none | Background shown on hover |
| `PressedBackground` | `UIBackground?` | none | Background shown while pressed |
| `HoverScale` | `bool` | `false` | Slightly scale up on hover (1.0 → 1.05) |
| `BoxSizing` | `BoxSizing` | `ContentBox` | Whether padding is inside or outside declared size |
| + all `ITextElement` and `UIElement` properties | | | |

**Handling clicks on the server:**

```csharp
// Register handler
InterfaceManager.OnCommand("shop.buy", (player, args) => {
  // args[0] = "sword" from ".shop buy sword"
});

// Or use the ScarletCore command system
[Command("shop buy")]
public static void BuyItem(CommandContext ctx, string item) { ... }
```

**Using Input values in commands:**

When an `Input` element has an `Id`, that value can be referenced in button commands with `{InputId}`:

```csharp
new Input { Id = "qty", Placeholder = "Quantity" },
new Button { Label = "Buy", Command = ".shop buy sword {qty}" }
```

---

## CloseButton

A pre-styled × button that closes the current window when clicked. No configuration needed.

```csharp
new CloseButton()

// With custom color
new CloseButton { TextColor = UIColor.Hex("#aaa") }
```

---

## Image

An image loaded from a URL.

```csharp
new Image {
  Src = "https://example.com/icon.png",
  Width = 64, Height = 64,
  Fit = ImageFit.Contain
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Src` | `string` | — | HTTP/HTTPS URL of the image |
| `Fit` | `ImageFit` | `Stretch` | How the image fills its bounds |

**`ImageFit` values:** `Stretch`, `Contain`, `Cover`, `Fill`

> Use `InterfaceManager.PreCacheImages()` at plugin load to warm up images before windows are opened.

---

## Input

A text field. Its typed value can be passed to button commands using `{Id}`.

```csharp
new Input {
  Id = "playername",
  Placeholder = "Enter player name...",
  Width = "100%", Height = 32,
  InputType = InputType.String,
  MaxLength = 20,
  FocusBorder = new Border(UIColor.Hex("#7070ff"), 1, 4)
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Id` | `string` | — | Identifier — referenced as `{Id}` in button/dropdown commands |
| `Placeholder` | `string` | — | Hint text shown when empty |
| `PlaceholderColor` | `UIColor?` | system | Placeholder text color |
| `Value` | `string` | — | Pre-filled value |
| `InputType` | `InputType` | `String` | `String` (any text) or `Number` (digits only) |
| `MaxLength` | `int` | `0` (unlimited) | Maximum character count |
| `TextAlign` | `TextAlignment` | `Left` | Text alignment inside the field |
| `BoxSizing` | `BoxSizing` | `BorderBox` | Padding inside vs outside declared size |
| `FocusBackground` | `UIBackground?` | none | Background while the field is focused |
| `FocusBorder` | `Border?` | none | Border while the field is focused |
| `CaretColor` | `UIColor?` | system | Blinking caret color |
| `SelectionColor` | `UIColor?` | system | Text selection highlight color |
| `SelectionTextColor` | `UIColor?` | system | Color of selected text |
| + all `ITextElement` and `UIElement` properties | | | |

---

## Dropdown

A selector that sends a command with the chosen value when an option is picked.

```csharp
new Dropdown {
  Id = "difficulty",
  Options = "Easy:easy|Normal:normal|Hard:hard",
  Command = ".game setdifficulty {difficulty}",
  Placeholder = "Select difficulty...",
  Width = "100%", Height = 32
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Id` | `string` | — | Identifier used in `Command` as `{Id}` |
| `Options` | `string` | — | Pipe-separated options: `"Label:value\|Label2:value2"`. If no colon, label = value |
| `Command` | `string` | — | Command sent on selection. `{Id}` is replaced with the selected value |
| `Placeholder` | `string` | `"Select..."` | Text shown when nothing is selected |
| `Value` | `string` | — | Pre-selected value |
| `DropdownBackgroundColor` | `UIColor?` | system | Background of the popup panel |
| `DropdownTextColor` | `UIColor?` | system | Option label color inside the popup |
| `DropdownHoverColor` | `UIColor?` | system | Option highlight color on hover |
| `MaxHeight` | `float` | `200` | Max popup height before scrolling |
| `BoxSizing` | `BoxSizing` | `ContentBox` | Padding inside vs outside declared size |
| + all `ITextElement` and `UIElement` properties | | | |

---

## ProgressBar

A horizontal progress bar.

```csharp
new ProgressBar {
  Value = 75,
  Min = 0, Max = 100,
  Width = "100%", Height = 12,
  BarFill = UIBackground.FromColor(UIColor.Hex("#4caf50")),
  Background = UIBackground.FromColor(UIColor.Hex("#333")),
  Border = new Border(UIColor.Hex("#555"), 1, 6),
  AnimateValue = true
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Value` | `float` | `0` | Current progress value |
| `Min` | `float` | `0` | Minimum value |
| `Max` | `float` | `100` | Maximum value |
| `BarFill` | `UIBackground?` | system | Fill color/image of the bar portion |
| `AnimateValue` | `bool` | `false` | Animate value changes smoothly |
| `AnimationDuration` | `float` | `0.3` | Duration of value-change animation in seconds |
| + all `UIElement` properties | | | |

To update the bar value in real-time, use `Window.SendUpdate`:

```csharp
Window.SendUpdate(player, "myplugin", "hud", "hp-bar",
  new ProgressBar { Value = currentHp, Max = maxHp, AnimateValue = true });
```

---

## Accordion

A collapsible section. The header is always visible; clicking it expands or collapses the content — all handled client-side without a server round-trip.

```csharp
new Accordion {
  Title = "Advanced Settings",
  Expanded = false,
  HeaderBackground = UIBackground.FromColor(UIColor.Hex("#2a2a4a")),
  Gap = 6,
  Children = {
    new Row {
      Children = {
        new Text { Content = "Option A" },
        new Button { Label = "Set", Command = ".config seta" }
      }
    }
  }
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Title` | `string` | — | Header title text |
| `Expanded` | `bool` | `false` | Whether the accordion starts open |
| `HeaderHeight` | `float` | `32` | Header row height in pixels |
| `HeaderBackground` | `UIBackground?` | none | Header background |
| `HeaderTextColor` | `UIColor?` | white | Header title color |
| `FontSize` | `float` | `0` | Header title font size (0 = inherit) |
| `ContentBackground` | `UIBackground?` | none | Content area background |
| `Gap` | `float` | `0` | Gap between children inside the content area |
| `ChevronColor` | `UIColor?` | white | Expand/collapse arrow color |
| `ChevronIcon` | `string` | default | Custom chevron icon |
| `ShowChevron` | `bool` | `true` | Show or hide the chevron icon |

---

## AnimatedSheet

An element that cycles through a sequence of image frames to produce a sprite-sheet animation.

```csharp
new AnimatedSheet {
  FrameUrls = new[] {
    "https://example.com/frame1.png",
    "https://example.com/frame2.png",
    "https://example.com/frame3.png"
  },
  Duration = 0.5f,
  Trigger = AnimationTrigger.Always,
  LoopType = AnimationLoopType.Loop,
  Width = 64, Height = 64
}

// Hover-activated from native game sprites
new AnimatedSheet {
  FrameSprites = new[] { "SpriteA", "SpriteB", "SpriteC" },
  Duration = 0.3f,
  Trigger = AnimationTrigger.Hover,
  ReleaseMode = AnimationReleaseMode.Pause,
  Width = 48, Height = 48
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `FrameUrls` | `string[]` | — | Image URLs for each frame (takes priority over `FrameSprites`) |
| `FrameSprites` | `string[]` | — | Native game sprite names for each frame |
| `Duration` | `float` | `1` | Duration of one full cycle in seconds |
| `Trigger` | `AnimationTrigger` | `Always` | What starts the animation |
| `LoopType` | `AnimationLoopType` | `Loop` | `Loop` (wrap) or `Bounce` (ping-pong) |
| `LoopCount` | `int` | `0` (infinite) | Number of cycles to play (0 = infinite) |
| `ReleaseMode` | `AnimationReleaseMode` | `Pause` | Behavior when the trigger is released |
| `Playing` | `bool` | `true` | Start playing immediately (relevant for `Manual` trigger) |
| `Fit` | `ImageFit` | `Stretch` | How frames fill the element bounds |

**`AnimationTrigger` values:** `Always`, `WindowOpen`, `Hover`, `Pressed`, `Click`, `Manual` (flags — can be combined)

**`AnimationLoopType` values:** `Loop`, `Bounce`

---

## PortraitCamera

Renders the local player's character in 3D inside the UI element.

```csharp
new PortraitCamera {
  Width = 200, Height = 300,
  FieldOfView = 50,
  OrbitAngle = 15,
  Distance = 1.2f,
  BackgroundColor = UIColor.Hex("#111"),
  BackgroundUrl = "https://example.com/portrait-bg.png"
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `FieldOfView` | `float` | `60` | Camera field of view in degrees |
| `OrbitAngle` | `float` | `0` | Orbit angle in degrees around the anchor bone |
| `Distance` | `float` | `1` | Distance multiplier from the anchor bone |
| `AnchorBone` | `string` | `"Head_JNT"` | Bone the camera orbits around |
| `BackgroundUrl` | `string` | — | URL of the texture rendered behind the character |
| `BackgroundColor` | `UIColor?` | none | Solid tint on the background quad |
| `BackgroundSize` | `float` | `1.6` | World-space size of the background quad (metres) |
| `BackgroundOffsetX` | `float` | `0` | UV offset X for panning |
| `BackgroundOffsetY` | `float` | `0` | UV offset Y for panning |
| `BackgroundScaleX` | `float` | `1` | UV scale X for zooming |
| `BackgroundScaleY` | `float` | `1` | UV scale Y for zooming |
