# Project 3: String Processing Utilities

**Focus**: Extension Methods & Fluent APIs  
**Time**: ~2 hours  
**Deliverable**: Text manipulation library with fluent API

## Learning Objectives

By completing this project, you'll understand:

- How extension methods work (`this` parameter)
- Creating fluent interfaces and method chaining
- When and why to use extension methods
- Extending existing .NET types like `string`
- Building discoverable APIs that feel "native"
- Method chaining patterns and immutability

## Project Overview

Build a string processing library that extends `string` with game development utilities like text formatting, color codes, template processing, and validation. You'll learn to make APIs that feel like they belong in .NET itself.

## What You'll Build

A library that extends `string` with:

- Text formatting (padding, alignment, truncation)
- Color and markup processing (for game UIs)
- Template string processing (`"Hello {name}"`)
- Validation utilities (email, game tags, etc.)
- Text generation (random names, lorem ipsum)
- All chainable with fluent syntax

## Core Concepts to Learn

### Extension Methods

```csharp
// This method "extends" string
public static string Reverse(this string input)
{
    // Can be called as: "hello".Reverse()
}
```

### Fluent Interface Design

```csharp
// Method chaining for readable APIs
var result = "  hello world  "
    .Trim()
    .ToTitleCase()
    .Truncate(10)
    .WithColor("#FF0000");
```

## Implementation Guide

### Step 1: Project Setup (10 minutes)

```bash
# Create new class library project
dotnet new classlib -n StringUtilities
cd StringUtilities
```

### Step 2: Basic Extension Methods (30 minutes)

**Core String Extensions**

```csharp
public static class StringExtensions
{
    // Simple extension method
    public static string Reverse(this string input)
    {
        ArgumentNullException.ThrowIfNull(input);
        return new string(input.ToCharArray().Reverse().ToArray());
    }

    // Extension with parameters
    public static string Truncate(this string input, int maxLength, string suffix = "...")
    {
        ArgumentNullException.ThrowIfNull(input);
        ArgumentOutOfRangeException.ThrowIfNegative(maxLength);

        if (input.Length <= maxLength) return input;

        var truncated = input[..Math.Max(0, maxLength - suffix.Length)];
        return truncated + suffix;
    }

    // Extension that returns same type (for chaining)
    public static string ToTitleCase(this string input)
    {
        ArgumentNullException.ThrowIfNull(input);
        if (string.IsNullOrWhiteSpace(input)) return input;

        return CultureInfo.CurrentCulture.TextInfo.ToTitleCase(input.ToLower());
    }
}
```

### Step 3: Fluent Interface Pattern (40 minutes)

**Text Formatting Builder**

```csharp
// Immutable fluent interface
public static class StringFormatExtensions
{
    public static string PadCenter(this string input, int totalWidth, char padChar = ' ')
    {
        ArgumentNullException.ThrowIfNull(input);
        if (input.Length >= totalWidth) return input;

        int padding = totalWidth - input.Length;
        int leftPad = padding / 2;
        int rightPad = padding - leftPad;

        return new string(padChar, leftPad) + input + new string(padChar, rightPad);
    }

    public static string WithPrefix(this string input, string prefix)
    {
        ArgumentNullException.ThrowIfNull(input);
        ArgumentNullException.ThrowIfNull(prefix);
        return prefix + input;
    }

    public static string WithSuffix(this string input, string suffix)
    {
        ArgumentNullException.ThrowIfNull(input);
        ArgumentNullException.ThrowIfNull(suffix);
        return input + suffix;
    }

    // Chaining example usage:
    // var formatted = "Hello"
    //     .ToTitleCase()
    //     .PadCenter(20, '-')
    //     .WithPrefix("[")
    //     .WithSuffix("]");
}
```

**Game-Specific Extensions**

```csharp
public static class GameStringExtensions
{
    // Color markup for game UIs
    public static string WithColor(this string input, string hexColor)
    {
        ArgumentNullException.ThrowIfNull(input);
        ArgumentNullException.ThrowIfNull(hexColor);
        return $"<color={hexColor}>{input}</color>";
    }

    public static string Bold(this string input)
    {
        ArgumentNullException.ThrowIfNull(input);
        return $"<b>{input}</b>";
    }

    public static string Italic(this string input)
    {
        ArgumentNullException.ThrowIfNull(input);
        return $"<i>{input}</i>";
    }

    // Chaining: "Critical Hit!".Bold().WithColor("#FF0000")
}
```

### Step 4: Template Processing (30 minutes)

**Simple Template Engine**

```csharp
public static class TemplateExtensions
{
    // Process templates like "Hello {name}, you have {score} points"
    public static string Process(this string template, object data)
    {
        ArgumentNullException.ThrowIfNull(template);
        ArgumentNullException.ThrowIfNull(data);

        var result = template;
        var properties = data.GetType().GetProperties();

        foreach (var prop in properties)
        {
            var placeholder = $"{{{prop.Name}}}";
            var value = prop.GetValue(data)?.ToString() ?? "";
            result = result.Replace(placeholder, value, StringComparison.OrdinalIgnoreCase);
        }

        return result;
    }

    // Dictionary-based processing
    public static string Process(this string template, IDictionary<string, object> values)
    {
        ArgumentNullException.ThrowIfNull(template);
        ArgumentNullException.ThrowIfNull(values);

        var result = template;
        foreach (var kvp in values)
        {
            var placeholder = $"{{{kvp.Key}}}";
            var value = kvp.Value?.ToString() ?? "";
            result = result.Replace(placeholder, value, StringComparison.OrdinalIgnoreCase);
        }

        return result;
    }
}

// Usage:
// var message = "Hello {Name}, your level is {Level}!".Process(new { Name = "Alice", Level = 5 });
```

### Step 5: Validation Extensions (30 minutes)

**Fluent Validation Patterns**

```csharp
public static class ValidationExtensions
{
    public static bool IsValidEmail(this string input)
    {
        if (string.IsNullOrWhiteSpace(input)) return false;

        try
        {
            var addr = new System.Net.Mail.MailAddress(input);
            return addr.Address == input;
        }
        catch
        {
            return false;
        }
    }

    public static bool IsValidGameTag(this string input)
    {
        // Game tags: alphanumeric, 3-16 chars, no spaces
        if (string.IsNullOrWhiteSpace(input)) return false;
        return Regex.IsMatch(input, @"^[a-zA-Z0-9]{3,16}$");
    }

    public static bool HasMinLength(this string input, int minLength)
    {
        return input?.Length >= minLength;
    }

    public static bool HasMaxLength(this string input, int maxLength)
    {
        return input?.Length <= maxLength;
    }

    // Chaining validation:
    // bool isValid = userInput
    //     .HasMinLength(3)
    //     && userInput.HasMaxLength(20)
    //     && userInput.IsValidGameTag();
}
```

## Key Extension Method Patterns

### 1. Null Safety First

```csharp
public static string SafeMethod(this string input, string parameter)
{
    ArgumentNullException.ThrowIfNull(input);      // Check 'this' parameter
    ArgumentNullException.ThrowIfNull(parameter);  // Check other parameters

    // Your logic here
    return result;
}
```

### 2. Immutable Chaining

```csharp
// ✅ Return new strings (immutable)
public static string Transform(this string input)
{
    return input + "transformed"; // New string
}

// ❌ Don't modify the original (strings are immutable anyway)
// But this pattern applies to mutable types too
```

### 3. Logical Grouping

```csharp
// Group related extensions in focused classes
public static class StringFormatExtensions { /* formatting */ }
public static class StringValidationExtensions { /* validation */ }
public static class GameStringExtensions { /* game-specific */ }
```

### 4. Discoverability

```csharp
// Use clear, intention-revealing names
public static string ToGameDisplayName(this string input) // Clear purpose
public static string Process(this string template)       // Too generic
```

## Testing Your Understanding

Create a console app to test your fluent API:

```csharp
// Basic chaining
var formatted = "  hello world  "
    .Trim()
    .ToTitleCase()
    .PadCenter(20, '=')
    .WithPrefix(">>> ")
    .WithSuffix(" <<<");

Console.WriteLine(formatted);

// Game UI formatting
var gameMessage = "CRITICAL HIT"
    .Bold()
    .WithColor("#FF0000")
    .PadCenter(30, '-');

Console.WriteLine(gameMessage);

// Template processing
var playerStats = "Player {Name} - Level {Level} - HP: {Health}".Process(new
{
    Name = "Alice",
    Level = 42,
    Health = 100
});

Console.WriteLine(playerStats);

// Validation chaining
var isValidInput = userInput
    .HasMinLength(3) &&
    userInput.HasMaxLength(16) &&
    userInput.IsValidGameTag();

Console.WriteLine($"Input valid: {isValidInput}");
```

## Advanced Extension Patterns

### Generic Extension Methods

```csharp
public static class ObjectExtensions
{
    // Works with any type
    public static T Tap<T>(this T obj, Action<T> action)
    {
        action(obj);
        return obj; // For chaining
    }

    // Usage: player.Tap(p => Console.WriteLine($"Player: {p.Name}")).Move(newPosition);
}
```

### Conditional Extensions

```csharp
public static class ConditionalExtensions
{
    public static string If(this string input, bool condition, Func<string, string> transform)
    {
        return condition ? transform(input) : input;
    }

    // Usage: text.If(isHighlighted, s => s.Bold().WithColor("#FFFF00"))
}
```

## Extension Challenges

1. **Rich Text Builder**: Create a fluent builder for complex markup
2. **Localization Extensions**: Add multi-language support with fluent syntax
3. **Performance Extensions**: Add StringBuilder-based extensions for efficiency
4. **Parsing Extensions**: Add `ParseInt()`, `ParseEnum<T>()`, etc.
5. **Security Extensions**: Add sanitization and encoding extensions

## Common Extension Method Pitfalls

1. **Over-extension**: Don't extend everything

   ```csharp
   // ❌ Too specific, not generally useful
   public static string ToUpperCaseIfTuesday(this string input)

   // ✅ Generally useful
   public static string ToTitleCase(this string input)
   ```

2. **Breaking immutability**:

   ```csharp
   // ✅ Return new instances for value types
   public static string Transform(this string input) => newString;

   // ⚠️ Be careful with reference types
   public static List<T> Shuffle<T>(this List<T> list)
   {
       // Should this modify the original or return a new list?
   }
   ```

3. **Namespace pollution**:
   ```csharp
   // Keep extensions in focused namespaces
   namespace MyGame.StringUtils; // Not just MyGame
   ```

## Success Criteria

✅ Created multiple extension method classes  
✅ Built fluent interfaces with method chaining  
✅ Demonstrated understanding of `this` parameter  
✅ Applied null safety patterns consistently  
✅ Created discoverable, intention-revealing APIs  
✅ Implemented both simple and complex extension patterns  
✅ Built a working template processing system  
✅ Created game-specific string utilities

## Next Project Preview

Project 4 covers pattern matching and switch expressions by building a state machine for turn-based games. You'll learn modern C# pattern matching syntax and how to manage complex game state transitions elegantly.

## 📚 Essential Documentation for This Project

**Master extension methods by studying Microsoft's guidance.** These docs directly support your learning objectives:

### Extension Methods Fundamentals

- [Extension Methods Guide](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) - Core concepts and syntax
- [Extension Methods Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/extension-methods) - When and how to use them
- [this keyword](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/this) - Understanding the extension syntax

### String Processing APIs

- [String Class](https://docs.microsoft.com/en-us/dotnet/api/system.string) - Core string methods you'll extend
- [StringBuilder](https://docs.microsoft.com/en-us/dotnet/api/system.text.stringbuilder) - Efficient string building for complex operations
- [String.Join](https://docs.microsoft.com/en-us/dotnet/api/system.string.join) - Combining strings efficiently
- [String interpolation](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated) - Modern string formatting

### Method Chaining Patterns

- [Fluent Interface](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/fluent-interfaces) - Design guidelines for chainable APIs
- [Method overloading](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods) - Creating multiple method signatures
- [Optional parameters](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments) - Making APIs flexible

### Text Processing Features

- [Regular Expressions](https://docs.microsoft.com/en-us/dotnet/standard/base-types/regular-expressions) - Pattern matching for validation
- [CultureInfo](https://docs.microsoft.com/en-us/dotnet/api/system.globalization.cultureinfo) - Locale-aware text processing
- [Text encoding](https://docs.microsoft.com/en-us/dotnet/api/system.text.encoding) - Character encoding considerations

### Generic Programming

- [Generic Methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/generic-methods) - Making extensions work with any type
- [Type Constraints](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters) - Restricting generic parameters
- [Action and Func delegates](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/) - Function parameters for flexibility

### Validation and Safety

- [ArgumentNullException](https://docs.microsoft.com/en-us/dotnet/api/system.argumentnullexception) - Proper parameter validation
- [null-conditional operators](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/member-access-operators#null-conditional-operators--and-) - Safe null handling
- [nullable reference types](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references) - Compile-time null safety

### Performance Considerations

- [String performance](https://docs.microsoft.com/en-us/dotnet/standard/base-types/best-practices-strings) - Efficient string operations
- [Memory allocation](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals) - Understanding allocation patterns
- [Span<T> and ReadOnlySpan<T>](https://docs.microsoft.com/en-us/dotnet/standard/memory-and-spans/) - Zero-allocation string processing

**💡 Reading Strategy**: Start with extension methods fundamentals, then dive into string APIs. The fluent interface guidelines will teach you to design chainable APIs that feel natural to .NET developers.

---

**Ready to extend the world? Begin with Step 1!** 🔧
