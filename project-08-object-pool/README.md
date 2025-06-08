# Project 8: Generic Object Pool

**Focus**: Generics & Constraints  
**Time**: ~2 hours  
**Deliverable**: `ObjectPool<T>` library with advanced generic patterns

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Generic Design Guidelines](../microsoft-design-guidelines.md#generic-design-guidelines)** - Proper generic type design
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - Memory allocation optimization
- **[Interface Design](../microsoft-design-guidelines.md#interface-design)** - Generic interface patterns

## Learning Objectives

- Master generic types and constraints
- Understand generic type variance (covariance/contravariance)
- Learn performance optimization through object pooling
- Practice advanced generic patterns and interfaces
- Implement thread-safe generic collections

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Generic Design Guidelines](../microsoft-design-guidelines.md#generic-design-guidelines) before starting

2. **Start Coding** 💻  
   Follow the implementation guide below for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-08-object-pool/
├── README.md                   # This file
├── ObjectPool.sln              # Solution file
├── ObjectPool.Core/            # Main library implementation
├── ObjectPool.App/             # Console application
└── ObjectPool.Tests/           # Unit tests
```

## What You'll Build

A high-performance object pooling system featuring:

- Generic object pools with type constraints
- Thread-safe object borrowing and returning
- Configurable pool sizing and growth policies
- Generic factory patterns for object creation
- Performance monitoring and pool statistics
- Variance support for related object types

## Key Concepts

- **Generic Types**: Creating reusable generic classes and interfaces
- **Type Constraints**: Restricting generic type parameters appropriately
- **Variance**: Understanding covariance and contravariance
- **Factory Patterns**: Generic object creation strategies
- **Thread Safety**: Concurrent access to generic collections
- **Performance**: Memory allocation optimization techniques

## Project Template

**✅ Template Available**: This project includes a pre-configured solution template to streamline development!

The template includes:

- `ObjectPool.sln` - Solution file with all project references configured
- `ObjectPool.Core/` - Main library project (.NET 8.0 class library)
- `ObjectPool.App/` - Console application for testing and demonstration
- `ObjectPool.Tests/` - Unit test project with MSTest framework

**To get started**: Open `ObjectPool.sln` in your IDE and begin implementing in the `ObjectPool.Core` project!

## Project Setup Instructions (Manual Setup - Optional)

**Note**: The following instructions are for educational purposes and manual setup. The template above provides these pre-configured.

### Step 1: Manual Solution Creation (10 minutes)

If you need to recreate the solution structure manually, use these commands:

```bash
# Create the main solution file
dotnet new sln -n ObjectPool

# Create the main library project
dotnet new classlib -n ObjectPool.Core -f net8.0

# Create the console application
dotnet new console -n ObjectPool.App -f net8.0

# Create the test project
dotnet new mstest -n ObjectPool.Tests -f net8.0

# Add projects to solution
dotnet sln add ObjectPool.Core/ObjectPool.Core.csproj
dotnet sln add ObjectPool.App/ObjectPool.App.csproj
dotnet sln add ObjectPool.Tests/ObjectPool.Tests.csproj

# Add project references
dotnet add ObjectPool.App/ObjectPool.App.csproj reference ObjectPool.Core/ObjectPool.Core.csproj
dotnet add ObjectPool.Tests/ObjectPool.Tests.csproj reference ObjectPool.Core/ObjectPool.Core.csproj
```

### Step 2: Configure Project Files

Update the `ObjectPool.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>ObjectPool.Core</PackageId>
    <Title>Generic Object Pool</Title>
    <Description>A library for high-performance object pooling with advanced generic patterns</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your object pooling code:

```
ObjectPool.Core/
├── Interfaces/        # Generic interfaces and contracts
├── Pools/            # Pool implementations
├── Factories/        # Generic factory patterns
├── Policies/         # Pool management policies
├── Statistics/       # Performance monitoring
└── Extensions/       # Generic extension methods
```

Create these directories:

```bash
mkdir ObjectPool.Core/Interfaces
mkdir ObjectPool.Core/Pools
mkdir ObjectPool.Core/Factories
mkdir ObjectPool.Core/Policies
mkdir ObjectPool.Core/Statistics
mkdir ObjectPool.Core/Extensions
```

## Implementation Guide

### Step 4: Generic Interface Design (20 minutes)

**Core Pooling Interfaces with Constraints**

```csharp
// Generic interface foundation
namespace ObjectPool.Core.Interfaces;

// Basic poolable object constraint
public interface IPoolable
{
    void Reset();
    DateTime CreatedAt { get; }
    DateTime LastUsed { get; set; }
    int UseCount { get; }
}

// Factory pattern for object creation
public interface IObjectFactory<T> where T : class
{
    T Create();
    bool Validate(T obj);
    void Reset(T obj);
}

// Main pool interface with generic constraints
public interface IObjectPool<T> : IDisposable where T : class
{
    // Basic pool operations
    T Get();
    bool Return(T item);
    void Clear();

    // Pool statistics
    int Count { get; }
    int ActiveCount { get; }
    int MaxSize { get; }

    // Events for monitoring
    event EventHandler<ObjectPoolEventArgs<T>>? ObjectCreated;
    event EventHandler<ObjectPoolEventArgs<T>>? ObjectReturned;
    event EventHandler<ObjectPoolEventArgs<T>>? ObjectDestroyed;
}

// Pool with async support
public interface IAsyncObjectPool<T> : IObjectPool<T> where T : class
{
    Task<T> GetAsync(CancellationToken cancellationToken = default);
    ValueTask<bool> ReturnAsync(T item, CancellationToken cancellationToken = default);
}

// Pool policy interface for configuration
public interface IPoolPolicy<T> where T : class
{
    int MaximumRetained { get; }
    bool ShouldReturn(T obj);
    void OnReturn(T obj);
    void OnDestroy(T obj);
}

// Advanced pool with covariance for read operations
public interface IReadOnlyPool<out T> where T : class
{
    int Count { get; }
    int ActiveCount { get; }
    IEnumerable<T> GetSnapshot();
}

// Contravariant interface for pool management
public interface IPoolManager<in T> where T : class
{
    void RegisterPool(IObjectPool<T> pool);
    void UnregisterPool(IObjectPool<T> pool);
    void ClearAll();
}

// Generic pool with factory constraint
public interface IFactoryPool<T, TFactory> : IObjectPool<T>
    where T : class
    where TFactory : IObjectFactory<T>
{
    TFactory Factory { get; }
    void SetFactory(TFactory factory);
}

// Event arguments for pool monitoring
public class ObjectPoolEventArgs<T> : EventArgs where T : class
{
    public T Object { get; }
    public DateTime Timestamp { get; }
    public string PoolName { get; }

    public ObjectPoolEventArgs(T obj, string poolName)
    {
        Object = obj;
        PoolName = poolName;
        Timestamp = DateTime.UtcNow;
    }
}
```

### Step 5: Thread-Safe Generic Pool Implementation (35 minutes)

**High-Performance Pool with Advanced Generics**

```csharp
using System.Collections.Concurrent;
using ObjectPool.Core.Interfaces;

namespace ObjectPool.Core.Pools;

public class ThreadSafeObjectPool<T> : IAsyncObjectPool<T> where T : class, new()
{
    private readonly ConcurrentQueue<T> _pool;
    private readonly ConcurrentDictionary<T, DateTime> _activeObjects;
    private readonly IObjectFactory<T> _factory;
    private readonly IPoolPolicy<T> _policy;
    private readonly SemaphoreSlim _semaphore;
    private readonly Timer _cleanupTimer;
    private readonly string _poolName;

    private volatile int _count;
    private volatile bool _disposed;

    // Events for monitoring
    public event EventHandler<ObjectPoolEventArgs<T>>? ObjectCreated;
    public event EventHandler<ObjectPoolEventArgs<T>>? ObjectReturned;
    public event EventHandler<ObjectPoolEventArgs<T>>? ObjectDestroyed;

    public int Count => _count;
    public int ActiveCount => _activeObjects.Count;
    public int MaxSize { get; }
    public string Name => _poolName;

    public ThreadSafeObjectPool(
        IObjectFactory<T> factory,
        IPoolPolicy<T> policy,
        int maxSize = 100,
        string? name = null)
    {
        _factory = factory ?? throw new ArgumentNullException(nameof(factory));
        _policy = policy ?? throw new ArgumentNullException(nameof(policy));
        MaxSize = maxSize;
        _poolName = name ?? typeof(T).Name + "Pool";

        _pool = new ConcurrentQueue<T>();
        _activeObjects = new ConcurrentDictionary<T, DateTime>();
        _semaphore = new SemaphoreSlim(maxSize, maxSize);

        // Setup cleanup timer to remove stale objects
        _cleanupTimer = new Timer(CleanupStaleObjects, null,
            TimeSpan.FromMinutes(1), TimeSpan.FromMinutes(1));
    }

    // Synchronous get with timeout
    public T Get()
    {
        return GetAsync(CancellationToken.None).GetAwaiter().GetResult();
    }

    // Asynchronous get with cancellation support
    public async Task<T> GetAsync(CancellationToken cancellationToken = default)
    {
        ThrowIfDisposed();

        // Wait for available slot
        await _semaphore.WaitAsync(cancellationToken);

        try
        {
            T obj;

            // Try to get from pool first
            if (_pool.TryDequeue(out var pooledObj) && _factory.Validate(pooledObj))
            {
                obj = pooledObj;
                Interlocked.Decrement(ref _count);
            }
            else
            {
                // Create new object if pool is empty or validation failed
                obj = _factory.Create();
                ObjectCreated?.Invoke(this, new ObjectPoolEventArgs<T>(obj, _poolName));
            }

            // Track active object
            _activeObjects[obj] = DateTime.UtcNow;

            // Update usage statistics if object implements IPoolable
            if (obj is IPoolable poolable)
            {
                poolable.LastUsed = DateTime.UtcNow;
            }

            return obj;
        }
        catch
        {
            _semaphore.Release();
            throw;
        }
    }

    // Synchronous return
    public bool Return(T item)
    {
        return ReturnAsync(item).GetAwaiter().GetResult();
    }

    // Asynchronous return
    public async ValueTask<bool> ReturnAsync(T item, CancellationToken cancellationToken = default)
    {
        if (item == null || _disposed)
            return false;

        try
        {
            // Remove from active tracking
            _activeObjects.TryRemove(item, out _);

            // Check pool policy
            if (!_policy.ShouldReturn(item))
            {
                DestroyObject(item);
                return false;
            }

            // Reset object state
            _factory.Reset(item);
            _policy.OnReturn(item);

            // Return to pool if there's space
            if (_count < _policy.MaximumRetained)
            {
                _pool.Enqueue(item);
                Interlocked.Increment(ref _count);

                ObjectReturned?.Invoke(this, new ObjectPoolEventArgs<T>(item, _poolName));
                return true;
            }
            else
            {
                // Pool is full, destroy object
                DestroyObject(item);
                return false;
            }
        }
        finally
        {
            _semaphore.Release();
        }
    }

    public void Clear()
    {
        ThrowIfDisposed();

        // Clear the pool
        while (_pool.TryDequeue(out var obj))
        {
            DestroyObject(obj);
        }

        Interlocked.Exchange(ref _count, 0);

        // Note: We don't clear active objects as they're still in use
    }

    // Generic method with additional constraints
    public TResult ProcessWithPool<TResult>(Func<T, TResult> processor) where TResult : class
    {
        var obj = Get();
        try
        {
            return processor(obj);
        }
        finally
        {
            Return(obj);
        }
    }

    // Async version with constraints
    public async Task<TResult> ProcessWithPoolAsync<TResult>(
        Func<T, Task<TResult>> processor,
        CancellationToken cancellationToken = default) where TResult : class
    {
        var obj = await GetAsync(cancellationToken);
        try
        {
            return await processor(obj);
        }
        finally
        {
            await ReturnAsync(obj, cancellationToken);
        }
    }

    private void CleanupStaleObjects(object? state)
    {
        if (_disposed) return;

        var now = DateTime.UtcNow;
        var staleThreshold = TimeSpan.FromMinutes(30);

        // Remove stale objects from active tracking
        var staleObjects = _activeObjects
            .Where(kvp => now - kvp.Value > staleThreshold)
            .Select(kvp => kvp.Key)
            .ToList();

        foreach (var obj in staleObjects)
        {
            if (_activeObjects.TryRemove(obj, out _))
            {
                // Object was likely leaked, release semaphore
                _semaphore.Release();
            }
        }
    }

    private void DestroyObject(T obj)
    {
        try
        {
            _policy.OnDestroy(obj);

            if (obj is IDisposable disposable)
            {
                disposable.Dispose();
            }

            ObjectDestroyed?.Invoke(this, new ObjectPoolEventArgs<T>(obj, _poolName));
        }
        catch (Exception ex)
        {
            // Log destruction error but don't throw
            Console.WriteLine($"Error destroying pooled object: {ex.Message}");
        }
    }

    private void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(ThreadSafeObjectPool<T>));
    }

    public void Dispose()
    {
        if (_disposed) return;

        _disposed = true;

        _cleanupTimer?.Dispose();

        // Clear all pooled objects
        Clear();

        // Dispose active objects if possible
        foreach (var obj in _activeObjects.Keys)
        {
            if (obj is IDisposable disposable)
            {
                try
                {
                    disposable.Dispose();
                }
                catch
                {
                    // Ignore disposal errors
                }
            }
        }

        _activeObjects.Clear();
        _semaphore?.Dispose();
    }
}
```

### Step 6: Generic Factory Patterns (25 minutes)

**Advanced Factory with Constraints**

```csharp
namespace ObjectPool.Core.Factories;

// Default factory using parameterless constructor constraint
public class DefaultFactory<T> : IObjectFactory<T> where T : class, new()
{
    public virtual T Create()
    {
        return new T();
    }

    public virtual bool Validate(T obj)
    {
        return obj != null;
    }

    public virtual void Reset(T obj)
    {
        if (obj is IPoolable poolable)
        {
            poolable.Reset();
        }

        // Use reflection to reset common properties
        ResetCommonProperties(obj);
    }

    private static void ResetCommonProperties(T obj)
    {
        var type = typeof(T);

        // Reset properties that commonly need resetting
        var properties = type.GetProperties(BindingFlags.Public | BindingFlags.Instance)
            .Where(p => p.CanWrite && IsResettableType(p.PropertyType));

        foreach (var prop in properties)
        {
            try
            {
                var defaultValue = GetDefaultValue(prop.PropertyType);
                prop.SetValue(obj, defaultValue);
            }
            catch
            {
                // Ignore reset errors for individual properties
            }
        }
    }

    private static bool IsResettableType(Type type)
    {
        return type == typeof(string) ||
               type == typeof(int) ||
               type == typeof(bool) ||
               type == typeof(DateTime) ||
               type.IsValueType;
    }

    private static object? GetDefaultValue(Type type)
    {
        return type.IsValueType ? Activator.CreateInstance(type) : null;
    }
}

// Factory with custom creation delegate
public class DelegateFactory<T> : IObjectFactory<T> where T : class
{
    private readonly Func<T> _creator;
    private readonly Action<T>? _resetter;
    private readonly Func<T, bool>? _validator;

    public DelegateFactory(
        Func<T> creator,
        Action<T>? resetter = null,
        Func<T, bool>? validator = null)
    {
        _creator = creator ?? throw new ArgumentNullException(nameof(creator));
        _resetter = resetter;
        _validator = validator;
    }

    public T Create() => _creator();

    public bool Validate(T obj) => _validator?.Invoke(obj) ?? (obj != null);

    public void Reset(T obj)
    {
        if (_resetter != null)
        {
            _resetter(obj);
        }
        else if (obj is IPoolable poolable)
        {
            poolable.Reset();
        }
    }
}

// Factory for types with specific constructor constraints
public class ParameterizedFactory<T, TParam> : IObjectFactory<T>
    where T : class
{
    private readonly Func<TParam, T> _creator;
    private readonly TParam _parameter;
    private readonly Action<T>? _resetter;

    public ParameterizedFactory(Func<TParam, T> creator, TParam parameter, Action<T>? resetter = null)
    {
        _creator = creator ?? throw new ArgumentNullException(nameof(creator));
        _parameter = parameter;
        _resetter = resetter;
    }

    public T Create() => _creator(_parameter);

    public bool Validate(T obj) => obj != null;

    public void Reset(T obj)
    {
        _resetter?.Invoke(obj);
    }
}

// Generic factory registry with variance
public class FactoryRegistry<T> where T : class
{
    private readonly ConcurrentDictionary<Type, IObjectFactory<T>> _factories;

    public FactoryRegistry()
    {
        _factories = new ConcurrentDictionary<Type, IObjectFactory<T>>();
    }

    public void Register<TImplementation>(IObjectFactory<TImplementation> factory)
        where TImplementation : class, T
    {
        _factories[typeof(TImplementation)] = factory as IObjectFactory<T> ??
            throw new InvalidOperationException($"Factory for {typeof(TImplementation)} is not compatible");
    }

    public IObjectFactory<T>? GetFactory<TImplementation>() where TImplementation : class, T
    {
        _factories.TryGetValue(typeof(TImplementation), out var factory);
        return factory;
    }

    public IObjectFactory<T> GetOrCreateFactory<TImplementation>()
        where TImplementation : class, T, new()
    {
        return GetFactory<TImplementation>() ?? new DefaultFactory<TImplementation>() as IObjectFactory<T>
            ?? throw new InvalidOperationException("Cannot create factory");
    }
}
```

### Step 7: Pool Manager with Variance (20 minutes)

**Advanced Pool Management with Covariance/Contravariance**

```csharp
namespace ObjectPool.Core.Pools;

// Pool manager demonstrating variance
public class PoolManager : IDisposable
{
    private readonly ConcurrentDictionary<Type, object> _pools;
    private readonly ConcurrentDictionary<string, object> _namedPools;
    private readonly object _lock = new();
    private bool _disposed;

    public PoolManager()
    {
        _pools = new ConcurrentDictionary<Type, object>();
        _namedPools = new ConcurrentDictionary<string, object>();
    }

    // Generic method with constraint
    public IObjectPool<T> GetOrCreatePool<T>(
        IObjectFactory<T>? factory = null,
        IPoolPolicy<T>? policy = null,
        int maxSize = 100,
        string? name = null) where T : class, new()
    {
        var type = typeof(T);
        var key = name ?? type.Name;

        if (name != null && _namedPools.TryGetValue(key, out var namedPool))
        {
            return (IObjectPool<T>)namedPool;
        }

        if (name == null && _pools.TryGetValue(type, out var existingPool))
        {
            return (IObjectPool<T>)existingPool;
        }

        lock (_lock)
        {
            // Double-check pattern
            var checkKey = name ?? type.Name;
            var checkDict = name != null ? _namedPools : _pools;
            var checkType = name != null ? checkKey : (object)type;

            if (checkDict.TryGetValue(checkType, out var doubleCheckPool))
            {
                return (IObjectPool<T>)doubleCheckPool;
            }

            // Create new pool
            factory ??= new DefaultFactory<T>();
            policy ??= new DefaultPoolPolicy<T>();

            var newPool = new ThreadSafeObjectPool<T>(factory, policy, maxSize, name);

            if (name != null)
            {
                _namedPools[name] = newPool;
            }
            else
            {
                _pools[type] = newPool;
            }

            return newPool;
        }
    }

    // Variance example: covariant method
    public IReadOnlyCollection<IReadOnlyPool<T>> GetReadOnlyPools<T>() where T : class
    {
        var pools = new List<IReadOnlyPool<T>>();

        foreach (var pool in _pools.Values.OfType<IObjectPool<T>>())
        {
            pools.Add(pool); // Covariance: IObjectPool<T> -> IReadOnlyPool<T>
        }

        return pools;
    }

    // Generic method with multiple constraints
    public TPool GetPool<T, TPool>(string name)
        where T : class
        where TPool : class, IObjectPool<T>
    {
        if (_namedPools.TryGetValue(name, out var pool) && pool is TPool typedPool)
        {
            return typedPool;
        }

        throw new InvalidOperationException($"Pool '{name}' not found or not of type {typeof(TPool).Name}");
    }

    public void RegisterPool<T>(string name, IObjectPool<T> pool) where T : class
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(name);
        ArgumentNullException.ThrowIfNull(pool);

        _namedPools[name] = pool;
    }

    public bool RemovePool<T>(string? name = null) where T : class
    {
        if (name != null)
        {
            return _namedPools.TryRemove(name, out var removed) &&
                   (removed as IDisposable)?.Let(d => d.Dispose()) == null;
        }
        else
        {
            return _pools.TryRemove(typeof(T), out var removed) &&
                   (removed as IDisposable)?.Let(d => d.Dispose()) == null;
        }
    }

    public void ClearAll()
    {
        lock (_lock)
        {
            foreach (var pool in _pools.Values.OfType<IDisposable>())
            {
                try
                {
                    pool.Dispose();
                }
                catch
                {
                    // Ignore disposal errors
                }
            }

            foreach (var pool in _namedPools.Values.OfType<IDisposable>())
            {
                try
                {
                    pool.Dispose();
                }
                catch
                {
                    // Ignore disposal errors
                }
            }

            _pools.Clear();
            _namedPools.Clear();
        }
    }

    public PoolStatistics GetStatistics()
    {
        var stats = new PoolStatistics();

        foreach (var pool in _pools.Values.OfType<IObjectPool<object>>())
        {
            stats.TotalPools++;
            stats.TotalObjects += pool.Count;
            stats.ActiveObjects += pool.ActiveCount;
        }

        foreach (var pool in _namedPools.Values.OfType<IObjectPool<object>>())
        {
            stats.TotalPools++;
            stats.TotalObjects += pool.Count;
            stats.ActiveObjects += pool.ActiveCount;
        }

        return stats;
    }

    public void Dispose()
    {
        if (_disposed) return;

        ClearAll();
        _disposed = true;
    }
}

// Extension method for functional-style operations
public static class ObjectExtensions
{
    public static TResult Let<T, TResult>(this T obj, Func<T, TResult> func)
    {
        return func(obj);
    }
}

public class PoolStatistics
{
    public int TotalPools { get; set; }
    public int TotalObjects { get; set; }
    public int ActiveObjects { get; set; }
    public int AvailableObjects => TotalObjects - ActiveObjects;
    public double UtilizationPercentage => TotalObjects > 0 ? (double)ActiveObjects / TotalObjects * 100 : 0;
}

// Default pool policy
public class DefaultPoolPolicy<T> : IPoolPolicy<T> where T : class
{
    public int MaximumRetained { get; } = 100;

    public bool ShouldReturn(T obj)
    {
        // Basic validation - can be overridden
        return obj != null;
    }

    public void OnReturn(T obj)
    {
        // Default: do nothing
    }

    public void OnDestroy(T obj)
    {
        if (obj is IDisposable disposable)
        {
            disposable.Dispose();
        }
    }
}
```

## Key Generic Programming Patterns

### 1. Constraint Composition

```csharp
// ✅ Multiple constraints for type safety
public class PoolableFactory<T> : IObjectFactory<T>
    where T : class, IPoolable, new()
{
    public T Create() => new T();
    public void Reset(T obj) => obj.Reset(); // Can call Reset() due to constraint
}

// ✅ Constraint combinations
public interface IAdvancedPool<T> : IObjectPool<T>
    where T : class, IDisposable, IComparable<T>
{
    T GetBest();
}
```

### 2. Variance Usage

```csharp
// ✅ Covariance for read-only operations
IReadOnlyPool<object> readOnlyPool = new ThreadSafeObjectPool<string>(...);

// ✅ Contravariance for input operations
IPoolManager<object> manager = new SpecificPoolManager<string>();
```

### 3. Generic Factory Patterns

```csharp
// ✅ Factory with constraints
public static IObjectPool<T> CreatePool<T>()
    where T : class, new()
{
    return new ThreadSafeObjectPool<T>(
        new DefaultFactory<T>(),
        new DefaultPoolPolicy<T>()
    );
}
```

## Testing Your Understanding

Create a comprehensive object pool demonstration:

```csharp
using ObjectPool.Core.Pools;
using ObjectPool.Core.Factories;
using ObjectPool.Core.Interfaces;

// Example poolable class
public class GameEntity : IPoolable, IDisposable
{
    public string Name { get; set; } = string.Empty;
    public Vector3 Position { get; set; }
    public bool IsActive { get; set; }
    public DateTime CreatedAt { get; }
    public DateTime LastUsed { get; set; }
    public int UseCount { get; private set; }

    public GameEntity()
    {
        CreatedAt = DateTime.UtcNow;
        Reset();
    }

    public void Reset()
    {
        Name = string.Empty;
        Position = Vector3.Zero;
        IsActive = false;
        LastUsed = DateTime.UtcNow;
        UseCount++;
    }

    public void Dispose()
    {
        // Cleanup resources
        IsActive = false;
    }
}

// Usage demonstration
var manager = new PoolManager();

// Create pools with different configurations
var entityPool = manager.GetOrCreatePool<GameEntity>(
    factory: new DefaultFactory<GameEntity>(),
    policy: new DefaultPoolPolicy<GameEntity>(),
    maxSize: 1000,
    name: "GameEntities"
);

var stringPool = manager.GetOrCreatePool<StringBuilder>(
    factory: new DelegateFactory<StringBuilder>(
        creator: () => new StringBuilder(),
        resetter: sb => sb.Clear(),
        validator: sb => sb != null
    ),
    maxSize: 50
);

// Subscribe to pool events
entityPool.ObjectCreated += (sender, e) =>
    Console.WriteLine($"Created entity at {e.Timestamp}");

entityPool.ObjectReturned += (sender, e) =>
    Console.WriteLine($"Returned entity: {e.Object.Name}");

// Use the pools
Console.WriteLine("=== Object Pool Demo ===");

// Get objects from pool
var entity1 = await entityPool.GetAsync();
entity1.Name = "Player";
entity1.Position = new Vector3(10, 0, 5);
entity1.IsActive = true;

var entity2 = entityPool.Get();
entity2.Name = "Enemy";
entity2.Position = new Vector3(-5, 0, 10);

var sb = stringPool.Get();
sb.Append("Hello from pooled StringBuilder!");

Console.WriteLine($"Entity 1: {entity1.Name} at {entity1.Position}");
Console.WriteLine($"Entity 2: {entity2.Name} at {entity2.Position}");
Console.WriteLine($"StringBuilder: {sb}");

// Process with automatic return
var result = await entityPool.ProcessWithPoolAsync(async entity =>
{
    entity.Name = "Temporary";
    await Task.Delay(100); // Simulate processing
    return $"Processed {entity.Name}";
});

Console.WriteLine($"Process result: {result}");

// Return objects to pool
await entityPool.ReturnAsync(entity1);
entityPool.Return(entity2);
stringPool.Return(sb);

// Check pool statistics
var stats = manager.GetStatistics();
Console.WriteLine($"\nPool Statistics:");
Console.WriteLine($"Total Pools: {stats.TotalPools}");
Console.WriteLine($"Total Objects: {stats.TotalObjects}");
Console.WriteLine($"Active Objects: {stats.ActiveObjects}");
Console.WriteLine($"Utilization: {stats.UtilizationPercentage:F1}%");

// Cleanup
manager.Dispose();
```

## Extension Challenges

1. **Smart Pool Sizing**: Implement dynamic pool sizing based on usage patterns
2. **Pool Warming**: Pre-populate pools with objects during initialization
3. **Memory Pressure**: Automatically shrink pools when memory is low
4. **Pool Analytics**: Add detailed performance metrics and reporting
5. **Distributed Pooling**: Create pools that work across multiple processes
6. **Type Hierarchies**: Support pooling with inheritance hierarchies
7. **Pool Serialization**: Save and restore pool states across application restarts

## Common Pitfalls & Solutions

1. **Generic Constraint Violations**:

   ```csharp
   // ❌ Missing required constraints
   public class WeakPool<T> : IObjectPool<T>
   {
       public T Get() => new T(); // Error: T doesn't have new() constraint
   }

   // ✅ Proper constraints
   public class StrongPool<T> : IObjectPool<T> where T : class, new()
   {
       public T Get() => new T(); // Works!
   }
   ```

2. **Memory Leaks in Pools**:

   ```csharp
   // ❌ Not disposing pooled objects
   public void BadReturn(T obj)
   {
       _pool.Enqueue(obj); // Might leak if obj holds resources
   }

   // ✅ Proper cleanup
   public void GoodReturn(T obj)
   {
       if (obj is IPoolable poolable)
           poolable.Reset(); // Clear references

       _pool.Enqueue(obj);
   }
   ```

3. **Thread Safety Issues**:

   ```csharp
   // ❌ Not thread-safe
   private readonly List<T> _pool = new();

   public T Get()
   {
       if (_pool.Count > 0)
       {
           var obj = _pool[0]; // Race condition!
           _pool.RemoveAt(0);
           return obj;
       }
       return new T();
   }

   // ✅ Thread-safe with concurrent collections
   private readonly ConcurrentQueue<T> _pool = new();

   public T Get()
   {
       return _pool.TryDequeue(out var obj) ? obj : new T();
   }
   ```

4. **Generic Variance Misuse**:

   ```csharp
   // ❌ Incorrect variance usage
   IObjectPool<string> stringPool = ...;
   IObjectPool<object> objectPool = stringPool; // Compilation error

   // ✅ Use appropriate variance
   IReadOnlyPool<object> readOnlyPool = stringPool; // OK if IReadOnlyPool<out T>
   ```

## Resources

- 📚 **[Progress Tracker](../progress-tracker.md)** - Track your learning progress
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Generics Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)**
- 📘 **[Generic Constraints](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)**

## Building on Previous Projects

This project advances concepts from all previous projects:

- **Project 1-2**: Advanced type design with generic constraints
- **Project 3**: Extension methods for generic types
- **Project 4**: Generic state machines and pattern matching
- **Project 5**: LINQ with generic collections
- **Project 6**: Async operations with generic types
- **Project 7**: Event-driven programming with generic delegates
- **New Skills**: Advanced generics, constraints, and performance optimization

## Example Usage

```csharp
// Basic object pool usage
var stringPool = new ObjectPool<string>(() => string.Empty, maxSize: 100);

using var borrowed = stringPool.Get();
var str = borrowed.Value; // Use the pooled string
// Automatically returned to pool when disposed

// Generic pool with constraints
var gameObjectPool = new ObjectPool<T>() where T : IGameObject, new()
{
    Factory = () => new T(),
    Reset = obj => obj.Reset(),
    MaxSize = 50
};

// Variance example
IObjectPool<IReadOnlyList<string>> readOnlyPool =
    new ObjectPool<List<string>>();  // Covariant conversion

// Generic factory pattern
var poolManager = new PoolManager()
    .RegisterPool<Bullet>(maxSize: 1000)
    .RegisterPool<Particle>(maxSize: 5000)
    .RegisterPool<AudioSource>(maxSize: 32);

var bullet = poolManager.Get<Bullet>();
```

## Performance Focus

This project emphasizes performance optimization:

- Reducing garbage collection pressure through object reuse
- Minimizing allocations in hot code paths
- Understanding when object pooling provides benefits
- Measuring and monitoring pool performance

## Advanced Generic Patterns

- **Generic Constraints**: `where T : class, IDisposable, new()`
- **Variance**: `IEnumerable<out T>`, `IComparer<in T>`
- **Generic Delegates**: `Func<T, TResult>`, `Action<T>`
- **Conditional Constraints**: Using constraint combinations effectively

## Contributing

When implementing this project:

1. Use appropriate generic constraints for type safety
2. Implement thread-safe operations correctly
3. Consider performance implications of design choices
4. Use variance appropriately for interface design
5. Write comprehensive benchmarks and performance tests
