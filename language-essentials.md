# .NET Language Essentials

_Core concepts you need to know for the curriculum projects_

**💡 This is your starting point!** For complete details, see [Microsoft Design Guidelines](microsoft-design-guidelines.md).

---

## 🏗️ Fundamental Building Blocks

### Types: Value vs Reference

```csharp
// Value types (stack allocated, copied by value)
struct Point { public int X, Y; }           // ✅ Use for small, immutable data
int number = 42;                            // ✅ Built-in value types
bool isActive = true;

// Reference types (heap allocated, copied by reference)
class Player { public string Name; }        // ✅ Use for objects with identity
string text = "Hello";                      // ✅ String is a reference type
```

**When to use which?** → See [Class vs Struct Decision Matrix](microsoft-design-guidelines.md#class-vs-struct-decision-matrix)

### Properties vs Fields vs Methods

```csharp
class GameCharacter
{
    // ❌ Public field - avoid in APIs
    public int health;

    // ✅ Property - preferred for data access
    public int Health { get; set; }

    // ✅ Computed property - derived values
    public bool IsAlive => Health > 0;

    // ✅ Method - for actions and operations
    public void TakeDamage(int amount) => Health -= amount;
}
```

**Full guidelines** → See [Property Design](microsoft-design-guidelines.md#property-design)

---

## 📦 Collections and Generics

### Essential Collection Types

```csharp
// Fixed size arrays
int[] numbers = { 1, 2, 3 };

// Dynamic lists
List<string> names = new() { "Alice", "Bob" };

// Key-value pairs
Dictionary<string, int> scores = new() { ["Alice"] = 100 };

// Unique items
HashSet<string> tags = new() { "action", "rpg" };
```

### Generic Constraints

```csharp
// Constrain to reference types
public class Repository<T> where T : class { }

// Constrain to specific interface
public class Manager<T> where T : IDisposable { }

// Multiple constraints
public class Pool<T> where T : class, new() { }
```

**Full collection guidance** → See [Collection Guidelines](microsoft-design-guidelines.md#collection-guidelines)

---

## 🔄 Modern C# Patterns

### Pattern Matching

```csharp
// Switch expressions (C# 8+)
string GetPlayerStatus(int health) => health switch
{
    > 80 => "Healthy",
    > 50 => "Injured",
    > 0 => "Critical",
    _ => "Dead"
};

// Pattern matching with when
string DescribeItem(object item) => item switch
{
    string s when s.Length > 10 => "Long text",
    string s => $"Text: {s}",
    int n when n < 0 => "Negative number",
    int n => $"Number: {n}",
    _ => "Unknown"
};
```

### Records (C# 9+)

```csharp
// Immutable data with value equality
public record Player(string Name, int Score);

// Usage
var player1 = new Player("Alice", 100);
var player2 = player1 with { Score = 150 };  // Create modified copy
```

**Full pattern guidance** → See [Type Design Guidelines](microsoft-design-guidelines.md#type-design-guidelines)

---

## ⚡ Async Programming

### Basic Async/Await

```csharp
// Async method signature
public async Task<string> LoadDataAsync()
{
    // Use ConfigureAwait(false) in libraries
    var data = await httpClient.GetStringAsync(url).ConfigureAwait(false);
    return ProcessData(data);
}

// Calling async methods
public async Task UseDataAsync()
{
    var data = await LoadDataAsync();
    Console.WriteLine(data);
}
```

### Common Patterns

```csharp
// Return Task for async methods without return value
public async Task SaveDataAsync(string data) { }

// Use Task.Run for CPU-bound work
public Task<int> CalculateAsync(int[] numbers) =>
    Task.Run(() => numbers.Sum(x => x * x));
```

**Complete async guidance** → See [Async Guidelines](microsoft-design-guidelines.md#async-guidelines)

---

## 🎯 LINQ Essentials

### Method Syntax (Preferred)

```csharp
var results = players
    .Where(p => p.Score > 100)
    .OrderByDescending(p => p.Score)
    .Select(p => p.Name)
    .ToList();
```

### Query Syntax (Alternative)

```csharp
var results = (from p in players
               where p.Score > 100
               orderby p.Score descending
               select p.Name).ToList();
```

### Deferred Execution

```csharp
// Query is not executed yet!
var query = players.Where(p => p.IsActive);

// Now it executes
var activePlayersList = query.ToList();
var activePlayersArray = query.ToArray();
```

**LINQ best practices** → See [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)

---

## 🔗 Extension Methods

### Creating Extensions

```csharp
public static class StringExtensions
{
    // 'this' makes it an extension method
    public static bool IsValidEmail(this string email)
    {
        return email.Contains("@") && email.Contains(".");
    }

    // Fluent chaining
    public static string Capitalize(this string text)
    {
        return char.ToUpper(text[0]) + text[1..].ToLower();
    }
}

// Usage
string email = "user@example.com";
bool valid = email.IsValidEmail();           // Extension method
string name = "alice".Capitalize();          // Fluent chaining
```

**Extension method guidelines** → See [Method Design](microsoft-design-guidelines.md#method-design)

---

## 📢 Events and Delegates

### Event Pattern

```csharp
public class GameSession
{
    // Event declaration
    public event EventHandler<PlayerJoinedEventArgs>? PlayerJoined;

    // Event args class
    public class PlayerJoinedEventArgs : EventArgs
    {
        public string PlayerName { get; init; }
    }

    // Trigger event
    protected virtual void OnPlayerJoined(string playerName)
    {
        PlayerJoined?.Invoke(this, new PlayerJoinedEventArgs { PlayerName = playerName });
    }
}
```

### Delegates and Lambdas

```csharp
// Built-in delegate types
Action<string> logger = message => Console.WriteLine(message);
Func<int, int, int> add = (x, y) => x + y;
Predicate<string> isEmpty = text => string.IsNullOrEmpty(text);

// Usage
logger("Hello World");
int result = add(5, 3);
bool empty = isEmpty("");
```

**Event design patterns** → See [Event Design](microsoft-design-guidelines.md#event-design)

---

## 🏛️ Interfaces and Architecture

### Interface Design

```csharp
// ✅ Good interface - focused, cohesive
public interface IRepository<T>
{
    Task<T?> GetByIdAsync(int id);
    Task<IEnumerable<T>> GetAllAsync();
    Task SaveAsync(T entity);
    Task DeleteAsync(int id);
}

// ❌ Avoid large interfaces
public interface IGameService  // Too many responsibilities
{
    void StartGame();
    void SaveGame();
    void LoadGame();
    void ProcessInput();
    void UpdateGraphics();
    void PlaySound();
}
```

### Dependency Injection

```csharp
// Service registration (in Program.cs or Startup)
services.AddScoped<IPlayerRepository, PlayerRepository>();
services.AddSingleton<IGameConfig, GameConfig>();
services.AddTransient<IEmailService, EmailService>();

// Constructor injection
public class PlayerService
{
    private readonly IPlayerRepository _repository;

    public PlayerService(IPlayerRepository repository)
    {
        _repository = repository;
    }
}
```

**Complete architecture guidance** → See [Abstract Classes vs Interfaces](microsoft-design-guidelines.md#abstract-classes-vs-interfaces)

---

## 📛 Naming Quick Reference

| Element                      | Convention   | Example                                   |
| ---------------------------- | ------------ | ----------------------------------------- |
| Classes, Interfaces, Methods | `PascalCase` | `GamePlayer`, `IRepository`, `GetScore()` |
| Parameters, Local Variables  | `camelCase`  | `playerName`, `scoreValue`                |
| Private Fields               | `_camelCase` | `_gameState`, `_playerList`               |
| Constants                    | `PascalCase` | `MaxPlayers`, `DefaultTimeout`            |

**Complete naming rules** → See [Basic Naming Rules](microsoft-design-guidelines.md#basic-naming-rules)

---

## 🔧 Error Handling

### Exception Guidelines

```csharp
// ✅ Throw specific exceptions
public void SetHealth(int health)
{
    if (health < 0)
        throw new ArgumentOutOfRangeException(nameof(health), "Health cannot be negative");
}

// ✅ Try pattern for operations that might fail
public bool TryParsePlayerData(string data, out Player player)
{
    player = null;
    try
    {
        player = JsonSerializer.Deserialize<Player>(data);
        return true;
    }
    catch (JsonException)
    {
        return false;
    }
}
```

**Exception handling best practices** → See [Exception Design](microsoft-design-guidelines.md#exception-design)

---

## 🚀 Next Steps

This covers the essentials you need for the curriculum. For deeper understanding:

1. **📏 [Microsoft Design Guidelines](microsoft-design-guidelines.md)** - Complete reference
2. **📚 [Curriculum Details](curriculum-details.md)** - See how concepts apply to projects
3. **📖 [Reference Directory](reference/)** - Deep-dive when needed
4. **🎯 [Progress Tracker](progress-tracker.md)** - Track your learning

**Ready to start?** Begin with [Project 1: Dice Rolling Library](project-01-dice-roller/) and apply these concepts!
