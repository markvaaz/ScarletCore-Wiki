# Your First Mod

Now that you have a configured project, let's create practical functionality using ScarletCore.

---

## What We'll Build

In this tutorial, you'll learn to:

1. **Create commands** for players to use in chat
2. **Use services** from ScarletCore to interact with the game
3. **Respond to events** from the game
4. **Save player data**

---

## Creating Commands

ScarletCore has an attribute-based command system that is simple and powerful.

### Basic Command: Hello World

Add this class inside your `Plugin.cs`:

```csharp
[CommandGroup("mymod", Language.English, adminOnly: false, aliases: new[] { "mm" })]
public static class MyModCommands
{
    [Command("hello", Language.English, description: "Say hello")]
    public static void HelloCommand(CommandContext ctx)
    {
        ctx.ReplySuccess($"Hello, {ctx.Sender.Name}! Welcome to my mod!");
    }
}
```

**How to use in-game:**
```
.mymod hello
```
or use the alias:
```
.mm hello
```

> **Note:** Use `aliases: []` for command shorthands. Use `CommandGroupAlias` only for multi-language support (e.g., Portuguese translation).

### Command with Parameters

Add a command that receives a name as parameter:

The system automatically converts player names or Steam ids to `PlayerData`:

```csharp
[Command("greet", Language.English, description: "Greet a player")]
public static void GreetCommand(CommandContext ctx, PlayerData player)
{
    if (player == null)
    {
        ctx.ReplyError("Player not found.");
        return;
    }

    MessageService.Send(player, "Hello from {ctx.Sender.Name}!");
    ctx.ReplySuccess($"Sending greetings to {playerName}!");
}
```

**How to use:**
```
.mymod greet John
```

### Admin-Only Command

To create commands only for administrators:

```csharp
[CommandGroup("admin", Language.English, adminOnly: true)]
public static class AdminCommands
{
    [Command("heal", Language.English, description: "Heal a player")]
    public static void HealCommand(CommandContext ctx, PlayerData player)
    {
        // Player has been automatically validated
        var entity = player.Character.Entity;
        
        // Fully heal the player
        if (entity.TryGetComponent(out Health health))
        {
            health.Value = health.MaxHealth;
            ctx.ReplySuccess($"{player.Name} has been healed!");
        }
    }
}
```

### Multi-Language Commands

Use `CommandGroupAlias` and `CommandAlias` for multi-language support:

```csharp
[CommandGroup("teleport", Language.English, adminOnly: false, aliases: new[] { "tp" })]
[CommandGroupAlias("teletransporte", Language.Portuguese, aliases: new[] { "tt" })]
public static class TeleportCommands
{
    [Command("home", Language.English, description: "Teleport to your home")]
    [CommandAlias("casa", Language.Portuguese, description: "Teleportar para sua casa")]
    public static void HomeCommand(CommandContext ctx)
    {
        // Teleport logic
        ctx.ReplySuccess("Teleported home!");
    }
}
```

**Usage:**
- English: `.teleport home` or `.tp home`
- Portuguese: `.teletransporte casa` or `.tt casa`

---

## 🔧 Using Services

ScarletCore provides high-level services for common tasks.

### MessageService: Sending Messages

```csharp
using ScarletCore.Services;

[Command("announce", Language.English, description: "Send announcement")]
public static void AnnounceCommand(CommandContext ctx, string message)
{
    // Send to all players
    MessageService.SendAll(message);
    
    // Send only to command author
    MessageService.Send(ctx.Sender, "Announcement sent!");
}
```

### PlayerService: Managing Players

```csharp
[Command("online", Language.English, description: "List online players")]
public static void OnlineCommand(CommandContext ctx)
{
    var onlinePlayers = PlayerService.GetOnlinePlayers();
    
    ctx.ReplySuccess($"Online players: {onlinePlayers.Count()}");
    
    foreach (var player in onlinePlayers)
    {
        ctx.Reply($"- {player.Name} (Level {player.Level})");
    }
}
```

### TeleportService: Teleportation

```csharp
[Command("tp", Language.English, description: "Teleport to coordinates")]
public static void TeleportCommand(CommandContext ctx, float3 position)
{
    if (TeleportService.Teleport(ctx.Sender.CharacterEntity, position))
    {
        ctx.ReplySuccess($"Teleported to ({x}, {z})!");
    }
    else
    {
        ctx.ReplyError("Failed to teleport.");
    }
}
```

**Usage:**
- `.tp 100,10,200`

### BuffService: Applying Buffs

```csharp
using ProjectM;

[Command("buff", Language.English, description: "Apply a buff")]
public static void BuffCommand(CommandContext ctx, PlayerData player, PrefabGUID buffGuid, float duration)
{
    BuffService.ApplyBuff(player.Character.Entity, buffGuid, duration);
    
    ctx.ReplySuccess($"Buff applied to {player.Name} for {duration} seconds!");
}
```

---

## Responding to Events

Events allow you to react to game actions. You need to subscribe to events using `EventManager.On()`.

### Subscribing to Events

In your `Initialize()` method in `Plugin.cs`, subscribe to events:

```csharp
private void Initialize()
{
    // Subscribe to player join/leave events
    EventManager.On(PlayerEvents.PlayerJoined, OnPlayerJoined);
    EventManager.On(PlayerEvents.PlayerLeft, OnPlayerLeft);
    
    // Subscribe to death event
    EventManager.On(PrefixEvents.OnDeath, OnEntityDeath);
    
    // Subscribe to damage event
    EventManager.On(PrefixEvents.OnDealDamage, OnDealDamage);
    
    Log.Info("Events registered!");
}
```

### Event: When Player Joins/Leaves

Handle player connections:

```csharp
private static void OnPlayerJoined(PlayerData player)
{
    MessageService.SendAll($"🎉 {player.Name} joined the server!");
    Log.Info($"Player connected: {player.Name}");
}

private static void OnPlayerLeft(PlayerData player)
{
    MessageService.SendAll($"👋 {player.Name} left the server.");
    Log.Info($"Player disconnected: {player.Name}");
}
```

### Event: When Entity Dies

Handle death of any entity (players, NPCs, etc.):

```csharp
private static void OnEntityDeath(NativeArray<Entity> entities)
{
    foreach (var entity in entities)
    {
        var ev = entity.Read<DeathEvent>();
        var killer = ev.Killer;
        var killed = ev.Died;

        if (!killed.IsPlayer()) continue;

        var playerData = killed.GetPlayerData();

        if (playerData == null) continue;
        
        MessageService.SendAll($"💀 {playerData.Name} has died");
        
        // You can add custom logic here
        // e.g., track deaths, apply penalties, etc.
    }
}
```

### Event: Custom Event Handler Class

Organize events in a separate class:

```csharp
// Events/GameEventHandler.cs
using ScarletCore.Events;
using ScarletCore.Services;
using Unity.Collections;
using Unity.Entities;

namespace MyMod.Events;

public static class GameEventHandler
{
    public static void Initialize()
    {
        EventManager.On(PlayerEvents.PlayerJoined, OnPlayerJoined);
        EventManager.On(PlayerEvents.PlayerLeft, OnPlayerLeft);
        EventManager.On(PrefixEvents.OnDeath, OnDeath);
    }
    
    private static void OnPlayerJoined(PlayerData player)
    {
        // Handle player join
    }
    
    private static void OnPlayerLeft(PlayerData player)
    {
        // Handle player leave
    }
    
    private static void OnDeath(NativeArray<Entity> entities)
    {
        // Handle deaths
    }
    
    public static void Cleanup()
    {
        EventManager.Off(PlayerEvents.PlayerJoined, OnPlayerJoined);
        EventManager.Off(PlayerEvents.PlayerLeft, OnPlayerLeft);
        EventManager.Off(PrefixEvents.OnDeath, OnDeath);
    }
}
```

Then call it from your main `Initialize()`:

```csharp
private void Initialize()
{
    GameEventHandler.Initialize();
}
```

### Available Event Types

ScarletCore provides several event categories:

- **PlayerEvents** - `PlayerJoined`, `PlayerLeft`, `CharacterCreated`
- **ServerEvents** - `OnInitialize`, `OnSave`
- **PrefixEvents** - `OnDeath`, `OnDealDamage`, and more

Check the [Event Types documentation](Events/EventTypes.md) and [Event Examples](Events/Examples.md) for complete list and usage patterns.

> **Important:** 
> - Always subscribe to events in the `Initialize()` method, not in `Load()`
> - Use `EventManager.Off()` or `EventManager.UnregisterAssembly()` to cleanup when unloading
> - Prefix events receive `NativeArray<Entity>`, player events receive `PlayerData`

---

## Saving Data

Use ScarletCore's database system to persist information.

### Define Data Model

Create `Data/PlayerStats.cs`:

```csharp
namespace MyMod.Data;

public class PlayerStats
{
    public ulong PlatformId { get; set; }
    public string Name { get; set; }
    public int TotalKills { get; set; }
    public int TotalDeaths { get; set; }
    public DateTime FirstJoin { get; set; }
    public DateTime LastSeen { get; set; }
}
```

### Save and Load Data

In `Plugin.cs`, add:

```csharp
using MyMod.Data;

// Save player data
public static void SavePlayerStats(PlayerData player, int kills, int deaths)
{
    var stats = new PlayerStats
    {
        PlatformId = player.PlatformId,
        Name = player.Name,
        TotalKills = kills,
        TotalDeaths = deaths,
        LastSeen = DateTime.Now
    };
    
    Database.Set($"player_{player.PlatformId}", stats);
}

// Load player data
public static PlayerStats GetPlayerStats(ulong PlatformId)
{
    return Database.Get<PlayerStats>($"player_{PlatformId}");
}
```

### Command to View Stats

```csharp
[Command("stats", Language.English, description: "View your stats")]
public static void StatsCommand(CommandContext ctx)
{
    var stats = Plugin.GetPlayerStats(ctx.Sender.PlatformId);
    
    if (stats == null)
    {
        ctx.ReplyError("No statistics found.");
        return;
    }
    
    ctx.ReplySuccess("📊 Your Statistics:");
    ctx.Reply($"Kills: {stats.TotalKills}");
    ctx.Reply($"Deaths: {stats.TotalDeaths}");
    ctx.Reply($"K/D Ratio: {(float)stats.TotalKills / Math.Max(1, stats.TotalDeaths):F2}");
    ctx.Reply($"First joined: {stats.FirstJoin:MM/dd/yyyy}");
}
```

---

## Scheduling Actions

Execute code with precise timing control using `ActionScheduler`. It supports time-based and frame-based scheduling.

### Basic Scheduling

#### Execute After Delay

Use `Delayed()` to run code once after a time delay:

```csharp
[Command("bomb", Language.English, description: "Plant a bomb")]
public static void BombCommand(CommandContext ctx)
{
    ctx.ReplySuccess("Bomb planted! Exploding in 5 seconds...");
    
    ActionScheduler.Delayed(() => {
        MessageService.SendAll($"💥 {ctx.Sender.Name}'s bomb exploded!");
    }, 5f);
}
```

#### Execute on Next Frame

Use `NextFrame()` to defer an operation by one frame:

```csharp
[Command("spawn", Language.English, description: "Spawn an enemy")]
public static void SpawnCommand(CommandContext ctx, PrefabGUID enemyGUID)
{
    var position = ctx.Sender.CharacterEntity.Read<Translation>().Value;
    var entity = SpawnerService.ImmediateSpawn(enemyGUID, position);
    
    // Modify on next frame to ensure entity is fully initialized
    ActionScheduler.NextFrame(() => {
        if (entity.Exists())
        {
            // Apply modifications safely
            StatModifierService.ApplyModifiers(entity, modifiers);
        }
    });
    
    ctx.ReplySuccess("Enemy spawned!");
}
```

### Repeating Actions

#### Periodic Execution

Use `Repeating()` for actions that repeat at time intervals:

```csharp
private void Initialize()
{
    // Auto-save every 5 minutes
    ActionScheduler.Repeating(() => {
        SaveAllPlayerData();
        MessageService.SendAll("🎮 Server auto-saved!");
    }, 300f); // 300 seconds = 5 minutes
    
    // Limited repeats (10 times only)
    ActionScheduler.Repeating(() => {
        MessageService.SendAll("Wave incoming!");
        SpawnWave();
    }, 30f, maxExecutions: 10);
}
```

#### Random Intervals

Use `RepeatingRandom()` for randomized timing:

### Self-Cancelling Actions

Actions can cancel themselves using the cancel callback:

```csharp
[Command("regen", Language.English, description: "Start regeneration")]
public static void RegenCommand(CommandContext ctx)
{
    ctx.ReplySuccess("Regeneration started! Lasts until full health.");
    
    ActionScheduler.Repeating(cancel => {
        var health = ctx.Sender.CharacterEntity.Read<Health>();
        
        if (health.Value >= health.MaxHealth)
        {
            MessageService.Send(ctx.Sender, "✨ Fully healed!");
            cancel(); // Stop repeating
            return;
        }
        
        // Heal 10 HP
        health.Value = Math.Min(health.Value + 10, health.MaxHealth);
        ctx.Sender.CharacterEntity.Write(health);
        
    }, 1f); // Every second
}
```

### Action Control

You can control scheduled actions:

```csharp
// Store the action ID
private static ActionId _regenerationId;

[Command("startregen", Language.English, description: "Start continuous regeneration")]
public static void StartRegenCommand(CommandContext ctx)
{
    _regenerationId = ActionScheduler.Repeating(() => {
        var players = PlayerService.GetAllConnected();
        foreach (var player in players)
        {
            BuffService.TryApplyBuff(player.CharacterEntity, healBuffGUID);
        }
    }, 5f);
    
    ctx.ReplySuccess("Regeneration aura started!");
}

[Command("stopregen", Language.English, description: "Stop regeneration")]
public static void StopRegenCommand(CommandContext ctx)
{
    if (ActionScheduler.CancelAction(_regenerationId))
    {
        ctx.ReplySuccess("Regeneration aura stopped!");
    }
    else
    {
        ctx.ReplyError("No active regeneration aura.");
    }
}
```

### Countdown Example

Use `CreateSequence()` to chain actions with delays:

```csharp
[Command("countdown", Language.English, description: "Start a countdown")]
public static void CountdownCommand(CommandContext ctx, int seconds)
{
    ctx.ReplySuccess($"Countdown started: {seconds} seconds");
    
    // Build the countdown sequence
    var sequence = ActionScheduler.CreateSequence();
    
    // Add countdown steps
    for (int i = seconds; i > 0; i--)
    {
        int count = i; // Capture for closure
        sequence = sequence
            .Then(() => MessageService.SendAll($"⏰ {count}..."))
            .ThenWait(1f);
    }
    
    // Add final message
    sequence = sequence.Then(() => MessageService.SendAll("🎉 GO!"));
    
    // Execute the sequence
    sequence.Execute();
}
```

Or use a more complex sequence with different wait times:

```csharp
[Command("event", Language.English, description: "Start a timed event")]
public static void EventCommand(CommandContext ctx)
{
    ActionScheduler.CreateSequence()
        .Then(() => MessageService.SendAll("🎮 Event starting soon..."))
        .ThenWait(5f)
        .Then(() => MessageService.SendAll("⏰ 10 seconds!"))
        .ThenWait(5f)
        .Then(() => MessageService.SendAll("⏰ 5 seconds!"))
        .ThenWait(3f)
        .Then(() => MessageService.SendAll("⏰ 2 seconds!"))
        .ThenWait(2f)
        .Then(() => {
            MessageService.SendAll("🎉 EVENT STARTED!");
            StartEvent();
        })
        .Execute();
    
    ctx.ReplySuccess("Event sequence started!");
}
```

### Cleanup

Always clean up scheduled actions when your mod unloads:

```csharp
public override bool Unload()
{
    // Clear all scheduled actions from this assembly
    ActionScheduler.UnregisterAssembly();
    
    _harmony?.UnpatchSelf();
    EventManager.UnregisterAssembly();
    CommandHandler.UnregisterAssembly();
    return true;
}
```

> **Note:** Check the [ActionScheduler documentation](Systems/ActionScheduler.md) for advanced features like frame-based scheduling, action sequences, and more control options.

---

## ✅ Complete Mod: Points System

Let's combine everything into a functional points system.

### 1. Data Model

```csharp
// Data/PlayerPoints.cs
public class PlayerPoints
{
    public ulong PlatformId { get; set; }
    public string Name { get; set; }
    public int Points { get; set; }
}
```

### 2. Points Manager

```csharp
// Managers/PointsManager.cs
public static class PointsManager
{
    public static void AddPoints(PlayerData player, int amount)
    {
        var data = Plugin.Database.Get<PlayerPoints>($"points_{player.PlatformId}") 
                   ?? new PlayerPoints { PlatformId = player.PlatformId, Name = player.Name };
        
        data.Points += amount;
        Plugin.Database.Set($"points_{player.PlatformId}", data);
        
        MessageService.Send(player, $"💰 +{amount} points! Total: {data.Points}");
    }
    
    public static int GetPoints(PlayerData player)
    {
        var data = Plugin.Database.Get<PlayerPoints>($"points_{player.PlatformId}");
        return data?.Points ?? 0;
    }
}
```

### 3. Points Events

Subscribe to events to award points:

```csharp
// In Plugin.cs Initialize() method
private void Initialize()
{
    EventManager.On(PrefixEvents.OnDeath, OnEntityDeath);
    EventManager.On(PlayerEvents.PlayerJoined, OnPlayerJoined);
}

private static void OnEntityDeath(NativeArray<Entity> entities)
{
    foreach (var entity in entities)
    {
        var ev = entity.Read<DeathEvent>();
        var killer = ev.Killer;
        var killed = ev.Died;
        
        // Skip if killed entity is a player (we only want NPC kills)
        if (killed.IsPlayer()) continue;
        
        // Check if killer is a player
        if (!killer.IsPlayer()) continue;
        
        var killerData = killer.GetPlayerData();
        if (killerData == null) continue;
        
        // Award points for killing an NPC
        PointsManager.AddPoints(killerData, 10);
    }
}

private static void OnPlayerJoined(PlayerData player)
{
    // Award daily login bonus
    var lastSeen = Plugin.Database.Get<DateTime>($"lastseen_{player.PlatformId}");
    var today = DateTime.Now.Date;
    
    if (lastSeen.Date != today)
    {
        PointsManager.AddPoints(player, 50);
        MessageService.Send(player, "🎁 Daily login bonus: +50 points!");
    }
    
    Plugin.Database.Set($"lastseen_{player.PlatformId}", DateTime.Now);
}
```

### 4. Points Commands

```csharp
[CommandGroup("points", Language.English)]
public static class PointsCommands
{
    [Command("balance", Language.English, description: "Check your points")]
    public static void BalanceCommand(CommandContext ctx)
    {
        int points = PointsManager.GetPoints(ctx.Sender);
        ctx.ReplySuccess($"💰 You have {points} points");
    }
    
    [Command("top", Language.English, description: "View top players")]
    public static void TopCommand(CommandContext ctx)
    {
        // Implement ranking
        ctx.ReplySuccess("🏆 Top 10 Players");
        // ... list top players
    }
}
```

---

## Next Steps

Congratulations! You've created your first functional mod. Keep learning:

- [**Practical Examples**](GettingStarted/Examples.md) — More mod examples
- [**Best Practices**](GettingStarted/BestPractices.md) — How to write clean code
- [**Service Documentation**](Services/) — Complete APIs
- [**Event System**](Events/) — All available events
