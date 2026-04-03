# Interface Examples

A collection of practical, copy-paste examples for common UI patterns.

## Table of Contents

- [Simple notification](#simple-notification)
- [Shop window](#shop-window)
- [Player stats HUD](#player-stats-hud)
- [Confirmation dialog](#confirmation-dialog)
- [Form with inputs](#form-with-inputs)
- [Scrollable list](#scrollable-list)
- [Accordion settings panel](#accordion-settings-panel)
- [Updating elements in real-time](#updating-elements-in-real-time)
- [Animated button background](#animated-button-background)
- [Inline icons in text](#inline-icons-in-text)
- [Portrait card](#portrait-card)
- [Modifying native UI](#modifying-native-ui)

---

## Simple notification

A brief centered notification that closes after the player clicks it.

```csharp
new Window(player, "myplugin", "notify") {
  Width = 320,
  Background = UIBackground.FromColor(UIColor.Hex("#1e1e2e")),
  Border = new Border(UIColor.Hex("#555"), 1, 8),
  Padding = Spacing.All(16),
  Gap = 10,
  OpenAnimation = WindowAnimation.Scale,
  AnimationDuration = 0.2f,
  Children = {
    new Text {
      Content = "You have been promoted to Officer!",
      FontSize = 14,
      TextAlign = TextAlignment.Center,
      Wrap = true,
      Width = "100%"
    },
    new Button {
      Label = "OK",
      Command = ".notify_close",
      Width = "100%", Height = 34,
      Background = UIBackground.FromColor(UIColor.Hex("#3a3a7a")),
      HoverBackground = UIBackground.FromColor(UIColor.Hex("#5a5aaa"))
    }
  }
}.Send();

// Close on button press
InterfaceManager.OnCommand("notify_close", (p, _) => {
  InterfaceManager.CloseWindow(p, "myplugin", "notify");
});
```

---

## Shop window

A shop with multiple rows of items.

```csharp
void OpenShop(PlayerData player) {
  var items = new[] {
    ("Iron Sword", "sword", 50),
    ("Leather Armor", "armor", 80),
    ("Health Potion", "potion", 20),
  };

  var rows = new List<UIElement>();
  foreach (var (name, id, price) in items) {
    rows.Add(new Row {
      Height = 40,
      AlignItems = AlignItems.Center,
      Gap = 8,
      Padding = new Spacing(vertical: 4, horizontal: 8),
      Children = {
        new Text { Content = name, Width = "60%", FontSize = 13 },
        new Text {
          Content = $"{price}g",
          Width = 50,
          TextAlign = TextAlignment.Center,
          TextColor = UIColor.Hex("#f0c040")
        },
        new Button {
          Label = "Buy",
          Command = $".shop buy {id}",
          Width = 60, Height = 28,
          Background = UIBackground.FromColor(UIColor.Hex("#2a5c2a")),
          HoverBackground = UIBackground.FromColor(UIColor.Hex("#3a7c3a")),
          FontSize = 12
        }
      }
    });
  }

  var window = new Window(player, "myplugin", "shop") {
    Width = 380,
    Background = UIBackground.FromColor(UIColor.Hex("#1a1a2e")),
    Border = new Border(UIColor.Hex("#444"), 1, 8),
    Gap = 4,
    OpenAnimation = WindowAnimation.SlideDown
  };

  // Header row
  window.Add(new Row {
    Height = 40,
    AlignItems = AlignItems.Center,
    Padding = new Spacing(vertical: 0, horizontal: 12),
    Background = UIBackground.FromColor(UIColor.Hex("#2a2a4a")),
    Children = {
      new Text { Content = "Shop", FontSize = 16, Width = "100%" },
      new CloseButton()
    }
  });

  foreach (var row in rows)
    window.Add(row);

  window.Send();
}

// Handle purchase
InterfaceManager.OnCommand("shop.buy", (player, args) => {
  var item = args[0];
  // process purchase...
  InterfaceManager.CloseWindow(player, "myplugin", "shop");
});
```

---

## Player stats HUD

A persistent HUD widget that stays on screen and updates in real-time using `ElemId`.

```csharp
void ShowHud(PlayerData player, int hp, int maxHp) {
  new Window(player, "myplugin", "hud") {
    Anchor = Anchor.TopLeft,
    Position = new Position(x: 20, y: -20),
    Width = 200,
    Background = UIBackground.FromColor(UIColor.RGBA(0f, 0f, 0f, 0.6f)),
    Border = new Border(UIColor.Hex("#555"), 1, 6),
    Padding = Spacing.All(8),
    Gap = 4,
    Draggable = false,
    HideOnMenuOpen = false,
    Children = {
      new Text { Content = "Health", FontSize = 11, TextColor = UIColor.Hex("#aaa") },
      new ProgressBar {
        ElemId = "hp-bar",
        Value = hp, Max = maxHp,
        Width = "100%", Height = 10,
        BarFill = UIBackground.FromColor(UIColor.Hex("#c0392b")),
        Background = UIBackground.FromColor(UIColor.Hex("#333")),
        Border = new Border(UIColor.Transparent, 0, 5),
        AnimateValue = true
      },
      new Text {
        ElemId = "hp-text",
        Content = $"{hp} / {maxHp}",
        FontSize = 11,
        TextAlign = TextAlignment.Right,
        Width = "100%"
      }
    }
  }.Send();
}

// Update the bar and text without re-sending the full window
void UpdateHud(PlayerData player, int hp, int maxHp) {
  var win = new Window(player, "myplugin", "hud");
  win.SendUpdate("hp-bar", new ProgressBar { Value = hp, Max = maxHp, AnimateValue = true });
  win.SendUpdate("hp-text", new Text { Content = $"{hp} / {maxHp}", FontSize = 11,
    TextAlign = TextAlignment.Right, Width = "100%" });
}
```

---

## Confirmation dialog

A two-button "Are you sure?" dialog.

```csharp
void ShowConfirmDialog(PlayerData player, string action, string confirmCmd, string cancelCmd) {
  new Window(player, "myplugin", "confirm") {
    Width = 300,
    Background = UIBackground.FromColor(UIColor.Hex("#1e1e2e")),
    Border = new Border(UIColor.Hex("#666"), 1, 8),
    Padding = Spacing.All(16),
    Gap = 12,
    OpenAnimation = WindowAnimation.Fade,
    Children = {
      new Text {
        Content = $"Are you sure you want to {action}?",
        Wrap = true, Width = "100%",
        TextAlign = TextAlignment.Center, FontSize = 13
      },
      new Row {
        Gap = 8,
        JustifyContent = JustifyContent.Center,
        Children = {
          new Button {
            Label = "Confirm",
            Command = confirmCmd,
            Width = 110, Height = 34,
            Background = UIBackground.FromColor(UIColor.Hex("#7a2a2a")),
            HoverBackground = UIBackground.FromColor(UIColor.Hex("#aa3a3a"))
          },
          new Button {
            Label = "Cancel",
            Command = cancelCmd,
            Width = 110, Height = 34,
            Background = UIBackground.FromColor(UIColor.Hex("#3a3a3a")),
            HoverBackground = UIBackground.FromColor(UIColor.Hex("#555"))
          }
        }
      }
    }
  }.Send();
}
```

---

## Form with inputs

A form that collects data from multiple input fields and submits them together in one button command.

```csharp
new Window(player, "myplugin", "create-clan") {
  Width = 360,
  Background = UIBackground.FromColor(UIColor.Hex("#1a1a2e")),
  Border = new Border(UIColor.Hex("#444"), 1, 8),
  Padding = Spacing.All(16),
  Gap = 10,
  Children = {
    new Row {
      AlignItems = AlignItems.Center,
      Children = {
        new Text { Content = "Create Clan", FontSize = 16, Width = "100%" },
        new CloseButton()
      }
    },
    new Text { Content = "Clan Name", FontSize = 12, TextColor = UIColor.Hex("#aaa") },
    new Input {
      Id = "clanname",
      Placeholder = "Enter clan name...",
      Width = "100%", Height = 32,
      Background = UIBackground.FromColor(UIColor.Hex("#252545")),
      Border = new Border(UIColor.Hex("#555"), 1, 4),
      FocusBorder = new Border(UIColor.Hex("#7070ff"), 1, 4),
      TextColor = UIColor.White,
      FontSize = 13,
      MaxLength = 20
    },
    new Text { Content = "Motto (optional)", FontSize = 12, TextColor = UIColor.Hex("#aaa") },
    new Input {
      Id = "motto",
      Placeholder = "Enter a motto...",
      Width = "100%", Height = 32,
      Background = UIBackground.FromColor(UIColor.Hex("#252545")),
      Border = new Border(UIColor.Hex("#555"), 1, 4),
      FocusBorder = new Border(UIColor.Hex("#7070ff"), 1, 4),
      TextColor = UIColor.White,
      FontSize = 13,
      MaxLength = 50
    },
    new Button {
      Label = "Create",
      // {clanname} and {motto} are replaced with the current input values
      Command = ".clan create {clanname} {motto}",
      Width = "100%", Height = 36,
      Background = UIBackground.FromColor(UIColor.Hex("#3a3a8a")),
      HoverBackground = UIBackground.FromColor(UIColor.Hex("#5a5aaa"))
    }
  }
}.Send();
```

---

## Scrollable list

A window with a tall scrollable content area.

```csharp
var entries = GetOnlinePlayers(); // returns List<PlayerData>

var rows = new List<UIElement>();
foreach (var p in entries) {
  rows.Add(new Row {
    Height = 36,
    AlignItems = AlignItems.Center,
    Gap = 8,
    Padding = new Spacing(vertical: 4, horizontal: 8),
    Children = {
      new Text { Content = p.Name, Width = "100%", FontSize = 13 },
      new Button {
        Label = "Kick",
        Command = $".admin kick {p.Name}",
        Width = 50, Height = 24,
        Background = UIBackground.FromColor(UIColor.Hex("#7a2a2a")),
        FontSize = 11
      }
    }
  });
}

var contentWindow = new Window(player, "myplugin", "player-list") {
  Width = 320,
  Background = UIBackground.FromColor(UIColor.Hex("#1a1a2e")),
  Border = new Border(UIColor.Hex("#444"), 1, 8),
  Gap = 4
};

// Fixed header
contentWindow.Add(new Row {
  Height = 40,
  AlignItems = AlignItems.Center,
  Padding = Spacing.Horizontal(12),
  Background = UIBackground.FromColor(UIColor.Hex("#2a2a4a")),
  Children = {
    new Text { Content = "Online Players", FontSize = 15, Width = "100%" },
    new CloseButton()
  }
});

// Scrollable container — a row that is itself scrollable
contentWindow.Add(new Row {
  Height = 300,
  Overflow = OverflowMode.ScrollY,
  ScrollbarColor = UIColor.Hex("#7070ff"),
  ScrollbarBackgroundColor = UIColor.Hex("#333"),
  ScrollbarWidth = 6,
  // Wrap items in a column-like structure using nested rows
  Children = rows
    .Select(r => r)
    .ToList()
    // For a true vertical list, wrap each row in another row
    // or use JustifyContent.Start (default)
});

contentWindow.Add(new Row {}); // padding at bottom

contentWindow.Send();
```

---

## Accordion settings panel

A panel where each section is collapsible.

```csharp
new Window(player, "myplugin", "settings") {
  Width = 380,
  Background = UIBackground.FromColor(UIColor.Hex("#1a1a2e")),
  Border = new Border(UIColor.Hex("#444"), 1, 8),
  Gap = 4,
  Padding = Spacing.All(8),
  Children = {
    new Text { Content = "Settings", FontSize = 18, Padding = new Spacing(0, 0, 8, 0) },

    new Accordion {
      Title = "Gameplay",
      Expanded = true,
      HeaderBackground = UIBackground.FromColor(UIColor.Hex("#2a2a4a")),
      Gap = 6,
      Children = {
        new Row {
          AlignItems = AlignItems.Center,
          Children = {
            new Text { Content = "Difficulty", Width = "60%", FontSize = 13 },
            new Dropdown {
              Id = "diff",
              Options = "Easy:easy|Normal:normal|Hard:hard",
              Command = ".settings set difficulty {diff}",
              Width = "40%", Height = 28, FontSize = 12
            }
          }
        }
      }
    },

    new Accordion {
      Title = "Notifications",
      Expanded = false,
      HeaderBackground = UIBackground.FromColor(UIColor.Hex("#2a2a4a")),
      Gap = 6,
      Children = {
        new Row {
          AlignItems = AlignItems.Center,
          Children = {
            new Text { Content = "Kill notifications", Width = "70%", FontSize = 13 },
            new Button {
              Label = "Toggle",
              Command = ".settings toggle kill_notify",
              Height = 26, Padding = Spacing.Horizontal(12), FontSize = 12,
              Background = UIBackground.FromColor(UIColor.Hex("#3a3a6a"))
            }
          }
        }
      }
    }
  }
}.Send();
```

---

## Updating elements in real-time

Send partial updates to an open window without re-building it.

```csharp
// Initial send
new Window(player, "myplugin", "status") {
  Width = 180,
  Anchor = Anchor.TopRight,
  Position = new Position(x: -20, y: -20),
  Background = UIBackground.FromColor(UIColor.RGBA(0, 0, 0, 0.7f)),
  Padding = Spacing.All(8), Gap = 4,
  Draggable = false,
  Children = {
    new Row { Children = {
      new Text { ElemId = "status-label", Content = "Status: Online", FontSize = 12 }
    }}
  }
}.Send();

// Later — update just the label
Window.SendUpdate(player, "myplugin", "status", "status-label",
  new Text { Content = "Status: In Combat", FontSize = 12, TextColor = UIColor.Red });

// Remove it entirely
Window.SendDelete(player, "myplugin", "status", "status-label");

// Close the window
InterfaceManager.CloseWindow(player, "myplugin", "status");
```

---

## Animated button background

```csharp
new Button {
  Label = "Start",
  Command = ".game start",
  Width = 160, Height = 44,
  Background = UIBackground.AnimatedFromUrls(new[] {
    "https://example.com/btn_idle.png",
    "https://example.com/btn_glow1.png",
    "https://example.com/btn_glow2.png"
  }, duration: 1.2f)
    .WithColor(UIColor.Hex("#1a1a3a")),  // fallback while loading
  HoverBackground = UIBackground.AnimatedFromUrls(new[] {
    "https://example.com/btn_hover1.png",
    "https://example.com/btn_hover2.png"
  }, duration: 0.4f)
}
```

---

## Inline icons in text

Embed in-game item icons inside any text content.

```csharp
// UIIcons.Icon() takes the PrefabGUID integer hash
var ironIngot = -2128818369;  // find these in the Prefabs Browser

new Text {
  Content = $"Recipe: {UIIcons.Icon(ironIngot, 20f)} × 5 Iron Ingot",
  FontSize = 14
}

// Icon with spacing
new Button {
  Label = $"{UIIcons.Icon(ironIngot, 18f, 4f)} Craft",
  Command = ".craft iron_sword"
}
```

---

## Portrait card

Display the player's 3D character inside a UI card.

```csharp
new Window(player, "myplugin", "profile") {
  Width = 260,
  Background = UIBackground.FromColor(UIColor.Hex("#1a1a2e")),
  Border = new Border(UIColor.Hex("#555"), 1, 8),
  Padding = Spacing.All(12),
  Gap = 8,
  Children = {
    new PortraitCamera {
      Width = "100%", Height = 200,
      FieldOfView = 45,
      OrbitAngle = 10,
      Distance = 1.1f,
      BackgroundColor = UIColor.Hex("#111"),
      Border = new Border(UIColor.Hex("#444"), 1, 4)
    },
    new Text {
      Content = player.Name,
      FontSize = 16,
      TextAlign = TextAlignment.Center,
      Width = "100%"
    },
    new CloseButton()
  }
}.Send();
```

---

## Modifying native UI

Move, recolor, or hide parts of the game's own UI.

```csharp
// Hide the minimap for this player
InterfaceManager.Native(player, "myplugin", "HUDMenuParent/Minimap")
  .SetActive(false)
  .Send();

// Replace a UI sprite with a custom image for all players
InterfaceManager.ReplaceSpriteAll("myplugin", "StatBG")
  .WithUrl("https://example.com/custom-stat-bg.png")
  .Send();

// Move a native element and disable its parent layout group
InterfaceManager.Native(player, "myplugin",
    "HUDMenuParent/CharacterMenu/SubMenu/InventoryMenu/MenuParent/" +
    "CharacterInventorySubMenu/MotionRoot/EquipmentTab/Slot_Head")
  .DisableParentLayout()
  .SetPosition(100f, -200f)
  .Send();

// Remove the modification
InterfaceManager.NativeClear(player, "myplugin",
  "HUDMenuParent/CharacterMenu/SubMenu/InventoryMenu/MenuParent/" +
  "CharacterInventorySubMenu/MotionRoot/EquipmentTab/Slot_Head");
```
