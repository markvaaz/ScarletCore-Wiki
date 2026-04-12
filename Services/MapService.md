# MapService

The `MapService` provides utility methods for revealing and hiding the in-game map for players, as well as querying world regions. It supports full map reveal, custom region control (radius and rectangle), advanced byte array pattern-based reveals, and region detection.

## Namespace
```csharp
using ScarletCore.Services;
```

## Overview

The `MapService` is a static utility class that provides:
- Full map reveal/hide functionality
- Circular area reveals (radius-based)
- Rectangular area reveals
- Custom map pattern reveals using byte arrays
- Precise pixel-level map control
- World region detection and display name lookup

## Table of Contents

- [Basic Operations](#basic-operations)
  - [RevealFullMap](#revealfullmap)
  - [HideFullMap](#hidefullmap)
- [Region-Based Reveals](#region-based-reveals)
  - [RevealMapRadius](#revealmapradius)
  - [RevealMapRectangle](#revealmaprectangle)
- [Advanced Operations](#advanced-operations)
  - [RevealMapFromByteArray](#revealmapfrombytearray)
- [World Regions](#world-regions)
  - [GetRegion](#getregion)
  - [GetRegionTypeByName](#getregiontypebyname)
  - [GetRegionDisplayName](#getregiondisplayname)
- [Technical Details](#technical-details)
- [Complete Examples](#complete-examples)
- [Best Practices](#best-practices)
- [Related Services](#related-services)

## Constants

The service uses several important constants for map calculations:

```csharp
MAP_CANVAS_SIZE = 256        // The client map canvas size (256x256 pixels)
BYTES_PER_ROW = 32           // 256 pixels per row, 8 pixels per byte = 32 bytes
BITS_PER_BYTE = 8            // Each byte represents 8 pixels
MAP_BUFFER_SIZE = 8192       // Total buffer size (256 rows × 32 bytes)
MAP_BOUNDS.Min = (-2880, 640)   // World coordinate bounds
MAP_BOUNDS.Max = (160, -2400)   // World coordinate bounds
MAP_SIZE = (3050, 3050)      // Map size in world units
```

## Methods

### Basic Operations

#### RevealFullMap
```csharp
public static void RevealFullMap(PlayerData playerData)
```

Reveals the entire map for a player.

**Parameters:**
- `playerData` (PlayerData): The player data object

**Example:**
```csharp
if (PlayerService.TryGetByName("DragonSlayer", out var player)) {
    MapService.RevealFullMap(player);
    MessageService.SendSuccess(player, "Full map revealed!");
}
```

**Notes:**
- Reveals all map zones for the player
- Automatically sends updated map data to the client
- Instant operation, no delay

---

#### HideFullMap
```csharp
public static void HideFullMap(PlayerData player)
```

Hides the entire map for a player, returning it to unexplored state.

**Parameters:**
- `player` (PlayerData): The player data object

**Example:**
```csharp
MapService.HideFullMap(player);
MessageService.SendWarning(player, "Map has been hidden!");
```

**Notes:**
- Clears all revealed areas
- Useful for resetting exploration progress
- Does not affect waypoints or other map markers

---

### Region-Based Reveals

#### RevealMapRadius
```csharp
public static void RevealMapRadius(PlayerData player, float3 centerPos, float radius)
```

Reveals a circular area of the map for a player.

**Parameters:**
- `player` (PlayerData): The player data object
- `centerPos` (float3): The center position in world coordinates
- `radius` (float): The radius in world units

**Example:**
```csharp
using Unity.Mathematics;

// Reveal 500 units around player's current position
var playerPos = player.Position;
MapService.RevealMapRadius(player, playerPos, 500f);

// Reveal area around a specific location
var castlePos = new float3(100f, 0f, 200f);
MapService.RevealMapRadius(player, castlePos, 1000f);

MessageService.SendInfo(player, "Map area revealed!");
```

**Notes:**
- Radius is in world units (game distance)
- Automatically converts world coordinates to map coordinates
- Can be called multiple times to reveal different areas
- Existing reveals are preserved (additive operation)

---

#### RevealMapRectangle
```csharp
public static void RevealMapRectangle(PlayerData player, float3 centerPos, float width, float height)
```

Reveals a rectangular area of the map for a player.

**Parameters:**
- `player` (PlayerData): The player data object
- `centerPos` (float3): The center position in world coordinates
- `width` (float): The width of the rectangle in world units
- `height` (float): The height of the rectangle in world units

**Example:**
```csharp
using Unity.Mathematics;

// Reveal a rectangular area
var centerPos = new float3(0f, 0f, 0f);
MapService.RevealMapRectangle(player, centerPos, 1000f, 500f);

// Reveal an area along a path
var pathStart = new float3(-500f, 0f, 0f);
MapService.RevealMapRectangle(player, pathStart, 2000f, 200f);

MessageService.SendInfo(player, "Rectangular area revealed!");
```

**Notes:**
- Width and height are in world units
- Rectangle is centered on the provided position
- Useful for revealing roads, paths, or specific zones
- Additive operation (existing reveals preserved)

---

### Advanced Operations

#### RevealMapFromByteArray
```csharp
public static void RevealMapFromByteArray(PlayerData player, byte[] mapData)
```

Reveals map areas based on a byte array pattern. This allows for custom, complex map reveal patterns.

**Parameters:**
- `player` (PlayerData): The player data object
- `mapData` (byte[]): Byte array representing the entire map (must be exactly 8192 bytes)

**Returns:**
- No return value (void)

**Example:**
```csharp
// Create a custom map pattern
byte[] customPattern = new byte[8192];

// Reveal specific areas (example: reveal top-left quadrant)
for (int i = 0; i < 2048; i++) {
    customPattern[i] = 0xFF; // Fully revealed
}

MapService.RevealMapFromByteArray(player, customPattern);

// Load pattern from file
byte[] savedPattern = System.IO.File.ReadAllBytes("custom_map.bin");
if (savedPattern.Length == 8192) {
    MapService.RevealMapFromByteArray(player, savedPattern);
}
```

**Byte Array Format:**
- **Size**: Exactly 8192 bytes (MAP_BUFFER_SIZE)
- **Structure**: 256 rows × 32 bytes per row
- **Bit Mapping**: Each byte represents 8 pixels
  - Bit 0 (LSB) = leftmost pixel
  - Bit 7 (MSB) = rightmost pixel
- **Values**: 
  - `0xFF` = all 8 pixels revealed
  - `0x00` = all 8 pixels hidden
  - Other values = partial reveal (specific bits set)

**Notes:**
- Array must be exactly 8192 bytes or the operation fails silently
- Additive operation (uses bitwise OR)
- Preserves existing revealed areas
- Useful for complex patterns, saved states, or procedural generation

---

## World Regions

Methods for detecting and querying named world regions.

### GetRegion
```csharp
public static WorldRegionType GetRegion(float3 pos)
```

Returns the `WorldRegionType` of the region that contains the specified world position, using polygon ray-casting.

**Parameters:**
- `pos` (float3): A world position to query

**Returns:**
- `WorldRegionType`: The region containing this position, or `WorldRegionType.None` if outside all regions

**Example:**
```csharp
var region = MapService.GetRegion(player.Position);
Log.Message($"Player is in region: {region}");
```

---

### GetRegionTypeByName
```csharp
public static WorldRegionType GetRegionTypeByName(string name)
```

Looks up a `WorldRegionType` by its lowercase string key.

**Parameters:**
- `name` (string): Case-insensitive region name key

**Returns:**
- `WorldRegionType`: The corresponding `WorldRegionType`, or `WorldRegionType.None` if not found

**Valid names:**

| Key | Region |
|-----|--------|
| `"farbane"` | Farbane Woods |
| `"dunley"` | Dunley Farmlands |
| `"forest"` | Cursed Forest |
| `"mountains"` | Hallowed Mountains |
| `"silverlight"` | Silverlight Hills |
| `"gloomrot"` | Gloomrot (combined) |
| `"gloomrotsouth"` | Gloomrot South |
| `"gloomrotnorth"` | Gloomrot North |
| `"mortium"` | Mortium |
| `"oakveil"` | Oakveil Woodlands |

**Example:**
```csharp
var regionType = MapService.GetRegionTypeByName("farbane");
var displayName = MapService.GetRegionDisplayName(regionType, Language.English);
// displayName = "Farbane Woods"
```

---

### GetRegionDisplayName
```csharp
public static string GetRegionDisplayName(WorldRegionType regionType, Language language)
```

Returns the localized display name of a region in the specified language.

**Parameters:**
- `regionType` (WorldRegionType): The region to get the name for
- `language` (Language): The `Language` enum value for the desired locale

**Returns:**
- `string`: The localized region display name

**Example:**
```csharp
var region = MapService.GetRegion(player.Position);
var regionName = MapService.GetRegionDisplayName(region, player.Language);
MessageService.Send(player, $"You are in {regionName}");
```

---

## Technical Details

### Map Coordinate System

The service uses a coordinate transformation system:

1. **World Coordinates**: Game world positions (x, y, z)
2. **Normalized Coordinates**: Transformed to map canvas space (0-256)
3. **Buffer Index**: Final position in the reveal buffer

**Transformation Process:**
```csharp
// World position → Normalized map position
float normalizedX = (worldX - MAP_BOUNDS.Min.x) / (MAP_BOUNDS.Max.x - MAP_BOUNDS.Min.x) * 256;
float normalizedZ = (worldZ - MAP_BOUNDS.Min.y) / (MAP_BOUNDS.Max.y - MAP_BOUNDS.Min.y) * 256;

// Normalized position → Buffer index
int pixelY = (int)normalizedZ;
int pixelX = (int)normalizedX;
int bufferRow = 255 - pixelY; // Inverted Y-axis
int byteCol = pixelX / 8;
int bufferIndex = bufferRow * 32 + byteCol;
```

### Pixel Packing

Each byte in the map buffer represents 8 horizontal pixels:

```
Byte value: 0b10110001
           ┌─┴─┬─┬─┬─┬─┬─┬─┐
Bit index: 7 6 5 4 3 2 1 0
Pixel:     □ ■ □ ■ ■ □ □ ■
           └─────────────┘
           Left → Right
```

### Map Buffer Layout

```
Buffer Index 0-31:     Row 255 (top of map)
Buffer Index 32-63:    Row 254
...
Buffer Index 8160-8191: Row 0 (bottom of map)
```

### Radius Conversion

World radius to map radius conversion:

```csharp
float worldScale = (3050 + 3050) / 2; // Average map size
float mapRadius = worldRadius * (256 / worldScale);
```

---

## Complete Examples

### Example 1: Map Reveal Command System

```csharp
using ScarletCore.Services;
using ScarletCore.Commanding;
using Unity.Mathematics;

public class MapRevealCommands {
    [Command("revealmap", description: "Reveal the entire map", adminOnly: true)]
    public static void RevealMapCommand(CommandContext ctx) {
        MapService.RevealFullMap(ctx.Player);
        MessageService.SendSuccess(ctx.Player, "Full map revealed!");
    }
    
    [Command("hidemap", description: "Hide the entire map", adminOnly: true)]
    public static void HideMapCommand(CommandContext ctx) {
        MapService.HideFullMap(ctx.Player);
        MessageService.SendWarning(ctx.Player, "Map has been hidden!");
    }
    
    [Command("revealradius", description: "Reveal map in a radius", adminOnly: true)]
    public static void RevealRadiusCommand(CommandContext ctx, float radius) {
        var pos = ctx.Player.Position;
        MapService.RevealMapRadius(ctx.Player, pos, radius);
        MessageService.SendSuccess(ctx.Player, $"Revealed {radius} units around you!");
    }
    
    [Command("revealat", description: "Reveal map at coordinates", adminOnly: true)]
    public static void RevealAtCommand(CommandContext ctx, float x, float z, float radius) {
        var pos = new float3(x, 0f, z);
        MapService.RevealMapRadius(ctx.Player, pos, radius);
        MessageService.SendSuccess(ctx.Player, $"Revealed area at ({x}, {z})!");
    }
}
```

### Example 2: Progressive Map Reveal System

```csharp
using ScarletCore.Services;
using ScarletCore.Systems;
using Unity.Mathematics;

public class ProgressiveMapReveal {
    private static Dictionary<ulong, float3> lastPositions = new();
    
    public static void OnPlayerMove(PlayerData player) {
        var currentPos = player.Position;
        
        // Check if player moved significantly
        if (lastPositions.TryGetValue(player.PlatformId, out var lastPos)) {
            var distance = math.distance(currentPos, lastPos);
            
            // Reveal map every 100 units traveled
            if (distance > 100f) {
                MapService.RevealMapRadius(player, currentPos, 150f);
                lastPositions[player.PlatformId] = currentPos;
            }
        } else {
            // First position
            lastPositions[player.PlatformId] = currentPos;
            MapService.RevealMapRadius(player, currentPos, 150f);
        }
    }
    
    public static void StartAutoReveal(PlayerData player) {
        ActionScheduler.RunRepeating(() => {
            if (player.IsConnected) {
                OnPlayerMove(player);
            }
        }, 5000); // Check every 5 seconds
    }
}
```

### Example 3: Zone-Based Map Reveal

```csharp
using Unity.Mathematics;

public class ZoneMapReveal {
    private static Dictionary<string, ZoneInfo> zones = new() {
        { "Farbane Woods", new ZoneInfo(new float3(-1500, 0, -1500), 800f) },
        { "Dunley Farmlands", new ZoneInfo(new float3(500, 0, -1000), 1000f) },
        { "Cursed Forest", new ZoneInfo(new float3(-500, 0, 500), 700f) },
        { "Silverlight Hills", new ZoneInfo(new float3(1000, 0, 1000), 900f) }
    };
    
    public static void RevealZone(PlayerData player, string zoneName) {
        if (zones.TryGetValue(zoneName, out var zone)) {
            MapService.RevealMapRadius(player, zone.Center, zone.Radius);
            MessageService.SendSuccess(player, $"Revealed {zoneName}!");
        } else {
            MessageService.SendError(player, $"Zone '{zoneName}' not found!");
        }
    }
    
    public static void RevealAllZones(PlayerData player) {
        foreach (var zone in zones) {
            MapService.RevealMapRadius(player, zone.Value.Center, zone.Value.Radius);
        }
        MessageService.SendSuccess(player, "All zones revealed!");
    }
}

public class ZoneInfo {
    public float3 Center { get; set; }
    public float Radius { get; set; }
    
    public ZoneInfo(float3 center, float radius) {
        Center = center;
        Radius = radius;
    }
}
```

### Example 4: Path Reveal System

```csharp
using Unity.Mathematics;

public class PathRevealSystem {
    public static void RevealPath(PlayerData player, List<float3> pathPoints, float width) {
        for (int i = 0; i < pathPoints.Count - 1; i++) {
            var start = pathPoints[i];
            var end = pathPoints[i + 1];
            
            // Calculate path segment
            var distance = math.distance(start, end);
            var steps = (int)(distance / 50f); // Reveal every 50 units
            
            for (int s = 0; s <= steps; s++) {
                var t = s / (float)steps;
                var pos = math.lerp(start, end, t);
                
                // Reveal small circles along the path
                MapService.RevealMapRadius(player, pos, width / 2);
            }
        }
        
        MessageService.SendSuccess(player, "Path revealed on map!");
    }
    
    public static void RevealRoad(PlayerData player, float3 start, float3 end) {
        // Calculate center and dimensions
        var center = (start + end) / 2;
        var direction = end - start;
        var length = math.length(direction);
        
        // Reveal as rectangle
        MapService.RevealMapRectangle(player, center, length, 100f);
        MessageService.SendSuccess(player, "Road revealed!");
    }
}
```

### Example 5: Custom Map Pattern Generator

```csharp
public class MapPatternGenerator {
    public static byte[] CreateCheckerboardPattern(int squareSize) {
        byte[] pattern = new byte[8192];
        
        for (int row = 0; row < 256; row++) {
            for (int byteCol = 0; byteCol < 32; byteCol++) {
                int bufferIndex = row * 32 + byteCol;
                byte value = 0;
                
                for (int bit = 0; bit < 8; bit++) {
                    int pixelX = byteCol * 8 + bit;
                    int pixelY = row;
                    
                    // Checkerboard logic
                    int squareX = pixelX / squareSize;
                    int squareY = pixelY / squareSize;
                    
                    if ((squareX + squareY) % 2 == 0) {
                        value |= (byte)(1 << bit);
                    }
                }
                
                pattern[bufferIndex] = value;
            }
        }
        
        return pattern;
    }
    
    public static byte[] CreateCircularPattern(float3 center, float radius) {
        byte[] pattern = new byte[8192];
        
        for (int row = 0; row < 256; row++) {
            for (int byteCol = 0; byteCol < 32; byteCol++) {
                int bufferIndex = row * 32 + byteCol;
                byte value = 0;
                
                for (int bit = 0; bit < 8; bit++) {
                    int pixelX = byteCol * 8 + bit;
                    int pixelY = row;
                    
                    float distance = math.distance(
                        new float2(pixelX, pixelY),
                        new float2(center.x, center.z)
                    );
                    
                    if (distance <= radius) {
                        value |= (byte)(1 << bit);
                    }
                }
                
                pattern[bufferIndex] = value;
            }
        }
        
        return pattern;
    }
    
    public static void ApplyCustomPattern(PlayerData player, byte[] pattern) {
        if (pattern.Length == 8192) {
            MapService.RevealMapFromByteArray(player, pattern);
            MessageService.SendSuccess(player, "Custom map pattern applied!");
        } else {
            MessageService.SendError(player, "Invalid pattern size!");
        }
    }
}
```

### Example 6: Map Reveal on Achievement

```csharp
public class AchievementMapReveals {
    public static void OnBossDefeated(PlayerData player, string bossName) {
        switch (bossName) {
            case "Alpha Wolf":
                RevealZoneAroundBoss(player, new float3(-1500, 0, -1500), 500f, "Farbane Woods");
                break;
            case "Lidia":
                RevealZoneAroundBoss(player, new float3(500, 0, -1000), 600f, "Dunley Farmlands");
                break;
            case "Solarus":
                // Reveal full map as final reward
                MapService.RevealFullMap(player);
                MessageService.SendSuccess(player, "Full map revealed for defeating Solarus!");
                return;
        }
    }
    
    private static void RevealZoneAroundBoss(PlayerData player, float3 position, float radius, string zoneName) {
        MapService.RevealMapRadius(player, position, radius);
        MessageService.SendSuccess(player, $"Map area revealed: {zoneName}!");
    }
}
```

---

## Best Practices

### 1. Use Appropriate Reveal Methods

```csharp
// Good - Use full reveal for admin commands
if (player.IsAdmin) {
    MapService.RevealFullMap(player);
}

// Good - Use radius for exploration
MapService.RevealMapRadius(player, player.Position, 200f);

// Good - Use rectangle for roads/paths
MapService.RevealMapRectangle(player, pathCenter, 1000f, 100f);
```

### 2. Validate Positions

```csharp
// Good - Check if position is within map bounds
bool IsValidPosition(float3 pos) {
    return pos.x >= -2880 && pos.x <= 160 &&
           pos.z >= -2400 && pos.z <= 640;
}

if (IsValidPosition(targetPos)) {
    MapService.RevealMapRadius(player, targetPos, radius);
}
```

### 3. Progressive Reveals

```csharp
// Good - Reveal gradually for better gameplay
public static void RevealProgressively(PlayerData player, float3 center, float maxRadius) {
    float currentRadius = 100f;
    
    ActionScheduler.RunRepeating(() => {
        MapService.RevealMapRadius(player, center, currentRadius);
        currentRadius += 100f;
        
        if (currentRadius > maxRadius) {
            return false; // Stop repeating
        }
        return true;
    }, 1000); // Every second
}
```

### 4. Byte Array Validation

```csharp
// Good - Always validate byte array size
public static bool ApplySafePattern(PlayerData player, byte[] pattern) {
    if (pattern == null || pattern.Length != 8192) {
        MessageService.SendError(player, "Invalid map pattern!");
        return false;
    }
    
    MapService.RevealMapFromByteArray(player, pattern);
    return true;
}
```

### 5. Combine Multiple Reveals

```csharp
// Good - Combine reveals for complex shapes
public static void RevealCrossPattern(PlayerData player, float3 center, float size) {
    // Horizontal bar
    MapService.RevealMapRectangle(player, center, size, size / 10);
    
    // Vertical bar
    MapService.RevealMapRectangle(player, center, size / 10, size);
    
    MessageService.SendInfo(player, "Cross pattern revealed!");
}
```

### 6. Performance Considerations

```csharp
// Good - Batch reveals when possible
public static void RevealMultipleAreas(PlayerData player, List<(float3 pos, float radius)> areas) {
    foreach (var area in areas) {
        MapService.RevealMapRadius(player, area.pos, area.radius);
    }
    // Only one map update sent at the end
}

// Avoid - Revealing with very high frequency
// Bad: Revealing every frame or multiple times per second
```

---

## Performance Notes

- **Full Map Reveal**: Fast operation (single buffer write)
- **Radius Reveals**: Performance depends on radius size
  - Small radius (< 500 units): Very fast
  - Medium radius (500-1500 units): Fast
  - Large radius (> 1500 units): May take a few milliseconds
- **Rectangle Reveals**: Generally faster than radius reveals for long paths
- **Byte Array**: Fast direct buffer write
- **Map Update**: Automatically sent once per operation

---

## Coordinate Reference

### World Bounds
```
Top-Left:     (-2880, 640)
Top-Right:    (160, 640)
Bottom-Left:  (-2880, -2400)
Bottom-Right: (160, -2400)
Center:       (-1360, -880)
```

### Common Locations (Example)
```csharp
var farbaneWoods = new float3(-1500, 0, -1500);
var dunleyFarmlands = new float3(500, 0, -1000);
var cursedForest = new float3(-500, 0, 500);
var silverlightHills = new float3(1000, 0, 1000);
```

---

## Troubleshooting

### Map Not Revealing
- Ensure player entity is valid and connected
- Check if position is within map bounds
- Verify radius/dimensions are positive values

### Byte Array Not Working
- Array must be exactly 8192 bytes
- Check for null array
- Ensure bytes are in correct format

### Partial Reveals
- Operations are additive (use bitwise OR)
- Use `HideFullMap` first if you want to reset
- Check coordinate transformations

---

## Related Services
- [PlayerService](PlayerService.md) - For player data retrieval
- [MessageService](MessageService.md) - For notifying players
- [TeleportService](TeleportService.md) - For position-based features
- [AdminService](AdminService.md) - For admin commands

## Notes
- Map reveals are persistent for the player's session
- Reveals are additive (existing reveals are preserved)
- Full map hide clears all revealed areas
- Map updates are sent automatically after each operation
- Thread-safe: Must be called from main thread only
