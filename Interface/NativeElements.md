# Native Elements

The Native API lets you modify existing game UI elements directly — repositioning them, changing their color, swapping sprites, hiding them, or adding child objects — without replacing the game's own rendering.

All modifications are persistent: the server re-applies them automatically whenever the UI reloads (e.g. when the player opens a menu or reconnects).

## Overview

```csharp
// Target a single player
InterfaceManager.Native(player, "myplugin", "HUDCanvas/BottomBar/SomeElement")
  .SetPosition(100f, -50f)
  .SetActive(true)
  .Send();

// Broadcast to all players
InterfaceManager.NativeAll("myplugin", "HUDCanvas/BottomBar/SomeElement")
  .SetColorTint(UIColor.Hex("#ff4444"))
  .Send();

// Remove the persistent modification
InterfaceManager.NativeClear(player, "myplugin", "HUDCanvas/BottomBar/SomeElement");
InterfaceManager.NativeClearAll("myplugin", "HUDCanvas/BottomBar/SomeElement");
```

The **path** is the normalized Unity hierarchy path to the target GameObject, without `(Clone)` suffixes.

---

## NativeElementBuilder Methods

All methods return the builder so they can be chained. Call `.Send()` at the end.

### Visibility

```csharp
.SetActive(bool active)
```
Shows or hides the target GameObject.

```csharp
InterfaceManager.Native(player, "myplugin", "HUDCanvas/HealthBar")
  .SetActive(false)
  .Send();
```

---

### Position

```csharp
.SetPosition(float x, float y)   // set both X and Y
.SetPositionX(float x)           // set only X
.SetPositionY(float y)           // set only Y
```

Moves the element's `RectTransform` anchored position.

```csharp
InterfaceManager.Native(player, "myplugin", "HUDCanvas/HealthBar")
  .SetPosition(0f, 50f)   // move 50px up
  .Send();
```

---

### Relative positioning

```csharp
.SetPositionRelativeTo(
  string refPath,
  float offsetX = 0f, float offsetY = 0f,
  float selfWidthFactor = 0f, float selfHeightFactor = 0f
)
```

Positions the element relative to another element's anchored position.

```
Final position = (ref.x + offsetX + self.width * selfWidthFactor,
                  ref.y + offsetY + self.height * selfHeightFactor)
```

```csharp
// Place slot45 directly below slot4, with an 18px gap
InterfaceManager.Native(player, "myplugin", slot45Path)
  .SetPositionRelativeTo(slot4Path, offsetY: -18f, selfHeightFactor: -1f)
  .Send();
```

---

### Anchor (screen-relative)

```csharp
.SetAnchor(Anchor anchor, Dimension x = default, Dimension y = default, Pivot? pivot = null)
```

Pins the element to an anchor point of its parent (or screen for root elements).

```csharp
// Pin to screen top-left with a 10px inset
InterfaceManager.Native(player, "myplugin", "HUDCanvas/SomePanel")
  .SetAnchor(Anchor.TopLeft, x: 10f, y: -10f)
  .Send();
```

---

### Size

```csharp
.SetSizeDelta(float w, float h)   // set both width and height
.SetWidth(float w)                 // set only width
.SetHeight(float h)                // set only height
```

Sets the `RectTransform` `sizeDelta`.

---

### RectTransform (advanced)

```csharp
.SetAnchorMin(float x, float y)   // Unity anchorMin (0–1 range)
.SetAnchorMax(float x, float y)   // Unity anchorMax (0–1 range)
.SetPivot(float x, float y)       // Unity pivot (0–1 range)
```

---

### Layout control

```csharp
.DisableParentLayout(bool disable = true)
.DisableLayout(bool disable = true)
```

Disables (or re-enables) the `LayoutGroup` component on the **parent** or the **element itself**. Required when the parent is a `GridLayoutGroup`, `HorizontalLayoutGroup`, or `VerticalLayoutGroup` that would override your position changes.

```csharp
InterfaceManager.Native(player, "myplugin", "HUDCanvas/GridContainer/Slot_45")
  .DisableParentLayout()
  .SetPosition(100f, -200f)
  .Send();
```

---

### Opacity

```csharp
.SetOpacity(float alpha)   // 0 = fully transparent, 1 = fully opaque
```

Sets the opacity via a `CanvasGroup` component (added automatically if missing).

---

### Rotation

```csharp
.SetRotation(float degrees)   // -360 to 360
```

---

### Color tint

```csharp
.SetColorTint(UIColor color, bool deep = false)
```

Applies a multiplicative color tint to `Graphic` components (Image, RawImage, etc.) on the element. Set `deep: true` to also tint all descendants.

```csharp
InterfaceManager.Native(player, "myplugin", "HUDCanvas/HealthBar/Fill")
  .SetColorTint(UIColor.Hex("#ff4444"))
  .Send();
```

---

### Sprite replacement

```csharp
.SetSpriteUrl(string url)
```

Replaces the sprite on the element's `Image` component (or `RawImage` texture) with an image downloaded from the URL. The texture is cached for the session.

---

### Z-index

```csharp
.SetZIndex(int zIndex)
```

Sets the Canvas sorting order for this element. Adds a `Canvas` component with `overrideSorting = true` if one doesn't exist. Higher values render on top.

---

### Reparenting

```csharp
.ChangeParent(string newParentPath)
```

Moves the element to a different parent in the hierarchy.

```csharp
InterfaceManager.Native(player, "myplugin", "HUDCanvas/SomePanel")
  .ChangeParent("HUDCanvas/BottomBarCanvas/BottomBar/Content")
  .Send();
```

---

### Child elements

Create or update a named child inside the target element:

```csharp
.AddOrUpdateChild(string childName)   // returns a ChildElementBuilder
.DestroyChild(string childName)        // destroys a named child
```

`AddOrUpdateChild` returns a `ChildElementBuilder`. Chain operations on the child, then call `.Done()` to return to the parent builder.

```csharp
InterfaceManager.Native(player, "myplugin", "HUDCanvas/HealthBar")
  .AddOrUpdateChild("Label")
    .SetPosition(0f, 0f)
    .SetSizeDelta(100f, 20f)
    .AddText("100 / 200", fontSize: 12f, alignment: "Center")
    .Done()
  .Send();
```

**`ChildElementBuilder` methods:**

| Method | Description |
|--------|-------------|
| `.SetPosition(x, y)` | Anchored position |
| `.SetSizeDelta(w, h)` | Size delta |
| `.SetWidth(w)` / `.SetHeight(h)` | Individual size setters |
| `.SetAnchorMin(x, y)` | Unity anchorMin |
| `.SetAnchorMax(x, y)` | Unity anchorMax |
| `.SetPivot(x, y)` | Unity pivot |
| `.SetAnchor(anchor, x, y, pivot)` | Anchor-relative positioning |
| `.SetActive(bool)` | Show / hide |
| `.SetOpacity(alpha)` | Opacity via CanvasGroup |
| `.SetRotation(degrees)` | Rotation |
| `.SetSpriteUrl(url)` | Replace sprite texture |
| `.ChangeParent(path)` | Reparent the child |
| `.AddText(text, ...)` | Add/update a TextMeshProUGUI component |
| `.Done()` | Return to the parent `NativeElementBuilder` |

**`.AddText()` parameters:**

```csharp
.AddText(
  text:          "Hello",
  fontSize:      14f,
  fontStyle:     "Bold",       // "Normal" | "Bold" | "Italic" | "BoldAndItalic"
  alignment:     "Center",     // "Left" | "Center" | "Right" | "TopLeft" | "TopCenter" | …
  colorR:        1f,
  colorG:        1f,
  colorB:        1f,
  colorA:        1f,
  raycastTarget: false
)
```

---

### Send and Clear

```csharp
.Send()    // apply all queued operations to the target
.Clear()   // remove the persistent modification for this path
```

---

## Sprite Replacement (by sprite name)

Replace every `Image` component whose `sprite.name` matches a given name with a texture from a URL. Useful for re-skinning entire UI sections at once.

```csharp
// For a specific player
InterfaceManager.ReplaceSprite(player, "myplugin", "StatBG")
  .WithUrl("https://example.com/my-stat-bg.png")
  .Send();

// For all players
InterfaceManager.ReplaceSpriteAll("myplugin", "StatBG")
  .WithUrl("https://example.com/my-stat-bg.png")
  .Send();

// Remove the replacement
InterfaceManager.ReplaceSprite(player, "myplugin", "StatBG").Clear();
```

The client finds every `Image` with a matching sprite name and swaps it. The replacement is re-applied after UI reloads.

---

## Font Bundles

Load custom TMP fonts from a `.fonts.bin` bundle URL. After loading, fonts can be referenced by name in elements via the `Font` property.

```csharp
// Call once at plugin load — cached on disk per server
InterfaceManager.LoadFontBundleAll("myplugin", "https://example.com/fonts.bin");

// For a specific player only
InterfaceManager.LoadFontBundle(player, "myplugin", "https://example.com/fonts.bin");

// Then use the font by name
new Text { Content = "Hello", Font = "MyCustomFont SDF" }
```

---

## Image Pre-caching

Pre-download images to the client before any window uses them. Reduces load time when windows are first opened.

```csharp
// Call once at plugin load
InterfaceManager.PreCacheImages("myplugin", new[] {
  "https://example.com/icon1.png",
  "https://example.com/icon2.png",
  "https://example.com/bg.png"
});

// For a specific player
InterfaceManager.PreCacheImages(player, "myplugin", new[] {
  "https://example.com/icon1.png"
});
```

Images are stored per-server on disk and reused across sessions. Outdated images (size changed) are re-downloaded automatically.

---

## Event Callbacks

Register server-side handlers for UI interactions without using the full ScarletCore command system.

### OnMessage — raw chat prefix

```csharp
// Fires when a player sends a chat message starting with "mymod_confirm"
InterfaceManager.OnMessage("mymod_confirm", (player, args) => {
  // args = tokens after the prefix
  Log.Message($"{player.Name} confirmed with: {string.Join(", ", args)}");
});
```

### OnCommand — ScarletCore command name

```csharp
// Fires when a player runs the ".shop buy" command (via a button or chat)
InterfaceManager.OnCommand("shop.buy", (player, args) => {
  string item = args.Length > 0 ? args[0] : "";
  Log.Message($"{player.Name} wants to buy: {item}");
});
```
