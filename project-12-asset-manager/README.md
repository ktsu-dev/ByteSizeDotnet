# Project 12: Game Asset Manager

**Focus**: Repository Pattern & Unit of Work  
**Time**: ~2 hours  
**Deliverable**: `AssetManager` library with data access patterns

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Repository Pattern Guidelines](../microsoft-design-guidelines.md#repository-pattern)** - Data access abstraction
- **[Unit of Work Guidelines](../microsoft-design-guidelines.md#unit-of-work-pattern)** - Transaction management
- **[Caching Guidelines](../microsoft-design-guidelines.md#caching-patterns)** - Performance optimization

## Learning Objectives

- Master repository and unit of work patterns
- Understand data access layer abstraction
- Learn caching strategies and cache invalidation
- Practice transaction management patterns
- Build performant asset loading systems

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Repository Pattern Guidelines](../microsoft-design-guidelines.md#repository-pattern) before starting

2. **Start Coding** 💻  
   See [detailed project guide](asset-manager-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-12-asset-manager/
├── README.md                    # This file
├── asset-manager-project-guide.md # Detailed implementation guide
├── src/                         # Your implementation
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
```

## What You'll Build

A sophisticated asset management system featuring:

- **Repository Pattern** - Clean abstraction for asset data access and queries
- **Unit of Work Pattern** - Transaction management and coordinated asset operations
- **Specification Pattern** - Flexible asset querying and filtering criteria
- **Caching Strategy** - Multi-level caching with LRU, LFU, and TTL eviction policies
- **Dependency Resolution** - Automatic loading of asset dependencies and references
- **Async Loading Pipeline** - Non-blocking asset loading with progress tracking
- **Hot-Reload System** - Development-time asset reloading and change detection
- **Asset Versioning** - Version management and compatibility checking

## Key Concepts

### Repository Pattern

```csharp
// Clean abstraction for asset data access
public interface IAssetRepository<T> where T : class, IAsset
{
    Task<T?> GetByIdAsync(string id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(ISpecification<T> specification);
    Task AddAsync(T asset);
    Task UpdateAsync(T asset);
    Task DeleteAsync(string id);
}

// Usage
var textureRepo = new AssetRepository<Texture>(dataSource, cache);
var playerTexture = await textureRepo.GetByIdAsync("player_idle.png");
```

### Unit of Work Pattern

```csharp
// Coordinate multiple repository operations as single transaction
using var unitOfWork = new AssetUnitOfWork();
var textureRepo = unitOfWork.GetRepository<Texture>();
var modelRepo = unitOfWork.GetRepository<Model>();

await textureRepo.LoadAsync("character_diffuse.png");
await modelRepo.LoadAsync("character_model.fbx");
await unitOfWork.CommitAsync(); // All or nothing
```

### Specification Pattern

```csharp
// Flexible query building
var largeTextures = new AndSpecification<Texture>(
    new TextureSizeSpecification(minWidth: 1024),
    new FileFormatSpecification("png", "jpg")
);

var assets = await repository.FindAsync(largeTextures);
```

### Core Data Access Patterns Applied

- **Repository** - Abstracting asset storage and retrieval
- **Unit of Work** - Transaction boundaries for asset operations
- **Specification** - Composable query logic
- **Cache-Aside** - Performance optimization through caching
- **Observer** - Change notifications and hot-reload
- **Strategy** - Pluggable loading and caching strategies

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-12-asset-manager/` directory:

```bash
# Create the main solution file
dotnet new sln -n AssetManager

# Create the main library project
dotnet new classlib -n AssetManager.Core -f net8.0

# Create the console application
dotnet new console -n AssetManager.App -f net8.0

# Create the test project
dotnet new mstest -n AssetManager.Tests -f net8.0

# Add projects to solution
dotnet sln add AssetManager.Core/AssetManager.Core.csproj
dotnet sln add AssetManager.App/AssetManager.App.csproj
dotnet sln add AssetManager.Tests/AssetManager.Tests.csproj

# Add project references
dotnet add AssetManager.App/AssetManager.App.csproj reference AssetManager.Core/AssetManager.Core.csproj
dotnet add AssetManager.Tests/AssetManager.Tests.csproj reference AssetManager.Core/AssetManager.Core.csproj
```

### Step 2: Configure Project Files

Update the `AssetManager.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>AssetManager.Core</PackageId>
    <Title>Game Asset Manager</Title>
    <Description>A library for game asset management using repository and unit of work patterns</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="System.Text.Json" Version="8.0.0" />
  </ItemGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your asset management code:

```
AssetManager.Core/
├── Models/           # Asset models and DTOs
├── Repositories/     # Repository implementations
├── UnitOfWork/       # Unit of work implementation
├── Specifications/   # Query specification pattern
├── Caching/          # Caching strategies and implementations
├── Loading/          # Asset loading pipeline
├── Dependencies/     # Dependency resolution
└── Events/           # Asset change events and notifications
```

Create these directories:

```bash
mkdir AssetManager.Core/Models
mkdir AssetManager.Core/Repositories
mkdir AssetManager.Core/UnitOfWork
mkdir AssetManager.Core/Specifications
mkdir AssetManager.Core/Caching
mkdir AssetManager.Core/Loading
mkdir AssetManager.Core/Dependencies
mkdir AssetManager.Core/Events
```

## Implementation Guide

### Step 4: Core Asset Models (25 minutes)

**Foundation Asset Types and Interfaces**

```csharp
namespace AssetManager.Core.Models;

public interface IAsset
{
    string Id { get; }
    string Name { get; }
    string FilePath { get; }
    AssetType Type { get; }
    long FileSize { get; }
    DateTime CreatedAt { get; }
    DateTime ModifiedAt { get; }
    DateTime? LoadedAt { get; }
    AssetStatus Status { get; }
    IDictionary<string, object> Metadata { get; }
    IReadOnlyList<string> Dependencies { get; }
}

public enum AssetType
{
    Unknown,
    Texture,
    Model,
    Audio,
    Video,
    Font,
    Shader,
    Material,
    Animation,
    Scene,
    Prefab,
    Script
}

public enum AssetStatus
{
    Unloaded,
    Loading,
    Loaded,
    Failed,
    Outdated
}

public abstract class Asset : IAsset
{
    private readonly List<string> _dependencies = new();
    private readonly Dictionary<string, object> _metadata = new();

    protected Asset(string id, string name, string filePath, AssetType type)
    {
        Id = id ?? throw new ArgumentNullException(nameof(id));
        Name = name ?? throw new ArgumentNullException(nameof(name));
        FilePath = filePath ?? throw new ArgumentNullException(nameof(filePath));
        Type = type;
        CreatedAt = DateTime.UtcNow;
        ModifiedAt = DateTime.UtcNow;
        Status = AssetStatus.Unloaded;
    }

    public string Id { get; }
    public string Name { get; }
    public string FilePath { get; }
    public AssetType Type { get; }
    public long FileSize { get; protected set; }
    public DateTime CreatedAt { get; }
    public DateTime ModifiedAt { get; protected set; }
    public DateTime? LoadedAt { get; protected set; }
    public AssetStatus Status { get; protected set; }
    public IDictionary<string, object> Metadata => _metadata;
    public IReadOnlyList<string> Dependencies => _dependencies;

    public void AddDependency(string assetId)
    {
        if (!string.IsNullOrEmpty(assetId) && !_dependencies.Contains(assetId))
        {
            _dependencies.Add(assetId);
        }
    }

    public void RemoveDependency(string assetId)
    {
        _dependencies.Remove(assetId);
    }

    public virtual void SetLoaded()
    {
        Status = AssetStatus.Loaded;
        LoadedAt = DateTime.UtcNow;
    }

    public virtual void SetFailed()
    {
        Status = AssetStatus.Failed;
        LoadedAt = null;
    }

    public void UpdateModifiedTime()
    {
        ModifiedAt = DateTime.UtcNow;
        if (Status == AssetStatus.Loaded)
        {
            Status = AssetStatus.Outdated;
        }
    }

    public override string ToString() => $"{Type} '{Name}' ({Id})";
}

// Concrete asset implementations
public class Texture : Asset
{
    public Texture(string id, string name, string filePath)
        : base(id, name, filePath, AssetType.Texture)
    {
    }

    public int Width { get; set; }
    public int Height { get; set; }
    public TextureFormat Format { get; set; } = TextureFormat.RGBA32;
    public bool HasMipMaps { get; set; }
    public byte[]? ImageData { get; set; }

    public override void SetLoaded()
    {
        base.SetLoaded();
        Metadata["Width"] = Width;
        Metadata["Height"] = Height;
        Metadata["Format"] = Format.ToString();
        Metadata["HasMipMaps"] = HasMipMaps;
    }
}

public class Model : Asset
{
    public Model(string id, string name, string filePath)
        : base(id, name, filePath, AssetType.Model)
    {
    }

    public int VertexCount { get; set; }
    public int TriangleCount { get; set; }
    public IList<string> MaterialReferences { get; set; } = new List<string>();
    public IList<string> TextureReferences { get; set; } = new List<string>();
    public byte[]? ModelData { get; set; }

    public override void SetLoaded()
    {
        base.SetLoaded();

        // Add material and texture dependencies
        foreach (var material in MaterialReferences)
        {
            AddDependency(material);
        }

        foreach (var texture in TextureReferences)
        {
            AddDependency(texture);
        }

        Metadata["VertexCount"] = VertexCount;
        Metadata["TriangleCount"] = TriangleCount;
        Metadata["MaterialCount"] = MaterialReferences.Count;
        Metadata["TextureCount"] = TextureReferences.Count;
    }
}

public class AudioClip : Asset
{
    public AudioClip(string id, string name, string filePath)
        : base(id, name, filePath, AssetType.Audio)
    {
    }

    public TimeSpan Duration { get; set; }
    public int SampleRate { get; set; }
    public int Channels { get; set; }
    public AudioFormat Format { get; set; } = AudioFormat.PCM16;
    public byte[]? AudioData { get; set; }

    public override void SetLoaded()
    {
        base.SetLoaded();
        Metadata["Duration"] = Duration.TotalSeconds;
        Metadata["SampleRate"] = SampleRate;
        Metadata["Channels"] = Channels;
        Metadata["Format"] = Format.ToString();
    }
}

public enum TextureFormat
{
    RGBA32,
    RGB24,
    DXT1,
    DXT5,
    BC7,
    ASTC_4x4
}

public enum AudioFormat
{
    PCM16,
    PCM24,
    PCM32,
    MP3,
    OGG,
    WAV
}

// Asset reference for lazy loading
public class AssetReference<T> where T : class, IAsset
{
    private readonly Func<Task<T?>> _loader;
    private T? _asset;
    private bool _isLoaded;

    public AssetReference(string id, Func<Task<T?>> loader)
    {
        Id = id ?? throw new ArgumentNullException(nameof(id));
        _loader = loader ?? throw new ArgumentNullException(nameof(loader));
    }

    public string Id { get; }
    public bool IsLoaded => _isLoaded;
    public T? Asset => _asset;

    public async Task<T?> LoadAsync()
    {
        if (!_isLoaded)
        {
            _asset = await _loader();
            _isLoaded = true;
        }
        return _asset;
    }

    public void Unload()
    {
        _asset = null;
        _isLoaded = false;
    }

    public static implicit operator string(AssetReference<T> reference) => reference.Id;
    public override string ToString() => $"AssetRef<{typeof(T).Name}>({Id})";
}
```

### Step 5: Repository Pattern Implementation (35 minutes)

**Generic Asset Repository with Caching**

```csharp
namespace AssetManager.Core.Repositories;

// Repository pattern interface
public interface IAssetRepository<T> where T : class, IAsset
{
    Task<T?> GetByIdAsync(string id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<IEnumerable<T>> FindAsync(ISpecification<T> specification);
    Task<bool> ExistsAsync(string id);
    Task AddAsync(T asset);
    Task UpdateAsync(T asset);
    Task DeleteAsync(string id);
    Task<int> CountAsync();
    Task<int> CountAsync(ISpecification<T> specification);
}

// Generic repository implementation with caching
public class AssetRepository<T> : IAssetRepository<T> where T : class, IAsset
{
    private readonly IAssetDataSource<T> _dataSource;
    private readonly IAssetCache<T> _cache;
    private readonly IAssetLoader<T> _loader;

    public AssetRepository(IAssetDataSource<T> dataSource, IAssetCache<T> cache, IAssetLoader<T> loader)
    {
        _dataSource = dataSource ?? throw new ArgumentNullException(nameof(dataSource));
        _cache = cache ?? throw new ArgumentNullException(nameof(cache));
        _loader = loader ?? throw new ArgumentNullException(nameof(loader));
    }

    public async Task<T?> GetByIdAsync(string id)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(id);

        // Check cache first (Cache-Aside pattern)
        var cachedAsset = await _cache.GetAsync(id);
        if (cachedAsset != null)
        {
            return cachedAsset;
        }

        // Load from data source
        var asset = await _dataSource.GetByIdAsync(id);
        if (asset != null)
        {
            // Load asset content if needed
            if (asset.Status == AssetStatus.Unloaded)
            {
                await _loader.LoadAsync(asset);
            }

            // Cache the loaded asset
            await _cache.SetAsync(id, asset);
        }

        return asset;
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        var assets = await _dataSource.GetAllAsync();
        var loadedAssets = new List<T>();

        foreach (var asset in assets)
        {
            // Check cache first
            var cachedAsset = await _cache.GetAsync(asset.Id);
            if (cachedAsset != null)
            {
                loadedAssets.Add(cachedAsset);
            }
            else
            {
                // Load if needed and cache
                if (asset.Status == AssetStatus.Unloaded)
                {
                    await _loader.LoadAsync(asset);
                }
                await _cache.SetAsync(asset.Id, asset);
                loadedAssets.Add(asset);
            }
        }

        return loadedAssets;
    }

    public async Task<IEnumerable<T>> FindAsync(ISpecification<T> specification)
    {
        ArgumentNullException.ThrowIfNull(specification);

        var allAssets = await GetAllAsync();
        return allAssets.Where(specification.IsSatisfiedBy);
    }

    public async Task<bool> ExistsAsync(string id)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(id);
        return await _dataSource.ExistsAsync(id);
    }

    public async Task AddAsync(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);

        await _dataSource.AddAsync(asset);
        await _cache.SetAsync(asset.Id, asset);
    }

    public async Task UpdateAsync(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);

        await _dataSource.UpdateAsync(asset);
        await _cache.RemoveAsync(asset.Id); // Invalidate cache
    }

    public async Task DeleteAsync(string id)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(id);

        await _dataSource.DeleteAsync(id);
        await _cache.RemoveAsync(id);
    }

    public async Task<int> CountAsync()
    {
        return await _dataSource.CountAsync();
    }

    public async Task<int> CountAsync(ISpecification<T> specification)
    {
        ArgumentNullException.ThrowIfNull(specification);

        var assets = await _dataSource.GetAllAsync();
        return assets.Count(specification.IsSatisfiedBy);
    }
}

// Data source abstraction
public interface IAssetDataSource<T> where T : class, IAsset
{
    Task<T?> GetByIdAsync(string id);
    Task<IEnumerable<T>> GetAllAsync();
    Task<bool> ExistsAsync(string id);
    Task AddAsync(T asset);
    Task UpdateAsync(T asset);
    Task DeleteAsync(string id);
    Task<int> CountAsync();
}

// File system data source implementation
public class FileSystemAssetDataSource<T> : IAssetDataSource<T> where T : class, IAsset
{
    private readonly string _basePath;
    private readonly Dictionary<string, T> _assetRegistry = new();
    private readonly IAssetSerializer<T> _serializer;

    public FileSystemAssetDataSource(string basePath, IAssetSerializer<T> serializer)
    {
        _basePath = basePath ?? throw new ArgumentNullException(nameof(basePath));
        _serializer = serializer ?? throw new ArgumentNullException(nameof(serializer));

        if (!Directory.Exists(_basePath))
        {
            Directory.CreateDirectory(_basePath);
        }

        InitializeRegistry();
    }

    public async Task<T?> GetByIdAsync(string id)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(id);

        if (_assetRegistry.TryGetValue(id, out var asset))
        {
            return asset;
        }

        // Try to load from metadata file
        var metadataPath = Path.Combine(_basePath, $"{id}.meta");
        if (File.Exists(metadataPath))
        {
            var json = await File.ReadAllTextAsync(metadataPath);
            asset = _serializer.Deserialize(json);
            if (asset != null)
            {
                _assetRegistry[id] = asset;
                return asset;
            }
        }

        return null;
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        // Scan for new assets that might have been added
        await ScanForNewAssets();
        return _assetRegistry.Values.ToList();
    }

    public Task<bool> ExistsAsync(string id)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(id);
        return Task.FromResult(_assetRegistry.ContainsKey(id));
    }

    public async Task AddAsync(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);

        _assetRegistry[asset.Id] = asset;

        // Save metadata
        var metadataPath = Path.Combine(_basePath, $"{asset.Id}.meta");
        var json = _serializer.Serialize(asset);
        await File.WriteAllTextAsync(metadataPath, json);
    }

    public async Task UpdateAsync(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);
        await AddAsync(asset); // Same operation for file system
    }

    public async Task DeleteAsync(string id)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(id);

        _assetRegistry.Remove(id);

        var metadataPath = Path.Combine(_basePath, $"{id}.meta");
        if (File.Exists(metadataPath))
        {
            File.Delete(metadataPath);
        }
    }

    public Task<int> CountAsync()
    {
        return Task.FromResult(_assetRegistry.Count);
    }

    private void InitializeRegistry()
    {
        // Scan existing metadata files
        var metadataFiles = Directory.GetFiles(_basePath, "*.meta");

        foreach (var file in metadataFiles)
        {
            try
            {
                var json = File.ReadAllText(file);
                var asset = _serializer.Deserialize(json);
                if (asset != null)
                {
                    _assetRegistry[asset.Id] = asset;
                }
            }
            catch (Exception ex)
            {
                // Log error but continue loading other assets
                Console.WriteLine($"Error loading asset metadata from {file}: {ex.Message}");
            }
        }
    }

    private async Task ScanForNewAssets()
    {
        // Implementation would scan for new asset files and create metadata
        // This is a simplified version
        await Task.CompletedTask;
    }
}

// Asset serializer for persistence
public interface IAssetSerializer<T> where T : class, IAsset
{
    string Serialize(T asset);
    T? Deserialize(string data);
}

public class JsonAssetSerializer<T> : IAssetSerializer<T> where T : class, IAsset
{
    public string Serialize(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);

        // Create a simplified DTO for serialization
        var dto = new AssetDto
        {
            Id = asset.Id,
            Name = asset.Name,
            FilePath = asset.FilePath,
            Type = asset.Type.ToString(),
            FileSize = asset.FileSize,
            CreatedAt = asset.CreatedAt,
            ModifiedAt = asset.ModifiedAt,
            Dependencies = asset.Dependencies.ToList(),
            Metadata = asset.Metadata.ToDictionary(kvp => kvp.Key, kvp => kvp.Value?.ToString() ?? "")
        };

        return System.Text.Json.JsonSerializer.Serialize(dto, new JsonSerializerOptions { WriteIndented = true });
    }

    public T? Deserialize(string data)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(data);

        try
        {
            var dto = System.Text.Json.JsonSerializer.Deserialize<AssetDto>(data);
            if (dto == null) return null;

            // Factory method to create specific asset types
            return CreateAssetFromDto(dto);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error deserializing asset: {ex.Message}");
            return null;
        }
    }

    private T? CreateAssetFromDto(AssetDto dto)
    {
        if (!Enum.TryParse<AssetType>(dto.Type, out var assetType))
        {
            return null;
        }

        IAsset? asset = assetType switch
        {
            AssetType.Texture => new Texture(dto.Id, dto.Name, dto.FilePath),
            AssetType.Model => new Model(dto.Id, dto.Name, dto.FilePath),
            AssetType.Audio => new AudioClip(dto.Id, dto.Name, dto.FilePath),
            _ => null
        };

        if (asset is T typedAsset)
        {
            // Restore dependencies
            foreach (var dependency in dto.Dependencies)
            {
                asset.AddDependency(dependency);
            }

            // Restore metadata
            foreach (var kvp in dto.Metadata)
            {
                asset.Metadata[kvp.Key] = kvp.Value;
            }

            return typedAsset;
        }

        return null;
    }

    private class AssetDto
    {
        public string Id { get; set; } = "";
        public string Name { get; set; } = "";
        public string FilePath { get; set; } = "";
        public string Type { get; set; } = "";
        public long FileSize { get; set; }
        public DateTime CreatedAt { get; set; }
        public DateTime ModifiedAt { get; set; }
        public List<string> Dependencies { get; set; } = new();
        public Dictionary<string, string> Metadata { get; set; } = new();
    }
}
```

### Step 6: Unit of Work Pattern (40 minutes)

**Coordinated Repository Operations**

```csharp
namespace AssetManager.Core.UnitOfWork;

// Unit of Work pattern interface
public interface IAssetUnitOfWork : IDisposable
{
    IAssetRepository<T> GetRepository<T>() where T : class, IAsset;
    Task<int> CommitAsync();
    Task RollbackAsync();
    void RegisterNew<T>(T asset) where T : class, IAsset;
    void RegisterDirty<T>(T asset) where T : class, IAsset;
    void RegisterDeleted<T>(T asset) where T : class, IAsset;
    bool HasChanges { get; }
}

public class AssetUnitOfWork : IAssetUnitOfWork
{
    private readonly Dictionary<Type, object> _repositories = new();
    private readonly Dictionary<Type, IAssetDataSource<IAsset>> _dataSources = new();
    private readonly List<IAsset> _newAssets = new();
    private readonly List<IAsset> _dirtyAssets = new();
    private readonly List<IAsset> _deletedAssets = new();
    private readonly IAssetCache<IAsset> _cache;
    private bool _disposed;

    public AssetUnitOfWork(IDictionary<Type, IAssetDataSource<IAsset>> dataSources, IAssetCache<IAsset> cache)
    {
        _dataSources = new Dictionary<Type, IAssetDataSource<IAsset>>(dataSources);
        _cache = cache ?? throw new ArgumentNullException(nameof(cache));
    }

    public bool HasChanges => _newAssets.Count > 0 || _dirtyAssets.Count > 0 || _deletedAssets.Count > 0;

    public IAssetRepository<T> GetRepository<T>() where T : class, IAsset
    {
        var type = typeof(T);

        if (_repositories.TryGetValue(type, out var repository))
        {
            return (IAssetRepository<T>)repository;
        }

        // Create repository if it doesn't exist
        if (!_dataSources.TryGetValue(type, out var dataSource))
        {
            throw new InvalidOperationException($"No data source registered for type {type.Name}");
        }

        var typedDataSource = (IAssetDataSource<T>)dataSource;
        var typedCache = (IAssetCache<T>)_cache;
        var loader = CreateLoader<T>();

        var newRepository = new UnitOfWorkRepository<T>(typedDataSource, typedCache, loader, this);
        _repositories[type] = newRepository;

        return newRepository;
    }

    public void RegisterNew<T>(T asset) where T : class, IAsset
    {
        ArgumentNullException.ThrowIfNull(asset);

        if (!_newAssets.Contains(asset))
        {
            _newAssets.Add(asset);
        }
    }

    public void RegisterDirty<T>(T asset) where T : class, IAsset
    {
        ArgumentNullException.ThrowIfNull(asset);

        if (!_dirtyAssets.Contains(asset) && !_newAssets.Contains(asset))
        {
            _dirtyAssets.Add(asset);
        }
    }

    public void RegisterDeleted<T>(T asset) where T : class, IAsset
    {
        ArgumentNullException.ThrowIfNull(asset);

        if (!_deletedAssets.Contains(asset))
        {
            _deletedAssets.Add(asset);
            _newAssets.Remove(asset);
            _dirtyAssets.Remove(asset);
        }
    }

    public async Task<int> CommitAsync()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(AssetUnitOfWork));

        int changeCount = 0;

        try
        {
            // Process deletions first
            foreach (var asset in _deletedAssets)
            {
                var dataSource = GetDataSourceForAsset(asset);
                await dataSource.DeleteAsync(asset.Id);
                await _cache.RemoveAsync(asset.Id);
                changeCount++;
            }

            // Process new assets
            foreach (var asset in _newAssets)
            {
                var dataSource = GetDataSourceForAsset(asset);
                await dataSource.AddAsync(asset);
                await _cache.SetAsync(asset.Id, asset);
                changeCount++;
            }

            // Process dirty assets
            foreach (var asset in _dirtyAssets)
            {
                var dataSource = GetDataSourceForAsset(asset);
                await dataSource.UpdateAsync(asset);
                await _cache.RemoveAsync(asset.Id); // Invalidate cache
                changeCount++;
            }

            // Clear change tracking
            _newAssets.Clear();
            _dirtyAssets.Clear();
            _deletedAssets.Clear();

            return changeCount;
        }
        catch (Exception)
        {
            // In a real implementation, you might want to implement rollback logic
            await RollbackAsync();
            throw;
        }
    }

    public async Task RollbackAsync()
    {
        // Clear all pending changes
        _newAssets.Clear();
        _dirtyAssets.Clear();
        _deletedAssets.Clear();

        // In a more sophisticated implementation, you might restore previous states
        await Task.CompletedTask;
    }

    private IAssetDataSource<IAsset> GetDataSourceForAsset(IAsset asset)
    {
        var assetType = asset.GetType();

        // Try exact type first
        if (_dataSources.TryGetValue(assetType, out var dataSource))
        {
            return dataSource;
        }

        // Try base interfaces
        var baseType = assetType.BaseType;
        while (baseType != null && baseType != typeof(object))
        {
            if (_dataSources.TryGetValue(baseType, out dataSource))
            {
                return dataSource;
            }
            baseType = baseType.BaseType;
        }

        throw new InvalidOperationException($"No data source found for asset type {assetType.Name}");
    }

    private IAssetLoader<T> CreateLoader<T>() where T : class, IAsset
    {
        // Factory method to create appropriate loader for asset type
        return new DefaultAssetLoader<T>();
    }

    public void Dispose()
    {
        if (!_disposed)
        {
            _repositories.Clear();
            _newAssets.Clear();
            _dirtyAssets.Clear();
            _deletedAssets.Clear();
            _disposed = true;
        }
    }
}

// Unit of Work aware repository
internal class UnitOfWorkRepository<T> : IAssetRepository<T> where T : class, IAsset
{
    private readonly IAssetDataSource<T> _dataSource;
    private readonly IAssetCache<T> _cache;
    private readonly IAssetLoader<T> _loader;
    private readonly AssetUnitOfWork _unitOfWork;

    public UnitOfWorkRepository(IAssetDataSource<T> dataSource, IAssetCache<T> cache,
        IAssetLoader<T> loader, AssetUnitOfWork unitOfWork)
    {
        _dataSource = dataSource;
        _cache = cache;
        _loader = loader;
        _unitOfWork = unitOfWork;
    }

    public async Task<T?> GetByIdAsync(string id)
    {
        // Same as regular repository but within unit of work context
        var cachedAsset = await _cache.GetAsync(id);
        if (cachedAsset != null)
        {
            return cachedAsset;
        }

        var asset = await _dataSource.GetByIdAsync(id);
        if (asset != null)
        {
            if (asset.Status == AssetStatus.Unloaded)
            {
                await _loader.LoadAsync(asset);
            }
            await _cache.SetAsync(id, asset);
        }

        return asset;
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        var assets = await _dataSource.GetAllAsync();
        var loadedAssets = new List<T>();

        foreach (var asset in assets)
        {
            var cachedAsset = await _cache.GetAsync(asset.Id);
            if (cachedAsset != null)
            {
                loadedAssets.Add(cachedAsset);
            }
            else
            {
                if (asset.Status == AssetStatus.Unloaded)
                {
                    await _loader.LoadAsync(asset);
                }
                await _cache.SetAsync(asset.Id, asset);
                loadedAssets.Add(asset);
            }
        }

        return loadedAssets;
    }

    public async Task<IEnumerable<T>> FindAsync(ISpecification<T> specification)
    {
        var allAssets = await GetAllAsync();
        return allAssets.Where(specification.IsSatisfiedBy);
    }

    public async Task<bool> ExistsAsync(string id)
    {
        return await _dataSource.ExistsAsync(id);
    }

    public Task AddAsync(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);
        _unitOfWork.RegisterNew(asset);
        return Task.CompletedTask;
    }

    public Task UpdateAsync(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);
        _unitOfWork.RegisterDirty(asset);
        return Task.CompletedTask;
    }

    public Task DeleteAsync(string id)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(id);

        // You would need to load the asset first to register it for deletion
        // This is a simplified implementation
        return Task.CompletedTask;
    }

    public async Task<int> CountAsync()
    {
        return await _dataSource.CountAsync();
    }

    public async Task<int> CountAsync(ISpecification<T> specification)
    {
        var assets = await _dataSource.GetAllAsync();
        return assets.Count(specification.IsSatisfiedBy);
    }
}

// Asset loader interface
public interface IAssetLoader<T> where T : class, IAsset
{
    Task LoadAsync(T asset);
    Task UnloadAsync(T asset);
    bool CanLoad(T asset);
}

// Default asset loader implementation
public class DefaultAssetLoader<T> : IAssetLoader<T> where T : class, IAsset
{
    public async Task LoadAsync(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);

        if (!CanLoad(asset))
        {
            asset.SetFailed();
            return;
        }

        try
        {
            // Simulate loading based on asset type
            await Task.Delay(100); // Simulate I/O operation

            if (File.Exists(asset.FilePath))
            {
                var fileInfo = new FileInfo(asset.FilePath);
                asset.GetType().GetProperty(nameof(Asset.FileSize))?.SetValue(asset, fileInfo.Length);

                // Type-specific loading
                await LoadAssetContent(asset);

                asset.SetLoaded();
            }
            else
            {
                asset.SetFailed();
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error loading asset {asset.Id}: {ex.Message}");
            asset.SetFailed();
        }
    }

    public Task UnloadAsync(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);

        // Clear loaded content
        ClearAssetContent(asset);

        return Task.CompletedTask;
    }

    public bool CanLoad(T asset)
    {
        ArgumentNullException.ThrowIfNull(asset);
        return File.Exists(asset.FilePath);
    }

    private async Task LoadAssetContent(T asset)
    {
        switch (asset)
        {
            case Texture texture:
                await LoadTextureContent(texture);
                break;
            case Model model:
                await LoadModelContent(model);
                break;
            case AudioClip audio:
                await LoadAudioContent(audio);
                break;
        }
    }

    private async Task LoadTextureContent(Texture texture)
    {
        // Simplified texture loading
        var imageData = await File.ReadAllBytesAsync(texture.FilePath);
        texture.ImageData = imageData;

        // In a real implementation, you would parse image headers to get dimensions
        texture.Width = 512; // Placeholder
        texture.Height = 512; // Placeholder
        texture.Format = TextureFormat.RGBA32;
    }

    private async Task LoadModelContent(Model model)
    {
        // Simplified model loading
        var modelData = await File.ReadAllBytesAsync(model.FilePath);
        model.ModelData = modelData;

        // In a real implementation, you would parse the model file
        model.VertexCount = 1000; // Placeholder
        model.TriangleCount = 500; // Placeholder
    }

    private async Task LoadAudioContent(AudioClip audio)
    {
        // Simplified audio loading
        var audioData = await File.ReadAllBytesAsync(audio.FilePath);
        audio.AudioData = audioData;

        // In a real implementation, you would parse audio headers
        audio.Duration = TimeSpan.FromSeconds(30); // Placeholder
        audio.SampleRate = 44100;
        audio.Channels = 2;
    }

    private void ClearAssetContent(T asset)
    {
        switch (asset)
        {
            case Texture texture:
                texture.ImageData = null;
                break;
            case Model model:
                model.ModelData = null;
                break;
            case AudioClip audio:
                audio.AudioData = null;
                break;
        }
    }
}
```

## Testing Your Understanding

Create a console app to test your asset management system:

```csharp
using AssetManager.Core.Models;
using AssetManager.Core.Repositories;
using AssetManager.Core.UnitOfWork;
using AssetManager.Core.Caching;

static async Task Main(string[] args)
{
    // Setup asset management system
    var cache = new LRUCache<Texture>(maxSize: 10);
    var serializer = new JsonAssetSerializer<Texture>();
    var dataSource = new FileSystemAssetDataSource<Texture>("./assets", serializer);
    var loader = new DefaultAssetLoader<Texture>();
    var repository = new AssetRepository<Texture>(dataSource, cache, loader);

    // Create and add some test textures
    var playerTexture = new Texture("player_idle", "Player Idle", "./assets/player_idle.png")
    {
        Width = 64,
        Height = 64,
        Format = TextureFormat.RGBA32
    };

    var enemyTexture = new Texture("enemy_walk", "Enemy Walk", "./assets/enemy_walk.png")
    {
        Width = 32,
        Height = 32,
        Format = TextureFormat.RGB24
    };

    // Test repository operations
    await repository.AddAsync(playerTexture);
    await repository.AddAsync(enemyTexture);

    Console.WriteLine($"Added {await repository.CountAsync()} textures");

    // Test retrieval
    var retrievedTexture = await repository.GetByIdAsync("player_idle");
    Console.WriteLine($"Retrieved: {retrievedTexture}");

    // Test Unit of Work
    var dataSources = new Dictionary<Type, IAssetDataSource<IAsset>>
    {
        { typeof(Texture), (IAssetDataSource<IAsset>)dataSource }
    };

    using var unitOfWork = new AssetUnitOfWork(dataSources, (IAssetCache<IAsset>)cache);

    var textureRepo = unitOfWork.GetRepository<Texture>();

    // Create new texture within unit of work
    var newTexture = new Texture("ui_button", "UI Button", "./assets/ui_button.png");
    await textureRepo.AddAsync(newTexture);

    Console.WriteLine($"Has changes: {unitOfWork.HasChanges}");

    // Commit all changes
    var changeCount = await unitOfWork.CommitAsync();
    Console.WriteLine($"Committed {changeCount} changes");
}
```

## Extension Challenges

1. **Advanced Dependency Resolution**: Implement circular dependency detection
2. **Asset Streaming**: Progressive loading for large assets
3. **Asset Bundles**: Package related assets together
4. **Version Management**: Asset versioning and migration
5. **Memory Management**: Automatic asset unloading based on memory pressure
6. **Asset Validation**: Content validation pipeline
7. **Performance Monitoring**: Asset loading performance analytics

## Resources

- 📚 **[Detailed Guide](asset-manager-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Repository Pattern Documentation](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)**
- 📘 **[Caching Documentation](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/memory)**

## Building on Previous Projects

This project completes Phase 3 by integrating all architectural patterns:

- **Projects 1-8**: All foundational skills in advanced architectural context
- **Project 9**: Dependency injection for asset system components
- **Project 10**: Factory patterns for asset creation
- **Project 11**: Observer patterns for asset events
- **New Skills**: Data access patterns and caching strategies

## Example Usage

```csharp
// Repository pattern for asset access
var assetRepository = new AssetRepository<Texture>(
    dataSource: new FileSystemDataSource("./assets/textures"),
    cache: new LRUCache<string, Texture>(maxSize: 100)
);

var texture = await assetRepository.GetByIdAsync("player_sprite.png");
var allTextures = await assetRepository.GetAllAsync();

// Unit of work for batch operations
using var unitOfWork = new AssetUnitOfWork();

var textureRepo = unitOfWork.GetRepository<Texture>();
var audioRepo = unitOfWork.GetRepository<AudioClip>();
var modelRepo = unitOfWork.GetRepository<Model>();

// Load related assets in transaction
await textureRepo.LoadAsync("character_diffuse.png");
await textureRepo.LoadAsync("character_normal.png");
await modelRepo.LoadAsync("character_model.fbx");

await unitOfWork.CommitAsync(); // All assets loaded together

// Asset manager with dependency resolution
var assetManager = new AssetManager()
    .RegisterLoader<Texture, TextureLoader>()
    .RegisterLoader<Model, ModelLoader>()
    .RegisterLoader<Material, MaterialLoader>()
    .ConfigureCache(cache => cache
        .SetMaxMemorySize(256_000_000) // 256MB
        .SetEvictionPolicy(EvictionPolicy.LRU)
        .SetTTL(TimeSpan.FromMinutes(30)));

// Load asset with dependencies
var characterMaterial = await assetManager.LoadAsync<Material>("character.mat");
// Automatically loads dependent textures and shaders

// Specification pattern for querying
var largeTextures = await assetRepository.FindAsync(
    new TextureSizeSpecification(minWidth: 1024, minHeight: 1024)
);

var recentAssets = await assetRepository.FindAsync(
    new AssetModifiedAfterSpecification(DateTime.Now.AddDays(-7))
);
```

## Pattern Integration Focus

This project demonstrates integration of multiple patterns:

- **Repository**: Clean data access abstraction
- **Unit of Work**: Transaction boundary management
- **Specification**: Flexible querying
- **Strategy**: Different caching and loading strategies
- **Observer**: Asset change notifications

## Advanced Features

- **Smart Preloading**: Predictive asset loading based on usage patterns
- **Streaming**: Loading large assets in chunks
- **Asset Bundles**: Packaging related assets together
- **Version Management**: Handling asset updates and compatibility
- **Memory Monitoring**: Tracking asset memory usage and optimization

## Contributing

When implementing this project:

1. Abstract data access behind repository interfaces
2. Implement proper transaction boundaries with unit of work
3. Design efficient caching strategies for different asset types
4. Create comprehensive integration tests
5. Document performance characteristics and trade-offs
