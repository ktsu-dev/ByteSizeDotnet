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
├── DungeonGenerator.sln         # Solution file
├── DungeonGenerator.Core/       # Main library implementation
├── DungeonGenerator.App/        # Console application
└── DungeonGenerator.Tests/      # Unit tests
├── tests/                          # Unit tests
└── docs/                           # Additional documentation
```

## What You'll Build

A sophisticated dungeon generation system featuring:

- **Factory Method Pattern** - Creating different room types (treasure, boss, puzzle rooms)
- **Abstract Factory Pattern** - Themed dungeon element families (medieval, sci-fi, horror)
- **Builder Pattern** - Step-by-step dungeon construction with fluent API
- **Configurable Algorithms** - BSP, cellular automata, maze generation
- **Room Templates** - Reusable room designs with inheritance
- **Export System** - Multiple output formats (JSON, ASCII art, game data)
- **Validation System** - Ensuring generated dungeons meet criteria
- **Statistics & Analytics** - Room distribution, connectivity analysis

## Key Concepts

### Factory Method Pattern

```csharp
// Create different room types without specifying concrete classes
IRoomFactory factory = new ThemeRoomFactory(DungeonTheme.Medieval);
IRoom treasureRoom = factory.CreateRoom(RoomType.Treasure, new Size(8, 6));
```

### Abstract Factory Pattern

```csharp
// Create families of related objects
IDungeonElementFactory factory = new MedievalDungeonFactory();
IRoom room = factory.CreateRoom();
ICorridor corridor = factory.CreateCorridor();
ITrap trap = factory.CreateTrap();
```

### Builder Pattern

```csharp
// Step-by-step construction with fluent interface
var dungeon = new DungeonBuilder()
    .SetSize(100, 80)
    .SetTheme(DungeonTheme.Underground)
    .AddRoomType<TreasureRoom>(probability: 0.1f)
    .SetAlgorithm<BSPGenerationAlgorithm>()
    .Build();
```

### Core Design Patterns Applied

- **Factory Method** - Abstracting room creation
- **Abstract Factory** - Themed element families
- **Builder** - Complex dungeon configuration
- **Strategy** - Pluggable generation algorithms
- **Template Method** - Common generation workflow
- **Observer** - Generation progress notifications

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-10-dungeon-generator/` directory:

```bash
# Create the main solution file
dotnet new sln -n DungeonGenerator

# Create the main library project
dotnet new classlib -n DungeonGenerator.Core -f net8.0

# Create the console application
dotnet new console -n DungeonGenerator.App -f net8.0

# Create the test project
dotnet new mstest -n DungeonGenerator.Tests -f net8.0

# Add projects to solution
dotnet sln add DungeonGenerator.Core/DungeonGenerator.Core.csproj
dotnet sln add DungeonGenerator.App/DungeonGenerator.App.csproj
dotnet sln add DungeonGenerator.Tests/DungeonGenerator.Tests.csproj

# Add project references
dotnet add DungeonGenerator.App/DungeonGenerator.App.csproj reference DungeonGenerator.Core/DungeonGenerator.Core.csproj
dotnet add DungeonGenerator.Tests/DungeonGenerator.Tests.csproj reference DungeonGenerator.Core/DungeonGenerator.Core.csproj
```

### Step 2: Configure Project Files

Update the `DungeonGenerator.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>DungeonGenerator.Core</PackageId>
    <Title>Procedural Dungeon Generator</Title>
    <Description>A library for generating dungeons using creational design patterns</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your dungeon generation code:

```
DungeonGenerator.Core/
├── Models/           # Core data models (Room, Corridor, Dungeon)
├── Factories/        # Factory pattern implementations
├── Builders/         # Builder pattern implementations
├── Algorithms/       # Generation algorithm strategies
├── Themes/           # Abstract factory themes
├── Validators/       # Dungeon validation logic
└── Exporters/        # Export format implementations
```

Create these directories:

```bash
mkdir DungeonGenerator.Core/Models
mkdir DungeonGenerator.Core/Factories
mkdir DungeonGenerator.Core/Builders
mkdir DungeonGenerator.Core/Algorithms
mkdir DungeonGenerator.Core/Themes
mkdir DungeonGenerator.Core/Validators
mkdir DungeonGenerator.Core/Exporters
```

## Implementation Guide

### Step 4: Core Data Models (25 minutes)

**Foundation Types and Enums**

```csharp
namespace DungeonGenerator.Core.Models;

public enum RoomType
{
    Basic,
    Treasure,
    Boss,
    Puzzle,
    Shop,
    Rest,
    Secret,
    Trap
}

public enum DungeonTheme
{
    Medieval,
    SciFi,
    Horror,
    Fantasy,
    Modern,
    Underground
}

public enum ConnectionType
{
    Door,
    Archway,
    SecretPassage,
    Portal,
    Stairs
}

public readonly struct Position : IEquatable<Position>
{
    public int X { get; }
    public int Y { get; }

    public Position(int x, int y)
    {
        X = x;
        Y = y;
    }

    public bool Equals(Position other) => X == other.X && Y == other.Y;
    public override bool Equals(object? obj) => obj is Position other && Equals(other);
    public override int GetHashCode() => HashCode.Combine(X, Y);
    public override string ToString() => $"({X}, {Y})";

    public static Position operator +(Position a, Position b) => new(a.X + b.X, a.Y + b.Y);
    public static Position operator -(Position a, Position b) => new(a.X - b.X, a.Y - b.Y);
}

public readonly struct Size : IEquatable<Size>
{
    public int Width { get; }
    public int Height { get; }

    public Size(int width, int height)
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(width);
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(height);
        Width = width;
        Height = height;
    }

    public int Area => Width * Height;
    public bool Equals(Size other) => Width == other.Width && Height == other.Height;
    public override bool Equals(object? obj) => obj is Size other && Equals(other);
    public override int GetHashCode() => HashCode.Combine(Width, Height);
    public override string ToString() => $"{Width}x{Height}";
}

public readonly struct Rectangle : IEquatable<Rectangle>
{
    public Position Position { get; }
    public Size Size { get; }

    public Rectangle(Position position, Size size)
    {
        Position = position;
        Size = size;
    }

    public Rectangle(int x, int y, int width, int height)
        : this(new Position(x, y), new Size(width, height)) { }

    public int Left => Position.X;
    public int Top => Position.Y;
    public int Right => Position.X + Size.Width;
    public int Bottom => Position.Y + Size.Height;
    public Position Center => new(Position.X + Size.Width / 2, Position.Y + Size.Height / 2);

    public bool Contains(Position point) =>
        point.X >= Left && point.X < Right && point.Y >= Top && point.Y < Bottom;

    public bool Intersects(Rectangle other) =>
        Left < other.Right && Right > other.Left && Top < other.Bottom && Bottom > other.Top;

    public bool Equals(Rectangle other) => Position.Equals(other.Position) && Size.Equals(other.Size);
    public override bool Equals(object? obj) => obj is Rectangle other && Equals(other);
    public override int GetHashCode() => HashCode.Combine(Position, Size);
}
```

**Room and Corridor Models**

```csharp
namespace DungeonGenerator.Core.Models;

public interface IRoom
{
    string Id { get; }
    RoomType Type { get; }
    Rectangle Bounds { get; }
    DungeonTheme Theme { get; }
    IReadOnlyList<IConnection> Connections { get; }
    IDictionary<string, object> Properties { get; }

    void AddConnection(IConnection connection);
    bool CanConnect(IRoom other);
    IRoom Clone();
}

public interface IConnection
{
    string Id { get; }
    ConnectionType Type { get; }
    Position Position { get; }
    IRoom FromRoom { get; }
    IRoom ToRoom { get; }
    bool IsPassable { get; set; }
}

public abstract class Room : IRoom
{
    private readonly List<IConnection> _connections = new();

    protected Room(string id, RoomType type, Rectangle bounds, DungeonTheme theme)
    {
        Id = id ?? throw new ArgumentNullException(nameof(id));
        Type = type;
        Bounds = bounds;
        Theme = theme;
        Properties = new Dictionary<string, object>();
    }

    public string Id { get; }
    public RoomType Type { get; }
    public Rectangle Bounds { get; }
    public DungeonTheme Theme { get; }
    public IReadOnlyList<IConnection> Connections => _connections;
    public IDictionary<string, object> Properties { get; }

    public virtual void AddConnection(IConnection connection)
    {
        ArgumentNullException.ThrowIfNull(connection);
        if (!_connections.Contains(connection))
        {
            _connections.Add(connection);
        }
    }

    public virtual bool CanConnect(IRoom other)
    {
        ArgumentNullException.ThrowIfNull(other);

        // Basic connectivity rules
        if (other == this) return false;
        if (Connections.Any(c => c.ToRoom == other || c.FromRoom == other)) return false;

        // Check if rooms are adjacent
        return IsAdjacent(other);
    }

    private bool IsAdjacent(IRoom other)
    {
        var thisBounds = Bounds;
        var otherBounds = other.Bounds;

        // Check if rooms share a wall
        bool horizontallyAdjacent =
            (thisBounds.Right == otherBounds.Left || thisBounds.Left == otherBounds.Right) &&
            !(thisBounds.Bottom <= otherBounds.Top || thisBounds.Top >= otherBounds.Bottom);

        bool verticallyAdjacent =
            (thisBounds.Bottom == otherBounds.Top || thisBounds.Top == otherBounds.Bottom) &&
            !(thisBounds.Right <= otherBounds.Left || thisBounds.Left >= otherBounds.Right);

        return horizontallyAdjacent || verticallyAdjacent;
    }

    public abstract IRoom Clone();

    public override string ToString() => $"{Type} Room '{Id}' at {Bounds}";
}

// Concrete room implementations
public class BasicRoom : Room
{
    public BasicRoom(string id, Rectangle bounds, DungeonTheme theme)
        : base(id, RoomType.Basic, bounds, theme) { }

    public override IRoom Clone() => new BasicRoom(Id + "_copy", Bounds, Theme);
}

public class TreasureRoom : Room
{
    public TreasureRoom(string id, Rectangle bounds, DungeonTheme theme)
        : base(id, RoomType.Treasure, bounds, theme)
    {
        Properties["TreasureValue"] = 100;
        Properties["HasChest"] = true;
    }

    public int TreasureValue
    {
        get => (int)Properties["TreasureValue"];
        set => Properties["TreasureValue"] = value;
    }

    public bool HasChest
    {
        get => (bool)Properties["HasChest"];
        set => Properties["HasChest"] = value;
    }

    public override IRoom Clone() => new TreasureRoom(Id + "_copy", Bounds, Theme)
    {
        TreasureValue = TreasureValue,
        HasChest = HasChest
    };
}

public class BossRoom : Room
{
    public BossRoom(string id, Rectangle bounds, DungeonTheme theme)
        : base(id, RoomType.Boss, bounds, theme)
    {
        Properties["BossLevel"] = 5;
        Properties["HasSpecialLoot"] = true;
    }

    public int BossLevel
    {
        get => (int)Properties["BossLevel"];
        set => Properties["BossLevel"] = value;
    }

    public override bool CanConnect(IRoom other)
    {
        // Boss rooms should have limited connections
        return Connections.Count < 2 && base.CanConnect(other);
    }

    public override IRoom Clone() => new BossRoom(Id + "_copy", Bounds, Theme)
    {
        BossLevel = BossLevel
    };
}
```

### Step 5: Factory Method Pattern (30 minutes)

**Room Factory Interface and Implementation**

```csharp
namespace DungeonGenerator.Core.Factories;

// Factory Method Pattern
public interface IRoomFactory
{
    IRoom CreateRoom(RoomType type, Rectangle bounds);
    IRoom CreateRoom(RoomType type, Position position, Size size);
    bool CanCreateRoom(RoomType type);
    IEnumerable<RoomType> SupportedRoomTypes { get; }
}

public class StandardRoomFactory : IRoomFactory
{
    private readonly DungeonTheme _theme;
    private readonly Random _random;
    private int _roomCounter = 0;

    public StandardRoomFactory(DungeonTheme theme, Random? random = null)
    {
        _theme = theme;
        _random = random ?? new Random();
    }

    public IEnumerable<RoomType> SupportedRoomTypes => new[]
    {
        RoomType.Basic, RoomType.Treasure, RoomType.Boss, RoomType.Puzzle
    };

    public IRoom CreateRoom(RoomType type, Rectangle bounds)
    {
        var id = GenerateRoomId(type);

        return type switch
        {
            RoomType.Basic => new BasicRoom(id, bounds, _theme),
            RoomType.Treasure => CreateTreasureRoom(id, bounds),
            RoomType.Boss => CreateBossRoom(id, bounds),
            RoomType.Puzzle => CreatePuzzleRoom(id, bounds),
            _ => throw new ArgumentException($"Unsupported room type: {type}")
        };
    }

    public IRoom CreateRoom(RoomType type, Position position, Size size) =>
        CreateRoom(type, new Rectangle(position, size));

    public bool CanCreateRoom(RoomType type) => SupportedRoomTypes.Contains(type);

    private string GenerateRoomId(RoomType type) => $"{type}_{++_roomCounter:D3}";

    private TreasureRoom CreateTreasureRoom(string id, Rectangle bounds)
    {
        var room = new TreasureRoom(id, bounds, _theme);

        // Customize based on theme
        switch (_theme)
        {
            case DungeonTheme.Medieval:
                room.TreasureValue = _random.Next(50, 200);
                room.Properties["HasGoldCoins"] = true;
                break;
            case DungeonTheme.SciFi:
                room.TreasureValue = _random.Next(100, 300);
                room.Properties["HasTechArtifacts"] = true;
                break;
        }

        return room;
    }

    private BossRoom CreateBossRoom(string id, Rectangle bounds)
    {
        var room = new BossRoom(id, bounds, _theme);
        room.BossLevel = _random.Next(3, 8);
        return room;
    }

    private IRoom CreatePuzzleRoom(string id, Rectangle bounds)
    {
        // For now, return a basic room with puzzle properties
        var room = new BasicRoom(id, bounds, _theme);
        room.Properties["PuzzleType"] = "Riddle";
        room.Properties["Difficulty"] = _random.Next(1, 5);
        return room;
    }
}

// Specialized factory for themed rooms
public class ThemedRoomFactory : IRoomFactory
{
    private readonly DungeonTheme _theme;
    private readonly IRoomFactory _baseFactory;

    public ThemedRoomFactory(DungeonTheme theme)
    {
        _theme = theme;
        _baseFactory = new StandardRoomFactory(theme);
    }

    public IEnumerable<RoomType> SupportedRoomTypes => _baseFactory.SupportedRoomTypes;

    public IRoom CreateRoom(RoomType type, Rectangle bounds)
    {
        var room = _baseFactory.CreateRoom(type, bounds);
        ApplyThemeDecorations(room);
        return room;
    }

    public IRoom CreateRoom(RoomType type, Position position, Size size) =>
        CreateRoom(type, new Rectangle(position, size));

    public bool CanCreateRoom(RoomType type) => _baseFactory.CanCreateRoom(type);

    private void ApplyThemeDecorations(IRoom room)
    {
        switch (_theme)
        {
            case DungeonTheme.Medieval:
                room.Properties["WallMaterial"] = "Stone";
                room.Properties["Lighting"] = "Torches";
                break;
            case DungeonTheme.SciFi:
                room.Properties["WallMaterial"] = "Metal";
                room.Properties["Lighting"] = "LED Panels";
                break;
            case DungeonTheme.Horror:
                room.Properties["WallMaterial"] = "Decaying Wood";
                room.Properties["Lighting"] = "Flickering Candles";
                room.Properties["Atmosphere"] = "Creepy";
                break;
        }
    }
}
```

### Step 6: Abstract Factory Pattern (35 minutes)

**Dungeon Element Abstract Factory**

```csharp
namespace DungeonGenerator.Core.Themes;

// Abstract Factory Pattern - Creates families of related objects
public interface IDungeonElementFactory
{
    IRoom CreateRoom(RoomType type, Rectangle bounds);
    IConnection CreateConnection(ConnectionType type, Position position, IRoom from, IRoom to);
    ITrap CreateTrap(string type, Position position);
    IDecoration CreateDecoration(string type, Position position);
    DungeonTheme Theme { get; }
}

public interface ITrap
{
    string Id { get; }
    string Type { get; }
    Position Position { get; }
    int Damage { get; }
    bool IsActive { get; set; }
}

public interface IDecoration
{
    string Id { get; }
    string Type { get; }
    Position Position { get; }
    IDictionary<string, object> Properties { get; }
}

// Medieval theme factory
public class MedievalDungeonFactory : IDungeonElementFactory
{
    private readonly IRoomFactory _roomFactory;
    private int _elementCounter = 0;

    public MedievalDungeonFactory()
    {
        _roomFactory = new StandardRoomFactory(DungeonTheme.Medieval);
    }

    public DungeonTheme Theme => DungeonTheme.Medieval;

    public IRoom CreateRoom(RoomType type, Rectangle bounds) => _roomFactory.CreateRoom(type, bounds);

    public IConnection CreateConnection(ConnectionType type, Position position, IRoom from, IRoom to)
    {
        return type switch
        {
            ConnectionType.Door => new MedievalDoor(GenerateId("Door"), position, from, to),
            ConnectionType.Archway => new MedievalArchway(GenerateId("Archway"), position, from, to),
            ConnectionType.SecretPassage => new MedievalSecretPassage(GenerateId("Secret"), position, from, to),
            _ => throw new ArgumentException($"Unsupported connection type: {type}")
        };
    }

    public ITrap CreateTrap(string type, Position position)
    {
        return type.ToLower() switch
        {
            "spike" => new SpikeTrap(GenerateId("SpikeTrap"), position),
            "arrow" => new ArrowTrap(GenerateId("ArrowTrap"), position),
            "pit" => new PitTrap(GenerateId("PitTrap"), position),
            _ => throw new ArgumentException($"Unknown trap type: {type}")
        };
    }

    public IDecoration CreateDecoration(string type, Position position)
    {
        return type.ToLower() switch
        {
            "torch" => new TorchDecoration(GenerateId("Torch"), position),
            "banner" => new BannerDecoration(GenerateId("Banner"), position),
            "statue" => new StatueDecoration(GenerateId("Statue"), position),
            _ => throw new ArgumentException($"Unknown decoration type: {type}")
        };
    }

    private string GenerateId(string prefix) => $"{prefix}_{++_elementCounter:D3}";
}

// SciFi theme factory
public class SciFiDungeonFactory : IDungeonElementFactory
{
    private readonly IRoomFactory _roomFactory;
    private int _elementCounter = 0;

    public SciFiDungeonFactory()
    {
        _roomFactory = new StandardRoomFactory(DungeonTheme.SciFi);
    }

    public DungeonTheme Theme => DungeonTheme.SciFi;

    public IRoom CreateRoom(RoomType type, Rectangle bounds) => _roomFactory.CreateRoom(type, bounds);

    public IConnection CreateConnection(ConnectionType type, Position position, IRoom from, IRoom to)
    {
        return type switch
        {
            ConnectionType.Door => new AutomaticDoor(GenerateId("AutoDoor"), position, from, to),
            ConnectionType.Portal => new EnergyPortal(GenerateId("Portal"), position, from, to),
            _ => new BasicConnection(GenerateId("Connection"), type, position, from, to)
        };
    }

    public ITrap CreateTrap(string type, Position position)
    {
        return type.ToLower() switch
        {
            "laser" => new LaserTrap(GenerateId("LaserTrap"), position),
            "force" => new ForceFieldTrap(GenerateId("ForceTrap"), position),
            "electric" => new ElectricTrap(GenerateId("ElectricTrap"), position),
            _ => throw new ArgumentException($"Unknown trap type: {type}")
        };
    }

    public IDecoration CreateDecoration(string type, Position position)
    {
        return type.ToLower() switch
        {
            "computer" => new ComputerTerminal(GenerateId("Computer"), position),
            "hologram" => new HologramProjector(GenerateId("Hologram"), position),
            "reactor" => new ReactorCore(GenerateId("Reactor"), position),
            _ => throw new ArgumentException($"Unknown decoration type: {type}")
        };
    }

    private string GenerateId(string prefix) => $"{prefix}_{++_elementCounter:D3}";
}

// Concrete implementations for Medieval theme
public class MedievalDoor : IConnection
{
    public string Id { get; }
    public ConnectionType Type => ConnectionType.Door;
    public Position Position { get; }
    public IRoom FromRoom { get; }
    public IRoom ToRoom { get; }
    public bool IsPassable { get; set; } = true;

    public bool IsLocked { get; set; } = false;
    public string Material { get; set; } = "Oak";

    public MedievalDoor(string id, Position position, IRoom from, IRoom to)
    {
        Id = id;
        Position = position;
        FromRoom = from;
        ToRoom = to;
    }
}

public class SpikeTrap : ITrap
{
    public string Id { get; }
    public string Type => "Spike Trap";
    public Position Position { get; }
    public int Damage { get; set; } = 25;
    public bool IsActive { get; set; } = true;

    public SpikeTrap(string id, Position position)
    {
        Id = id;
        Position = position;
    }
}

// Similar implementations for other concrete types...
```

### Step 7: Builder Pattern Implementation (40 minutes)

**Fluent Dungeon Builder**

```csharp
namespace DungeonGenerator.Core.Builders;

public interface IDungeonBuilder
{
    IDungeonBuilder SetSize(int width, int height);
    IDungeonBuilder SetTheme(DungeonTheme theme);
    IDungeonBuilder SetAlgorithm<T>() where T : class, IDungeonGenerationAlgorithm, new();
    IDungeonBuilder AddRoomType<T>(float probability) where T : Room;
    IDungeonBuilder SetRoomCount(int min, int max);
    IDungeonBuilder SetConnectivity(float density);
    IDungeonBuilder AddValidator<T>() where T : class, IDungeonValidator, new();
    IDungeonBuilder ConfigureGeneration(Action<GenerationOptions> configure);
    Dungeon Build();
}

public class DungeonBuilder : IDungeonBuilder
{
    private Size _size = new(50, 50);
    private DungeonTheme _theme = DungeonTheme.Medieval;
    private IDungeonGenerationAlgorithm? _algorithm;
    private readonly Dictionary<Type, float> _roomTypeProbabilities = new();
    private int _minRooms = 5;
    private int _maxRooms = 15;
    private float _connectivity = 0.3f;
    private readonly List<IDungeonValidator> _validators = new();
    private readonly GenerationOptions _options = new();

    public IDungeonBuilder SetSize(int width, int height)
    {
        _size = new Size(width, height);
        return this;
    }

    public IDungeonBuilder SetTheme(DungeonTheme theme)
    {
        _theme = theme;
        return this;
    }

    public IDungeonBuilder SetAlgorithm<T>() where T : class, IDungeonGenerationAlgorithm, new()
    {
        _algorithm = new T();
        return this;
    }

    public IDungeonBuilder AddRoomType<T>(float probability) where T : Room
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(probability);
        _roomTypeProbabilities[typeof(T)] = probability;
        return this;
    }

    public IDungeonBuilder SetRoomCount(int min, int max)
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(min);
        ArgumentOutOfRangeException.ThrowIfLessThan(max, min);
        _minRooms = min;
        _maxRooms = max;
        return this;
    }

    public IDungeonBuilder SetConnectivity(float density)
    {
        ArgumentOutOfRangeException.ThrowIfNegative(density);
        _connectivity = Math.Min(density, 1.0f);
        return this;
    }

    public IDungeonBuilder AddValidator<T>() where T : class, IDungeonValidator, new()
    {
        _validators.Add(new T());
        return this;
    }

    public IDungeonBuilder ConfigureGeneration(Action<GenerationOptions> configure)
    {
        ArgumentNullException.ThrowIfNull(configure);
        configure(_options);
        return this;
    }

    public Dungeon Build()
    {
        // Validate builder state
        ValidateBuilderState();

        // Create generation context
        var context = new DungeonGenerationContext
        {
            Size = _size,
            Theme = _theme,
            Algorithm = _algorithm!,
            RoomTypeProbabilities = new Dictionary<Type, float>(_roomTypeProbabilities),
            MinRooms = _minRooms,
            MaxRooms = _maxRooms,
            Connectivity = _connectivity,
            Options = _options,
            Validators = new List<IDungeonValidator>(_validators)
        };

        // Generate the dungeon
        var generator = new DungeonGenerator(context);
        var dungeon = generator.Generate();

        // Validate the result
        ValidateDungeon(dungeon);

        return dungeon;
    }

    private void ValidateBuilderState()
    {
        if (_algorithm == null)
        {
            throw new InvalidOperationException("Generation algorithm must be specified");
        }

        if (_roomTypeProbabilities.Count == 0)
        {
            // Add default room types
            _roomTypeProbabilities[typeof(BasicRoom)] = 0.7f;
            _roomTypeProbabilities[typeof(TreasureRoom)] = 0.2f;
            _roomTypeProbabilities[typeof(BossRoom)] = 0.1f;
        }

        // Normalize probabilities
        var totalProbability = _roomTypeProbabilities.Values.Sum();
        if (totalProbability > 1.0f)
        {
            var scale = 1.0f / totalProbability;
            var keys = _roomTypeProbabilities.Keys.ToList();
            foreach (var key in keys)
            {
                _roomTypeProbabilities[key] *= scale;
            }
        }
    }

    private void ValidateDungeon(Dungeon dungeon)
    {
        var validationResults = new List<ValidationResult>();

        foreach (var validator in _validators)
        {
            var result = validator.Validate(dungeon);
            validationResults.Add(result);
        }

        var errors = validationResults.Where(r => !r.IsValid).ToList();
        if (errors.Count > 0)
        {
            var errorMessages = errors.SelectMany(e => e.Errors);
            throw new DungeonValidationException($"Dungeon validation failed: {string.Join(", ", errorMessages)}");
        }
    }
}

// Fluent configuration for specific aspects
public class RoomConfigurationBuilder
{
    private Size _minSize = new(3, 3);
    private Size _maxSize = new(10, 10);
    private bool _allowOverlapping = false;
    private int _minConnections = 1;
    private int _maxConnections = 4;

    public RoomConfigurationBuilder SetSizeRange(Size min, Size max)
    {
        _minSize = min;
        _maxSize = max;
        return this;
    }

    public RoomConfigurationBuilder AllowOverlapping(bool allow = true)
    {
        _allowOverlapping = allow;
        return this;
    }

    public RoomConfigurationBuilder SetConnectionRange(int min, int max)
    {
        _minConnections = min;
        _maxConnections = max;
        return this;
    }

    internal RoomConfiguration Build() => new()
    {
        MinSize = _minSize,
        MaxSize = _maxSize,
        AllowOverlapping = _allowOverlapping,
        MinConnections = _minConnections,
        MaxConnections = _maxConnections
    };
}

// Usage with nested builders
public static class DungeonBuilderExtensions
{
    public static IDungeonBuilder ConfigureRooms(this IDungeonBuilder builder,
        Func<RoomConfigurationBuilder, RoomConfigurationBuilder> configure)
    {
        var roomBuilder = new RoomConfigurationBuilder();
        var roomConfig = configure(roomBuilder).Build();

        return builder.ConfigureGeneration(options => options.RoomConfiguration = roomConfig);
    }
}
```

## Resources

- 📚 **[Design Patterns: Gang of Four](https://en.wikipedia.org/wiki/Design_Patterns)** - Classic design patterns reference
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Factory Pattern Documentation](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/)** - .NET design guidelines
- 📘 **[Builder Pattern Examples](https://refactoring.guru/design-patterns/builder)** - Builder pattern implementation guide

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
