# Quick Start with ScarletTemplate

**ScarletTemplate** is a .NET template that automatically creates the complete structure of a V Rising mod with all dependencies configured.

---

## Installing the Template

Open a terminal and run:

```bash
dotnet new install ScarletTemplate
```

This will install the template globally on your system.

---

## Creating Your First Mod

Create a new mod project:

```bash
dotnet new scarlettemplate -n MyFirstMod
```

This creates a project with:
- **ScarletCore** — Main framework with commands, events, and services
- **BepInEx** — Plugin system
- **V Rising Bindings** — Game APIs

---

## Generated Structure

After creating the project, you'll have:

```
MyFirstMod/
├── MyFirstMod.csproj    # Project file
├── MyFirstMod.sln       # Visual Studio solution
├── Plugin.cs            # Main plugin class
├── MyPluginInfo.cs      # Auto-generated metadata
├── nuget.config         # NuGet package sources
└── LICENSE              # AGPL-3.0 License
```

---

## Opening the Project

### Visual Studio 2022

1. Double-click `MyFirstMod.sln`
2. Wait for Visual Studio to load dependencies
3. Ready to develop!

### Visual Studio Code

```bash
cd MyFirstMod
code .
```

---

## Building the Mod

In the terminal, inside the project folder:

```bash
dotnet build
```

Or use the shortcut in Visual Studio: **Ctrl+Shift+B**

The compiled mod will be automatically copied to:
```
C:\Program Files (x86)\Steam\steamapps\common\VRisingDedicatedServer\BepInEx\plugins\
```

> **Note:** The installation path can be customized in the `.csproj` file

---

## Testing the Mod

1. **Start the V Rising Dedicated Server**
2. **Check the BepInEx logs** to confirm the mod was loaded:
   ```
   [Info: Plugin MyFirstMod] Plugin com.author.myfirstmod version 1.0.0 is loaded!
   ```
3. **Join the server** and test the functionality

---

## Basic Plugin Structure

The `Plugin.cs` file contains the basic structure:

```csharp
using System.Reflection;
using BepInEx;
using BepInEx.Logging;
using BepInEx.Unity.IL2CPP;
using HarmonyLib;
using ScarletCore.Commanding;
using ScarletCore.Data;
using ScarletCore.Events;
using ScarletCore.Systems;

namespace ScarletTemplate;

[BepInPlugin(MyPluginInfo.PLUGIN_GUID, MyPluginInfo.PLUGIN_NAME, MyPluginInfo.PLUGIN_VERSION)]
[BepInDependency("markvaaz.ScarletCore")]
public class Plugin : BasePlugin
{
    static Harmony _harmony;
    public static Harmony Harmony => _harmony;
    public static Plugin Instance { get; private set; }
    public static ManualLogSource LogInstance { get; private set; }
    public static Settings Settings { get; private set; }
    public static Database Database { get; private set; }

    public override void Load()
    {
        Instance = this;
        LogInstance = Log;

        Log.LogInfo($"Plugin {MyPluginInfo.PLUGIN_GUID} version {MyPluginInfo.PLUGIN_VERSION} is loaded!");

        _harmony = new Harmony(MyPluginInfo.PLUGIN_GUID);
        _harmony.PatchAll(Assembly.GetExecutingAssembly());

        Settings = new Settings(MyPluginInfo.PLUGIN_GUID, Instance);
        Database = new Database(MyPluginInfo.PLUGIN_GUID);
        GameSystems.OnInitialize(Initialize);
        CommandHandler.RegisterAll();
    }

    private void Initialize()
    {
      // Your initialization code here
    }

    public override bool Unload()
    {
        _harmony?.UnpatchSelf();
        ActionScheduler.UnregisterAssembly();
        EventManager.UnregisterAssembly();
        CommandHandler.UnregisterAssembly();
        return true;
    }
}
```

---

## Next Steps

Now that you have a working project:

- [**Create Commands**](GettingStarted/FirstMod.md#creating-commands) — Add in-game commands
- [**Use Services**](GettingStarted/FirstMod.md#using-services) — Access game APIs
- [**Handle Events**](GettingStarted/FirstMod.md#responding-to-events) — Respond to game actions
- [**View Examples**](GettingStarted/Examples.md) — Learn from real code

---


## Troubleshooting

### Mod doesn't appear in logs

1. Verify the `.dll` file is in the `BepInEx/plugins` folder
2. Confirm BepInEx is installed correctly
3. Check the BepInEx logs at `BepInEx/LogOutput.log`

### Dependency errors

```bash
dotnet restore
```

### Wrong installation path

Edit the `.csproj` file and adjust the `GameDirectory` property:

```xml
<PropertyGroup>
  <GameDirectory>C:\YOUR\PATH\VRisingDedicatedServer</GameDirectory>
</PropertyGroup>
```


