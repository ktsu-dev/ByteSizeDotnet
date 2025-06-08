# Project 13: Configuration System

**Focus**: Reflection & Attributes  
**Time**: ~2 hours  
**Deliverable**: `ConfigurationSystem` library with reflection-based configuration

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Reflection Guidelines](../microsoft-design-guidelines.md#reflection-guidelines)** - Safe and efficient reflection usage
- **[Attribute Guidelines](../microsoft-design-guidelines.md#attribute-design)** - Custom attribute design
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - Reflection performance optimization

## Learning Objectives

- Master reflection and runtime type inspection
- Understand custom attribute design and usage
- Learn configuration binding and validation
- Practice metadata-driven programming
- Build flexible and extensible configuration systems

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Reflection Guidelines](../microsoft-design-guidelines.md#reflection-guidelines) before starting

2. **Start Coding** 💻  
   Follow the implementation guide below for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-13-configuration-system/
├── README.md                       # This file
├── configuration-system-project-guide.md # Detailed implementation guide
├── src/                            # Your implementation
├── tests/                          # Unit tests
└── docs/                           # Additional documentation
```

## What You'll Build

A sophisticated configuration system featuring:

- **Reflection-Based Binding** - Automatic mapping from configuration sources to strongly-typed objects
- **Custom Attributes** - Metadata annotations for validation, defaults, and behavior control
- **Multi-Source Support** - JSON files, environment variables, command line arguments, and custom sources
- **Automatic Validation** - Attribute-driven validation with detailed error reporting
- **Type Conversion** - Intelligent conversion between configuration types with custom converters
- **Change Detection** - Hot-reloading and configuration change notifications
- **Hierarchical Sections** - Nested configuration with inheritance and overrides
- **Performance Optimization** - Cached reflection with expression tree compilation

## Key Concepts

### Reflection in Configuration

```csharp
// Discovering configuration properties at runtime
public static class ConfigurationBinder
{
    public static T Bind<T>(IConfigurationData data) where T : new()
    {
        var instance = new T();
        var type = typeof(T);

        foreach (var property in type.GetProperties())
        {
            var configKey = GetConfigurationKey(property);
            if (data.TryGetValue(configKey, out var value))
            {
                SetPropertyValue(property, instance, value);
            }
        }

        return instance;
    }
}
```

### Custom Attributes for Metadata

```csharp
// Configuration attributes for validation and behavior
[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : ValidationAttribute
{
    public override bool IsValid(object? value)
        => value is not null && !string.Empty.Equals(value);
}

[AttributeUsage(AttributeTargets.Property)]
public class RangeAttribute : ValidationAttribute
{
    public RangeAttribute(double minimum, double maximum) { /* ... */ }
    public override bool IsValid(object? value) { /* ... */ }
}

// Usage on configuration classes
public class GameConfig
{
    [Required]
    [Range(800, 3840)]
    public int ScreenWidth { get; set; }

    [ConfigurationKey("audio.master_volume")]
    [Range(0.0, 1.0)]
    public float MasterVolume { get; set; } = 0.7f;
}
```

### Performance-Optimized Property Access

```csharp
// Compiled property accessors for high-performance binding
public class PropertyAccessor<T>
{
    private static readonly ConcurrentDictionary<string, Func<T, object>> _getters = new();
    private static readonly ConcurrentDictionary<string, Action<T, object>> _setters = new();

    public static object GetValue(T instance, string propertyName)
    {
        var getter = _getters.GetOrAdd(propertyName, CreateGetter);
        return getter(instance);
    }

    public static void SetValue(T instance, string propertyName, object value)
    {
        var setter = _setters.GetOrAdd(propertyName, CreateSetter);
        setter(instance, value);
    }

    private static Func<T, object> CreateGetter(string propertyName)
    {
        // Expression tree compilation for performance
        var parameter = Expression.Parameter(typeof(T));
        var property = Expression.Property(parameter, propertyName);
        var convert = Expression.Convert(property, typeof(object));
        return Expression.Lambda<Func<T, object>>(convert, parameter).Compile();
    }
}
```

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-13-configuration-system/` directory:

```bash
# Create the main solution file
dotnet new sln -n ConfigurationSystem

# Create the main library project
dotnet new classlib -n ConfigurationSystem.Core -f net8.0

# Create the console application
dotnet new console -n ConfigurationSystem.App -f net8.0

# Create the test project
dotnet new mstest -n ConfigurationSystem.Tests -f net8.0

# Add projects to solution
dotnet sln add ConfigurationSystem.Core/ConfigurationSystem.Core.csproj
dotnet sln add ConfigurationSystem.App/ConfigurationSystem.App.csproj
dotnet sln add ConfigurationSystem.Tests/ConfigurationSystem.Tests.csproj

# Add project references
dotnet add ConfigurationSystem.App/ConfigurationSystem.App.csproj reference ConfigurationSystem.Core/ConfigurationSystem.Core.csproj
dotnet add ConfigurationSystem.Tests/ConfigurationSystem.Tests.csproj reference ConfigurationSystem.Core/ConfigurationSystem.Core.csproj
```

### Step 2: Configure Project Files

Update the `ConfigurationSystem.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>ConfigurationSystem.Core</PackageId>
    <Title>Reflection-Based Configuration System</Title>
    <Description>A library for binding and validating configuration using reflection and attributes</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="System.Text.Json" Version="8.0.0" />
    <PackageReference Include="System.ComponentModel.Annotations" Version="5.0.0" />
  </ItemGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your configuration code:

```
ConfigurationSystem.Core/
├── Attributes/          # Custom validation and metadata attributes
├── Binding/             # Configuration binding and conversion
├── Sources/             # Configuration data sources
├── Validation/          # Validation logic and results
├── Reflection/          # Reflection utilities and caching
├── Conversion/          # Type conversion system
└── Models/              # Configuration models and metadata
```

Create these directories:

```bash
mkdir ConfigurationSystem.Core/Attributes
mkdir ConfigurationSystem.Core/Binding
mkdir ConfigurationSystem.Core/Sources
mkdir ConfigurationSystem.Core/Validation
mkdir ConfigurationSystem.Core/Reflection
mkdir ConfigurationSystem.Core/Conversion
mkdir ConfigurationSystem.Core/Models
```

## Implementation Guide

### Step 4: Core Configuration Models (25 minutes)

**Foundation Configuration Interfaces and Models**

```csharp
namespace ConfigurationSystem.Core.Models;

public interface IConfigurationData
{
    bool TryGetValue(string key, out object? value);
    IEnumerable<string> GetAllKeys();
    IConfigurationData GetSection(string sectionName);
    bool HasSection(string sectionName);
}

public class ConfigurationData : IConfigurationData
{
    private readonly Dictionary<string, object?> _values = new();
    private readonly Dictionary<string, ConfigurationData> _sections = new();

    public bool TryGetValue(string key, out object? value)
    {
        // Support nested keys like "database.connectionString"
        var parts = key.Split('.');
        if (parts.Length == 1)
        {
            return _values.TryGetValue(key, out value);
        }

        var sectionName = parts[0];
        var remainingKey = string.Join('.', parts[1..]);

        if (_sections.TryGetValue(sectionName, out var section))
        {
            return section.TryGetValue(remainingKey, out value);
        }

        value = null;
        return false;
    }

    public IEnumerable<string> GetAllKeys()
    {
        foreach (var key in _values.Keys)
        {
            yield return key;
        }

        foreach (var (sectionName, section) in _sections)
        {
            foreach (var subKey in section.GetAllKeys())
            {
                yield return $"{sectionName}.{subKey}";
            }
        }
    }

    public IConfigurationData GetSection(string sectionName)
    {
        return _sections.TryGetValue(sectionName, out var section)
            ? section
            : new ConfigurationData();
    }

    public bool HasSection(string sectionName) => _sections.ContainsKey(sectionName);

    public void SetValue(string key, object? value) => _values[key] = value;

    public void SetSection(string name, ConfigurationData section) => _sections[name] = section;
}

public class ConfigurationMetadata
{
    public string PropertyName { get; init; } = string.Empty;
    public Type PropertyType { get; init; } = typeof(object);
    public string ConfigurationKey { get; init; } = string.Empty;
    public object? DefaultValue { get; init; }
    public bool IsRequired { get; init; }
    public IList<ValidationAttribute> ValidationAttributes { get; init; } = new List<ValidationAttribute>();
    public bool IsSection { get; init; }
    public bool IsCollection { get; init; }
}

public class ConfigurationResult<T>
{
    public T? Value { get; init; }
    public bool IsValid { get; init; }
    public IList<ValidationError> Errors { get; init; } = new List<ValidationError>();
    public IList<string> Warnings { get; init; } = new List<string>();
}

public class ValidationError
{
    public string PropertyName { get; init; } = string.Empty;
    public string Message { get; init; } = string.Empty;
    public object? AttemptedValue { get; init; }
    public string ErrorCode { get; init; } = string.Empty;
}
```

### Step 5: Custom Attributes (25 minutes)

**Validation and Metadata Attributes**

```csharp
namespace ConfigurationSystem.Core.Attributes;

using System.ComponentModel.DataAnnotations;

[AttributeUsage(AttributeTargets.Property)]
public class ConfigurationKeyAttribute : Attribute
{
    public ConfigurationKeyAttribute(string key)
    {
        Key = key ?? throw new ArgumentNullException(nameof(key));
    }

    public string Key { get; }
}

[AttributeUsage(AttributeTargets.Property)]
public class ConfigurationSectionAttribute : Attribute
{
    public ConfigurationSectionAttribute(string? sectionName = null)
    {
        SectionName = sectionName;
    }

    public string? SectionName { get; }
}

[AttributeUsage(AttributeTargets.Property)]
public class ConfigurationCollectionAttribute : Attribute
{
    public ConfigurationCollectionAttribute(string? itemKey = null)
    {
        ItemKey = itemKey ?? "item";
    }

    public string ItemKey { get; }
}

[AttributeUsage(AttributeTargets.Property)]
public class DefaultValueAttribute : Attribute
{
    public DefaultValueAttribute(object? defaultValue)
    {
        DefaultValue = defaultValue;
    }

    public object? DefaultValue { get; }
}

[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : ValidationAttribute
{
    public override bool IsValid(object? value)
    {
        return value switch
        {
            null => false,
            string str => !string.IsNullOrWhiteSpace(str),
            _ => true
        };
    }

    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (IsValid(value))
        {
            return ValidationResult.Success;
        }

        var memberName = validationContext.MemberName ?? "Property";
        return new ValidationResult($"{memberName} is required.", new[] { memberName });
    }
}

[AttributeUsage(AttributeTargets.Property)]
public class RangeAttribute : ValidationAttribute
{
    public RangeAttribute(double minimum, double maximum)
    {
        Minimum = minimum;
        Maximum = maximum;
    }

    public RangeAttribute(int minimum, int maximum)
    {
        Minimum = minimum;
        Maximum = maximum;
    }

    public double Minimum { get; }
    public double Maximum { get; }

    public override bool IsValid(object? value)
    {
        if (value == null) return true; // Let Required handle null checking

        var numericValue = Convert.ToDouble(value);
        return numericValue >= Minimum && numericValue <= Maximum;
    }

    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (IsValid(value))
        {
            return ValidationResult.Success;
        }

        var memberName = validationContext.MemberName ?? "Property";
        return new ValidationResult(
            $"{memberName} must be between {Minimum} and {Maximum}.",
            new[] { memberName });
    }
}

[AttributeUsage(AttributeTargets.Property)]
public class RegexPatternAttribute : ValidationAttribute
{
    private readonly Regex _regex;

    public RegexPatternAttribute(string pattern)
    {
        Pattern = pattern ?? throw new ArgumentNullException(nameof(pattern));
        _regex = new Regex(pattern, RegexOptions.Compiled);
    }

    public string Pattern { get; }

    public override bool IsValid(object? value)
    {
        if (value == null) return true; // Let Required handle null checking
        return _regex.IsMatch(value.ToString() ?? string.Empty);
    }

    protected override ValidationResult? IsValid(object? value, ValidationContext validationContext)
    {
        if (IsValid(value))
        {
            return ValidationResult.Success;
        }

        var memberName = validationContext.MemberName ?? "Property";
        return new ValidationResult(
            $"{memberName} must match the pattern: {Pattern}",
            new[] { memberName });
    }
}
```

## Building on Previous Projects

This project begins Phase 4 by integrating all previous concepts with advanced .NET features:

- **Projects 1-12**: All previous skills in runtime programming context
- **New Skills**: Reflection, attributes, and metadata-driven programming

## Example Usage

```csharp
// Configuration class with attributes
public class GameConfiguration
{
    [Required]
    [Range(800, 3840)]
    public int ScreenWidth { get; set; }

    [Required]
    [Range(600, 2160)]
    public int ScreenHeight { get; set; }

    [DefaultValue(true)]
    public bool FullScreen { get; set; }

    [Range(0.0, 1.0)]
    [DefaultValue(0.7)]
    public float MasterVolume { get; set; }

    [ConfigurationSection("Graphics")]
    public GraphicsSettings Graphics { get; set; }

    [ConfigurationCollection]
    public List<InputBinding> InputBindings { get; set; }
}

// Configuration binding
var configSystem = new ConfigurationSystem()
    .AddSource(new JsonConfigurationSource("config.json"))
    .AddSource(new EnvironmentVariableSource("GAME_"))
    .AddSource(new CommandLineSource(args));

var config = configSystem.Bind<GameConfiguration>();

// Validation happens automatically
if (!config.IsValid)
{
    foreach (var error in config.ValidationErrors)
    {
        Console.WriteLine($"Config error: {error.PropertyName} - {error.Message}");
    }
}

// Hot-reloading support
configSystem.ConfigurationChanged += (sender, e) =>
{
    var updatedConfig = e.GetConfiguration<GameConfiguration>();
    gameEngine.UpdateConfiguration(updatedConfig);
};
```

## Advanced Reflection Patterns

This project explores advanced reflection scenarios:

- **Type Discovery**: Finding types with specific attributes
- **Dynamic Binding**: Runtime property mapping
- **Expression Trees**: Compiled property accessors for performance
- **Generic Type Handling**: Working with generic configuration types
- **Caching**: Optimizing repeated reflection operations

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-13-configuration-system/` directory and create the project structure:

```bash
dotnet new sln -n ConfigurationSystem
dotnet new classlib -n ConfigurationSystem.Core -f net8.0
dotnet new console -n ConfigurationSystem.App -f net8.0
dotnet new mstest -n ConfigurationSystem.Tests -f net8.0

dotnet sln add ConfigurationSystem.Core/ConfigurationSystem.Core.csproj
dotnet sln add ConfigurationSystem.App/ConfigurationSystem.App.csproj
dotnet sln add ConfigurationSystem.Tests/ConfigurationSystem.Tests.csproj

dotnet add ConfigurationSystem.App/ConfigurationSystem.App.csproj reference ConfigurationSystem.Core/ConfigurationSystem.Core.csproj
dotnet add ConfigurationSystem.Tests/ConfigurationSystem.Tests.csproj reference ConfigurationSystem.Core/ConfigurationSystem.Core.csproj
```

### Step 2: Directory Structure

Create these directories in ConfigurationSystem.Core:

```bash
mkdir ConfigurationSystem.Core/Attributes
mkdir ConfigurationSystem.Core/Binding
mkdir ConfigurationSystem.Core/Sources
mkdir ConfigurationSystem.Core/Validation
mkdir ConfigurationSystem.Core/Reflection
mkdir ConfigurationSystem.Core/Conversion
mkdir ConfigurationSystem.Core/Models
```

### Step 3: Configuration Attributes (30 minutes)

Create `ConfigurationSystem.Core/Attributes/ConfigurationAttributes.cs`:

```csharp
using System.ComponentModel.DataAnnotations;
using System.Text.RegularExpressions;

namespace ConfigurationSystem.Core.Attributes;

[AttributeUsage(AttributeTargets.Property)]
public class ConfigurationKeyAttribute : Attribute
{
    public ConfigurationKeyAttribute(string key) => Key = key;
    public string Key { get; }
}

[AttributeUsage(AttributeTargets.Property)]
public class ConfigurationSectionAttribute : Attribute
{
    public ConfigurationSectionAttribute(string? sectionName = null)
        => SectionName = sectionName;
    public string? SectionName { get; }
}

[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : ValidationAttribute
{
    public override bool IsValid(object? value) => value switch
    {
        null => false,
        string str => !string.IsNullOrWhiteSpace(str),
        _ => true
    };

    protected override ValidationResult? IsValid(object? value, ValidationContext context)
    {
        if (IsValid(value)) return ValidationResult.Success;
        return new ValidationResult($"{context.MemberName} is required.",
            new[] { context.MemberName ?? "Property" });
    }
}

[AttributeUsage(AttributeTargets.Property)]
public class RangeAttribute : ValidationAttribute
{
    public RangeAttribute(double min, double max) => (Minimum, Maximum) = (min, max);
    public double Minimum { get; }
    public double Maximum { get; }

    public override bool IsValid(object? value)
    {
        if (value == null) return true;
        var num = Convert.ToDouble(value);
        return num >= Minimum && num <= Maximum;
    }
}
```

### Step 4: Configuration Binding Engine (45 minutes)

Create `ConfigurationSystem.Core/Binding/ConfigurationBinder.cs`:

```csharp
using System.Reflection;
using ConfigurationSystem.Core.Attributes;
using ConfigurationSystem.Core.Models;

namespace ConfigurationSystem.Core.Binding;

public class ConfigurationBinder
{
    public static ConfigurationResult<T> Bind<T>(IConfigurationData data) where T : new()
    {
        var result = new ConfigurationResult<T>();
        var instance = new T();
        var errors = new List<ValidationError>();

        var metadata = ReflectionCache.GetMetadata<T>();

        foreach (var property in metadata.Properties)
        {
            try
            {
                var value = GetConfigurationValue(data, property);
                SetPropertyValue(instance, property, value);

                var validationErrors = ValidateProperty(property, value);
                errors.AddRange(validationErrors);
            }
            catch (Exception ex)
            {
                errors.Add(new ValidationError
                {
                    PropertyName = property.PropertyName,
                    Message = ex.Message,
                    ErrorCode = "BINDING_ERROR"
                });
            }
        }

        return new ConfigurationResult<T>
        {
            Value = instance,
            IsValid = errors.Count == 0,
            Errors = errors
        };
    }

    private static object? GetConfigurationValue(IConfigurationData data, ConfigurationMetadata property)
    {
        var key = property.ConfigurationKey;

        if (data.TryGetValue(key, out var value))
        {
            return ConvertValue(value, property.PropertyType);
        }

        if (property.IsRequired)
        {
            throw new InvalidOperationException($"Required configuration '{key}' not found");
        }

        return property.DefaultValue;
    }

    private static void SetPropertyValue(object instance, ConfigurationMetadata property, object? value)
    {
        var propertyInfo = instance.GetType().GetProperty(property.PropertyName);
        propertyInfo?.SetValue(instance, value);
    }

    private static object? ConvertValue(object? value, Type targetType)
    {
        if (value == null) return null;
        if (targetType.IsAssignableFrom(value.GetType())) return value;

        // Handle nullable types
        if (targetType.IsGenericType && targetType.GetGenericTypeDefinition() == typeof(Nullable<>))
        {
            targetType = Nullable.GetUnderlyingType(targetType)!;
        }

        return Convert.ChangeType(value, targetType);
    }

    private static List<ValidationError> ValidateProperty(ConfigurationMetadata property, object? value)
    {
        var errors = new List<ValidationError>();

        foreach (var validator in property.ValidationAttributes)
        {
            var context = new ValidationContext(new object()) { MemberName = property.PropertyName };
            var result = validator.GetValidationResult(value, context);

            if (result != ValidationResult.Success)
            {
                errors.Add(new ValidationError
                {
                    PropertyName = property.PropertyName,
                    Message = result?.ErrorMessage ?? "Validation failed",
                    AttemptedValue = value,
                    ErrorCode = "VALIDATION_ERROR"
                });
            }
        }

        return errors;
    }
}
```

### Step 5: Reflection Cache for Performance (30 minutes)

Create `ConfigurationSystem.Core/Reflection/ReflectionCache.cs`:

```csharp
using System.Collections.Concurrent;
using System.Reflection;
using ConfigurationSystem.Core.Attributes;
using ConfigurationSystem.Core.Models;

namespace ConfigurationSystem.Core.Reflection;

public static class ReflectionCache
{
    private static readonly ConcurrentDictionary<Type, TypeMetadata> _typeCache = new();

    public static TypeMetadata GetMetadata<T>() => GetMetadata(typeof(T));

    public static TypeMetadata GetMetadata(Type type)
    {
        return _typeCache.GetOrAdd(type, BuildMetadata);
    }

    private static TypeMetadata BuildMetadata(Type type)
    {
        var properties = new List<ConfigurationMetadata>();

        foreach (var property in type.GetProperties(BindingFlags.Public | BindingFlags.Instance))
        {
            if (!property.CanWrite) continue;

            var metadata = new ConfigurationMetadata
            {
                PropertyName = property.Name,
                PropertyType = property.PropertyType,
                ConfigurationKey = GetConfigurationKey(property),
                DefaultValue = GetDefaultValue(property),
                IsRequired = HasAttribute<RequiredAttribute>(property),
                ValidationAttributes = GetValidationAttributes(property).ToList(),
                IsSection = HasAttribute<ConfigurationSectionAttribute>(property),
                IsCollection = HasAttribute<ConfigurationCollectionAttribute>(property)
            };

            properties.Add(metadata);
        }

        return new TypeMetadata
        {
            Type = type,
            Properties = properties
        };
    }

    private static string GetConfigurationKey(PropertyInfo property)
    {
        var keyAttr = property.GetCustomAttribute<ConfigurationKeyAttribute>();
        return keyAttr?.Key ?? ToCamelCase(property.Name);
    }

    private static object? GetDefaultValue(PropertyInfo property)
    {
        var defaultAttr = property.GetCustomAttribute<DefaultValueAttribute>();
        return defaultAttr?.DefaultValue;
    }

    private static bool HasAttribute<T>(PropertyInfo property) where T : Attribute
    {
        return property.GetCustomAttribute<T>() != null;
    }

    private static IEnumerable<ValidationAttribute> GetValidationAttributes(PropertyInfo property)
    {
        return property.GetCustomAttributes<ValidationAttribute>();
    }

    private static string ToCamelCase(string input)
    {
        if (string.IsNullOrEmpty(input)) return input;
        return char.ToLowerInvariant(input[0]) + input[1..];
    }
}

public class TypeMetadata
{
    public Type Type { get; init; } = typeof(object);
    public IList<ConfigurationMetadata> Properties { get; init; } = new List<ConfigurationMetadata>();
}
```

## Testing Your Understanding

Create a comprehensive test in `ConfigurationSystem.App/Program.cs`:

```csharp
using ConfigurationSystem.Core.Attributes;
using ConfigurationSystem.Core.Binding;
using ConfigurationSystem.Core.Models;

// Example configuration classes
public class GameConfiguration
{
    [Required]
    [Range(800, 3840)]
    public int ScreenWidth { get; set; }

    [Required]
    [Range(600, 2160)]
    public int ScreenHeight { get; set; }

    [DefaultValue(true)]
    public bool FullScreen { get; set; }

    [Range(0.0, 1.0)]
    [DefaultValue(0.7)]
    public float MasterVolume { get; set; }

    [ConfigurationSection("graphics")]
    public GraphicsSettings Graphics { get; set; } = new();

    [ConfigurationKey("player.name")]
    [Required]
    public string PlayerName { get; set; } = string.Empty;
}

public class GraphicsSettings
{
    [Range(0, 4)]
    [DefaultValue(2)]
    public int QualityLevel { get; set; }

    [DefaultValue(true)]
    public bool EnableVSync { get; set; }

    [Range(1, 16)]
    [DefaultValue(4)]
    public int AntiAliasing { get; set; }
}

static void Main()
{
    // Create test configuration data
    var configData = new ConfigurationData();
    configData.SetValue("screenWidth", 1920);
    configData.SetValue("screenHeight", 1080);
    configData.SetValue("fullScreen", false);
    configData.SetValue("masterVolume", 0.8);
    configData.SetValue("player.name", "TestPlayer");

    var graphicsSection = new ConfigurationData();
    graphicsSection.SetValue("qualityLevel", 3);
    graphicsSection.SetValue("enableVSync", true);
    graphicsSection.SetValue("antiAliasing", 8);
    configData.SetSection("graphics", graphicsSection);

    // Bind configuration
    var result = ConfigurationBinder.Bind<GameConfiguration>(configData);

    if (result.IsValid)
    {
        var config = result.Value!;
        Console.WriteLine($"Screen: {config.ScreenWidth}x{config.ScreenHeight}");
        Console.WriteLine($"FullScreen: {config.FullScreen}");
        Console.WriteLine($"Volume: {config.MasterVolume}");
        Console.WriteLine($"Player: {config.PlayerName}");
        Console.WriteLine($"Graphics Quality: {config.Graphics.QualityLevel}");
        Console.WriteLine($"VSync: {config.Graphics.EnableVSync}");
        Console.WriteLine($"AA: {config.Graphics.AntiAliasing}");
    }
    else
    {
        Console.WriteLine("Configuration validation failed:");
        foreach (var error in result.Errors)
        {
            Console.WriteLine($"  {error.PropertyName}: {error.Message}");
        }
    }
}
```

## Extension Challenges

1. **JSON Configuration Source**: Load configuration from JSON files
2. **Environment Variable Source**: Map environment variables to configuration
3. **Command Line Source**: Parse command line arguments
4. **Configuration Change Detection**: Hot-reload support with file watchers
5. **Expression Tree Compilation**: Optimize property access for high-performance scenarios
6. **Custom Type Converters**: Support complex type conversions
7. **Configuration Validation**: Complex cross-property validation rules

## Resources

- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 📘 **[Reflection Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/)**
- 📘 **[Attributes Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/reflection-and-attributes/creating-custom-attributes)**
- 📘 **[Expression Trees](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/)**

## Contributing

When implementing this project:

1. Use reflection safely with proper error handling
2. Design intuitive and consistent attributes
3. Implement efficient caching for reflection operations
4. Create comprehensive validation logic
5. Write performance tests for reflection-heavy operations
