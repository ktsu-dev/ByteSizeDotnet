# .NET Curriculum - Detailed Project Guide

_Comprehensive breakdown of all 20 projects with learning objectives and Microsoft guideline references_

---

## Phase 1: .NET Fundamentals & Idioms (Projects 1-4)

### Project 1: .NET Type System & Conventions

**Project**: Build a **Dice Rolling Library**

- **Core Concept**: .NET naming conventions, properties, value types vs reference types
- **What You'll Learn**: C# idioms, `struct` vs `class`, properties, implicit typing with `var`
- **Deliverable**: `DiceRoller` NuGet package
- **Microsoft Guidelines Focus**:
  - [Naming Conventions](microsoft-design-guidelines.md#basic-naming-rules)
  - [Class vs Struct Decision Matrix](microsoft-design-guidelines.md#class-vs-struct-decision-matrix)

### Project 2: Collections & Enumerables

**Project**: Build a **Card Deck Shuffler**

- **Core Concept**: `IEnumerable<T>`, `List<T>`, `Array`, generic collections
- **What You'll Learn**: Generic collections, `foreach` mechanics, `IEnumerable` interface
- **Deliverable**: Card game foundation library
- **Microsoft Guidelines Focus**:
  - [Collection Guidelines](microsoft-design-guidelines.md#collection-guidelines)
  - [Generic Type Parameters](microsoft-design-guidelines.md#generic-type-parameters)

### Project 3: Extension Methods & Fluent APIs

**Project**: Build **String Processing Utilities**

- **Core Concept**: Extension methods and method chaining
- **What You'll Learn**: `this` parameter, fluent interface design, extending existing types
- **Deliverable**: Text manipulation library with fluent API
- **Microsoft Guidelines Focus**:
  - [Method Design](microsoft-design-guidelines.md#method-design)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

### Project 4: Pattern Matching & Switch Expressions

**Project**: Build a **State Machine for Turn-Based Games**

- **Core Concept**: Modern C# pattern matching, switch expressions
- **What You'll Learn**: Pattern matching, `switch` expressions, `when` clauses, tuple deconstruction
- **Deliverable**: Game state management library
- **Microsoft Guidelines Focus**:
  - [Immutability Guidelines](microsoft-design-guidelines.md#immutability-guidelines)

---

## Phase 2: Advanced Language Features (Projects 5-8)

### Project 5: LINQ & Deferred Execution

**Project**: Build a **Game Statistics Analyzer**

- **Core Concept**: LINQ queries, lazy evaluation
- **What You'll Learn**: Method syntax vs query syntax, deferred execution, `IQueryable` vs `IEnumerable`
- **Deliverable**: Statistical analysis library for game data
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Collection Guidelines](microsoft-design-guidelines.md#collection-guidelines)

### Project 6: Async/Await & Task-Based Programming

**Project**: Build a **File-Based Game Save System**

- **Core Concept**: Asynchronous programming, `Task<T>`
- **What You'll Learn**: `async`/`await`, `Task.Run`, `ConfigureAwait`, async best practices
- **Deliverable**: Async file I/O library for games
- **Microsoft Guidelines Focus**:
  - [Async Guidelines](microsoft-design-guidelines.md#async-guidelines)
  - [Exception Design](microsoft-design-guidelines.md#exception-design)

### Project 7: Delegates, Events & Functional Programming

**Project**: Build an **Event-Driven Audio Manager**

- **Core Concept**: `Action<T>`, `Func<T>`, events, functional concepts
- **What You'll Learn**: Delegates, events, lambda expressions, functional programming in C#
- **Deliverable**: Audio system with event-driven architecture
- **Microsoft Guidelines Focus**:
  - [Event Design](microsoft-design-guidelines.md#event-design)

### Project 8: Generics & Constraints

**Project**: Build a **Generic Object Pool**

- **Core Concept**: Generic types, constraints, covariance/contravariance
- **What You'll Learn**: Generic constraints (`where T : class`), generic collections, type safety
- **Deliverable**: Performance-optimized object pooling library
- **Microsoft Guidelines Focus**:
  - [Generic Type Parameters](microsoft-design-guidelines.md#generic-type-parameters)

---

## Phase 3: Architecture & Design Patterns (Projects 9-12)

### Project 9: Dependency Injection & IoC

**Project**: Build a **Modular Game Plugin System**

- **Core Concept**: Dependency injection, inversion of control
- **What You'll Learn**: `Microsoft.Extensions.DependencyInjection`, service lifetimes, interface segregation
- **Deliverable**: Plugin architecture for extensible games
- **Microsoft Guidelines Focus**:
  - [Abstract Classes vs Interfaces](microsoft-design-guidelines.md#abstract-classes-vs-interfaces)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

### Project 10: Builder Pattern & Factory Pattern

**Project**: Build a **Procedural Dungeon Generator**

- **Core Concept**: Creational design patterns
- **What You'll Learn**: Builder pattern, factory pattern, fluent configuration
- **Deliverable**: Configurable dungeon generation library
- **Microsoft Guidelines Focus**:
  - [Builder Pattern](microsoft-design-guidelines.md#builder-pattern)
  - [Factory Pattern](microsoft-design-guidelines.md#factory-pattern)

### Project 11: Observer Pattern & Command Pattern

**Project**: Build a **Game Input Manager**

- **Core Concept**: Behavioral design patterns
- **What You'll Learn**: Observer pattern, command pattern, undo/redo functionality
- **Deliverable**: Flexible input handling system
- **Microsoft Guidelines Focus**:
  - [Event Design](microsoft-design-guidelines.md#event-design)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

### Project 12: Repository Pattern & Unit of Work

**Project**: Build a **Game Asset Manager**

- **Core Concept**: Data access patterns, SOLID principles
- **What You'll Learn**: Repository pattern, unit of work, separation of concerns
- **Deliverable**: Asset loading and caching system
- **Microsoft Guidelines Focus**:
  - [Abstract Classes vs Interfaces](microsoft-design-guidelines.md#abstract-classes-vs-interfaces)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

---

## Phase 4: Advanced .NET Features (Projects 13-16)

### Project 13: Reflection & Attributes

**Project**: Build a **Configuration System**

- **Core Concept**: Runtime type inspection, metadata
- **What You'll Learn**: `Type.GetType()`, custom attributes, attribute-driven configuration
- **Deliverable**: Flexible configuration library using attributes
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

### Project 14: Expression Trees & Dynamic Code

**Project**: Build a **Dynamic Query Builder**

- **Core Concept**: Expression trees, dynamic code generation
- **What You'll Learn**: `Expression<T>`, dynamic method creation, code as data
- **Deliverable**: Runtime query builder for game data
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Exception Design](microsoft-design-guidelines.md#exception-design)

### Project 15: Memory Management & Spans

**Project**: Build a **High-Performance Game Buffer**

- **Core Concept**: Memory management, `Span<T>`, `Memory<T>`
- **What You'll Learn**: Stack allocation, memory pools, `Span<T>` for performance
- **Deliverable**: Zero-allocation buffer management system
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Collection Guidelines](microsoft-design-guidelines.md#collection-guidelines)

### Project 16: Unsafe Code & Interop

**Project**: Build a **Native Library Wrapper**

- **Core Concept**: P/Invoke, unsafe code, interoperability
- **What You'll Learn**: `DllImport`, unsafe blocks, marshaling, calling native libraries
- **Deliverable**: C# wrapper for a native game physics library
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Exception Design](microsoft-design-guidelines.md#exception-design)

---

## Phase 5: Tooling & Distribution (Projects 17-20)

### Project 17: NuGet Package Creation

**Project**: **Package Your Best Library**

- **Core Concept**: NuGet ecosystem, package management
- **What You'll Learn**: `.csproj` configuration, versioning, package metadata, publishing
- **Deliverable**: Published NuGet package
- **Microsoft Guidelines Focus**:
  - [Versioning and Compatibility](microsoft-design-guidelines.md#versioning-and-compatibility)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

### Project 18: Unit Testing & Mocking

**Project**: **Test Suite for Game Logic**

- **Core Concept**: Unit testing, test-driven development
- **What You'll Learn**: MSTest, Moq, test organization, mocking dependencies, test attributes
- **Deliverable**: Comprehensive test suite
- **Microsoft Guidelines Focus**:
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)
  - [Exception Design](microsoft-design-guidelines.md#exception-design)

### Project 19: Source Generators

**Project**: **Code Generator for Game Entities**

- **Core Concept**: Compile-time code generation
- **What You'll Learn**: `ISourceGenerator`, Roslyn analyzers, compile-time metaprogramming
- **Deliverable**: Source generator that creates boilerplate code
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

### Project 20: Performance Profiling & Optimization

**Project**: **Optimize Your Game Libraries**

- **Core Concept**: Performance analysis, optimization techniques
- **What You'll Learn**: dotMemory, PerfView, BenchmarkDotNet, performance best practices
- **Deliverable**: Performance-optimized versions of previous projects
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Collection Guidelines](microsoft-design-guidelines.md#collection-guidelines)

---

## Final Project Options (Projects 21-22)

Choose one capstone project that combines multiple concepts:

### Option A: **Complete 2D Game Engine**

Combine all learned concepts into a lightweight 2D game engine

- **Combines**: All phases - type design, collections, async patterns, DI, design patterns, performance optimization
- **Focus**: Large-scale architecture, plugin systems, performance-critical code
- **Deliverable**: Production-ready game engine with documentation and examples

### Option B: **Game Development Toolkit**

Build a comprehensive toolkit with generators, analyzers, and utilities

- **Combines**: Source generators, reflection, expression trees, tooling, NuGet packaging
- **Focus**: Developer productivity, code generation, static analysis
- **Deliverable**: Published toolkit that other developers can use

### Option C: **Real-time Strategy Game Framework**

Create a framework for building RTS games with all architectural patterns

- **Combines**: Event systems, command patterns, async programming, performance optimization
- **Focus**: Real-time systems, scalability, network programming
- **Deliverable**: Framework with reference implementation of an RTS game

---

## Learning Support Features

### Microsoft Guidelines Integration

Each project explicitly references specific sections of the [Microsoft Design Guidelines](microsoft-design-guidelines.md), ensuring you:

- Learn industry-standard practices from the start
- Understand the "why" behind design decisions
- Build muscle memory for professional .NET development
- Create code that follows Microsoft's own standards

### Progressive Skill Building

Projects are carefully sequenced to build upon previous knowledge:

1. **Fundamentals First** - Master basic .NET concepts and conventions
2. **Language Features** - Explore modern C# capabilities
3. **Architecture Patterns** - Learn professional software design
4. **Advanced Features** - Tackle performance and runtime topics
5. **Professional Tools** - Master the development ecosystem

### Practical Focus

Every project produces a real, usable library:

- Build your portfolio with tangible deliverables
- Practice packaging and distribution
- Learn by solving real problems
- Create tools you'll actually use

### Cross-References and Connections

- Each project links to relevant Microsoft documentation
- Progress tracker includes guideline checklists
- Reference materials tie back to project examples
- Clear progression from basic to advanced topics
