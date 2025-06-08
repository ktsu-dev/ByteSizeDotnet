# Project 3: String Processing Utilities

**Focus**: Extension Methods & Fluent APIs  
**Time**: ~2 hours  
**Deliverable**: Text manipulation library with fluent API

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Extension Method Guidelines](../microsoft-design-guidelines.md#extension-method-guidelines)** - Proper extension method design
- **[API Design](../microsoft-design-guidelines.md#api-design-guidelines)** - Fluent interface patterns
- **[Naming Conventions](../microsoft-design-guidelines.md#naming-conventions)** - Method and parameter naming
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - String manipulation efficiency

## Learning Objectives

By completing this project, you'll understand:

- How extension methods work (`this` parameter)
- Creating fluent interfaces and method chaining
- When and why to use extension methods
- Extending existing .NET types like `string`
- Building discoverable APIs that feel "native"
- Method chaining patterns and immutability
- Performance-optimized string operations
- Template processing and validation patterns

## What You'll Build

A comprehensive string processing library featuring:

- Fluent text transformation methods
- Advanced parsing and validation
- Performance-optimized operations using `Span<T>`
- Game-specific string utilities
- Chainable formatting operations
- Template string processing (`"Hello {name}"`)
- Color and markup processing (for game UIs)
- Text generation utilities

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

## Project Setup Instructions

### Step 1: Project Setup (10 minutes)

Navigate to the `project-03-string-utilities/src` directory and create the solution:

```bash
# Create the main solution file
dotnet new sln -n StringUtilities

# Create the main library project
dotnet new classlib -n StringUtilities.Core -f net8.0

# Create the console application
dotnet new console -n StringUtilities.App -f net8.0

# Create the test project
dotnet new mstest -n StringUtilities.Tests -f net8.0

# Add projects to solution
dotnet sln add StringUtilities.Core/StringUtilities.Core.csproj
dotnet sln add StringUtilities.App/StringUtilities.App.csproj
dotnet sln add StringUtilities.Tests/StringUtilities.Tests.csproj

# Add project references
dotnet add StringUtilities.App/StringUtilities.App.csproj reference StringUtilities.Core/StringUtilities.Core.csproj
dotnet add StringUtilities.Tests/StringUtilities.Tests.csproj reference StringUtilities.Core/StringUtilities.Core.csproj
```

Create directory structure:

```bash
mkdir StringUtilities.Core/Extensions
mkdir StringUtilities.Core/Builders
mkdir StringUtilities.Core/Validators
```

## Implementation Guide

### Step 2: Basic Extension Methods (30 minutes)

**Core String Extensions**

```csharp
using System.Globalization;

namespace StringUtilities.Core.Extensions;

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

    public static string ToPascalCase(this string input)
    {
        ArgumentNullException.ThrowIfNull(input);
        if (string.IsNullOrWhiteSpace(input)) return input;

        var words = input.Split(new[] { ' ', '_', '-' }, StringSplitOptions.RemoveEmptyEntries);
        return string.Concat(words.Select(word =>
            char.ToUpper(word[0]) + word[1..].ToLower()));
    }

    public static string ToCamelCase(this string input)
    {
        var pascal = input.ToPascalCase();
        if (string.IsNullOrEmpty(pascal)) return pascal;
        return char.ToLower(pascal[0]) + pascal[1..];
    }
}
```

### Step 3: Fluent Interface Pattern (40 minutes)

**Text Formatting Builder**

```csharp
namespace StringUtilities.Core.Extensions;

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

    public static string Wrap(this string input, string wrapper)
    {
        ArgumentNullException.ThrowIfNull(input);
        ArgumentNullException.ThrowIfNull(wrapper);
        return wrapper + input + wrapper;
    }

    public static string Wrap(this string input, string prefix, string suffix)
    {
        ArgumentNullException.ThrowIfNull(input);
        ArgumentNullException.ThrowIfNull(prefix);
        ArgumentNullException.ThrowIfNull(suffix);
        return prefix + input + suffix;
    }

    public static string Repeat(this string input, int count, string separator = "")
    {
        ArgumentNullException.ThrowIfNull(input);
        ArgumentOutOfRangeException.ThrowIfNegative(count);

        if (count == 0) return string.Empty;
        if (count == 1) return input;

        return string.Join(separator, Enumerable.Repeat(input, count));
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
namespace StringUtilities.Core.Extensions;

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

    public static string Underline(this string input)
    {
        ArgumentNullException.ThrowIfNull(input);
        return $"<u>{input}</u>";
    }

    public static string WithSize(this string input, int size)
    {
        ArgumentNullException.ThrowIfNull(input);
        ArgumentOutOfRangeException.ThrowIfNegativeOrZero(size);
        return $"<size={size}>{input}</size>";
    }

    // Chaining: "Critical Hit!".Bold().WithColor("#FF0000").WithSize(24)
}
```

### Step 4: Template Processing (30 minutes)

**Simple Template Engine**

```csharp
using System.Reflection;
using System.Text.RegularExpressions;

namespace StringUtilities.Core.Extensions;

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

    // Advanced template processing with format specifiers
    public static string ProcessAdvanced(this string template, object data)
    {
        ArgumentNullException.ThrowIfNull(template);
        ArgumentNullException.ThrowIfNull(data);

        var properties = data.GetType().GetProperties().ToDictionary(p => p.Name, p => p);

        return Regex.Replace(template, @"\{(\w+)(?::([^}]+))?\}", match =>
        {
            var propertyName = match.Groups[1].Value;
            var format = match.Groups[2].Value;

            if (properties.TryGetValue(propertyName, out var property))
            {
                var value = property.GetValue(data);
                if (value == null) return "";

                if (!string.IsNullOrEmpty(format))
                {
                    if (value is IFormattable formattable)
                        return formattable.ToString(format, null);
                }

                return value.ToString();
            }

            return match.Value; // Return original if property not found
        }, RegexOptions.IgnoreCase);
    }
}

// Usage:
// var message = "Hello {Name}, your level is {Level}!".Process(new { Name = "Alice", Level = 5 });
// var formatted = "Score: {Score:N0}, Time: {Time:mm\\:ss}".ProcessAdvanced(new { Score = 1234, Time = TimeSpan.FromSeconds(75) });
```

### Step 5: Validation Extensions (30 minutes)

**Fluent Validation Patterns**

```csharp
using System.Net.Mail;
using System.Text.RegularExpressions;

namespace StringUtilities.Core.Validators;

public static class ValidationExtensions
{
    public static bool IsValidEmail(this string input)
    {
        if (string.IsNullOrWhiteSpace(input)) return false;

        try
        {
            var addr = new MailAddress(input);
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

    public static bool IsAlphanumeric(this string input)
    {
        if (string.IsNullOrEmpty(input)) return false;
        return input.All(char.IsLetterOrDigit);
    }

    public static bool IsNumeric(this string input)
    {
        if (string.IsNullOrEmpty(input)) return false;
        return input.All(char.IsDigit);
    }

    public static bool ContainsOnly(this string input, string allowedChars)
    {
        if (string.IsNullOrEmpty(input)) return false;
        ArgumentNullException.ThrowIfNull(allowedChars);
        return input.All(allowedChars.Contains);
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
using StringUtilities.Core.Extensions;
using StringUtilities.Core.Validators;

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

// Advanced template with formatting
var scoreDisplay = "Score: {Score:N0} - Time: {Time:mm\\:ss}".ProcessAdvanced(new
{
    Score = 123456,
    Time = TimeSpan.FromMinutes(5).Add(TimeSpan.FromSeconds(23))
});

Console.WriteLine(scoreDisplay);

// Validation chaining
string userInput = "PlayerName123";
var isValidInput = userInput
    .HasMinLength(3) &&
    userInput.HasMaxLength(16) &&
    userInput.IsValidGameTag();

Console.WriteLine($"Input '{userInput}' is valid: {isValidInput}");

// Complex chaining example
var result = "hello world"
    .ToPascalCase()
    .Wrap("[", "]")
    .Repeat(3, " | ");

Console.WriteLine(result);
```

## Advanced Extension Patterns

### Generic Extension Methods

```csharp
namespace StringUtilities.Core.Extensions;

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
namespace StringUtilities.Core.Extensions;

public static class ConditionalExtensions
{
    public static string If(this string input, bool condition, Func<string, string> transform)
    {
        return condition ? transform(input) : input;
    }

    public static string IfNotEmpty(this string input, Func<string, string> transform)
    {
        return string.IsNullOrEmpty(input) ? input : transform(input);
    }

    // Usage: text.If(isHighlighted, s => s.Bold().WithColor("#FFFF00"))
}
```

## Example Usage

```csharp
// Fluent API example
string result = "hello world"
    .ToPascalCase()
    .Wrap("[", "]")
    .Repeat(3, " | ");
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

## 📚 Essential Documentation

**Master extension methods by studying Microsoft's guidance:**

### Extension Methods Fundamentals

- [Extension Methods Guide](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) - Core concepts and syntax
- [Extension Methods Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/extension-methods) - When and how to use them
- [this keyword](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/this) - Understanding the extension syntax

### String Processing APIs

- [String Class](https://docs.microsoft.com/en-us/dotnet/api/system.string) - Core string methods you'll extend
- [StringBuilder](https://docs.microsoft.com/en-us/dotnet/api/system.text.stringbuilder) - Efficient string building for complex operations
- [String interpolation](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/tokens/interpolated) - Modern string formatting

## Resources

- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Extension Methods Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)**

## Building on Previous Projects

This project extends concepts from earlier projects:

- **Project 1**: Applies type design and naming conventions
- **Project 2**: Uses enumerable interfaces for string processing
- **New Skills**: Extension methods and fluent API design

---

**Ready to extend the world? Begin with Step 1!** 🔧
