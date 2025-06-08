# Project 17: Mini Game Engine

**Focus**: Integration Capstone  
**Time**: ~4 hours  
**Deliverable**: `MiniGameEngine` - A complete game engine integrating all previous concepts

## 📏 Microsoft Guidelines Applied

This project integrates all Microsoft guidelines and best practices from the entire curriculum:

- **All Design Guidelines** - Comprehensive application of .NET design principles
- **Performance Guidelines** - Optimized game engine architecture
- **Framework Integration** - Professional-grade system design

## Learning Objectives

- Integrate all 16 previous projects into a cohesive system
- Master large-scale software architecture and design
- Apply professional game development patterns
- Build a complete, extensible game engine
- Demonstrate mastery of .NET ecosystem

## Quick Start

1. **Review All Guidelines** 📖  
   This capstone integrates concepts from all previous projects

2. **Start Building** 💻  
   See [detailed project guide](game-engine-project-guide.md) for comprehensive instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) for final milestones

## Project Structure

```
project-17-game-engine/
├── README.md                    # This file
├── game-engine-project-guide.md # Detailed implementation guide
├── src/                         # Your implementation
│   ├── Core/                    # Engine core systems
│   ├── Rendering/               # Graphics and rendering
│   ├── Audio/                   # Audio management
│   ├── Input/                   # Input handling
│   ├── Physics/                 # Physics simulation
│   ├── Assets/                  # Asset management
│   ├── Scripting/              # Game logic scripting
│   └── Tools/                   # Engine tools
├── tests/                       # Comprehensive test suite
├── docs/                        # Engine documentation
└── samples/                     # Example games
```

## What You'll Build

A complete mini game engine featuring:

- **Modular Architecture** - Plugin-based, extensible design
- **High-Performance Rendering** - GPU-accelerated graphics pipeline
- **Audio System** - 3D positional audio with mixing
- **Physics Engine** - Real-time collision detection and response
- **Asset Pipeline** - Efficient resource loading and management
- **Input Management** - Multi-platform input handling
- **Scripting Support** - C# scripting for game logic
- **Editor Tools** - Basic game development tools

## Integration Points

This project integrates concepts from all previous projects:

### Phase 1 Integration (.NET Fundamentals)

- **Project 1**: Type system for engine components
- **Project 2**: Collections for entity management
- **Project 3**: String utilities for engine tools
- **Project 4**: State machines for game logic
- **Project 5**: LINQ for entity queries

### Phase 2 Integration (Advanced Features)

- **Project 6**: Async file I/O for asset loading
- **Project 7**: Event-driven audio system
- **Project 8**: Object pooling for performance

### Phase 3 Integration (Architecture & Patterns)

- **Project 9**: Dependency injection for engine services
- **Project 10**: Factory patterns for entity creation
- **Project 11**: Input system with command patterns
- **Project 12**: Asset management with repository pattern

### Phase 4 Integration (Advanced .NET)

- **Project 13**: Configuration system for engine settings
- **Project 14**: Dynamic query builder for entity systems
- **Project 15**: High-performance buffers for rendering
- **Project 16**: Native interop for platform features

## Project Setup Instructions

### Step 1: Create the Solution Structure (20 minutes)

Navigate to the `project-17-game-engine/` directory:

```bash
# Create the main solution
dotnet new sln -n GameEngine

# Create core engine projects
dotnet new classlib -n GameEngine.Core -f net8.0
dotnet new classlib -n GameEngine.Rendering -f net8.0
dotnet new classlib -n GameEngine.Audio -f net8.0
dotnet new classlib -n GameEngine.Physics -f net8.0
dotnet new classlib -n GameEngine.Input -f net8.0
dotnet new classlib -n GameEngine.Assets -f net8.0

# Create sample applications
dotnet new console -n GameEngine.Samples.Platformer -f net8.0
dotnet new console -n GameEngine.Samples.Explorer -f net8.0

# Create test project
dotnet new mstest -n GameEngine.Tests -f net8.0

# Add all projects to solution
dotnet sln add **/*.csproj

# Set up project references
dotnet add GameEngine.Rendering/GameEngine.Rendering.csproj reference GameEngine.Core/GameEngine.Core.csproj
dotnet add GameEngine.Audio/GameEngine.Audio.csproj reference GameEngine.Core/GameEngine.Core.csproj
dotnet add GameEngine.Physics/GameEngine.Physics.csproj reference GameEngine.Core/GameEngine.Core.csproj
dotnet add GameEngine.Input/GameEngine.Input.csproj reference GameEngine.Core/GameEngine.Core.csproj
dotnet add GameEngine.Assets/GameEngine.Assets.csproj reference GameEngine.Core/GameEngine.Core.csproj

# Sample apps reference all systems
dotnet add GameEngine.Samples.Platformer/GameEngine.Samples.Platformer.csproj reference GameEngine.Core/GameEngine.Core.csproj
dotnet add GameEngine.Samples.Platformer/GameEngine.Samples.Platformer.csproj reference GameEngine.Rendering/GameEngine.Rendering.csproj
dotnet add GameEngine.Samples.Platformer/GameEngine.Samples.Platformer.csproj reference GameEngine.Audio/GameEngine.Audio.csproj
dotnet add GameEngine.Samples.Platformer/GameEngine.Samples.Platformer.csproj reference GameEngine.Physics/GameEngine.Physics.csproj
dotnet add GameEngine.Samples.Platformer/GameEngine.Samples.Platformer.csproj reference GameEngine.Input/GameEngine.Input.csproj
dotnet add GameEngine.Samples.Platformer/GameEngine.Samples.Platformer.csproj reference GameEngine.Assets/GameEngine.Assets.csproj

# Tests reference core
dotnet add GameEngine.Tests/GameEngine.Tests.csproj reference GameEngine.Core/GameEngine.Core.csproj
```

### Step 2: Configure Core Project Files

Update `GameEngine.Core.csproj`:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Configuration" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging" Version="8.0.0" />
    <PackageReference Include="System.Numerics.Vectors" Version="4.5.0" />
    <PackageReference Include="System.Text.Json" Version="8.0.0" />
  </ItemGroup>
</Project>
```

## Implementation Guide

### Step 3: Core Engine Architecture (45 minutes)

**Create the foundational engine architecture**

```csharp
// File: GameEngine.Core/GameEngine.cs
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

namespace GameEngine.Core;

/// <summary>
/// Main game engine orchestrator integrating all systems
/// </summary>
public class MiniGameEngine : IDisposable
{
    private readonly IServiceProvider _services;
    private readonly ILogger<MiniGameEngine> _logger;
    private readonly List<IGameSystem> _systems;
    private readonly GameClock _gameClock;

    private bool _isRunning;
    private bool _disposed;

    public MiniGameEngine(IServiceProvider services)
    {
        _services = services ?? throw new ArgumentNullException(nameof(services));
        _logger = _services.GetRequiredService<ILogger<MiniGameEngine>>();
        _systems = new List<IGameSystem>();
        _gameClock = new GameClock();
    }

    public EntityManager Entities => _services.GetRequiredService<EntityManager>();
    public AssetManager Assets => _services.GetRequiredService<AssetManager>();
    public InputManager Input => _services.GetRequiredService<InputManager>();
    public AudioManager Audio => _services.GetRequiredService<AudioManager>();
    public PhysicsManager Physics => _services.GetRequiredService<PhysicsManager>();
    public RenderingManager Rendering => _services.GetRequiredService<RenderingManager>();

    public async Task InitializeAsync(EngineConfiguration config)
    {
        _logger.LogInformation("Initializing game engine...");

        try
        {
            // Register all systems
            RegisterSystems();

            // Initialize systems in dependency order
            foreach (var system in _systems.OrderBy(s => s.InitializationOrder))
            {
                _logger.LogDebug("Initializing system: {SystemName}", system.GetType().Name);
                await system.InitializeAsync(config);
            }

            _logger.LogInformation("Game engine initialized successfully");
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to initialize game engine");
            throw;
        }
    }

    public async Task RunAsync(CancellationToken cancellationToken = default)
    {
        _isRunning = true;
        _gameClock.Start();

        _logger.LogInformation("Starting game loop");

        try
        {
            while (_isRunning && !cancellationToken.IsCancellationRequested)
            {
                var gameTime = _gameClock.Update();

                // Update all systems
                await UpdateAsync(gameTime);

                // Render frame
                await RenderAsync(gameTime);

                // Maintain target frame rate
                await _gameClock.WaitForNextFrameAsync();
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Game loop error");
            throw;
        }
        finally
        {
            _logger.LogInformation("Game loop stopped");
        }
    }

    private async Task UpdateAsync(GameTime gameTime)
    {
        // Update systems in priority order
        foreach (var system in _systems.Where(s => s.IsEnabled).OrderBy(s => s.UpdateOrder))
        {
            try
            {
                await system.UpdateAsync(gameTime);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error updating system {SystemName}", system.GetType().Name);
                throw;
            }
        }
    }

    private async Task RenderAsync(GameTime gameTime)
    {
        try
        {
            await Rendering.RenderFrameAsync(gameTime);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Rendering error");
            throw;
        }
    }

    private void RegisterSystems()
    {
        // Get all registered systems from DI container
        _systems.AddRange(_services.GetServices<IGameSystem>());

        _logger.LogInformation("Registered {SystemCount} game systems", _systems.Count);
    }

    public void Stop()
    {
        _isRunning = false;
        _logger.LogInformation("Game engine stop requested");
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                Stop();

                // Dispose systems in reverse order
                foreach (var system in _systems.AsEnumerable().Reverse())
                {
                    if (system is IDisposable disposable)
                    {
                        disposable.Dispose();
                    }
                }

                _systems.Clear();
                _gameClock.Dispose();
            }

            _disposed = true;
        }
    }

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }
}

/// <summary>
/// Base interface for all engine systems
/// </summary>
public interface IGameSystem
{
    string Name { get; }
    int InitializationOrder { get; }
    int UpdateOrder { get; }
    bool IsEnabled { get; set; }

    Task InitializeAsync(EngineConfiguration config);
    Task UpdateAsync(GameTime gameTime);
}

/// <summary>
/// High-precision game timing
/// </summary>
public class GameClock : IDisposable
{
    private readonly Stopwatch _stopwatch;
    private TimeSpan _lastFrameTime;
    private TimeSpan _targetFrameTime;
    private bool _disposed;

    public GameClock(int targetFps = 60)
    {
        _stopwatch = Stopwatch.StartNew();
        _targetFrameTime = TimeSpan.FromSeconds(1.0 / targetFps);
        TargetFps = targetFps;
    }

    public int TargetFps { get; set; }
    public double DeltaTime { get; private set; }
    public double TotalTime { get; private set; }
    public int FrameCount { get; private set; }
    public double FramesPerSecond { get; private set; }

    public void Start()
    {
        _stopwatch.Restart();
        _lastFrameTime = TimeSpan.Zero;
    }

    public GameTime Update()
    {
        var currentTime = _stopwatch.Elapsed;
        var deltaTime = currentTime - _lastFrameTime;

        DeltaTime = deltaTime.TotalSeconds;
        TotalTime = currentTime.TotalSeconds;
        FrameCount++;

        // Calculate FPS
        if (FrameCount % 60 == 0)
        {
            FramesPerSecond = 1.0 / DeltaTime;
        }

        _lastFrameTime = currentTime;

        return new GameTime(TotalTime, DeltaTime, FrameCount);
    }

    public async Task WaitForNextFrameAsync()
    {
        var frameTime = _stopwatch.Elapsed - _lastFrameTime;
        var remainingTime = _targetFrameTime - frameTime;

        if (remainingTime > TimeSpan.Zero)
        {
            await Task.Delay(remainingTime);
        }
    }

    public void Dispose()
    {
        if (!_disposed)
        {
            _stopwatch?.Stop();
            _disposed = true;
        }
    }
}

/// <summary>
/// Immutable game timing information
/// </summary>
public readonly struct GameTime
{
    public GameTime(double totalTime, double deltaTime, int frameCount)
    {
        TotalTime = totalTime;
        DeltaTime = deltaTime;
        FrameCount = frameCount;
    }

    public double TotalTime { get; }
    public double DeltaTime { get; }
    public int FrameCount { get; }

    public float TotalTimeF => (float)TotalTime;
    public float DeltaTimeF => (float)DeltaTime;
}

/// <summary>
/// Engine configuration
/// </summary>
public class EngineConfiguration
{
    public WindowConfiguration Window { get; set; } = new();
    public RenderingConfiguration Rendering { get; set; } = new();
    public AudioConfiguration Audio { get; set; } = new();
    public PhysicsConfiguration Physics { get; set; } = new();
    public AssetConfiguration Assets { get; set; } = new();
    public InputConfiguration Input { get; set; } = new();

    public void Validate()
    {
        Window.Validate();
        Rendering.Validate();
        Audio.Validate();
        Physics.Validate();
        Assets.Validate();
        Input.Validate();
    }
}

public class WindowConfiguration
{
    public string Title { get; set; } = "Mini Game Engine";
    public int Width { get; set; } = 1280;
    public int Height { get; set; } = 720;
    public bool Fullscreen { get; set; } = false;
    public bool VSync { get; set; } = true;

    public void Validate()
    {
        if (Width <= 0) throw new ArgumentException("Width must be positive");
        if (Height <= 0) throw new ArgumentException("Height must be positive");
        if (string.IsNullOrEmpty(Title)) throw new ArgumentException("Title cannot be empty");
    }
}

public class RenderingConfiguration
{
    public int TargetFps { get; set; } = 60;
    public bool EnableMultisampling { get; set; } = true;
    public int MultisampleCount { get; set; } = 4;
    public bool EnableHDR { get; set; } = false;

    public void Validate()
    {
        if (TargetFps <= 0) throw new ArgumentException("Target FPS must be positive");
        if (MultisampleCount < 1) throw new ArgumentException("Multisample count must be at least 1");
    }
}

public class AudioConfiguration
{
    public int SampleRate { get; set; } = 44100;
    public int BufferSize { get; set; } = 1024;
    public int Channels { get; set; } = 2;
    public float MasterVolume { get; set; } = 1.0f;

    public void Validate()
    {
        if (SampleRate <= 0) throw new ArgumentException("Sample rate must be positive");
        if (BufferSize <= 0) throw new ArgumentException("Buffer size must be positive");
        if (Channels <= 0) throw new ArgumentException("Channel count must be positive");
        if (MasterVolume < 0 || MasterVolume > 1) throw new ArgumentException("Master volume must be between 0 and 1");
    }
}

public class PhysicsConfiguration
{
    public Vector3 Gravity { get; set; } = new(0, -9.81f, 0);
    public int MaxIterations { get; set; } = 10;
    public float TimeStep { get; set; } = 1f / 60f;

    public void Validate()
    {
        if (MaxIterations <= 0) throw new ArgumentException("Max iterations must be positive");
        if (TimeStep <= 0) throw new ArgumentException("Time step must be positive");
    }
}

public class AssetConfiguration
{
    public string RootPath { get; set; } = "Assets";
    public int CacheSize { get; set; } = 100;
    public bool PreloadCommonAssets { get; set; } = true;

    public void Validate()
    {
        if (string.IsNullOrEmpty(RootPath)) throw new ArgumentException("Root path cannot be empty");
        if (CacheSize <= 0) throw new ArgumentException("Cache size must be positive");
    }
}

public class InputConfiguration
{
    public bool EnableKeyboard { get; set; } = true;
    public bool EnableMouse { get; set; } = true;
    public bool EnableGamepad { get; set; } = true;
    public float MouseSensitivity { get; set; } = 1.0f;

    public void Validate()
    {
        if (MouseSensitivity <= 0) throw new ArgumentException("Mouse sensitivity must be positive");
    }
}
```

### Step 4: Entity-Component-System (40 minutes)

**Implement a high-performance ECS architecture**

```csharp
// File: GameEngine.Core/ECS/EntityManager.cs
using System.Collections.Concurrent;

namespace GameEngine.Core.ECS;

/// <summary>
/// High-performance entity-component-system manager
/// </summary>
public class EntityManager : IGameSystem, IDisposable
{
    private readonly ConcurrentDictionary<EntityId, Entity> _entities;
    private readonly Dictionary<Type, IComponentStore> _componentStores;
    private readonly List<IGameSystem> _systems;
    private readonly object _lockObject = new();
    private EntityId _nextEntityId;
    private bool _disposed;

    public EntityManager()
    {
        _entities = new ConcurrentDictionary<EntityId, Entity>();
        _componentStores = new Dictionary<Type, IComponentStore>();
        _systems = new List<IGameSystem>();
        _nextEntityId = EntityId.First;
    }

    public string Name => "Entity Manager";
    public int InitializationOrder => 0;
    public int UpdateOrder => 0;
    public bool IsEnabled { get; set; } = true;

    public Entity CreateEntity(string? name = null)
    {
        var id = Interlocked.Increment(ref _nextEntityId);
        var entity = new Entity(id, name ?? $"Entity_{id}", this);

        _entities[id] = entity;
        return entity;
    }

    public void DestroyEntity(EntityId entityId)
    {
        if (_entities.TryRemove(entityId, out var entity))
        {
            // Remove all components
            foreach (var store in _componentStores.Values)
            {
                store.RemoveComponent(entityId);
            }

            entity.MarkDestroyed();
        }
    }

    public Entity? GetEntity(EntityId entityId)
    {
        return _entities.TryGetValue(entityId, out var entity) ? entity : null;
    }

    public IEnumerable<Entity> GetEntities()
    {
        return _entities.Values.Where(e => !e.IsDestroyed);
    }

    public T AddComponent<T>(EntityId entityId) where T : class, IComponent, new()
    {
        var store = GetOrCreateComponentStore<T>();
        var component = store.AddComponent(entityId, new T());
        component.EntityId = entityId;
        return component;
    }

    public T? GetComponent<T>(EntityId entityId) where T : class, IComponent
    {
        var store = GetComponentStore<T>();
        return store?.GetComponent(entityId);
    }

    public bool RemoveComponent<T>(EntityId entityId) where T : class, IComponent
    {
        var store = GetComponentStore<T>();
        return store?.RemoveComponent(entityId) ?? false;
    }

    public bool HasComponent<T>(EntityId entityId) where T : class, IComponent
    {
        var store = GetComponentStore<T>();
        return store?.HasComponent(entityId) ?? false;
    }

    public IEnumerable<Entity> GetEntitiesWith<T>() where T : class, IComponent
    {
        var store = GetComponentStore<T>();
        if (store == null) yield break;

        foreach (var entityId in store.GetEntityIds())
        {
            if (_entities.TryGetValue(entityId, out var entity) && !entity.IsDestroyed)
            {
                yield return entity;
            }
        }
    }

    public IEnumerable<(Entity Entity, T Component)> GetEntitiesWithComponent<T>() where T : class, IComponent
    {
        var store = GetComponentStore<T>();
        if (store == null) yield break;

        foreach (var entityId in store.GetEntityIds())
        {
            if (_entities.TryGetValue(entityId, out var entity) && !entity.IsDestroyed)
            {
                var component = store.GetComponent(entityId);
                if (component != null)
                {
                    yield return (entity, component);
                }
            }
        }
    }

    private ComponentStore<T> GetOrCreateComponentStore<T>() where T : class, IComponent
    {
        lock (_lockObject)
        {
            var type = typeof(T);
            if (!_componentStores.TryGetValue(type, out var store))
            {
                store = new ComponentStore<T>();
                _componentStores[type] = store;
            }
            return (ComponentStore<T>)store;
        }
    }

    private ComponentStore<T>? GetComponentStore<T>() where T : class, IComponent
    {
        lock (_lockObject)
        {
            var type = typeof(T);
            return _componentStores.TryGetValue(type, out var store) ? (ComponentStore<T>)store : null;
        }
    }

    public Task InitializeAsync(EngineConfiguration config)
    {
        // Initialize component systems
        return Task.CompletedTask;
    }

    public Task UpdateAsync(GameTime gameTime)
    {
        // Clean up destroyed entities periodically
        if (gameTime.FrameCount % 60 == 0)
        {
            CleanupDestroyedEntities();
        }

        return Task.CompletedTask;
    }

    private void CleanupDestroyedEntities()
    {
        var toRemove = _entities.Where(kvp => kvp.Value.IsDestroyed).Select(kvp => kvp.Key).ToList();
        foreach (var entityId in toRemove)
        {
            _entities.TryRemove(entityId, out _);
        }
    }

    public void Dispose()
    {
        if (!_disposed)
        {
            foreach (var entity in _entities.Values)
            {
                entity.MarkDestroyed();
            }

            _entities.Clear();
            _componentStores.Clear();
            _disposed = true;
        }
    }
}

/// <summary>
/// Unique entity identifier
/// </summary>
public readonly struct EntityId : IEquatable<EntityId>
{
    public static readonly EntityId First = new(1);
    public static readonly EntityId Invalid = new(0);

    private readonly uint _value;

    public EntityId(uint value)
    {
        _value = value;
    }

    public bool IsValid => _value != 0;

    public bool Equals(EntityId other) => _value == other._value;
    public override bool Equals(object? obj) => obj is EntityId other && Equals(other);
    public override int GetHashCode() => _value.GetHashCode();
    public override string ToString() => _value.ToString();

    public static bool operator ==(EntityId left, EntityId right) => left.Equals(right);
    public static bool operator !=(EntityId left, EntityId right) => !(left == right);

    public static implicit operator uint(EntityId entityId) => entityId._value;
    public static implicit operator EntityId(uint value) => new(value);
}

/// <summary>
/// Game entity with components
/// </summary>
public class Entity
{
    private readonly EntityManager _entityManager;

    internal Entity(EntityId id, string name, EntityManager entityManager)
    {
        Id = id;
        Name = name;
        _entityManager = entityManager;
        Transform = _entityManager.AddComponent<Transform>(id);
    }

    public EntityId Id { get; }
    public string Name { get; set; }
    public Transform Transform { get; }
    public bool IsDestroyed { get; private set; }

    public T AddComponent<T>() where T : class, IComponent, new()
    {
        ThrowIfDestroyed();
        return _entityManager.AddComponent<T>(Id);
    }

    public T? GetComponent<T>() where T : class, IComponent
    {
        ThrowIfDestroyed();
        return _entityManager.GetComponent<T>(Id);
    }

    public bool RemoveComponent<T>() where T : class, IComponent
    {
        ThrowIfDestroyed();
        return _entityManager.RemoveComponent<T>(Id);
    }

    public bool HasComponent<T>() where T : class, IComponent
    {
        return !IsDestroyed && _entityManager.HasComponent<T>(Id);
    }

    public void Destroy()
    {
        _entityManager.DestroyEntity(Id);
    }

    internal void MarkDestroyed()
    {
        IsDestroyed = true;
    }

    private void ThrowIfDestroyed()
    {
        if (IsDestroyed)
            throw new InvalidOperationException("Entity has been destroyed");
    }

    public override string ToString() => $"Entity({Id}, '{Name}')";
}

/// <summary>
/// Base component interface
/// </summary>
public interface IComponent
{
    EntityId EntityId { get; set; }
}

/// <summary>
/// Transform component for position, rotation, and scale
/// </summary>
public class Transform : IComponent
{
    public EntityId EntityId { get; set; }

    public Vector3 Position { get; set; } = Vector3.Zero;
    public Quaternion Rotation { get; set; } = Quaternion.Identity;
    public Vector3 Scale { get; set; } = Vector3.One;

    public Matrix4x4 WorldMatrix
    {
        get
        {
            return Matrix4x4.CreateScale(Scale) *
                   Matrix4x4.CreateFromQuaternion(Rotation) *
                   Matrix4x4.CreateTranslation(Position);
        }
    }

    public Vector3 Forward => Vector3.Transform(Vector3.UnitZ, Rotation);
    public Vector3 Right => Vector3.Transform(Vector3.UnitX, Rotation);
    public Vector3 Up => Vector3.Transform(Vector3.UnitY, Rotation);

    public void LookAt(Vector3 target, Vector3 up)
    {
        var forward = Vector3.Normalize(target - Position);
        var right = Vector3.Normalize(Vector3.Cross(up, forward));
        var actualUp = Vector3.Cross(forward, right);

        Rotation = Quaternion.CreateFromRotationMatrix(new Matrix4x4(
            right.X, right.Y, right.Z, 0,
            actualUp.X, actualUp.Y, actualUp.Z, 0,
            forward.X, forward.Y, forward.Z, 0,
            0, 0, 0, 1));
    }
}

/// <summary>
/// Component storage interface
/// </summary>
internal interface IComponentStore
{
    bool HasComponent(EntityId entityId);
    bool RemoveComponent(EntityId entityId);
    IEnumerable<EntityId> GetEntityIds();
}

/// <summary>
/// Efficient component storage
/// </summary>
internal class ComponentStore<T> : IComponentStore where T : class, IComponent
{
    private readonly Dictionary<EntityId, T> _components = new();

    public T AddComponent(EntityId entityId, T component)
    {
        _components[entityId] = component;
        return component;
    }

    public T? GetComponent(EntityId entityId)
    {
        return _components.TryGetValue(entityId, out var component) ? component : null;
    }

    public bool HasComponent(EntityId entityId)
    {
        return _components.ContainsKey(entityId);
    }

    public bool RemoveComponent(EntityId entityId)
    {
        return _components.Remove(entityId);
    }

    public IEnumerable<EntityId> GetEntityIds()
    {
        return _components.Keys;
    }

    public IEnumerable<T> GetComponents()
    {
        return _components.Values;
    }
}
```

## Testing Your Understanding

Create a complete example in `GameEngine.Samples.Platformer/Program.cs`:

```csharp
using GameEngine.Core;
using GameEngine.Core.ECS;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

namespace GameEngine.Samples.Platformer;

class Program
{
    static async Task Main(string[] args)
    {
        Console.WriteLine("🎮 Mini Game Engine - 2D Platformer Sample");
        Console.WriteLine("==========================================\n");

        // Set up dependency injection
        var services = new ServiceCollection();
        ConfigureServices(services);

        var serviceProvider = services.BuildServiceProvider();

        // Create and configure engine
        using var engine = new MiniGameEngine(serviceProvider);

        var config = new EngineConfiguration
        {
            Window = { Title = "2D Platformer", Width = 1280, Height = 720 },
            Rendering = { TargetFps = 60 },
            Audio = { MasterVolume = 0.8f }
        };

        try
        {
            await engine.InitializeAsync(config);

            // Create sample game entities
            CreateSampleGame(engine);

            Console.WriteLine("Press any key to start the game simulation...");
            Console.ReadKey();

            // Run game simulation
            using var cts = new CancellationTokenSource();
            var gameTask = engine.RunAsync(cts.Token);

            // Run for 5 seconds then stop
            await Task.Delay(5000);
            cts.Cancel();

            await gameTask;

            Console.WriteLine("\n✅ Game simulation completed!");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"❌ Game error: {ex.Message}");
        }
    }

    static void ConfigureServices(ServiceCollection services)
    {
        // Add logging
        services.AddLogging(builder =>
        {
            builder.AddConsole();
            builder.SetMinimumLevel(LogLevel.Information);
        });

        // Register core engine services
        services.AddSingleton<EntityManager>();
        services.AddSingleton<AssetManager>();
        services.AddSingleton<InputManager>();
        services.AddSingleton<AudioManager>();
        services.AddSingleton<PhysicsManager>();
        services.AddSingleton<RenderingManager>();

        // Register all managers as game systems
        services.AddSingleton<IGameSystem>(sp => sp.GetRequiredService<EntityManager>());
        services.AddSingleton<IGameSystem>(sp => sp.GetRequiredService<AssetManager>());
        services.AddSingleton<IGameSystem>(sp => sp.GetRequiredService<InputManager>());
        services.AddSingleton<IGameSystem>(sp => sp.GetRequiredService<AudioManager>());
        services.AddSingleton<IGameSystem>(sp => sp.GetRequiredService<PhysicsManager>());
        services.AddSingleton<IGameSystem>(sp => sp.GetRequiredService<RenderingManager>());
    }

    static void CreateSampleGame(MiniGameEngine engine)
    {
        Console.WriteLine("Creating sample 2D platformer game...");

        // Create player entity
        var player = engine.Entities.CreateEntity("Player");
        player.Transform.Position = new Vector3(0, 1, 0);

        var playerMovement = player.AddComponent<PlayerMovement>();
        playerMovement.Speed = 5.0f;
        playerMovement.JumpHeight = 2.0f;

        var playerCollider = player.AddComponent<BoxCollider>();
        playerCollider.Size = new Vector3(1, 2, 1);

        var playerRenderer = player.AddComponent<SpriteRenderer>();
        playerRenderer.SpritePath = "Sprites/player.png";

        Console.WriteLine($"✅ Created player: {player}");

        // Create platforms
        for (int i = 0; i < 5; i++)
        {
            var platform = engine.Entities.CreateEntity($"Platform_{i}");
            platform.Transform.Position = new Vector3(i * 3, -1, 0);
            platform.Transform.Scale = new Vector3(2, 0.5f, 1);

            var platformCollider = platform.AddComponent<BoxCollider>();
            platformCollider.Size = new Vector3(2, 0.5f, 1);
            platformCollider.IsStatic = true;

            var platformRenderer = platform.AddComponent<SpriteRenderer>();
            platformRenderer.SpritePath = "Sprites/platform.png";

            Console.WriteLine($"✅ Created platform: {platform}");
        }

        // Create camera
        var camera = engine.Entities.CreateEntity("Camera");
        camera.Transform.Position = new Vector3(0, 2, -10);

        var cameraComponent = camera.AddComponent<CameraComponent>();
        cameraComponent.FieldOfView = 60f;
        cameraComponent.NearPlane = 0.1f;
        cameraComponent.FarPlane = 100f;

        var cameraFollow = camera.AddComponent<CameraFollow>();
        cameraFollow.Target = player.Id;
        cameraFollow.FollowSpeed = 2.0f;

        Console.WriteLine($"✅ Created camera: {camera}");

        Console.WriteLine($"Total entities created: {engine.Entities.GetEntities().Count()}");
    }
}

// Sample components for the platformer
public class PlayerMovement : IComponent
{
    public EntityId EntityId { get; set; }
    public float Speed { get; set; } = 5.0f;
    public float JumpHeight { get; set; } = 2.0f;
    public bool IsGrounded { get; set; }
    public Vector3 Velocity { get; set; }
}

public class BoxCollider : IComponent
{
    public EntityId EntityId { get; set; }
    public Vector3 Size { get; set; } = Vector3.One;
    public bool IsStatic { get; set; } = false;
    public bool IsTrigger { get; set; } = false;
}

public class SpriteRenderer : IComponent
{
    public EntityId EntityId { get; set; }
    public string SpritePath { get; set; } = string.Empty;
    public Vector4 Color { get; set; } = Vector4.One;
    public int SortingOrder { get; set; } = 0;
}

public class CameraComponent : IComponent
{
    public EntityId EntityId { get; set; }
    public float FieldOfView { get; set; } = 60f;
    public float NearPlane { get; set; } = 0.1f;
    public float FarPlane { get; set; } = 100f;
    public bool IsActive { get; set; } = true;
}

public class CameraFollow : IComponent
{
    public EntityId EntityId { get; set; }
    public EntityId Target { get; set; }
    public float FollowSpeed { get; set; } = 2.0f;
    public Vector3 Offset { get; set; } = new(0, 2, -10);
}

// Placeholder manager implementations for demo
public class AssetManager : IGameSystem
{
    public string Name => "Asset Manager";
    public int InitializationOrder => 1;
    public int UpdateOrder => 100;
    public bool IsEnabled { get; set; } = true;

    public Task InitializeAsync(EngineConfiguration config)
    {
        Console.WriteLine("Asset Manager initialized");
        return Task.CompletedTask;
    }

    public Task UpdateAsync(GameTime gameTime)
    {
        return Task.CompletedTask;
    }
}

public class InputManager : IGameSystem
{
    public string Name => "Input Manager";
    public int InitializationOrder => 2;
    public int UpdateOrder => 10;
    public bool IsEnabled { get; set; } = true;

    public Task InitializeAsync(EngineConfiguration config)
    {
        Console.WriteLine("Input Manager initialized");
        return Task.CompletedTask;
    }

    public Task UpdateAsync(GameTime gameTime)
    {
        return Task.CompletedTask;
    }
}

public class AudioManager : IGameSystem
{
    public string Name => "Audio Manager";
    public int InitializationOrder => 3;
    public int UpdateOrder => 80;
    public bool IsEnabled { get; set; } = true;

    public Task InitializeAsync(EngineConfiguration config)
    {
        Console.WriteLine("Audio Manager initialized");
        return Task.CompletedTask;
    }

    public Task UpdateAsync(GameTime gameTime)
    {
        return Task.CompletedTask;
    }
}

public class PhysicsManager : IGameSystem
{
    public string Name => "Physics Manager";
    public int InitializationOrder => 4;
    public int UpdateOrder => 20;
    public bool IsEnabled { get; set; } = true;

    public Task InitializeAsync(EngineConfiguration config)
    {
        Console.WriteLine("Physics Manager initialized");
        return Task.CompletedTask;
    }

    public Task UpdateAsync(GameTime gameTime)
    {
        return Task.CompletedTask;
    }
}

public class RenderingManager : IGameSystem
{
    public string Name => "Rendering Manager";
    public int InitializationOrder => 5;
    public int UpdateOrder => 90;
    public bool IsEnabled { get; set; } = true;

    public Task InitializeAsync(EngineConfiguration config)
    {
        Console.WriteLine("Rendering Manager initialized");
        return Task.CompletedTask;
    }

    public Task UpdateAsync(GameTime gameTime)
    {
        return Task.CompletedTask;
    }

    public Task RenderFrameAsync(GameTime gameTime)
    {
        // Simulate rendering work
        if (gameTime.FrameCount % 60 == 0)
        {
            Console.WriteLine($"Rendering frame {gameTime.FrameCount} (FPS: ~60)");
        }
        return Task.CompletedTask;
    }
}
```

## Performance Requirements

The engine must meet these performance targets:

- **Startup Time**: < 2 seconds on modern hardware
- **Frame Rate**: Maintain 60 FPS with 1000+ entities
- **Memory Usage**: < 100MB baseline memory footprint
- **Asset Loading**: Stream assets without frame drops
- **Input Latency**: < 16ms input-to-display latency

## Sample Games

Build at least two sample games to demonstrate engine capabilities:

1. **2D Platformer** - Physics, animation, audio
2. **3D First-Person Explorer** - 3D rendering, input, asset streaming

## Resources

- 📚 **[Detailed Guide](game-engine-project-guide.md)** - Complete implementation instructions
- 📖 **[All Previous Projects](../)** - Reference implementations
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Game Engine Architecture](https://www.gameenginebook.com/)** - Industry reference
- 📘 **[Real-Time Rendering](http://www.realtimerendering.com/)** - Graphics programming

## Professional Standards

This capstone project follows industry standards:

- **Clean Architecture** - Separation of concerns and SOLID principles
- **Test Coverage** - Comprehensive unit and integration tests
- **Documentation** - API documentation and usage guides
- **Performance Profiling** - Benchmarks and optimization analysis
- **Cross-Platform** - Windows, macOS, and Linux support

## Contributing

When implementing this capstone:

1. Design for modularity and extensibility
2. Implement comprehensive testing strategies
3. Create detailed API documentation
4. Profile and optimize critical paths
5. Build sample games to validate engine design
