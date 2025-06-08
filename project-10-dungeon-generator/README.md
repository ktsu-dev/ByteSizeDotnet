# Project 10: Procedural Dungeon Generator

**Focus**: Builder & Factory Patterns  
**Time**: ~2 hours  
**Deliverable**: `DungeonGenerator` library with creational design patterns

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Factory Pattern Guidelines](../microsoft-design-guidelines.md#factory-patterns)** - Object creation strategies
- **[Builder Pattern Guidelines](../microsoft-design-guidelines.md#builder-patterns)** - Complex object construction
- **[Design Pattern Usage](../microsoft-design-guidelines.md#design-patterns)** - Appropriate pattern selection

## Learning Objectives

- Master factory and builder design patterns
- Understand complex object construction strategies
- Learn procedural generation algorithms
- Practice fluent API design with builders
- Build configurable and extensible systems

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Factory Pattern Guidelines](../microsoft-design-guidelines.md#factory-patterns) before starting

2. **Start Coding** 💻  
   See [detailed project guide](dungeon-generator-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-10-dungeon-generator/
├── README.md                       # This file
├── dungeon-generator-project-guide.md # Detailed implementation guide
├── src/                            # Your implementation
├── tests/                          # Unit tests
└── docs/                           # Additional documentation
```

## What You'll Build

A sophisticated dungeon generation system featuring:

- Factory patterns for room and corridor creation
- Builder patterns for complex dungeon construction
- Configurable generation algorithms
- Fluent API for dungeon specification
- Plugin system for custom room types
- Export capabilities for different formats

## Key Concepts

- **Factory Method Pattern**: Creating objects without specifying exact classes
- **Abstract Factory Pattern**: Families of related objects
- **Builder Pattern**: Step-by-step construction of complex objects
- **Fluent Interfaces**: Method chaining for readable configuration
- **Strategy Pattern**: Pluggable algorithms for generation
- **Template Method**: Common generation workflow with customizable steps

## Resources

- 📚 **[Detailed Guide](dungeon-generator-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Design Patterns Documentation](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/)**
- 📘 **[Factory Pattern Examples](https://learn.microsoft.com/en-us/dotnet/api/system.activator)**

## Building on Previous Projects

This project advances Phase 3 architectural concepts:

- **Projects 1-8**: All foundational skills in pattern implementation
- **Project 9**: Dependency injection for factory registration
- **New Skills**: Creational design patterns and procedural generation

## Example Usage

```csharp
// Factory pattern for room creation
var roomFactory = new RoomFactory()
    .RegisterRoomType<TreasureRoom>("treasure")
    .RegisterRoomType<BossRoom>("boss")
    .RegisterRoomType<PuzzleRoom>("puzzle");

var treasureRoom = roomFactory.CreateRoom("treasure", size: new Size(10, 8));

// Builder pattern for dungeon construction
var dungeon = new DungeonBuilder()
    .WithSize(width: 100, height: 80)
    .WithDifficulty(DifficultyLevel.Hard)
    .WithRoomCount(15, 25)
    .AddRoomType("treasure", probability: 0.1f)
    .AddRoomType("boss", probability: 0.05f)
    .WithCorridorStyle(CorridorStyle.Winding)
    .WithTheme(DungeonTheme.Underground)
    .Build();

// Fluent configuration
var generator = DungeonGenerator.Create()
    .UseAlgorithm<MazeGenerationAlgorithm>()
    .ConfigureRooms(rooms => rooms
        .SetMinSize(4, 4)
        .SetMaxSize(12, 12)
        .AllowOverlapping(false))
    .ConfigureCorridors(corridors => corridors
        .SetWidth(2)
        .AllowBranching(true)
        .SetConnectionDensity(0.3f));

var result = await generator.GenerateAsync(seed: 12345);
```

## Pattern Focus

This project demonstrates multiple creational patterns:

- **Factory Method**: Different room type creation
- **Abstract Factory**: Themed dungeon element families
- **Builder**: Complex dungeon configuration
- **Prototype**: Cloning room templates

## Algorithm Examples

The project includes several generation algorithms:

- Binary Space Partitioning (BSP)
- Cellular Automata
- Maze Generation
- Voronoi Diagrams
- Wave Function Collapse

## Contributing

When implementing this project:

1. Use appropriate creational patterns for object construction
2. Design fluent interfaces for readable configuration
3. Implement pluggable algorithms through strategy pattern
4. Create comprehensive unit tests for generation logic
5. Document pattern usage and design decisions
