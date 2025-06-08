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
   Follow the implementation guide below for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-09-plugin-system/
├── README.md                    # This file - complete implementation guide
├── PluginSystem.sln             # Solution file
├── PluginSystem.Core/           # Main library implementation
├── PluginSystem.App/            # Console application
└── PluginSystem.Tests/          # Unit tests
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

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-09-plugin-system/` directory:

```bash
# Create the main solution file
dotnet new sln -n PluginSystem

# Create the main library project
dotnet new classlib -n PluginSystem.Core -f net8.0

# Create a sample plugin project
dotnet new classlib -n PluginSystem.SamplePlugin -f net8.0

# Create the host application
dotnet new console -n PluginSystem.Host -f net8.0

# Create the test project
dotnet new mstest -n PluginSystem.Tests -f net8.0

# Add projects to solution
dotnet sln add PluginSystem.Core/PluginSystem.Core.csproj
dotnet sln add PluginSystem.SamplePlugin/PluginSystem.SamplePlugin.csproj
dotnet sln add PluginSystem.Host/PluginSystem.Host.csproj
dotnet sln add PluginSystem.Tests/PluginSystem.Tests.csproj

# Add project references
dotnet add PluginSystem.SamplePlugin/PluginSystem.SamplePlugin.csproj reference PluginSystem.Core/PluginSystem.Core.csproj
dotnet add PluginSystem.Host/PluginSystem.Host.csproj reference PluginSystem.Core/PluginSystem.Core.csproj
dotnet add PluginSystem.Tests/PluginSystem.Tests.csproj reference PluginSystem.Core/PluginSystem.Core.csproj
```

### Step 2: Configure Project Files

Update the `PluginSystem.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>PluginSystem.Core</PackageId>
    <Title>Modular Plugin System</Title>
    <Description>A library for dependency injection and IoC container implementation</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging" Version="8.0.0" />
  </ItemGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your plugin system code:

```
PluginSystem.Core/
├── Interfaces/        # Plugin and service interfaces
├── Container/         # IoC container implementation
├── Loader/           # Plugin loading and discovery
├── Registry/         # Service registration and resolution
├── Lifetime/         # Service lifetime management
└── Plugins/          # Base plugin classes and metadata
```

Create these directories:

```bash
mkdir PluginSystem.Core/Interfaces
mkdir PluginSystem.Core/Container
mkdir PluginSystem.Core/Loader
mkdir PluginSystem.Core/Registry
mkdir PluginSystem.Core/Lifetime
mkdir PluginSystem.Core/Plugins
```

## Implementation Guide

### Step 4: Core Plugin Interfaces (25 minutes)

**Plugin Foundation and Metadata**

```csharp
// Core plugin interfaces
namespace PluginSystem.Core.Interfaces;

// Basic plugin interface
public interface IPlugin
{
    string Name { get; }
    string Version { get; }
    string Description { get; }
    Task InitializeAsync(IServiceProvider serviceProvider);
    Task ShutdownAsync();
}

// Plugin with configuration support
public interface IConfigurablePlugin : IPlugin
{
    void Configure(IConfiguration configuration);
    IDictionary<string, object> GetDefaultConfiguration();
}

// Plugin with dependency registration
public interface IServicePlugin : IPlugin
{
    void RegisterServices(IServiceCollection services);
    void ConfigureServices(IServiceProvider serviceProvider);
}

// Plugin lifecycle management
public interface IPluginLifecycle
{
    PluginState State { get; }
    Task StartAsync(CancellationToken cancellationToken = default);
    Task StopAsync(CancellationToken cancellationToken = default);
    Task RestartAsync(CancellationToken cancellationToken = default);
}

// Plugin metadata
[AttributeUsage(AttributeTargets.Class)]
public class PluginMetadataAttribute : Attribute
{
    public string Name { get; }
    public string Version { get; }
    public string Description { get; }
    public string[] Dependencies { get; }
    public bool IsOptional { get; }

    public PluginMetadataAttribute(
        string name,
        string version,
        string description,
        string[]? dependencies = null,
        bool isOptional = false)
    {
        Name = name;
        Version = version;
        Description = description;
        Dependencies = dependencies ?? Array.Empty<string>();
        IsOptional = isOptional;
    }
}

public enum PluginState
{
    Unloaded,
    Loading,
    Loaded,
    Initializing,
    Running,
    Stopping,
    Stopped,
    Error
}

// Service interfaces for common plugin types
public interface IGameServicePlugin : IServicePlugin
{
    int Priority { get; }
    bool IsRequired { get; }
}

public interface IDataPlugin : IPlugin
{
    Task<T> LoadDataAsync<T>(string key) where T : class;
    Task SaveDataAsync<T>(string key, T data) where T : class;
}

public interface IRendererPlugin : IPlugin
{
    void Render(IRenderContext context);
    bool SupportsFeature(string feature);
}

// Plugin context for communication
public interface IPluginContext
{
    IServiceProvider Services { get; }
    IConfiguration Configuration { get; }
    ILogger Logger { get; }
    CancellationToken ShutdownToken { get; }

    T GetPlugin<T>() where T : class, IPlugin;
    IEnumerable<T> GetPlugins<T>() where T : class, IPlugin;
    void PublishEvent<T>(T eventData) where T : class;
    void Subscribe<T>(Action<T> handler) where T : class;
}
```

### Step 5: IoC Container Implementation (35 minutes)

**Custom Dependency Injection Container**

```csharp
using Microsoft.Extensions.DependencyInjection;

namespace PluginSystem.Core.Container;

public class GameContainer : IServiceProvider, IDisposable
{
    private readonly IServiceCollection _services;
    private readonly Dictionary<Type, ServiceDescriptor> _serviceDescriptors;
    private readonly Dictionary<Type, object> _singletonInstances;
    private readonly Dictionary<Type, Func<IServiceProvider, object>> _factories;
    private readonly object _lock = new();
    private bool _disposed;

    public GameContainer()
    {
        _services = new ServiceCollection();
        _serviceDescriptors = new Dictionary<Type, ServiceDescriptor>();
        _singletonInstances = new Dictionary<Type, object>();
        _factories = new Dictionary<Type, Func<IServiceProvider, object>>();
    }

    // Registration methods with fluent interface
    public GameContainer RegisterSingleton<TInterface, TImplementation>()
        where TInterface : class
        where TImplementation : class, TInterface, new()
    {
        return RegisterSingleton<TInterface>(() => new TImplementation());
    }

    public GameContainer RegisterSingleton<TInterface>(Func<TInterface> factory)
        where TInterface : class
    {
        lock (_lock)
        {
            _factories[typeof(TInterface)] = provider => factory();
            var descriptor = ServiceDescriptor.Singleton<TInterface>(provider => (TInterface)factory());
            _services.Add(descriptor);
            _serviceDescriptors[typeof(TInterface)] = descriptor;
        }
        return this;
    }

    public GameContainer RegisterSingleton<TInterface>(TInterface instance)
        where TInterface : class
    {
        lock (_lock)
        {
            _singletonInstances[typeof(TInterface)] = instance;
            var descriptor = ServiceDescriptor.Singleton<TInterface>(instance);
            _services.Add(descriptor);
            _serviceDescriptors[typeof(TInterface)] = descriptor;
        }
        return this;
    }

    public GameContainer RegisterTransient<TInterface, TImplementation>()
        where TInterface : class
        where TImplementation : class, TInterface, new()
    {
        lock (_lock)
        {
            _factories[typeof(TInterface)] = provider => CreateInstance<TImplementation>(provider);
            var descriptor = ServiceDescriptor.Transient<TInterface, TImplementation>();
            _services.Add(descriptor);
            _serviceDescriptors[typeof(TInterface)] = descriptor;
        }
        return this;
    }

    public GameContainer RegisterTransient<TInterface>(Func<IServiceProvider, TInterface> factory)
        where TInterface : class
    {
        lock (_lock)
        {
            _factories[typeof(TInterface)] = provider => factory(provider);
            var descriptor = ServiceDescriptor.Transient<TInterface>(factory);
            _services.Add(descriptor);
            _serviceDescriptors[typeof(TInterface)] = descriptor;
        }
        return this;
    }

    public GameContainer RegisterScoped<TInterface, TImplementation>()
        where TInterface : class
        where TImplementation : class, TInterface, new()
    {
        lock (_lock)
        {
            var descriptor = ServiceDescriptor.Scoped<TInterface, TImplementation>();
            _services.Add(descriptor);
            _serviceDescriptors[typeof(TInterface)] = descriptor;
        }
        return this;
    }

    // Generic registration with conditional logic
    public GameContainer RegisterConditional<TInterface>(
        Func<IServiceProvider, bool> condition,
        Func<IServiceProvider, TInterface> factory)
        where TInterface : class
    {
        return RegisterTransient<TInterface>(provider =>
            condition(provider) ? factory(provider) : throw new InvalidOperationException($"Condition not met for {typeof(TInterface).Name}"));
    }

    // Decorator pattern registration
    public GameContainer RegisterDecorator<TInterface, TDecorator>()
        where TInterface : class
        where TDecorator : class, TInterface
    {
        if (!_serviceDescriptors.ContainsKey(typeof(TInterface)))
        {
            throw new InvalidOperationException($"Service {typeof(TInterface).Name} must be registered before decorating");
        }

        var originalDescriptor = _serviceDescriptors[typeof(TInterface)];

        // Replace with decorator
        _services.Remove(originalDescriptor);

        RegisterTransient<TInterface>(provider =>
        {
            // Create original service
            var originalService = CreateServiceFromDescriptor(originalDescriptor, provider);

            // Create decorator with original service
            return (TInterface)CreateInstance<TDecorator>(provider, originalService);
        });

        return this;
    }

    // Resolution methods
    public T Resolve<T>() where T : class
    {
        return (T)GetService(typeof(T)) ?? throw new InvalidOperationException($"Service {typeof(T).Name} not registered");
    }

    public T? TryResolve<T>() where T : class
    {
        return GetService(typeof(T)) as T;
    }

    public IEnumerable<T> ResolveAll<T>() where T : class
    {
        return GetServices(typeof(T)).Cast<T>();
    }

    // IServiceProvider implementation
    public object? GetService(Type serviceType)
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(GameContainer));

        // Check singleton cache first
        if (_singletonInstances.TryGetValue(serviceType, out var singletonInstance))
        {
            return singletonInstance;
        }

        // Check if we have a factory
        if (_factories.TryGetValue(serviceType, out var factory))
        {
            var instance = factory(this);

            // Cache if singleton
            if (_serviceDescriptors.TryGetValue(serviceType, out var descriptor) &&
                descriptor.Lifetime == ServiceLifetime.Singleton)
            {
                _singletonInstances[serviceType] = instance;
            }

            return instance;
        }

        // Fallback to built-in resolution
        using var scope = _services.BuildServiceProvider().CreateScope();
        return scope.ServiceProvider.GetService(serviceType);
    }

    public object GetRequiredService(Type serviceType)
    {
        return GetService(serviceType) ?? throw new InvalidOperationException($"Required service {serviceType.Name} not found");
    }

    public IEnumerable<object> GetServices(Type serviceType)
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(GameContainer));

        // For multiple registrations, use built-in container
        using var scope = _services.BuildServiceProvider().CreateScope();
        return scope.ServiceProvider.GetServices(serviceType);
    }

    // Advanced features
    public GameContainer RegisterFactory<T>(Func<IServiceProvider, T> factory) where T : class
    {
        return RegisterTransient<T>(factory);
    }

    public GameContainer RegisterLazy<T>() where T : class
    {
        return RegisterTransient<Lazy<T>>(provider => new Lazy<T>(() => provider.GetRequiredService<T>()));
    }

    public bool IsRegistered<T>() where T : class
    {
        return IsRegistered(typeof(T));
    }

    public bool IsRegistered(Type serviceType)
    {
        return _serviceDescriptors.ContainsKey(serviceType) ||
               _singletonInstances.ContainsKey(serviceType);
    }

    public void Unregister<T>() where T : class
    {
        Unregister(typeof(T));
    }

    public void Unregister(Type serviceType)
    {
        lock (_lock)
        {
            if (_serviceDescriptors.TryGetValue(serviceType, out var descriptor))
            {
                _services.Remove(descriptor);
                _serviceDescriptors.Remove(serviceType);
            }

            _factories.Remove(serviceType);

            if (_singletonInstances.TryRemove(serviceType, out var instance) &&
                instance is IDisposable disposable)
            {
                disposable.Dispose();
            }
        }
    }

    // Helper methods
    private T CreateInstance<T>(IServiceProvider provider, params object[] additionalArgs) where T : new()
    {
        var constructors = typeof(T).GetConstructors();
        var constructor = constructors.OrderByDescending(c => c.GetParameters().Length).First();
        var parameters = constructor.GetParameters();
        var args = new object[parameters.Length];

        var additionalIndex = 0;
        for (int i = 0; i < parameters.Length; i++)
        {
            var paramType = parameters[i].ParameterType;

            // Try additional args first
            if (additionalIndex < additionalArgs.Length &&
                paramType.IsAssignableFrom(additionalArgs[additionalIndex].GetType()))
            {
                args[i] = additionalArgs[additionalIndex++];
            }
            else
            {
                // Resolve from container
                args[i] = provider.GetRequiredService(paramType);
            }
        }

        return (T)Activator.CreateInstance(typeof(T), args)!;
    }

    private object CreateServiceFromDescriptor(ServiceDescriptor descriptor, IServiceProvider provider)
    {
        if (descriptor.ImplementationInstance != null)
            return descriptor.ImplementationInstance;

        if (descriptor.ImplementationFactory != null)
            return descriptor.ImplementationFactory(provider);

        if (descriptor.ImplementationType != null)
            return ActivatorUtilities.CreateInstance(provider, descriptor.ImplementationType);

        throw new InvalidOperationException("Invalid service descriptor");
    }

    public void Dispose()
    {
        if (_disposed) return;

        // Dispose all singleton instances
        foreach (var instance in _singletonInstances.Values)
        {
            if (instance is IDisposable disposable)
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

        _singletonInstances.Clear();
        _serviceDescriptors.Clear();
        _factories.Clear();
        _services.Clear();

        _disposed = true;
    }
}
```

### Step 6: Plugin Loader and Discovery (30 minutes)

**Dynamic Assembly Loading and Plugin Discovery**

```csharp
using System.Reflection;
using System.Runtime.Loader;

namespace PluginSystem.Core.Loader;

public class PluginLoader : IDisposable
{
    private readonly Dictionary<string, AssemblyLoadContext> _loadContexts;
    private readonly Dictionary<string, Assembly> _loadedAssemblies;
    private readonly Dictionary<string, IPlugin> _loadedPlugins;
    private readonly ILogger<PluginLoader> _logger;
    private readonly PluginLoadContext _defaultContext;
    private bool _disposed;

    public PluginLoader(ILogger<PluginLoader> logger)
    {
        _logger = logger;
        _loadContexts = new Dictionary<string, AssemblyLoadContext>();
        _loadedAssemblies = new Dictionary<string, Assembly>();
        _loadedPlugins = new Dictionary<string, IPlugin>();
        _defaultContext = new PluginLoadContext("DefaultPluginContext");
    }

    public async Task<IEnumerable<IPlugin>> DiscoverPluginsAsync(string pluginDirectory)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(pluginDirectory);

        if (!Directory.Exists(pluginDirectory))
        {
            _logger.LogWarning("Plugin directory {Directory} does not exist", pluginDirectory);
            return Enumerable.Empty<IPlugin>();
        }

        var plugins = new List<IPlugin>();
        var pluginFiles = Directory.GetFiles(pluginDirectory, "*.dll", SearchOption.AllDirectories);

        foreach (var pluginFile in pluginFiles)
        {
            try
            {
                var discoveredPlugins = await LoadPluginFromFileAsync(pluginFile);
                plugins.AddRange(discoveredPlugins);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Failed to load plugin from {File}", pluginFile);
            }
        }

        return plugins;
    }

    public async Task<IEnumerable<IPlugin>> LoadPluginFromFileAsync(string assemblyPath)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(assemblyPath);

        if (!File.Exists(assemblyPath))
        {
            throw new FileNotFoundException($"Plugin assembly not found: {assemblyPath}");
        }

        try
        {
            var assemblyName = Path.GetFileNameWithoutExtension(assemblyPath);

            // Create isolated load context for this plugin
            var loadContext = new PluginLoadContext(assemblyName);
            _loadContexts[assemblyName] = loadContext;

            // Load the assembly
            var assembly = loadContext.LoadFromAssemblyPath(assemblyPath);
            _loadedAssemblies[assemblyName] = assembly;

            _logger.LogInformation("Loaded assembly {Assembly} from {Path}", assembly.FullName, assemblyPath);

            // Discover plugin types
            var plugins = await DiscoverPluginTypesAsync(assembly);

            foreach (var plugin in plugins)
            {
                _loadedPlugins[plugin.Name] = plugin;
            }

            return plugins;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to load plugin assembly from {Path}", assemblyPath);
            throw;
        }
    }

    private async Task<IEnumerable<IPlugin>> DiscoverPluginTypesAsync(Assembly assembly)
    {
        var plugins = new List<IPlugin>();

        try
        {
            var pluginTypes = assembly.GetTypes()
                .Where(type => typeof(IPlugin).IsAssignableFrom(type) &&
                              !type.IsInterface &&
                              !type.IsAbstract)
                .ToList();

            foreach (var pluginType in pluginTypes)
            {
                try
                {
                    var plugin = await CreatePluginInstanceAsync(pluginType);
                    if (plugin != null)
                    {
                        plugins.Add(plugin);
                        _logger.LogInformation("Discovered plugin: {PluginName} v{Version}",
                            plugin.Name, plugin.Version);
                    }
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Failed to create instance of plugin type {Type}", pluginType.Name);
                }
            }
        }
        catch (ReflectionTypeLoadException ex)
        {
            _logger.LogError(ex, "Failed to load types from assembly {Assembly}", assembly.FullName);

            // Log loader exceptions for better debugging
            foreach (var loaderEx in ex.LoaderExceptions.Where(e => e != null))
            {
                _logger.LogError(loaderEx, "Loader exception");
            }
        }

        return plugins;
    }

    private async Task<IPlugin?> CreatePluginInstanceAsync(Type pluginType)
    {
        try
        {
            // Validate plugin type
            if (!typeof(IPlugin).IsAssignableFrom(pluginType))
            {
                _logger.LogWarning("Type {Type} does not implement IPlugin", pluginType.Name);
                return null;
            }

            // Check for metadata attribute
            var metadata = pluginType.GetCustomAttribute<PluginMetadataAttribute>();
            if (metadata == null)
            {
                _logger.LogWarning("Plugin type {Type} is missing PluginMetadataAttribute", pluginType.Name);
            }

            // Create instance
            var instance = Activator.CreateInstance(pluginType);
            if (instance is IPlugin plugin)
            {
                // Validate dependencies if metadata exists
                if (metadata != null && !await ValidatePluginDependenciesAsync(metadata))
                {
                    _logger.LogWarning("Plugin {Name} dependencies not satisfied", metadata.Name);
                    return metadata.IsOptional ? plugin : null;
                }

                return plugin;
            }

            _logger.LogError("Failed to cast instance to IPlugin for type {Type}", pluginType.Name);
            return null;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Exception creating plugin instance for type {Type}", pluginType.Name);
            return null;
        }
    }

    private async Task<bool> ValidatePluginDependenciesAsync(PluginMetadataAttribute metadata)
    {
        foreach (var dependency in metadata.Dependencies)
        {
            if (!_loadedPlugins.ContainsKey(dependency))
            {
                _logger.LogWarning("Plugin dependency {Dependency} not found for {Plugin}",
                    dependency, metadata.Name);
                return false;
            }
        }

        return true;
    }

    public async Task<bool> UnloadPluginAsync(string pluginName)
    {
        if (!_loadedPlugins.TryGetValue(pluginName, out var plugin))
        {
            _logger.LogWarning("Plugin {PluginName} not found for unloading", pluginName);
            return false;
        }

        try
        {
            // Shutdown plugin
            await plugin.ShutdownAsync();

            // Remove from loaded plugins
            _loadedPlugins.Remove(pluginName);

            // Find and unload assembly context
            var assemblyName = plugin.GetType().Assembly.GetName().Name;
            if (assemblyName != null && _loadContexts.TryGetValue(assemblyName, out var context))
            {
                context.Unload();
                _loadContexts.Remove(assemblyName);
                _loadedAssemblies.Remove(assemblyName);
            }

            _logger.LogInformation("Successfully unloaded plugin {PluginName}", pluginName);
            return true;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to unload plugin {PluginName}", pluginName);
            return false;
        }
    }

    public IPlugin? GetPlugin(string name)
    {
        _loadedPlugins.TryGetValue(name, out var plugin);
        return plugin;
    }

    public T? GetPlugin<T>() where T : class, IPlugin
    {
        return _loadedPlugins.Values.OfType<T>().FirstOrDefault();
    }

    public IEnumerable<T> GetPlugins<T>() where T : class, IPlugin
    {
        return _loadedPlugins.Values.OfType<T>();
    }

    public IEnumerable<IPlugin> GetAllPlugins()
    {
        return _loadedPlugins.Values.ToList();
    }

    public PluginInfo GetPluginInfo(string name)
    {
        if (!_loadedPlugins.TryGetValue(name, out var plugin))
        {
            throw new ArgumentException($"Plugin {name} not found", nameof(name));
        }

        var type = plugin.GetType();
        var metadata = type.GetCustomAttribute<PluginMetadataAttribute>();

        return new PluginInfo
        {
            Name = plugin.Name,
            Version = plugin.Version,
            Description = plugin.Description,
            Type = type,
            Assembly = type.Assembly,
            Metadata = metadata,
            State = plugin is IPluginLifecycle lifecycle ? lifecycle.State : PluginState.Running
        };
    }

    public void Dispose()
    {
        if (_disposed) return;

        // Shutdown all plugins
        var shutdownTasks = _loadedPlugins.Values.Select(async plugin =>
        {
            try
            {
                await plugin.ShutdownAsync();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error shutting down plugin {PluginName}", plugin.Name);
            }
        });

        Task.WhenAll(shutdownTasks).GetAwaiter().GetResult();

        // Unload all contexts
        foreach (var context in _loadContexts.Values)
        {
            try
            {
                context.Unload();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error unloading plugin context");
            }
        }

        _defaultContext?.Unload();

        _loadedPlugins.Clear();
        _loadedAssemblies.Clear();
        _loadContexts.Clear();

        _disposed = true;
    }
}

// Custom assembly load context for plugin isolation
public class PluginLoadContext : AssemblyLoadContext
{
    private readonly AssemblyDependencyResolver _resolver;

    public PluginLoadContext(string name) : base(name, isCollectible: true)
    {
        _resolver = new AssemblyDependencyResolver(AppDomain.CurrentDomain.BaseDirectory);
    }

    protected override Assembly? Load(AssemblyName assemblyName)
    {
        var assemblyPath = _resolver.ResolveAssemblyToPath(assemblyName);
        return assemblyPath != null ? LoadFromAssemblyPath(assemblyPath) : null;
    }

    protected override IntPtr LoadUnmanagedDll(string unmanagedDllName)
    {
        var libraryPath = _resolver.ResolveUnmanagedDllToPath(unmanagedDllName);
        return libraryPath != null ? LoadUnmanagedDllFromPath(libraryPath) : IntPtr.Zero;
    }
}

public class PluginInfo
{
    public string Name { get; set; } = string.Empty;
    public string Version { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public Type Type { get; set; } = null!;
    public Assembly Assembly { get; set; } = null!;
    public PluginMetadataAttribute? Metadata { get; set; }
    public PluginState State { get; set; }
}
```

## Key Dependency Injection Patterns

### 1. Service Registration Patterns

```csharp
// ✅ Fluent registration interface
var container = new GameContainer()
    .RegisterSingleton<ILogger, ConsoleLogger>()
    .RegisterTransient<IGameService, GameService>()
    .RegisterScoped<IPlayerService, PlayerService>();

// ✅ Conditional registration
container.RegisterConditional<IRenderer>(
    provider => provider.GetService<IConfiguration>()?.GetValue<bool>("UseAdvancedRenderer") == true,
    provider => new AdvancedRenderer()
);

// ✅ Decorator pattern
container.RegisterSingleton<IGameService, GameService>()
         .RegisterDecorator<IGameService, LoggingGameServiceDecorator>();
```

### 2. Plugin Integration

```csharp
// ✅ Plugin-based service registration
public class WeaponPlugin : IServicePlugin
{
    public void RegisterServices(IServiceCollection services)
    {
        services.AddTransient<IWeapon, Sword>();
        services.AddTransient<IWeapon, Bow>();
        services.AddSingleton<IWeaponFactory, WeaponFactory>();
    }
}
```

### 3. Lifetime Management

```csharp
// ✅ Proper disposal chain
public class GameEngine : IDisposable
{
    private readonly GameContainer _container;

    public GameEngine()
    {
        _container = new GameContainer()
            .RegisterSingleton<IGameEngine>(this)
            .RegisterTransient<IGameService, GameService>();
    }

    public void Dispose() => _container.Dispose(); // Cascades to all services
}
```

## Testing Your Understanding

Create a complete plugin system demonstration:

```csharp
using PluginSystem.Core.Container;
using PluginSystem.Core.Loader;
using PluginSystem.Core.Interfaces;

// Example plugin
[PluginMetadata("WeaponPlugin", "1.0.0", "Provides weapon functionality")]
public class WeaponPlugin : IServicePlugin, IPluginLifecycle
{
    public string Name => "WeaponPlugin";
    public string Version => "1.0.0";
    public string Description => "Provides weapon functionality";
    public PluginState State { get; private set; } = PluginState.Unloaded;

    public void RegisterServices(IServiceCollection services)
    {
        services.AddTransient<IWeapon, Sword>();
        services.AddTransient<IWeapon, Bow>();
        services.AddSingleton<IWeaponFactory, WeaponFactory>();
    }

    public async Task InitializeAsync(IServiceProvider serviceProvider)
    {
        State = PluginState.Initializing;

        // Initialize weapon data, load configs, etc.
        await Task.Delay(100); // Simulate initialization work

        State = PluginState.Running;
        Console.WriteLine($"✅ {Name} initialized successfully");
    }

    public async Task StartAsync(CancellationToken cancellationToken = default)
    {
        State = PluginState.Running;
        await Task.CompletedTask;
    }

    public async Task StopAsync(CancellationToken cancellationToken = default)
    {
        State = PluginState.Stopping;
        await Task.Delay(50, cancellationToken); // Cleanup work
        State = PluginState.Stopped;
    }

    public async Task RestartAsync(CancellationToken cancellationToken = default)
    {
        await StopAsync(cancellationToken);
        await StartAsync(cancellationToken);
    }

    public async Task ShutdownAsync()
    {
        await StopAsync();
        State = PluginState.Unloaded;
        Console.WriteLine($"🔄 {Name} shutdown complete");
    }

    public void ConfigureServices(IServiceProvider serviceProvider)
    {
        // Additional configuration after all services are registered
    }
}

// Plugin usage demonstration
public static async Task Main(string[] args)
{
    // Create container and plugin loader
    var container = new GameContainer();
    var logger = new ConsoleLogger<PluginLoader>();
    var pluginLoader = new PluginLoader(logger);

    try
    {
        Console.WriteLine("=== Plugin System Demo ===");

        // Register core services
        container.RegisterSingleton<ILogger>(logger)
                .RegisterSingleton<IPluginLoader>(pluginLoader);

        // Discover and load plugins
        var plugins = await pluginLoader.DiscoverPluginsAsync("./plugins");

        Console.WriteLine($"Discovered {plugins.Count()} plugins:");
        foreach (var plugin in plugins)
        {
            Console.WriteLine($"  • {plugin.Name} v{plugin.Version} - {plugin.Description}");
        }

        // Register plugin services
        var serviceCollection = new ServiceCollection();
        foreach (var plugin in plugins.OfType<IServicePlugin>())
        {
            plugin.RegisterServices(serviceCollection);
        }

        // Add services to our container
        foreach (var service in serviceCollection)
        {
            // This is simplified - in real implementation you'd properly transfer the services
            Console.WriteLine($"Registered service: {service.ServiceType.Name}");
        }

        // Initialize all plugins
        foreach (var plugin in plugins)
        {
            await plugin.InitializeAsync(container);
        }

        // Use plugin services
        var weapons = container.ResolveAll<IWeapon>();
        Console.WriteLine($"\nAvailable weapons from plugins:");
        foreach (var weapon in weapons)
        {
            Console.WriteLine($"  • {weapon.GetType().Name}");
        }

        // Demonstrate plugin lifecycle
        var weaponPlugin = pluginLoader.GetPlugin<WeaponPlugin>();
        if (weaponPlugin != null)
        {
            Console.WriteLine($"\nWeapon plugin state: {weaponPlugin.State}");

            await weaponPlugin.StopAsync();
            Console.WriteLine($"After stop: {weaponPlugin.State}");

            await weaponPlugin.StartAsync();
            Console.WriteLine($"After restart: {weaponPlugin.State}");
        }

        // Plugin information
        foreach (var plugin in plugins)
        {
            var info = pluginLoader.GetPluginInfo(plugin.Name);
            Console.WriteLine($"\nPlugin Info - {info.Name}:");
            Console.WriteLine($"  Assembly: {info.Assembly.GetName().Name}");
            Console.WriteLine($"  State: {info.State}");
            Console.WriteLine($"  Metadata: {info.Metadata?.Description ?? "None"}");
        }

        Console.WriteLine("\nPress any key to shutdown...");
        Console.ReadKey();
    }
    finally
    {
        // Cleanup
        pluginLoader.Dispose();
        container.Dispose();
    }
}

// Supporting interfaces and classes
public interface IWeapon
{
    string Name { get; }
    int Damage { get; }
    void Attack();
}

public class Sword : IWeapon
{
    public string Name => "Sword";
    public int Damage => 10;
    public void Attack() => Console.WriteLine("⚔️ Sword slash!");
}

public class Bow : IWeapon
{
    public string Name => "Bow";
    public int Damage => 8;
    public void Attack() => Console.WriteLine("🏹 Arrow shot!");
}

public interface IWeaponFactory
{
    IWeapon CreateWeapon(string type);
}

public class WeaponFactory : IWeaponFactory
{
    private readonly IServiceProvider _serviceProvider;

    public WeaponFactory(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public IWeapon CreateWeapon(string type)
    {
        return type.ToLower() switch
        {
            "sword" => _serviceProvider.GetRequiredService<Sword>(),
            "bow" => _serviceProvider.GetRequiredService<Bow>(),
            _ => throw new ArgumentException($"Unknown weapon type: {type}")
        };
    }
}
```

## Extension Challenges

1. **Hot Plugin Reloading**: Implement live plugin updates without restarting the application
2. **Plugin Dependencies**: Build a dependency resolution system for inter-plugin dependencies
3. **Plugin Security**: Add sandboxing and permission systems for untrusted plugins
4. **Plugin Marketplace**: Create a plugin discovery and installation system
5. **Configuration Management**: Build a hierarchical configuration system for plugins
6. **Plugin Communication**: Implement event bus for inter-plugin communication
7. **Plugin Analytics**: Add metrics and monitoring for plugin performance

## Common Pitfalls & Solutions

1. **Memory Leaks with Plugin Unloading**:

   ```csharp
   // ❌ Not properly disposing plugin resources
   public async Task UnloadPlugin(IPlugin plugin)
   {
       await plugin.ShutdownAsync(); // Not enough!
   }

   // ✅ Proper cleanup
   public async Task UnloadPlugin(IPlugin plugin)
   {
       await plugin.ShutdownAsync();

       if (plugin is IDisposable disposable)
           disposable.Dispose();

       // Unload assembly context if isolated
       if (_loadContexts.TryGetValue(plugin.Name, out var context))
       {
           context.Unload();
           _loadContexts.Remove(plugin.Name);
       }
   }
   ```

2. **Circular Dependencies**:

   ```csharp
   // ❌ Circular dependency
   container.RegisterTransient<IServiceA, ServiceA>(); // depends on IServiceB
   container.RegisterTransient<IServiceB, ServiceB>(); // depends on IServiceA

   // ✅ Break cycle with factory or event pattern
   container.RegisterTransient<IServiceA, ServiceA>();
   container.RegisterTransient<IServiceB, ServiceB>();
   container.RegisterSingleton<IEventBus, EventBus>(); // Mediator pattern
   ```

3. **Plugin Version Conflicts**:

   ```csharp
   // ✅ Version compatibility checking
   public async Task<bool> ValidatePluginCompatibility(IPlugin plugin)
   {
       var metadata = plugin.GetType().GetCustomAttribute<PluginMetadataAttribute>();
       if (metadata == null) return false;

       // Check version compatibility
       var version = Version.Parse(metadata.Version);
       var minVersion = new Version(1, 0, 0);
       var maxVersion = new Version(2, 0, 0);

       return version >= minVersion && version < maxVersion;
   }
   ```

4. **Assembly Loading Issues**:
   ```csharp
   // ✅ Proper error handling for assembly loading
   try
   {
       var assembly = loadContext.LoadFromAssemblyPath(pluginPath);
       return assembly;
   }
   catch (FileLoadException ex)
   {
       _logger.LogError(ex, "Failed to load assembly {Path}: {Message}", pluginPath, ex.Message);
       throw;
   }
   catch (BadImageFormatException ex)
   {
       _logger.LogError(ex, "Invalid assembly format {Path}", pluginPath);
       throw;
   }
   ```

## Resources

- 📚 **This README** - Complete implementation instructions
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
