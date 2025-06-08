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
