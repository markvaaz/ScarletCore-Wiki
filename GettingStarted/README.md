# Getting Started with ScarletCore

Welcome to the guide for creating V Rising mods with ScarletCore! This guide will teach you how to set up your development environment and create your first mod.

---

## Prerequisites

Before you begin, make sure you have installed:

- **.NET 6.0 SDK or later** — [Download here](https://dotnet.microsoft.com/download/dotnet/6.0)
- **V Rising Dedicated Server** — Required for testing your mods
- **Code Editor** — We recommend [Visual Studio 2022](https://visualstudio.microsoft.com/) or [Visual Studio Code](https://code.visualstudio.com/)
- **BepInEx 6.0+** — Modding framework for Unity ([installation guide](GettingStarted/BepInEx.md))

---

## What is ScarletCore?

**ScarletCore** is a comprehensive modding framework for V Rising that simplifies server-side mod development by providing:

- **High-level APIs** for player management, combat, inventory, and more
- **Command system** based on attributes with multi-language support
- **Event system** with priorities and filtering
- **Action scheduling** with time and frame control
- **Utilities** for logging, math, and text formatting
- **ECS Extensions** for fluent entity operations

---

## Getting Started

We'll use **ScarletTemplate** to create your mod quickly with all dependencies configured automatically.

### [Installing BepInEx](GettingStarted/BepInEx.md)
Install the modding framework required to run any V Rising server mod.

### [Quick Start with ScarletTemplate](GettingStarted/QuickStart.md)
Create a new mod project in minutes with the .NET template system.

---

## Next Steps

After creating your project:

1. [**Your First Mod**](GettingStarted/FirstMod.md) — Create commands, handle events, and save data
2. [**Practical Examples**](GettingStarted/Examples.md) — Learn from real examples
3. [**Best Practices**](GettingStarted/BestPractices.md) — Write clean and efficient code

---

## Useful Links

- [ScarletCore Repository](https://github.com/markvaaz/ScarletCore)
- [ScarletTemplate Repository](https://github.com/markvaaz/ScarletTemplate)
- [V Rising](https://store.steampowered.com/app/1604030/V_Rising/)
- [BepInEx](https://github.com/BepInEx/BepInEx)

---

## Need Help?

- [Report bugs or issues](https://github.com/markvaaz/ScarletCore/issues)
- [Report documentation issues](https://github.com/markvaaz/ScarletCore-Wiki/issues)
