# .NET Quick Syntax Reference

_Handy syntax examples and patterns for everyday .NET development_

**🎯 Quick Navigation**

- [Memory & Types](#memory--types) | [Collections](#collections-cheat-sheet) | [LINQ](#linq-quick-reference)
- [Async Patterns](#asyncawait-essentials) | [Functional](#functional-programming-essentials) | [Testing](#testing-patterns)

---

## 🧠 Memory & Types

### Stack vs Heap

**Stack** - Fast, automatic cleanup, small data:

```csharp
public void ExampleMethod()
{
    int number = 42;                    // Stack
    bool flag = true;                   // Stack
    var point = new Point(1, 2);        // Point struct on stack
    // Automatic cleanup when method exits
}
```

**Heap** - Larger capacity, garbage collected:

```csharp
public void ExampleMethod()
{
    var list = new List<int>();         // List object on heap
    var player = new Player();          // Player object on heap
    string name = "Alice";              // String on heap
    // Variables are references (on stack) pointing to heap objects
}
```

### Value vs Reference Semantics

**Value Semantics** - Copying behavior:

```csharp
int a = 5;
int b = a;  // b gets COPY of a's value
a = 10;     // a changes, b remains 5

var p1 = new Point(1, 2);   // Point is a struct
var p2 = p1;                // p2 gets COPY of p1's data
p1.X = 10;                  // p1 changes, p2 unaffected
```

**Reference Semantics** - Sharing behavior:

```csharp
var list1 = new List<int> { 1, 2, 3 };
var list2 = list1;          // list2 points to SAME object
list1.Add(4);               // Changes visible through list2

var player1 = new Player { Name = "Alice" };
var player2 = player1;      // Same object
player1.Name = "Bob";       // Changes player2.Name too
```

---

## 📚 Collections Cheat Sheet

```csharp
// Arrays - fixed size
int[] numbers = new int[5];                 // Empty array
int[] values = { 1, 2, 3, 4, 5 };          // Initialized array

// Lists - dynamic size
var list = new List<int> { 1, 2, 3 };       // Collection initializer
list.Add(4);                                // Add items
list.AddRange(new[] { 5, 6, 7 });          // Add multiple

// Dictionaries - key-value pairs
var dict = new Dictionary<string, int>
{
    ["one"] = 1,                            // Index initializer
    ["two"] = 2
};
dict.TryGetValue("one", out int value);     // Safe access

// Read-only collections - safe exposure
private readonly List<int> _values = new();
public IReadOnlyList<int> Values => _values;
```

---

## 🔄 LINQ Quick Reference

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Filtering
var evens = numbers.Where(n => n % 2 == 0);
var filtered = numbers.Where(IsEven);       // Method group

// Transformation
var doubled = numbers.Select(n => n * 2);
var strings = numbers.Select(n => n.ToString());

// Aggregation
var sum = numbers.Sum();
var max = numbers.Max();
var count = numbers.Count(n => n > 5);

// Chaining operations
var result = numbers
    .Where(n => n % 2 == 0)                 // Filter evens
    .Select(n => n * 2)                     // Double them
    .OrderByDescending(n => n)              // Sort descending
    .ToList();                              // Execute immediately
```

---

## ⚡ Async/Await Essentials

```csharp
// Async method signatures
public async Task<string> LoadDataAsync()
{
    var data = await File.ReadAllTextAsync("file.txt");
    return data.ToUpper();
}

// Multiple operations - parallel
var task1 = LoadDataAsync();
var task2 = LoadOtherDataAsync();
await Task.WhenAll(task1, task2);

// ConfigureAwait(false) in libraries
var data = await File.ReadAllTextAsync("file.txt").ConfigureAwait(false);
```

---

## 🎯 Functional Programming Essentials

```csharp
// Lambda expressions
Func<int, int> square = x => x * x;
Action<string> log = message => Console.WriteLine(message);
Predicate<int> isPositive = n => n > 0;

// Method groups
Func<string, int> parser = int.Parse;
Action<string> logger = Console.WriteLine;

// LINQ with lambdas
var evens = numbers.Where(n => n % 2 == 0);
var names = people.Select(p => p.Name);
```

---

## 📦 Records & Primary Constructors

```csharp
// Records (C# 9+)
public record Player(string Name, int Score);

var player = new Player("Alice", 100);
var updated = player with { Score = 150 };  // Immutable update

// Primary constructors (C# 12+)
public class DiceRoller(Random random)
{
    public int Roll(int sides) => random.Next(1, sides + 1);
}
```

---

## 🔧 Common Patterns

### Null Safety

```csharp
string? name = GetName();
var length = name?.Length ?? 0;     // Null-conditional
name ??= "Default";                 // Null-coalescing assignment
```

### String Interpolation

```csharp
var message = $"Rolled {count}d{sides} = {total}";
```

### Extension Methods

```csharp
public static class StringExtensions
{
    public static bool IsValidEmail(this string email)
    {
        return email.Contains("@") && email.Contains(".");
    }
}

// Usage: "user@example.com".IsValidEmail();
```

---

## 🧪 Testing Patterns

```csharp
[Fact]
public void DiceRoller_Roll_ShouldReturnValidResult()
{
    // Arrange
    var roller = new DiceRoller();

    // Act
    var result = roller.Roll(6);

    // Assert
    Assert.InRange(result, 1, 6);
}
```

---

## 🔗 Related Resources

- **[Language Essentials](../language-essentials.md)** - Core concepts with explanations
- **[Microsoft Design Guidelines](../microsoft-design-guidelines.md)** - Complete design reference
- **[Glossary](glossary.md)** - .NET terminology explained
- **[Language Comparison](language-comparison.md)** - How .NET maps to other languages
