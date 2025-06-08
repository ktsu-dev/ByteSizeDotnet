# .NET Language Comparison Guide

_Understanding how .NET maps to other programming languages_

**🎯 Quick Navigation**

- [Built-in Types](#built-in-types) | [Type System](#core-type-system) | [Collections](#containers-and-collections)
- [Functions](#methods-and-functions) | [Organization](#code-organization) | [Advanced Features](#advanced-features)

---

## Built-in Types

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

---

## Core Type System

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

---

## Methods and Functions

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

---

## Containers and Collections

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

---

## Code Organization

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

---

## Advanced Features

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

---

## Memory and Reference Semantics

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

---

## Key Differences to Note

### Properties

**Properties** are a .NET-specific concept that combines getters/setters into a unified syntax. Most languages require explicit getter/setter methods.

### Object Type

**Object Type** in .NET serves as the universal base type, similar to Java's `Object` but unlike C++ which has no universal base type. This enables polymorphism and reflection across all types.

### String Immutability

**String Immutability** in .NET means strings cannot be modified after creation, similar to Java and Python, but unlike C++ where `std::string` is mutable.

### Tuples

**Tuples** provide lightweight data structures for multiple values. .NET has both `Tuple<T1,T2>` (reference type) and `ValueTuple<T1,T2>` (value type), while languages like Python have built-in tuple support.

### Records

**Records** (C# 9+) provide immutable data types with value-based equality. Java added similar `record` types in Java 14+, while other languages achieve this through conventions or libraries.

### Structs

**Structs** in .NET are value types (stack-allocated), while in C++ they're just classes with public default access. C++ structs can have constructors, destructors, and virtual functions - they're essentially classes.

### Arrays vs Collections

**.NET distinguishes** between fixed arrays (`T[]`) and dynamic collections (`List<T>`), similar to Java but unlike JavaScript where `Array` is always dynamic.

### Span\<T\>

**Span\<T\>** provides zero-allocation slicing over memory, similar to Rust's slices (`&[T]`) or C++20's `std::span`, but unique among managed languages.

### Namespaces

**Namespaces** in .NET are more like Java packages than C++ namespaces. C++ namespaces are compile-time constructs that don't affect file organization, while .NET namespaces typically mirror folder structure.

### Memory Management

**Memory Management** differs significantly: .NET uses garbage collection, while C++ uses manual memory management or RAII (Resource Acquisition Is Initialization) with destructors.

### Templates vs Generics

**C++ templates** are compile-time code generation (can specialize for different types), while .NET generics maintain type information at runtime and have more restrictions but better runtime support.

### Virtual Functions

**C++** requires explicit `virtual` keyword and uses vtables, while .NET methods are virtual by default in most languages (Java, C#) unless explicitly sealed/final.

### Events

**Events** in .NET provide built-in observer pattern support, while C++ typically uses function pointers, callbacks, or signals/slots (Qt).

### LINQ

**LINQ** is a unique .NET feature that brings SQL-like queries to objects. C++20 introduced ranges which provide similar functionality but with different syntax.

### Resource Management

**.NET's `using` statements** provide automatic disposal, while C++ relies on RAII and destructors for automatic resource cleanup.

---

## 🔗 Related Resources

- **[Language Essentials](../language-essentials.md)** - Core .NET concepts with examples
- **[Microsoft Design Guidelines](../microsoft-design-guidelines.md)** - Official design patterns
- **[Glossary](glossary.md)** - .NET terminology explained
- **[Quick Syntax Reference](quick-syntax.md)** - Syntax examples and patterns
