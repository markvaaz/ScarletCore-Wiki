# Interface

ScarletCore Interface lets you build and send UI windows to players directly from your server-side plugin — no client mod code required. Players need the **ScarletInterface** client mod installed.

## Overview

The system has two main APIs:

| API | Purpose |
|-----|---------|
| **Window** | Build custom overlay windows with elements like text, buttons, images, and inputs |
| **Native** | Modify existing game UI elements (reposition, recolor, hide/show) |

Windows are constructed with C# object initializers and sent to one player or all players with a single `.Send()` call.

```csharp
new Window(player, "myplugin", "shop") {
  Width = 400, Height = 300,
  Background = UIBackground.FromColor(UIColor.Hex("#1a1a2e")),
  Border = new Border(UIColor.Hex("#444"), 1, 8),
  Padding = Spacing.All(12),
  Gap = 8,
  Children = {
    new Row {
      Children = {
        new Text { Content = "Shop", FontSize = 18 },
        new CloseButton()
      }
    },
    new Button {
      Label = "Buy Sword",
      Command = "shop buy sword",
      Background = UIBackground.FromColor(UIColor.Hex("#2a5c2a")),
      Width = "100%", Height = 36
    }
  }
}.Send();
```

## Quick Start

### 1. Create and send a window

```csharp
new Window(player, "myplugin", "hello") {
  Width = 300, Height = 100,
  Background = UIBackground.FromColor(UIColor.Hex("#222")),
  Padding = Spacing.All(12),
  Children = {
    new Text { Content = "Hello, World!", FontSize = 16 }
  }
}.Send();
```

### 2. Close a window

```csharp
InterfaceManager.CloseWindow(player, "myplugin", "hello");
```

### 3. React to button clicks

Buttons send a chat command when clicked. Register a handler on the server:

```csharp
InterfaceManager.OnCommand("shop.buy", (player, args) => {
  // args[0] = first argument after the command name
  Log.Message($"{player.Name} wants to buy: {args[0]}");
});
```

And on the button:

```csharp
new Button { Label = "Buy", Command = ".shop.buy sword" }
```

## Sections

- [Window](Interface/Window.md) — Window class, all properties, animations, and update methods
- [Elements](Interface/Elements.md) — Text, Button, Image, Input, Row, Accordion, ProgressBar, Dropdown, and more
- [Styling](Interface/Styling.md) — Colors, gradients, backgrounds, borders, spacing
- [Native Elements](Interface/NativeElements.md) — Modify existing game UI objects
- [Examples](Interface/Examples.md) — Full practical examples

## Requirements

- **ScarletInterface** must be installed on the client.
- Windows are silently ignored for players without the client mod.
- All API calls are server-side only.
