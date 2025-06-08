# Project 9: Modular Game Plugin System

**Focus**: Dependency Injection & IoC  
**Time**: ~2 hours  
**Deliverable**: `PluginSystem` framework with dependency injection

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Dependency Injection Guidelines](../microsoft-design-guidelines.md#dependency-injection-patterns)** - IoC container patterns
- **[Interface Design](../microsoft-design-guidelines.md#interface-design)** - Plugin interface contracts
- **[Assembly Loading](../microsoft-design-guidelines.md#assembly-guidelines)** - Dynamic plugin loading

## Learning Objectives

- Master dependency injection and inversion of control (IoC)
- Understand service lifetimes and scope management
- Learn plugin architecture and modular design
- Practice interface segregation and dependency inversion
- Build extensible and testable systems

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Dependency Injection Guidelines](../microsoft-design-guidelines.md#dependency-injection-patterns) before starting

2. **Start Coding** 💻  
   See [detailed project guide](plugin-system-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-09-plugin-system/
├── README.md                    # This file
├── plugin-system-project-guide.md # Detailed implementation guide
├── src/                         # Your implementation
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
```

## What You'll Build

A comprehensive plugin system featuring:

- Dependency injection container from scratch
- Dynamic plugin discovery and loading
- Service registration and lifetime management
- Plugin configuration and metadata
- Inter-plugin communication through services
- Hot-swappable plugin architecture

## Key Concepts

- **Dependency Injection**: Constructor, property, and method injection
- **IoC Container**: Service registration and resolution
- **Service Lifetimes**: Singleton, transient, and scoped services
- **Plugin Architecture**: Modular and extensible design
- **Interface Segregation**: Small, focused interfaces
- **Assembly Loading**: Dynamic type discovery and instantiation

## Resources

- 📚 **[Detailed Guide](plugin-system-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Dependency Injection Documentation](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)**
- 📘 **[Assembly Loading Documentation](https://learn.microsoft.com/en-us/dotnet/standard/assembly/)**

## Building on Previous Projects

This project begins Phase 3 by building on all previous concepts:

- **Projects 1-8**: All previous skills in a larger architectural context
- **New Skills**: Dependency injection, IoC containers, and plugin architecture

## Example Usage

```csharp
// Service registration
var container = new GameContainer()
    .RegisterSingleton<ILogger, ConsoleLogger>()
    .RegisterTransient<IGameService, GameService>()
    .RegisterScoped<IPlayerService, PlayerService>();

// Plugin registration
var pluginManager = new PluginManager(container)
    .DiscoverPlugins("./plugins")
    .LoadPlugin<IWeaponPlugin>("LaserWeapon.dll")
    .LoadPlugin<IAudioPlugin>("SpatialAudio.dll");

// Plugin usage with DI
var gameEngine = container.Resolve<IGameEngine>();
var weaponPlugins = container.ResolveAll<IWeaponPlugin>();

foreach (var plugin in weaponPlugins)
{
    plugin.Initialize(gameEngine);
}
```

## Architecture Focus

This project emphasizes software architecture principles:

- SOLID principles in practice
- Modular design for extensibility
- Separation of concerns through interfaces
- Testability through dependency injection

## Contributing

When implementing this project:

1. Follow SOLID principles throughout the design
2. Create focused, single-responsibility interfaces
3. Implement proper service lifetime management
4. Design for testability and mockability
5. Document plugin contracts clearly
