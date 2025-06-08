# Project 1: Dice Rolling Library

**Focus**: .NET Type System & Conventions  
**Time**: ~2 hours  
**Deliverable**: `DiceRoller` NuGet package

## Learning Objectives

By completing this project, you'll understand:

- .NET naming conventions (PascalCase, camelCase, etc.)
- Properties vs fields vs methods
- Value types (`struct`) vs reference types (`class`)
- When to use `var` vs explicit typing
- Basic .NET project structure
- Essential C# idioms

## Project Overview

Build a dice rolling library that game developers can use. This seems simple, but you'll learn fundamental .NET concepts while creating something genuinely useful.

## What You'll Build

A library that can:

- Roll various dice types (d4, d6, d8, d10, d12, d20, d100)
- Roll multiple dice at once (`3d6`, `2d20`, etc.)
- Apply modifiers (`1d20+5`, `3d6-2`)
- Parse dice notation strings (`"2d6+3"`)
- Provide roll statistics and history

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

## Implementation Guide

### Step 1: Project Setup (15 minutes)

```bash
# Create new class library project
dotnet new classlib -n DiceRoller
cd DiceRoller

# Update to latest C# version in .csproj
<TargetFramework>net8.0</TargetFramework>
<LangVersion>latest</LangVersion>
```

### Step 2: Core Types (30 minutes)

Create your fundamental types:

**`DiceResult` (struct)**

```csharp
// Value type - represents the result of a dice roll
public readonly struct DiceResult
{
    public int Value { get; }
    public int[] IndividualRolls { get; }
    public int Modifier { get; }

    // Constructor, ToString(), equality methods
}
```

**`Die` (struct)**

```csharp
// Value type - represents a single die
public readonly struct Die
{
    public int Sides { get; }
    // Methods: Roll(), IsValid(), common dice (D6, D20, etc.)
}
```

### Step 3: Main DiceRoller Class (45 minutes)

**`DiceRoller` (class)**

```csharp
public class DiceRoller
{
    private readonly Random _random;  // backing field
    private readonly List<DiceResult> _history;  // backing field

    public IReadOnlyList<DiceResult> History => _history;  // property
    public DiceResult LastResult => _history.LastOrDefault();  // computed property

    // Methods: Roll(), RollMultiple(), ParseAndRoll()
}
```

### Step 4: Dice Notation Parser (30 minutes)

Add ability to parse strings like `"3d6+2"`, `"1d20"`, `"2d10-1"`:

```csharp
public static class DiceNotation
{
    public static (int count, int sides, int modifier) Parse(string notation)
    {
        // Parse "3d6+2" into (3, 6, 2)
        // Handle edge cases and validation
    }
}
```

### Step 5: Extension Methods Preview (20 minutes)

Get a taste of extension methods (covered fully in Week 3):

```csharp
public static class IntExtensions
{
    public static Die D(this int sides) => new Die(sides);
    public static DiceResult Roll(this int count, Die die) => /* ... */;
}

// Usage: 6.D().Roll() or 3.Roll(20.D())
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
var roller = new DiceRoller();

// Test different ways to roll
var result1 = roller.Roll(new Die(6));
var result2 = roller.Roll(3, new Die(6), modifier: 2);
var result3 = roller.ParseAndRoll("2d20+5");

Console.WriteLine($"Single d6: {result1}");
Console.WriteLine($"3d6+2: {result2}");
Console.WriteLine($"Parsed 2d20+5: {result3}");
Console.WriteLine($"Roll history: {roller.History.Count} rolls");
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

## Next Project Preview

In Project 2, you'll learn about collections and `IEnumerable<T>` by building a card deck shuffler. Your dice library will remain standalone, but the concepts will build on what you learned about types and properties.

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
- [List<T>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1) - Storing roll history
- [IReadOnlyList<T>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ireadonlylist-1) - Exposing collections safely

### Modern C# Features

- [var keyword](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/var) - When to use implicit typing
- [String Interpolation](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated) - Modern string formatting
- [Null-conditional operators](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-) - Safe null handling

### Design Guidelines

- [Framework Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/) - How to design .NET libraries
- [Exception Handling Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/best-practices-for-exceptions) - Proper validation patterns
- [C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions) - Consistent code style

**💡 Study Strategy**: Before implementing each class, read the relevant documentation sections. Focus especially on the design guidelines - they'll teach you to think like a .NET library author.

---

**Ready to start? Create your project and begin with Step 1!** 🎲
