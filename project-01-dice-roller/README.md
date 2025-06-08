# Project 1: Dice Rolling Library

**Focus**: .NET Type System, Conventions & Project Setup  
**Time**: ~2 hours  
**Deliverable**: `DiceRoller` NuGet package

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Naming Conventions](../microsoft-design-guidelines.md#naming-guidelines)** - PascalCase for public members, camelCase for private fields
- **[Type Design](../microsoft-design-guidelines.md#type-design-guidelines)** - Value types for immutable data, reference types for mutable objects
- **[API Design](../microsoft-design-guidelines.md#api-design-guidelines)** - Intuitive method names, consistent parameter ordering
- **[Performance](../microsoft-design-guidelines.md#performance-guidelines)** - Struct usage for lightweight value types

## Learning Objectives

By completing this project, you'll understand:

1. **Master .NET Project Setup**: Learn to create solutions, projects, and manage dependencies
2. **Understand Type System**: Implement custom value types and reference types
3. **Apply Naming Conventions**: Follow Microsoft's naming guidelines throughout
4. **Structure Code Professionally**: Organize code with proper namespaces and file structure
5. **Set Up Testing**: Create and configure unit test projects
6. **.NET naming conventions** (PascalCase, camelCase, etc.)
7. **Properties vs fields vs methods**
8. **Value types (`struct`) vs reference types (`class`)**
9. **When to use `var` vs explicit typing**
10. **Basic .NET project structure**
11. **Essential C# idioms**

## What You'll Build

A dice rolling library that supports:

- Various dice types (d4, d6, d8, d10, d12, d20, d100)
- Multiple dice rolls (`3d6`, `2d20`)
- Modifiers (`1d20+5`, `3d6-2`)
- Dice notation parsing (`"2d6+3"`)
- Roll statistics and history

## Core Concepts to Learn

### .NET Naming Conventions

- **Classes/Interfaces**: `PascalCase` (`DiceRoller`, `IDice`)
- **Methods/Properties**: `PascalCase` (`Roll()`, `LastResult`)
- **Parameters/Variables**: `camelCase` (`diceCount`, `modifier`)
- **Constants**: `PascalCase` (`MaxSides`)
- **Private fields**: `_camelCase` (`_random`, `_history`)

### Value Types vs Reference Types

- Use `struct` for `DiceResult` (small, immutable data)
- Use `class` for `DiceRoller` (behavior, mutable state)

### Properties vs Fields

- Expose data through properties, not public fields
- Use auto-properties where appropriate
- Understand when to use full properties with backing fields

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

In this project, you'll learn to set up a complete .NET solution from scratch. Open your terminal and navigate to the `project-01-dice-roller/` directory.

```bash
# Create the main solution file
dotnet new sln -n DiceRoller

# Create the main library project
dotnet new classlib -n DiceRoller.Core -f net8.0

# Create the console application
dotnet new console -n DiceRoller.App -f net8.0

# Create the test project
dotnet new mstest -n DiceRoller.Tests -f net8.0

# Add projects to solution
dotnet sln add DiceRoller.Core/DiceRoller.Core.csproj
dotnet sln add DiceRoller.App/DiceRoller.App.csproj
dotnet sln add DiceRoller.Tests/DiceRoller.Tests.csproj

# Add project references
dotnet add DiceRoller.App/DiceRoller.App.csproj reference DiceRoller.Core/DiceRoller.Core.csproj
dotnet add DiceRoller.Tests/DiceRoller.Tests.csproj reference DiceRoller.Core/DiceRoller.Core.csproj
```

### Step 2: Configure Project Files

Update the `DiceRoller.Core.csproj` file to include proper metadata:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>DiceRoller.Core</PackageId>
    <Title>Dice Rolling Library</Title>
    <Description>A library for simulating dice rolls with various dice types</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your code with this structure:

```
DiceRoller.Core/
├── Models/          # Value types and data models
├── Services/        # Business logic classes
├── Enums/           # Enumeration types
└── Extensions/      # Extension methods
```

Create these directories:

```bash
mkdir DiceRoller.Core/Models
mkdir DiceRoller.Core/Services
mkdir DiceRoller.Core/Enums
mkdir DiceRoller.Core/Extensions
```

## Implementation Guide

### Step 4: Core Types (30 minutes)

Create your fundamental types:

**`DiceResult` (struct) - Value type for dice roll results**

```csharp
// Value type - represents the result of a dice roll
namespace DiceRoller.Core.Models;

public readonly struct DiceResult
{
    public int Value { get; }
    public int[] IndividualRolls { get; }
    public int Modifier { get; }
    public int TotalWithModifier => Value + Modifier;

    public DiceResult(int[] rolls, int modifier = 0)
    {
        IndividualRolls = rolls ?? throw new ArgumentNullException(nameof(rolls));
        Value = rolls.Sum();
        Modifier = modifier;
    }

    public override string ToString() =>
        $"{Value}{(Modifier >= 0 ? "+" : "")}{Modifier} = {TotalWithModifier}";

    // Implement IEquatable<DiceResult>, GetHashCode
}
```

**`Die` (struct) - Value type for individual dice**

```csharp
// Value type - represents a single die
namespace DiceRoller.Core.Models;

public readonly struct Die
{
    public int Sides { get; }
    public bool IsValid => Sides > 0;

    public Die(int sides)
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(sides);
        Sides = sides;
    }

    public int Roll(Random random) => random.Next(1, Sides + 1);

    // Common dice factory methods
    public static Die D4 => new(4);
    public static Die D6 => new(6);
    public static Die D8 => new(8);
    public static Die D10 => new(10);
    public static Die D12 => new(12);
    public static Die D20 => new(20);
    public static Die D100 => new(100);

    public override string ToString() => $"d{Sides}";
}
```

### Step 5: Main DiceRoller Class (45 minutes)

**`DiceRoller` (class) - Reference type for dice rolling operations**

```csharp
namespace DiceRoller.Core.Services;

public class DiceRoller
{
    private readonly Random _random;  // backing field
    private readonly List<DiceResult> _history;  // backing field

    public IReadOnlyList<DiceResult> History => _history;  // property
    public DiceResult? LastResult => _history.Count > 0 ? _history[^1] : null;  // computed property
    public int TotalRolls => _history.Count;

    public DiceRoller() : this(new Random()) { }

    public DiceRoller(Random random)
    {
        _random = random ?? throw new ArgumentNullException(nameof(random));
        _history = new List<DiceResult>();
    }

    // Roll a single die
    public DiceResult Roll(Die die, int modifier = 0)
    {
        var roll = die.Roll(_random);
        var result = new DiceResult([roll], modifier);
        _history.Add(result);
        return result;
    }

    // Roll multiple dice
    public DiceResult Roll(int count, Die die, int modifier = 0)
    {
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(count);

        var rolls = new int[count];
        for (int i = 0; i < count; i++)
        {
            rolls[i] = die.Roll(_random);
        }

        var result = new DiceResult(rolls, modifier);
        _history.Add(result);
        return result;
    }

    // Parse and roll dice notation like "3d6+2"
    public DiceResult ParseAndRoll(string notation)
    {
        var (count, sides, modifier) = DiceNotation.Parse(notation);
        return Roll(count, new Die(sides), modifier);
    }

    public void ClearHistory() => _history.Clear();
}
```

### Step 6: Dice Notation Parser (30 minutes)

Add ability to parse strings like `"3d6+2"`, `"1d20"`, `"2d10-1"`:

```csharp
namespace DiceRoller.Core.Services;

public static class DiceNotation
{
    public static (int count, int sides, int modifier) Parse(string notation)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(notation);

        // Remove spaces and convert to lowercase
        notation = notation.Replace(" ", "").ToLowerInvariant();

        // Split on 'd' to get count and rest
        var parts = notation.Split('d');
        if (parts.Length != 2)
            throw new ArgumentException($"Invalid dice notation: {notation}");

        // Parse count (optional, defaults to 1)
        var countPart = string.IsNullOrEmpty(parts[0]) ? "1" : parts[0];
        if (!int.TryParse(countPart, out int count) || count <= 0)
            throw new ArgumentException($"Invalid dice count: {countPart}");

        // Parse sides and modifier
        var rightPart = parts[1];
        int sides, modifier = 0;

        // Check for modifier
        if (rightPart.Contains('+') || rightPart.Contains('-'))
        {
            var modifierIndex = Math.Max(rightPart.IndexOf('+'), rightPart.IndexOf('-'));
            var sidesStr = rightPart[..modifierIndex];
            var modifierStr = rightPart[modifierIndex..];

            if (!int.TryParse(sidesStr, out sides) || sides <= 0)
                throw new ArgumentException($"Invalid dice sides: {sidesStr}");

            if (!int.TryParse(modifierStr, out modifier))
                throw new ArgumentException($"Invalid modifier: {modifierStr}");
        }
        else
        {
            if (!int.TryParse(rightPart, out sides) || sides <= 0)
                throw new ArgumentException($"Invalid dice sides: {rightPart}");
        }

        return (count, sides, modifier);
    }
}
```

### Step 7: Extension Methods Preview (20 minutes)

Get a taste of extension methods (covered fully in Week 3):

```csharp
namespace DiceRoller.Core.Extensions;

public static class IntExtensions
{
    public static Die D(this int sides) => new Die(sides);
}

public static class DiceExtensions
{
    public static DiceResult Roll(this Die die, DiceRoller roller, int modifier = 0)
        => roller.Roll(die, modifier);

    public static DiceResult Roll(this int count, Die die, DiceRoller roller, int modifier = 0)
        => roller.Roll(count, die, modifier);
}

// Usage examples:
// var roller = new DiceRoller();
// var result1 = 6.D().Roll(roller);
// var result2 = 3.Roll(6.D(), roller, 2);
```

## Key .NET Idioms to Practice

1. **Property Patterns**:

   ```csharp
   // Auto-property
   public int Sides { get; }

   // Computed property
   public bool IsValid => Sides > 0;

   // Property with backing field
   private int _lastRoll;
   public int LastRoll => _lastRoll;
   ```

2. **Constructor Patterns**:

   ```csharp
   // Traditional constructor
   public Die(int sides)
   {
       ArgumentOutOfRangeException.ThrowIfNegativeOrZero(sides);
       Sides = sides;
   }

   // Alternative: Primary constructor (C# 12+)
   // public readonly struct Die(int sides)
   // {
   //     public int Sides => ArgumentOutOfRangeException.ThrowIfNegativeOrZero(sides, nameof(sides)) == 0 ? sides : sides;
   //     // Note: Primary constructor parameters are available throughout the struct
   // }
   ```

3. **Null Safety**:

   ```csharp
   public DiceResult? LastResult => _history.Count > 0 ? _history[^1] : null;
   ```

4. **String Interpolation**:
   ```csharp
   public override string ToString() => $"{Count}d{Sides}{(Modifier >= 0 ? "+" : "")}{Modifier}";
   ```

## Testing Your Understanding

Create a simple console app to test your library:

```csharp
using DiceRoller.Core.Services;
using DiceRoller.Core.Models;
using DiceRoller.Core.Extensions;

var roller = new DiceRoller();

// Test different ways to roll
var result1 = roller.Roll(Die.D6);
var result2 = roller.Roll(3, Die.D6, modifier: 2);
var result3 = roller.ParseAndRoll("2d20+5");

Console.WriteLine($"Single d6: {result1}");
Console.WriteLine($"3d6+2: {result2}");
Console.WriteLine($"Parsed 2d20+5: {result3}");
Console.WriteLine($"Roll history: {roller.History.Count} rolls");

// Test extension methods
var result4 = 6.D().Roll(roller);
var result5 = 3.Roll(20.D(), roller, 5);

Console.WriteLine($"Extension d6: {result4}");
Console.WriteLine($"Extension 3d20+5: {result5}");
```

## Extension Challenges

If you finish early or want to explore more:

1. **Add Advantage/Disadvantage**: Roll twice, take highest/lowest
2. **Dice Pools**: Roll multiple dice, count successes above threshold
3. **Custom Dice**: Support non-standard dice (d3, d7, etc.)
4. **Roll Statistics**: Track averages, streaks, probability distributions
5. **Serialization**: Save/load roll history to JSON

## Common Pitfalls & Solutions

1. **Using `class` for `DiceResult`**: Should be `struct` - it's small, immutable data
2. **Public fields**: Use properties instead for better encapsulation
3. **Missing validation**: Always validate constructor parameters
4. **Inconsistent naming**: Follow .NET conventions religiously
5. **Mutable structs**: Keep structs readonly when possible

## Success Criteria

✅ Created a working dice rolling library  
✅ Used `struct` for value types, `class` for reference types  
✅ Followed .NET naming conventions throughout  
✅ Implemented properties correctly (auto, computed, backing field)  
✅ Added proper parameter validation  
✅ Created a simple test console application  
✅ Library feels "natural" to use (good API design)

## Building and Testing

```bash
# Build the solution
dotnet build

# Run all tests
dotnet test

# Run the demo application
dotnet run --project DiceRoller.App
```

## 📚 Essential Documentation for This Project

**Reading Microsoft's documentation is crucial for mastering .NET.** These links directly support this project's learning objectives:

### Core Language Concepts

- [.NET Naming Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines) - Official naming conventions you'll practice
- [Classes](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes) - Reference type fundamentals for `DiceRoller`
- [Structures](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct) - Value type fundamentals for `DiceResult` and `Die`
- [Properties](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties) - Property patterns used throughout

### Type System Deep Dive

- [Value Types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types) - Why `struct` for `DiceResult`
- [Reference Types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types) - Why `class` for `DiceRoller`
- [Choosing Between Class and Struct](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/choosing-between-class-and-struct) - Official guidance
- [readonly modifier](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/readonly) - Making structs immutable

### Specific APIs You'll Use

- [Random Class](https://docs.microsoft.com/en-us/dotnet/api/system.random) - Generating random dice rolls
- [ArgumentOutOfRangeException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentoutofrangeexception) - Parameter validation
- [List\<T\>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1) - Storing roll history
- [IReadOnlyList\<T\>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlylist-1) - Exposing collections safely

### Modern C# Features

- [var keyword](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/var) - When to use implicit typing
- [String Interpolation](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated) - Modern string formatting
- [Null-conditional operators](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-) - Safe null handling

### Design Guidelines

- [Framework Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/) - How to design .NET libraries
- [Exception Handling Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions) - Proper validation patterns
- [C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions) - Consistent code style

**💡 Study Strategy**: Before implementing each class, read the relevant documentation sections. Focus especially on the design guidelines - they'll teach you to think like a .NET library author.

## Resources

- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Type System Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)**

## Contributing

When implementing this project:

1. Follow the Microsoft design guidelines
2. Write clean, readable code with proper naming
3. Include XML documentation comments
4. Add unit tests for core functionality
5. Update the progress tracker with your learnings

---

**Ready to start? Create your project and begin with Step 1!** 🎲
