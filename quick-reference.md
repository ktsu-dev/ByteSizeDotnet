# .NET Quick Reference Guide

_Handy reference for concepts covered in this curriculum_

## 📝 Naming Conventions

| Element                                  | Convention   | Example                                       |
| ---------------------------------------- | ------------ | --------------------------------------------- |
| Classes, Interfaces, Methods, Properties | `PascalCase` | `DiceRoller`, `IDice`, `Roll()`, `LastResult` |
| Parameters, Local Variables              | `camelCase`  | `diceCount`, `modifier`, `result`             |
| Private Fields                           | `_camelCase` | `_random`, `_history`, `_lastRoll`            |
| Constants                                | `PascalCase` | `MaxSides`, `DefaultCount`                    |
| Namespaces                               | `PascalCase` | `MyCompany.GameLibraries`                     |

## 🌍 Language Translation Guide

Understanding how .NET terms map to generic programming concepts and other languages:

### Built-in Types

| .NET Term | Generic Concept | What it means                            |
| --------- | --------------- | ---------------------------------------- |
| `object`  | Base Type       | Root type that all types inherit from    |
| `string`  | Text            | Immutable sequence of Unicode characters |
| `int`     | Integer         | 32-bit signed integer                    |
| `bool`    | Boolean         | True/false value                         |
| `double`  | Float           | Double-precision floating point          |
| `decimal` | Precise Decimal | High-precision decimal for money/finance |
| `char`    | Character       | Single Unicode character                 |
| `byte`    | Byte            | 8-bit unsigned integer                   |

**Language Equivalents:**

- **Java**: `Object`, `String`, `int`, `boolean`, `double`, `BigDecimal`, `char`, `byte`
- **Python**: `object`, `str`, `int`, `bool`, `float`, `decimal.Decimal`, (no char), `bytes`
- **JavaScript**: `Object`, `string`, `number`, `boolean`, `number`, (no decimal), (no char), `number`
- **C++**: (no universal base), `std::string`, `int`, `bool`, `double`, (no built-in), `char`, `unsigned char`
- **Go**: `interface{}`, `string`, `int`, `bool`, `float64`, (no built-in), `rune`, `byte`
- **Rust**: (no universal base), `String`, `i32`, `bool`, `f64`, (external crate), `char`, `u8`

### Core Type System

| .NET Term   | Generic Concept   | What it means                                   |
| ----------- | ----------------- | ----------------------------------------------- |
| `class`     | Reference Type    | Object stored on heap, passed by reference      |
| `struct`    | Value Type        | Data stored on stack/inline, passed by copy     |
| `record`    | Immutable Data    | Immutable reference type with value semantics   |
| `interface` | Contract/Protocol | Defines behavior without implementation         |
| `enum`      | Enumeration       | Named constants representing discrete values    |
| `tuple`     | Product Type      | Lightweight data structure with multiple values |

**Language Equivalents:**

- **Java**: `class`, (no structs), `record` (Java 14+), `interface`, `enum`, (no built-in tuples)
- **Python**: `class`, `dataclass`, `dataclass(frozen=True)`, `Protocol` (3.8+), `Enum`, `tuple`
- **JavaScript**: `class`, (no structs), `Object.freeze()`, `interface` (TS), (no enums), `Array` (as tuple)
- **C++**: `class`, `struct` (but reference type), `const class`, `pure virtual class`, `enum class`, `std::tuple`
- **Go**: `struct` (no classes), `struct`, (manual immutability), `interface`, `const` blocks, (no built-in)
- **Rust**: `struct`, `struct`, `struct` (immutable by default), `trait`, `enum`, `tuple struct`

### Methods and Functions

| .NET Term   | Generic Concept   | What it means                   |
| ----------- | ----------------- | ------------------------------- |
| `method`    | Member Function   | Function belonging to a type    |
| `property`  | Getter/Setter     | Unified syntax for field access |
| `field`     | Instance Variable | Data stored in object instances |
| `parameter` | Function Argument | Input values passed to methods  |

**Language Equivalents:**

- **Java**: `method`, `getter/setter methods`, `field`, `parameter`
- **Python**: `method`, `@property decorator`, `attribute`, `parameter`
- **JavaScript**: `method`, `get/set keywords`, `property`, `parameter`
- **C++**: `member function`, `get/set methods`, `member variable`, `parameter`
- **Go**: `method`, (no built-in), `field`, `parameter`
- **Rust**: `method`, (no built-in), `field`, `parameter`

### Containers and Collections

| .NET Term         | Generic Concept | What it means                           |
| ----------------- | --------------- | --------------------------------------- |
| `T[]`             | Array           | Fixed-size sequence with indexed access |
| `List<T>`         | Dynamic Array   | Resizable array with indexed access     |
| `Dictionary<K,V>` | Hash Map        | Key-value pairs with fast lookup        |
| `HashSet<T>`      | Set             | Unique elements with fast membership    |
| `Queue<T>`        | FIFO Queue      | First-in, first-out data structure      |
| `Stack<T>`        | LIFO Stack      | Last-in, first-out data structure       |
| `IEnumerable<T>`  | Iterable        | Sequence that can be iterated over      |
| `Span<T>`         | Memory View     | Efficient slice over contiguous memory  |

**Language Equivalents:**

- **Java**: `T[]`, `ArrayList<T>`, `HashMap<K,V>`, `HashSet<T>`, `Queue<T>`, `Stack<T>`, `Iterable<T>`, (no equivalent)
- **Python**: `list`, `list`, `dict`, `set`, `collections.deque`, `list` (as stack), `Iterable`, `memoryview`
- **JavaScript**: `Array`, `Array`, `Map`, `Set`, `Array` (shift/unshift), `Array` (push/pop), `Iterable`, (no equivalent)
- **C++**: `T[]`, `std::vector<T>`, `std::unordered_map<K,V>`, `std::unordered_set<T>`, `std::queue<T>`, `std::stack<T>`, (STL ranges), `std::span<T>`
- **Go**: `[N]T`, `[]T` (slice), `map[K]V`, (no built-in set), (channel/slice), (slice), (range over), (slice)
- **Rust**: `[T; N]`, `Vec<T>`, `HashMap<K,V>`, `HashSet<T>`, `VecDeque<T>`, `Vec<T>` (as stack), `Iterator`, `&[T]`

### Code Organization

| .NET Term   | Generic Concept | What it means                     |
| ----------- | --------------- | --------------------------------- |
| `namespace` | Module/Package  | Logical grouping of related types |
| `using`     | Import/Include  | Bringing external code into scope |
| `assembly`  | Library/Package | Compiled unit of deployment       |

**Language Equivalents:**

- **Java**: `package`, `import`, JAR files
- **Python**: `module`, `import`, `pip packages`
- **JavaScript**: `module`, `import`, `npm packages`
- **C++**: `namespace`, `#include`, static/dynamic libraries
- **Go**: `package`, `import`, Go modules
- **Rust**: `module`, `use`, `crates`

### Advanced Features

| .NET Term     | Generic Concept          | What it means                             |
| ------------- | ------------------------ | ----------------------------------------- |
| `generic`     | Template/Parametric      | Type-safe code reuse with type parameters |
| `lambda`      | Anonymous Function       | Inline function expressions (x => x \* 2) |
| `delegate`    | Function Pointer         | Type-safe function references             |
| `event`       | Observer Pattern         | Decoupled notification system             |
| `async/await` | Asynchronous Programming | Non-blocking concurrent execution         |
| `LINQ`        | Query/Stream API         | SQL-like data querying in code            |

**Language Equivalents:**

- **Java**: `Generic<T>`, `lambda expressions`, `Functional Interface`, `Observable`, `CompletableFuture`, `Stream API`
- **Python**: `TypeVar`, `lambda`, `Callable`, (manual), `async/await`, `list comprehension`
- **JavaScript**: `Generic<T>` (TS), `arrow functions`, `Function`, `EventEmitter`, `async/await`, `Array methods`
- **C++**: `template<typename T>`, `lambda expressions`, `std::function`, (manual), `std::future`, `ranges` (C++20)
- **Go**: `type parameter`, `func literals`, `func type`, `channels`, `goroutine`, (manual loops)
- **Rust**: `generic`, `closures`, `Fn trait`, (manual), `async/await`, `Iterator`

### Memory and Reference Semantics

| .NET Term  | Generic Concept     | What it means                           |
| ---------- | ------------------- | --------------------------------------- |
| `ref`      | Reference Parameter | Pass by reference (can modify original) |
| `out`      | Output Parameter    | Method returns multiple values          |
| `nullable` | Optional Type       | Values that might not exist             |
| `using`    | Resource Management | Automatic cleanup of resources          |

**Language Equivalents:**

- **Java**: (pass by reference), (no out), `Optional<T>`, `try-with-resources`
- **Python**: (all references), (return tuple), `Optional`, `with` statement
- **JavaScript**: (all references), (return object), `T | null`, (manual cleanup)
- **C++**: `T&`, `T*`, `std::optional<T>`, `RAII/destructors`
- **Go**: `*T` (pointer), (return multiple), `*T` (pointer), `defer`
- **Rust**: `&mut T`, (return tuple), `Option<T>`, `Drop` trait

### Key Differences to Note:

**Properties** are a .NET-specific concept that combines getters/setters into a unified syntax. Most languages require explicit getter/setter methods.

**Object Type** in .NET serves as the universal base type, similar to Java's `Object` but unlike C++ which has no universal base type. This enables polymorphism and reflection across all types.

**String Immutability** in .NET means strings cannot be modified after creation, similar to Java and Python, but unlike C++ where `std::string` is mutable.

**Tuples** provide lightweight data structures for multiple values. .NET has both `Tuple<T1,T2>` (reference type) and `ValueTuple<T1,T2>` (value type), while languages like Python have built-in tuple support.

**Records** (C# 9+) provide immutable data types with value-based equality. Java added similar `record` types in Java 14+, while other languages achieve this through conventions or libraries.

**Structs** in .NET are value types (stack-allocated), while in C++ they're just classes with public default access. C++ structs can have constructors, destructors, and virtual functions - they're essentially classes.

**Arrays vs Collections**: .NET distinguishes between fixed arrays (`T[]`) and dynamic collections (`List<T>`), similar to Java but unlike JavaScript where `Array` is always dynamic.

**Span<T>** provides zero-allocation slicing over memory, similar to Rust's slices (`&[T]`) or C++20's `std::span`, but unique among managed languages.

**Namespaces** in .NET are more like Java packages than C++ namespaces. C++ namespaces are compile-time constructs that don't affect file organization, while .NET namespaces typically mirror folder structure.

**Memory Management** differs significantly: .NET uses garbage collection, while C++ uses manual memory management or RAII (Resource Acquisition Is Initialization) with destructors.

**Templates vs Generics**: C++ templates are compile-time code generation (can specialize for different types), while .NET generics maintain type information at runtime and have more restrictions but better runtime support.

**Virtual Functions**: C++ requires explicit `virtual` keyword and uses vtables, while .NET methods are virtual by default in most languages (Java, C#) unless explicitly sealed/final.

**Events** in .NET provide built-in observer pattern support, while C++ typically uses function pointers, callbacks, or signals/slots (Qt).

**LINQ** is a unique .NET feature that brings SQL-like queries to objects. C++20 introduced ranges which provide similar functionality but with different syntax.

**Resource Management**: .NET's `using` statements provide automatic disposal, while C++ relies on RAII and destructors for automatic resource cleanup.

## 🧠 Memory: Stack vs Heap

Understanding where data lives in memory is fundamental to .NET programming.

### The Stack

- **Purpose**: Stores method parameters, local variables, and return addresses
- **Characteristics**:
  - Fast allocation/deallocation (just move a pointer)
  - Automatic cleanup when method exits
  - Limited size (~1MB typically)
  - LIFO (Last In, First Out) structure
- **What goes here**: Value types (when declared as locals), method parameters, return addresses

```csharp
public void ExampleMethod()
{
    int number = 42;        // Stored on stack
    bool flag = true;       // Stored on stack
    var point = new Point(1, 2); // Point struct stored on stack

    // When method exits, all stack data is automatically cleaned up
}
```

### The Heap

- **Purpose**: Stores objects that need to persist beyond method scope
- **Characteristics**:
  - Slower allocation (find suitable memory block)
  - Managed by Garbage Collector for cleanup
  - Much larger than stack (limited by available RAM)
  - Random access to memory locations
- **What goes here**: Reference type objects (`class`, arrays, `string`)

```csharp
public void ExampleMethod()
{
    var list = new List<int>();     // List object stored on heap
    var player = new Player();      // Player object stored on heap
    string name = "Alice";          // String stored on heap

    // Variables 'list', 'player', 'name' are references stored on stack
    // They point to objects stored on heap
}
```

### Mixed Scenarios

```csharp
public class Container
{
    public int Value;           // Stored inline within Container object on heap
    public List<int> Numbers;   // Reference stored in Container, List object separately on heap
}

public void WorkWithData()
{
    var container = new Container();  // Container object on heap
    container.Value = 42;             // Sets value within heap object
    container.Numbers = new List<int>(); // New List object on heap
}
```

### Why This Matters

- **Performance**: Stack operations are faster than heap operations
- **Memory Management**: Stack is automatically managed, heap requires garbage collection
- **Lifetime**: Stack variables die with method scope, heap objects live until garbage collected
- **Size Limitations**: Large objects must go on heap due to stack size limits

## 🔄 Value vs Reference Semantics

Understanding the difference between value and reference semantics is crucial for effective C# programming.

### Value Semantics

- **Copy behavior**: Assignment copies the actual data
- **Memory**: Stored on the stack (for locals) or inline in containing objects
- **Equality**: Two objects are equal if their data is the same
- **Examples**: `int`, `bool`, `struct`, `enum`

```csharp
int a = 5;
int b = a;  // b gets a COPY of a's value
a = 10;     // a changes, but b remains 5
Console.WriteLine(b); // Output: 5

// Structs have value semantics
public struct Point
{
    public int X, Y;
    public Point(int x, int y) => (X, Y) = (x, y);
}

var p1 = new Point(1, 2);
var p2 = p1;  // p2 gets a COPY of p1's data
p1.X = 10;    // p1 changes, but p2 is unaffected
Console.WriteLine(p2.X); // Output: 1
```

### Reference Semantics

- **Copy behavior**: Assignment copies the reference (memory address/pointer), not the data
- **Memory**: Object stored on the heap, reference stored on stack
- **Equality**: Two references are equal if they point to the same object
- **Examples**: `class`, `interface`, `delegate`, arrays, `string`\*

```csharp
var list1 = new List<int> { 1, 2, 3 };
var list2 = list1;  // list2 points to the SAME object as list1
list1.Add(4);       // Adding to list1 also affects list2
Console.WriteLine(list2.Count); // Output: 4

// Classes have reference semantics
public class Player
{
    public string Name { get; set; }
    public int Score { get; set; }
}

var player1 = new Player { Name = "Alice", Score = 100 };
var player2 = player1;  // player2 points to the SAME object
player1.Score = 200;    // Changes both player1.Score and player2.Score
Console.WriteLine(player2.Score); // Output: 200
```

### Key Implications

**Performance**:

- Value types: Fast access, but expensive to copy if large
- Reference types: Cheap to copy reference, but heap allocation overhead

**Mutability**:

- Value types: Changes to copies don't affect original
- Reference types: Changes through any reference affect the shared object

**Null Safety**:

- Value types: Cannot be null (unless nullable: `int?`)
- Reference types: Can be null (unless non-nullable reference types enabled)

\*Note: `string` is technically a reference type but behaves like a value type due to immutability.

## 🏗️ Type System Quick Decisions

### When to use `struct` vs `class` vs `record`?

**Use `struct` when:**

- Small data (< 16 bytes typically)
- Immutable (readonly)
- Value semantics (copying makes sense)
- No inheritance needed
- Examples: `Point`, `Color`, `DiceResult`

**Use `class` when:**

- Mutable state
- Reference semantics (sharing makes sense)
- Inheritance needed
- Complex behavior
- Examples: `DiceRoller`, `GameEngine`, `Player`

**Use `record` when:**

- Immutable data transfer objects
- Value-based equality needed
- Simple data containers
- API models or DTOs
- Examples: `PlayerScore`, `GameAction`, `Configuration`

### Properties vs Fields vs Methods

```csharp
// ❌ Public field - avoid
public int Value;

// ✅ Auto-property - simple data
public int Value { get; set; }
public int Value { get; } // readonly

// ✅ Computed property - derived data
public bool IsEven => Value % 2 == 0;

// ✅ Property with backing field - validation/logic
private int _value;
public int Value
{
    get => _value;
    set => _value = value < 0 ? 0 : value;
}

// ✅ Method - actions or complex operations
public int CalculateTotal() => /* complex logic */;
```

## 🔧 Common Patterns

### Constructor Validation

```csharp
public Die(int sides)
{
    ArgumentOutOfRangeException.ThrowIfNegativeOrZero(sides);
    Sides = sides;
}
```

### Null Safety

```csharp
// Nullable reference types
public string? Name { get; set; }

// Null-conditional operators
var length = name?.Length ?? 0;
var first = collection?.FirstOrDefault();

// Null-coalescing assignment
name ??= "Default";
```

### String Interpolation

```csharp
// ✅ Preferred
var message = $"Rolled {count}d{sides} = {total}";

// ❌ Avoid
var message = "Rolled " + count + "d" + sides + " = " + total;
```

## 📚 Collections Cheat Sheet

```csharp
// Arrays - fixed size
int[] numbers = new int[5];
int[] values = { 1, 2, 3, 4, 5 };

// Lists - dynamic size
var list = new List<int> { 1, 2, 3 };
list.Add(4);

// Dictionaries - key-value pairs
var dict = new Dictionary<string, int>
{
    ["one"] = 1,
    ["two"] = 2
};

// Read-only collections - expose internal collections safely
private readonly List<int> _values = new();
public IReadOnlyList<int> Values => _values;
```

## 🔄 LINQ Quick Reference

```csharp
var numbers = new[] { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Filtering with predicates
var evens = numbers.Where(n => n % 2 == 0);
var filtered = numbers.Where(IsEven); // Method group as predicate

// Transformation with selectors
var doubled = numbers.Select(n => n * 2);
var strings = numbers.Select(n => n.ToString()); // Type conversion

// Aggregation
var sum = numbers.Sum();
var max = numbers.Max();
var average = numbers.Average();
var customAgg = numbers.Aggregate(0, (acc, n) => acc + n * n); // Custom aggregation

// Combination
var evenDoubled = numbers
    .Where(n => n % 2 == 0)
    .Select(n => n * 2)
    .ToList(); // Execute immediately

// Grouping
var byRemainder = numbers.GroupBy(n => n % 3);

// Method groups and delegates
Func<int, bool> isEven = n => n % 2 == 0;
Predicate<int> isPositive = n => n > 0;
var filtered2 = numbers.Where(isEven);

static bool IsEven(int n) => n % 2 == 0; // Method group usage
```

## 📦 Records & Primary Constructors

### Records (C# 9+)

Records provide immutable reference types with value semantics and built-in functionality:

```csharp
// Basic record with positional parameters
public record Player(string Name, int Score, DateTime JoinDate);

// Usage - automatic constructor, properties, ToString(), Equals(), GetHashCode()
var player = new Player("Alice", 100, DateTime.Now);
Console.WriteLine(player); // Player { Name = Alice, Score = 100, JoinDate = ... }

// Immutable updates with 'with' expression
var updatedPlayer = player with { Score = 150 };

// Value-based equality (not reference equality)
var player1 = new Player("Bob", 200, DateTime.Today);
var player2 = new Player("Bob", 200, DateTime.Today);
Console.WriteLine(player1 == player2); // True - same values

// Deconstruction
var (name, score, joinDate) = player;
```

### Record Classes vs Record Structs

```csharp
// Record class (reference type) - C# 9+
public record class GameSession(string Id, DateTime StartTime, List<string> Players);

// Record struct (value type) - C# 10+
public record struct Position(int X, int Y, int Z);

// Readonly record struct (immutable value type)
public readonly record struct Color(byte Red, byte Green, byte Blue);
```

### Records with Custom Members

```csharp
public record GameAction(ActionType Type, string PlayerId, DateTime Timestamp)
{
    // Custom properties
    public string Description => $"{PlayerId} performed {Type}";

    // Custom methods
    public bool IsExpired(TimeSpan maxAge) => DateTime.UtcNow - Timestamp > maxAge;

    // Custom constructor (calls primary constructor)
    public GameAction(ActionType type, string playerId) : this(type, playerId, DateTime.UtcNow) { }

    // Property with init-only setter
    public Dictionary<string, object> Data { get; init; } = new();
}
```

### Primary Constructors (C# 12+)

Primary constructors work with classes and structs, not just records:

```csharp
// Class with primary constructor
public class DiceRoller(Random random, ILogger logger)
{
    // Parameters are available throughout the class
    private readonly List<int> _history = new();

    public int Roll(int sides)
    {
        var result = random.Next(1, sides + 1);
        logger.LogInformation("Rolled {Result} on d{Sides}", result, sides);
        _history.Add(result);
        return result;
    }

    // Parameters accessible in all members
    public void Reset() => logger.LogInformation("Resetting dice roller");
}

// Struct with primary constructor
public readonly struct Die(int sides)
{
    // Automatic property from parameter
    public int Sides => sides;

    // Methods can use primary constructor parameters
    public int Roll(Random random) => random.Next(1, sides + 1);

    public bool IsValid => sides > 0;

    // ToString using parameter
    public override string ToString() => $"d{sides}";
}

// Usage
var roller = new DiceRoller(new Random(), logger);
var d20 = new Die(20);
```

### Advanced Record Patterns

```csharp
// Record inheritance
public abstract record GameEvent(DateTime Timestamp, string PlayerId);
public record PlayerJoined(DateTime Timestamp, string PlayerId, string PlayerName)
    : GameEvent(Timestamp, PlayerId);
public record PlayerLeft(DateTime Timestamp, string PlayerId, string Reason)
    : GameEvent(Timestamp, PlayerId);

// Pattern matching with records
public string ProcessEvent(GameEvent evt) => evt switch
{
    PlayerJoined(_, var playerId, var name) => $"Welcome {name}!",
    PlayerLeft(_, var playerId, var reason) => $"Goodbye {playerId}: {reason}",
    _ => "Unknown event"
};

// Nested records
public record GameState(
    Player CurrentPlayer,
    ImmutableList<Player> AllPlayers,
    GameBoard Board
)
{
    public record Player(string Id, string Name, int Score);
    public record GameBoard(int Width, int Height, ImmutableArray<Cell> Cells);
}
```

### When to Choose Each Type

| Type                              | Use Case                             | Example                       |
| --------------------------------- | ------------------------------------ | ----------------------------- |
| `record`                          | Immutable data, DTOs, configuration  | `PlayerScore`, `ApiResponse`  |
| `record struct`                   | Small immutable values               | `Position`, `Color`, `Money`  |
| `class` with primary constructor  | Services, controllers, complex logic | `GameEngine`, `PlayerService` |
| `struct` with primary constructor | Small value types with behavior      | `Die`, `CardValue`            |

## 🎯 Functional Programming Essentials

```csharp
// Lambda expressions (anonymous functions)
Func<int, int> square = x => x * x;
Action<string> log = message => Console.WriteLine(message);
Predicate<int> isPositive = n => n > 0;

// Method groups (converting methods to delegates)
Func<string, int> parser = int.Parse; // Method group
Action<string> logger = Console.WriteLine; // Method group

// Higher-order functions (functions that take other functions)
public static IEnumerable<T> Filter<T>(IEnumerable<T> source, Func<T, bool> predicate)
{
    foreach (var item in source)
        if (predicate(item))
            yield return item;
}

// Callbacks and event handlers
public event Action<string> OnDataReceived;
public void RegisterCallback(Action<string> callback) => OnDataReceived += callback;

// Functional composition
Func<int, int> addOne = x => x + 1;
Func<int, int> multiplyTwo = x => x * 2;
Func<int, int> composed = x => multiplyTwo(addOne(x)); // Composition

// Partial application with closures
public static Func<int, int> CreateMultiplier(int factor)
{
    return x => x * factor; // Closure captures 'factor'
}
var multiplyBy5 = CreateMultiplier(5);

// Selectors (projection functions)
Func<Person, string> nameSelector = p => p.Name;
Func<Person, int> ageSelector = p => p.Age;
var names = people.Select(nameSelector);

// Built-in delegate types
Action action = () => Console.WriteLine("No parameters, no return");
Action<int> actionWithParam = x => Console.WriteLine(x);
Func<int> funcNoParams = () => 42;
Func<int, string> funcWithParam = x => x.ToString();
Predicate<string> predicate = s => !string.IsNullOrEmpty(s);

// Event-driven programming
public class EventPublisher
{
    public event Action<string> MessageReceived;
    public event Func<string, bool> MessageValidating; // Can return values

    protected virtual void OnMessageReceived(string message)
    {
        MessageReceived?.Invoke(message); // Safe invocation
    }
}
```

## ⚡ Async/Await Essentials

```csharp
// Async method signature
public async Task<string> LoadDataAsync()
{
    // Await other async operations
    var data = await File.ReadAllTextAsync("file.txt");
    return data.ToUpper();
}

// Calling async methods
var result = await LoadDataAsync();

// Multiple async operations
var task1 = LoadDataAsync();
var task2 = LoadOtherDataAsync();
await Task.WhenAll(task1, task2);

// ConfigureAwait(false) in libraries
var data = await File.ReadAllTextAsync("file.txt").ConfigureAwait(false);
```

## 🎯 Extension Methods

```csharp
// Extension method definition
public static class StringExtensions
{
    public static bool IsNullOrWhiteSpace(this string? value)
    {
        return string.IsNullOrWhiteSpace(value);
    }

    public static string Repeat(this string value, int count)
    {
        return string.Concat(Enumerable.Repeat(value, count));
    }
}

// Usage
"hello".IsNullOrWhiteSpace(); // false
"*".Repeat(5); // "*****"
```

## 🏛️ SOLID Principles Quick Check

**S** - Single Responsibility: One reason to change  
**O** - Open/Closed: Open for extension, closed for modification  
**L** - Liskov Substitution: Derived classes must be substitutable  
**I** - Interface Segregation: Many specific interfaces > one general  
**D** - Dependency Inversion: Depend on abstractions, not concretions

## 🧪 Testing Patterns

```csharp
[Fact]
public void DiceRoller_Roll_ShouldReturnValidResult()
{
    // Arrange
    var roller = new DiceRoller();
    var die = new Die(6);

    // Act
    var result = roller.Roll(die);

    // Assert
    Assert.InRange(result.Value, 1, 6);
}

// Mocking with Moq
var mockService = new Mock<IGameService>();
mockService.Setup(s => s.GetPlayer(It.IsAny<int>()))
           .Returns(new Player { Name = "Test" });
```

## 🎨 Fluent API Design

```csharp
// Method chaining
var query = builder
    .Where(x => x.IsActive)
    .OrderBy(x => x.Name)
    .Select(x => x.DisplayName)
    .Build();

// Builder pattern
var config = new GameConfigBuilder()
    .WithPlayer("Alice")
    .WithDifficulty(Difficulty.Hard)
    .WithRules(rules => rules
        .AllowSaveStates()
        .EnableCheats(false))
    .Build();
```

## 📚 Quick Documentation Links

### Language Fundamentals

- [C# Language Reference](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/) - Complete C# syntax guide
- [Naming Conventions](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines) - Official .NET naming guidelines
- [Types](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/types/) - Value types, reference types, built-in types
- [Classes](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/types/classes) - Class declarations, members, inheritance
- [Structs](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/struct) - Value type declarations and usage
- [Interfaces](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/types/interfaces) - Interface declarations and implementation

### Memory Management

- [Memory Management](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/) - Garbage collection and memory concepts
- [Stack vs Heap](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types) - Value types and memory allocation
- [Span<T> and Memory<T>](https://docs.microsoft.com/en-us/dotnet/standard/memory-and-spans/) - High-performance memory management

### Properties and Methods

- [Properties](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties) - Auto-properties, computed properties, accessors
- [Methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods) - Method declarations, parameters, overloading
- [Extension Methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/extension-methods) - Extending existing types

### Collections and LINQ

- [Collections](https://docs.microsoft.com/en-us/dotnet/standard/collections/) - Generic collections overview
- [List<T>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1) - Dynamic arrays
- [Dictionary<TKey,TValue>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2) - Key-value pairs
- [IEnumerable<T>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1) - Enumeration interface
- [LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/) - Language Integrated Query
- [LINQ Methods](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) - Method vs query syntax

### Asynchronous Programming

- [Async/Await](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/) - Asynchronous programming patterns
- [Task<T>](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1) - Task-based asynchronous operations
- [ConfigureAwait](https://docs.microsoft.com/en-us/dotnet/api/system.threading.tasks.task.configureawait) - Context configuration for libraries

### Advanced Features

- [Generics](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/) - Generic types and methods
- [Constraints](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters) - Generic type constraints
- [Delegates](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/) - Function pointers and multicast
- [Events](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/events/) - Event-driven programming
- [Nullable Reference Types](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references) - Null safety

### Functional Programming

- [Lambda Expressions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions) - Anonymous functions and closures
- [Func<T> Delegate](https://docs.microsoft.com/en-us/dotnet/api/system.func-1) - Generic function delegates
- [Action<T> Delegate](https://docs.microsoft.com/en-us/dotnet/api/system.action-1) - Generic action delegates
- [Predicate<T> Delegate](https://docs.microsoft.com/en-us/dotnet/api/system.predicate-1) - Boolean-returning functions
- [Expression Trees](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/expression-trees/) - Code as data structures
- [Method Groups](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/using-delegates#method-groups) - Converting methods to delegates

### Pattern Matching

- [Pattern Matching](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching) - Modern pattern matching syntax
- [Switch Expressions](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression) - Switch expressions and patterns

### Design and Architecture

- [Dependency Injection](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection) - Built-in DI container
- [Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/) - Framework design guidelines
- [SOLID Principles](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/architectural-principles) - Object-oriented design principles

### Testing

- [Unit Testing](https://docs.microsoft.com/en-us/dotnet/core/testing/) - Testing in .NET
- [xUnit](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test) - xUnit testing framework
- [MSTest](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest) - MSTest framework

### Performance and Reflection

- [Reflection](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection) - Runtime type inspection
- [Attributes](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/attributes/) - Metadata and custom attributes
- [Unsafe Code](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code) - Pointers and memory management
- [P/Invoke](https://docs.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke) - Platform Invoke for native interop

### Package Management

- [NuGet](https://docs.microsoft.com/en-us/nuget/) - Package manager documentation
- [Creating Packages](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package) - Building and publishing packages
- [Package References](https://docs.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files) - Managing dependencies

### Project Configuration

- [Project Files](https://docs.microsoft.com/en-us/dotnet/core/project-sdk/overview) - Modern .csproj format
- [Target Frameworks](https://docs.microsoft.com/en-us/dotnet/standard/frameworks) - Framework targeting
- [Global.json](https://docs.microsoft.com/en-us/dotnet/core/tools/global-json) - SDK version selection

### Quick References

- [C# Keywords](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/) - Complete keyword reference
- [Operators](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/) - All C# operators
- [Built-in Types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/built-in-types) - Primitive types reference

---

_Keep this handy while working through the projects!_ 📖
