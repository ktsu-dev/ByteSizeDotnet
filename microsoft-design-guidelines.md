# Microsoft .NET Design and Style Guidelines

_Comprehensive reference for building well-designed .NET libraries and applications_

## 🎯 Quick Navigation by Curriculum Project

**Use this guide while working on curriculum projects:**

- **Projects 1-4**: [Naming Conventions](#basic-naming-rules), [Type Design](#type-design-guidelines)
- **Projects 5-8**: [Collection Design](#collection-design), [Async Patterns](#async-await-design)
- **Projects 9-12**: [Interface Design](#interface-design), [Design Patterns](#design-patterns)
- **Projects 13-16**: [Advanced Features](#advanced-net-features), [Performance](#performance-guidelines)
- **Projects 17-20**: [Testing](#testing-guidelines), [Package Design](#package-design)

## 📏 Official Naming Conventions

### Basic Naming Rules

| Element                                     | Convention       | Example                                        | Notes                               |
| ------------------------------------------- | ---------------- | ---------------------------------------------- | ----------------------------------- |
| Assemblies                                  | `PascalCase`     | `System.Collections`, `MyCompany.GameEngine`   | Use nouns or noun phrases           |
| Namespaces                                  | `PascalCase`     | `System.IO`, `Microsoft.Extensions.Logging`    | Company.Technology.Feature          |
| Types (Classes, Structs, Interfaces, Enums) | `PascalCase`     | `StringBuilder`, `IDisposable`, `FileMode`     | Use nouns or noun phrases           |
| Methods                                     | `PascalCase`     | `ToString()`, `GetHashCode()`, `ProcessData()` | Use verbs or verb phrases           |
| Properties                                  | `PascalCase`     | `Count`, `IsEmpty`, `LastModified`             | Use nouns, adjectives, or phrases   |
| Events                                      | `PascalCase`     | `Click`, `DataReceived`, `PropertyChanged`     | Use verbs or verb phrases           |
| Fields (Public/Protected)                   | `PascalCase`     | `Empty`, `MaxValue`                            | Use nouns or noun phrases           |
| Fields (Private)                            | `_camelCase`     | `_items`, `_isDisposed`, `_logger`             | Prefix with underscore              |
| Parameters                                  | `camelCase`      | `fileName`, `count`, `predicate`               | Use descriptive nouns               |
| Local Variables                             | `camelCase`      | `result`, `tempFile`, `userInput`              | Use descriptive nouns               |
| Constants                                   | `PascalCase`     | `MaxLength`, `DefaultTimeout`                  | Use nouns or noun phrases           |
| Type Parameters                             | `T{Description}` | `T`, `TKey`, `TValue`, `TElement`              | Single letter T, or T + description |

### Interface Naming

```csharp
// ✅ DO: Prefix interfaces with 'I'
public interface IDisposable { }
public interface IComparable<T> { }
public interface IServiceProvider { }

// ❌ DON'T: Use other prefixes or no prefix
public interface Disposable { }        // Missing 'I'
public interface CDisposable { }       // Wrong prefix
```

### Generic Type Parameters

```csharp
// ✅ DO: Use descriptive names for multiple parameters
public interface IDictionary<TKey, TValue> { }
public class EventHandler<TSender, TEventArgs> { }

// ✅ DO: Use 'T' for single parameter
public interface IComparable<T> { }
public class List<T> { }

// ✅ DO: Indicate constraints in name when helpful
public interface IRepository<TEntity> where TEntity : class { }
```

## 🎯 Type Design Guidelines

### Class vs Struct Decision Matrix

| Use `class` when:        | Use `struct` when:            |
| ------------------------ | ----------------------------- |
| Object identity matters  | Value equality is natural     |
| Need inheritance         | No inheritance needed         |
| Will be boxed frequently | Want to avoid heap allocation |
| Large data (>16 bytes)   | Small data (≤16 bytes)        |
| Mutable by design        | Immutable by design           |
| Need reference semantics | Want value semantics          |

### Abstract Classes vs Interfaces

```csharp
// ✅ USE Abstract Class when:
// - Need to provide some implementation
// - Want to version more easily
// - Need non-public members
public abstract class StreamWriter
{
    protected Stream _stream;
    public abstract void Write(string value);
    public virtual void Flush() => _stream.Flush(); // Default implementation
}

// ✅ USE Interface when:
// - Pure contract definition
// - Multiple inheritance needed
// - Cross-language compatibility important
public interface IDisposable
{
    void Dispose();
}
```

### Immutability Guidelines

```csharp
// ✅ DO: Make value types immutable
public readonly struct Point
{
    public Point(int x, int y) => (X, Y) = (x, y);
    public int X { get; }
    public int Y { get; }
}

// ✅ DO: Consider immutable reference types for data
public class Person
{
    public Person(string name, int age)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
        Age = age;
    }

    public string Name { get; }
    public int Age { get; }
}

// ✅ DO: Use init-only properties for flexibility
public class Configuration
{
    public string DatabaseUrl { get; init; } = "";
    public int TimeoutSeconds { get; init; } = 30;
}
```

## 🔧 Member Design Guidelines

### Property Design

```csharp
// ✅ DO: Use properties for simple data access
public class Rectangle
{
    public int Width { get; set; }
    public int Height { get; set; }

    // ✅ DO: Use computed properties for derived values
    public int Area => Width * Height;

    // ✅ DO: Validate in setters when needed
    private int _count;
    public int Count
    {
        get => _count;
        set => _count = value >= 0 ? value : throw new ArgumentOutOfRangeException();
    }
}

// ❌ DON'T: Use properties for expensive operations
public class FileService
{
    // ❌ BAD: Property that does I/O
    public string FileContent => File.ReadAllText(FileName);

    // ✅ GOOD: Method that clearly indicates work
    public string ReadFileContent() => File.ReadAllText(FileName);
}
```

### Method Design

```csharp
// ✅ DO: Use clear, descriptive method names
public class UserService
{
    // ✅ GOOD: Action is clear
    public User FindUserById(int id) { }
    public void DeleteUser(int id) { }
    public bool TryParseUser(string input, out User user) { }

    // ❌ BAD: Unclear what method does
    public User Get(int id) { }           // Get what?
    public void Process(int id) { }       // Process how?
    public User DoStuff(string input) { } // What stuff?
}

// ✅ DO: Use Try* pattern for operations that might fail
public static class Parser
{
    public static bool TryParseInt(string input, out int result)
    {
        return int.TryParse(input, out result);
    }

    // Pair with throwing version if needed
    public static int ParseInt(string input)
    {
        if (!TryParseInt(input, out int result))
            throw new FormatException($"Cannot parse '{input}' as integer");
        return result;
    }
}
```

### Event Design

```csharp
// ✅ DO: Use EventHandler<T> pattern
public class Document
{
    public event EventHandler<DocumentChangedEventArgs>? DocumentChanged;

    protected virtual void OnDocumentChanged(DocumentChangedEventArgs e)
    {
        DocumentChanged?.Invoke(this, e);
    }
}

public class DocumentChangedEventArgs : EventArgs
{
    public DocumentChangedEventArgs(string propertyName, object? oldValue, object? newValue)
    {
        PropertyName = propertyName;
        OldValue = oldValue;
        NewValue = newValue;
    }

    public string PropertyName { get; }
    public object? OldValue { get; }
    public object? NewValue { get; }
}

// ✅ DO: Use past tense for events that happened
public event EventHandler? FileOpened;    // After file opens
public event EventHandler? FileClosed;    // After file closes

// ✅ DO: Use present participle for cancellable events
public event EventHandler<CancelEventArgs>? FileOpening;  // Before file opens, can cancel
public event EventHandler<CancelEventArgs>? FileClosing; // Before file closes, can cancel
```

## 🛡️ Error Handling Guidelines

### Exception Design

```csharp
// ✅ DO: Create specific exception types
public class InvalidGameStateException : Exception
{
    public InvalidGameStateException(string message) : base(message) { }
    public InvalidGameStateException(string message, Exception innerException)
        : base(message, innerException) { }
}

// ✅ DO: Validate parameters and throw appropriate exceptions
public class GameEngine
{
    public void LoadGame(string fileName)
    {
        if (fileName == null)
            throw new ArgumentNullException(nameof(fileName));
        if (string.IsNullOrWhiteSpace(fileName))
            throw new ArgumentException("File name cannot be empty", nameof(fileName));
        if (!File.Exists(fileName))
            throw new FileNotFoundException($"Game file not found: {fileName}");
    }
}

// ✅ DO: Use standard exception types when appropriate
// ArgumentNullException     - null parameter
// ArgumentException         - invalid parameter value
// ArgumentOutOfRangeException - parameter outside valid range
// InvalidOperationException - object in wrong state for operation
// NotSupportedException     - operation not supported
// NotImplementedException   - not yet implemented
```

### Nullable Reference Types

```csharp
// ✅ DO: Enable nullable reference types
#nullable enable

public class UserService
{
    // ✅ DO: Be explicit about nullability
    public User? FindUser(string? userName)  // Can return null, can accept null
    {
        if (userName == null) return null;
        // ... search logic
        return foundUser;
    }

    // ✅ DO: Use null-forgiving operator when you know value isn't null
    public void ProcessUser(string userName)
    {
        var user = FindUser(userName);
        if (user != null)
        {
            // Compiler knows user isn't null here
            Console.WriteLine(user.Name);
        }
    }
}
```

## 🚀 Performance Guidelines

### Collection Guidelines

```csharp
// ✅ DO: Return IEnumerable<T> for read-only sequences
public IEnumerable<string> GetActiveUserNames()
{
    return _users.Where(u => u.IsActive).Select(u => u.Name);
}

// ✅ DO: Return IReadOnlyCollection<T> when count matters
public IReadOnlyCollection<User> GetUsers()
{
    return _users.AsReadOnly();
}

// ✅ DO: Use List<T> for modifiable collections
public List<User> GetMutableUsers()
{
    return new List<User>(_users);
}

// ✅ DO: Consider using ArraySegment<T> or Span<T> for performance
public void ProcessData(ReadOnlySpan<byte> data)
{
    // High-performance processing without allocations
}
```

### Async Guidelines

```csharp
// ✅ DO: Use async/await for I/O bound operations
public async Task<string> LoadDataAsync(string fileName)
{
    using var reader = new StreamReader(fileName);
    return await reader.ReadToEndAsync();
}

// ✅ DO: Use ConfigureAwait(false) in library code
public async Task<Data> ProcessDataAsync()
{
    var rawData = await LoadDataAsync("data.txt").ConfigureAwait(false);
    return ParseData(rawData);
}

// ✅ DO: Provide both sync and async versions when appropriate
public string LoadData(string fileName) => File.ReadAllText(fileName);
public Task<string> LoadDataAsync(string fileName) => File.ReadAllTextAsync(fileName);

// ❌ DON'T: Block on async code
public string BadLoadData(string fileName)
{
    return LoadDataAsync(fileName).Result; // Can cause deadlocks!
}

// ✅ DO: Return Task directly when possible
public Task SaveDataAsync(string fileName, string data)
{
    return File.WriteAllTextAsync(fileName, data);
}
```

## 📦 Library Design Guidelines

### API Surface Design

```csharp
// ✅ DO: Keep public surface area minimal
public class Calculator
{
    // ✅ Public interface
    public int Add(int a, int b) => PerformOperation(a, b, (x, y) => x + y);
    public int Subtract(int a, int b) => PerformOperation(a, b, (x, y) => x - y);

    // ✅ Implementation details are private
    private int PerformOperation(int a, int b, Func<int, int, int> operation)
    {
        ValidateInputs(a, b);
        return operation(a, b);
    }

    private void ValidateInputs(int a, int b)
    {
        // Validation logic
    }
}

// ✅ DO: Use dependency injection patterns
public class EmailService
{
    private readonly IEmailProvider _emailProvider;
    private readonly ILogger<EmailService> _logger;

    public EmailService(IEmailProvider emailProvider, ILogger<EmailService> logger)
    {
        _emailProvider = emailProvider ?? throw new ArgumentNullException(nameof(emailProvider));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}
```

### Versioning and Compatibility

```csharp
// ✅ DO: Use [Obsolete] for deprecated APIs
public class LegacyService
{
    [Obsolete("Use ProcessDataAsync instead. This method will be removed in v3.0.")]
    public void ProcessData() { }

    public async Task ProcessDataAsync() { }
}

// ✅ DO: Design for extensibility
public abstract class GameCharacter
{
    public abstract void Move();

    // ✅ Virtual methods allow overriding in derived classes
    public virtual void Attack() { /* default implementation */ }

    // ✅ Sealed methods prevent overriding when behavior must be fixed
    public sealed void Save() { /* critical save logic */ }
}
```

## 🏗️ Framework Design Patterns

### Builder Pattern

```csharp
// ✅ DO: Use builder pattern for complex object construction
public class QueryBuilder
{
    private readonly List<string> _conditions = new();
    private string? _orderBy;
    private int? _limit;

    public QueryBuilder Where(string condition)
    {
        _conditions.Add(condition);
        return this;
    }

    public QueryBuilder OrderBy(string column)
    {
        _orderBy = column;
        return this;
    }

    public QueryBuilder Limit(int count)
    {
        _limit = count;
        return this;
    }

    public string Build()
    {
        var query = "SELECT * FROM table";
        if (_conditions.Any())
            query += " WHERE " + string.Join(" AND ", _conditions);
        if (_orderBy != null)
            query += $" ORDER BY {_orderBy}";
        if (_limit.HasValue)
            query += $" LIMIT {_limit}";
        return query;
    }
}

// Usage:
var sql = new QueryBuilder()
    .Where("age > 18")
    .Where("status = 'active'")
    .OrderBy("name")
    .Limit(100)
    .Build();
```

### Factory Pattern

```csharp
// ✅ DO: Use factory pattern for complex object creation
public static class LoggerFactory
{
    public static ILogger CreateLogger(LogLevel level, string outputPath)
    {
        return level switch
        {
            LogLevel.Console => new ConsoleLogger(),
            LogLevel.File => new FileLogger(outputPath),
            LogLevel.Database => new DatabaseLogger(),
            _ => throw new ArgumentException($"Unsupported log level: {level}")
        };
    }
}

// ✅ DO: Use abstract factory for families of related objects
public abstract class UIFactory
{
    public abstract IButton CreateButton();
    public abstract ITextBox CreateTextBox();
}

public class WindowsUIFactory : UIFactory
{
    public override IButton CreateButton() => new WindowsButton();
    public override ITextBox CreateTextBox() => new WindowsTextBox();
}
```

## 📋 Code Organization Guidelines

### File and Project Structure

```
MyLibrary/
├── src/
│   └── MyLibrary/
│       ├── Models/           # Data models
│       ├── Services/         # Business logic
│       ├── Extensions/       # Extension methods
│       ├── Exceptions/       # Custom exceptions
│       └── Utilities/        # Helper classes
├── tests/
│   └── MyLibrary.Tests/
└── docs/
    └── api/                  # API documentation
```

### Namespace Organization

```csharp
// ✅ DO: Organize by feature, not by type
namespace MyCompany.GameEngine.Physics    // Feature-based
namespace MyCompany.GameEngine.Audio      // Feature-based
namespace MyCompany.GameEngine.Graphics   // Feature-based

// ❌ DON'T: Organize only by type
namespace MyCompany.GameEngine.Classes    // Type-based (avoid)
namespace MyCompany.GameEngine.Interfaces // Type-based (avoid)
namespace MyCompany.GameEngine.Enums      // Type-based (avoid)
```

## 📖 Documentation Guidelines

### XML Documentation

```csharp
/// <summary>
/// Calculates the distance between two points in 2D space.
/// </summary>
/// <param name="point1">The first point.</param>
/// <param name="point2">The second point.</param>
/// <returns>The Euclidean distance between the two points.</returns>
/// <exception cref="ArgumentNullException">
/// Thrown when <paramref name="point1"/> or <paramref name="point2"/> is null.
/// </exception>
/// <example>
/// <code>
/// var p1 = new Point(0, 0);
/// var p2 = new Point(3, 4);
/// double distance = MathUtilities.CalculateDistance(p1, p2);
/// // distance will be 5.0
/// </code>
/// </example>
public static double CalculateDistance(Point point1, Point point2)
{
    if (point1 == null) throw new ArgumentNullException(nameof(point1));
    if (point2 == null) throw new ArgumentNullException(nameof(point2));

    var dx = point2.X - point1.X;
    var dy = point2.Y - point1.Y;
    return Math.Sqrt(dx * dx + dy * dy);
}
```

## ⚡ Common Anti-Patterns to Avoid

### Design Anti-Patterns

```csharp
// ❌ DON'T: Use Hungarian notation
int iCount;              // Bad
string strName;          // Bad
bool bIsValid;           // Bad

// ✅ DO: Use meaningful names
int itemCount;           // Good
string customerName;     // Good
bool isValid;            // Good

// ❌ DON'T: Catch and ignore exceptions
try
{
    riskyOperation();
}
catch
{
    // Silently ignoring - BAD!
}

// ✅ DO: Handle exceptions appropriately
try
{
    riskyOperation();
}
catch (SpecificException ex)
{
    _logger.LogError(ex, "Failed to perform risky operation");
    // Handle or rethrow as appropriate
    throw;
}

// ❌ DON'T: Use magic numbers
if (user.Age > 65) { }   // What's special about 65?

// ✅ DO: Use named constants
private const int RetirementAge = 65;
if (user.Age > RetirementAge) { }

// ❌ DON'T: Create God classes
public class GameManager
{
    // 1000+ lines of code doing everything
    public void ProcessInput() { }
    public void UpdatePhysics() { }
    public void RenderGraphics() { }
    public void PlayAudio() { }
    public void SaveGame() { }
    // ... many more responsibilities
}

// ✅ DO: Use single responsibility principle
public class InputManager { public void ProcessInput() { } }
public class PhysicsEngine { public void UpdatePhysics() { } }
public class Renderer { public void RenderGraphics() { } }
public class AudioEngine { public void PlayAudio() { } }
public class SaveManager { public void SaveGame() { } }
```

## 🔗 Official Microsoft Resources

### Design Guidelines Documents

- [Framework Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/) - Complete official guidelines
- [Naming Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines) - Official naming conventions
- [Type Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/type) - Class, struct, interface design
- [Member Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/member) - Property, method, event design
- [Exception Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/exceptions) - Exception handling patterns

### API Design

- [API Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/usage-guidelines) - Usage guidelines for APIs
- [Dependency Injection Guidelines](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection-guidelines) - DI patterns and practices
- [Library Guidance](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/) - Best practices for .NET libraries
- [Async Guidance](https://docs.microsoft.com/en-us/dotnet/standard/async-in-depth) - Asynchronous programming best practices

### Performance Guidelines

- [Performance Best Practices](https://docs.microsoft.com/en-us/dotnet/framework/performance/performance-tips) - Performance optimization tips
- [Memory Management](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/optimization-for-shared-web-hosting) - GC optimization
- [Span\<T\> and Memory\<T\>](https://docs.microsoft.com/en-us/dotnet/standard/memory-and-spans/memory-t-usage-guidelines) - High-performance memory usage

### Security Guidelines

- [Secure Coding Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/security/secure-coding-guidelines) - Security best practices
- [Security Transparent Code](https://docs.microsoft.com/en-us/dotnet/framework/misc/security-transparent-code) - Security model understanding

### Testing Guidelines

- [Unit Testing Best Practices](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-best-practices) - Testing patterns and practices
- [Integration Testing](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests) - Integration testing approaches

### Package and Deployment

- [NuGet Package Guidelines](https://docs.microsoft.com/en-us/nuget/create-packages/package-authoring-best-practices) - Package creation best practices
- [.NET Standard Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/net-standard) - Multi-targeting guidance
- [Versioning Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/library-guidance/versioning) - Semantic versioning for .NET

### Code Analysis

- [Code Analysis Rules](https://docs.microsoft.com/en-us/dotnet/fundamentals/code-analysis/overview) - Built-in analyzers and rules
- [EditorConfig](https://docs.microsoft.com/en-us/dotnet/fundamentals/code-analysis/code-style-rule-options) - Code style configuration
- [StyleCop Rules](https://github.com/DotNetAnalyzers/StyleCopAnalyzers) - Additional style analysis

---

_Follow these guidelines to build robust, maintainable, and idiomatic .NET code!_ 🏗️
