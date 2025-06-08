# .NET Byte-Size Curriculum

_A progressive learning path for .NET development focused on libraries, tools, and games_

## Overview

This curriculum is designed for 2-hour evening sessions with ADHD-friendly, focused projects. Each project teaches one core concept while building practical applications.

## Learning Philosophy

- **Single Focus**: Each project teaches one main concept
- **Independent Projects**: No project dependencies to maintain focus
- **Practical Applications**: Libraries, tools, and games over web services
- **Progressive Complexity**: Concepts build upon each other logically

---

## Phase 1: .NET Fundamentals & Idioms (Projects 1-4)

### Project 1: .NET Type System & Conventions

**Project**: Build a **Dice Rolling Library**

- **Core Concept**: .NET naming conventions, properties, value types vs reference types
- **What You'll Learn**: C# idioms, `struct` vs `class`, properties, implicit typing with `var`
- **Deliverable**: `DiceRoller` NuGet package

### Project 2: Collections & Enumerables

**Project**: Build a **Card Deck Shuffler**

- **Core Concept**: `IEnumerable<T>`, `List<T>`, `Array`, generic collections
- **What You'll Learn**: Generic collections, `foreach` mechanics, `IEnumerable` interface
- **Deliverable**: Card game foundation library

### Project 3: Extension Methods & Fluent APIs

**Project**: Build **String Processing Utilities**

- **Core Concept**: Extension methods and method chaining
- **What You'll Learn**: `this` parameter, fluent interface design, extending existing types
- **Deliverable**: Text manipulation library with fluent API

### Project 4: Pattern Matching & Switch Expressions

**Project**: Build a **State Machine for Turn-Based Games**

- **Core Concept**: Modern C# pattern matching, switch expressions
- **What You'll Learn**: Pattern matching, `switch` expressions, `when` clauses, tuple deconstruction
- **Deliverable**: Game state management library

---

## Phase 2: Advanced Language Features (Projects 5-8)

### Project 5: LINQ & Deferred Execution

**Project**: Build a **Game Statistics Analyzer**

- **Core Concept**: LINQ queries, lazy evaluation
- **What You'll Learn**: Method syntax vs query syntax, deferred execution, `IQueryable` vs `IEnumerable`
- **Deliverable**: Statistical analysis library for game data

### Project 6: Async/Await & Task-Based Programming

**Project**: Build a **File-Based Game Save System**

- **Core Concept**: Asynchronous programming, `Task<T>`
- **What You'll Learn**: `async`/`await`, `Task.Run`, `ConfigureAwait`, async best practices
- **Deliverable**: Async file I/O library for games

### Project 7: Delegates, Events & Functional Programming

**Project**: Build an **Event-Driven Audio Manager**

- **Core Concept**: `Action<T>`, `Func<T>`, events, functional concepts
- **What You'll Learn**: Delegates, events, lambda expressions, functional programming in C#
- **Deliverable**: Audio system with event-driven architecture

### Project 8: Generics & Constraints

**Project**: Build a **Generic Object Pool**

- **Core Concept**: Generic types, constraints, covariance/contravariance
- **What You'll Learn**: Generic constraints (`where T : class`), generic collections, type safety
- **Deliverable**: Performance-optimized object pooling library

---

## Phase 3: Architecture & Design Patterns (Projects 9-12)

### Project 9: Dependency Injection & IoC

**Project**: Build a **Modular Game Plugin System**

- **Core Concept**: Dependency injection, inversion of control
- **What You'll Learn**: `Microsoft.Extensions.DependencyInjection`, service lifetimes, interface segregation
- **Deliverable**: Plugin architecture for extensible games

### Project 10: Builder Pattern & Factory Pattern

**Project**: Build a **Procedural Dungeon Generator**

- **Core Concept**: Creational design patterns
- **What You'll Learn**: Builder pattern, factory pattern, fluent configuration
- **Deliverable**: Configurable dungeon generation library

### Project 11: Observer Pattern & Command Pattern

**Project**: Build a **Game Input Manager**

- **Core Concept**: Behavioral design patterns
- **What You'll Learn**: Observer pattern, command pattern, undo/redo functionality
- **Deliverable**: Flexible input handling system

### Project 12: Repository Pattern & Unit of Work

**Project**: Build a **Game Asset Manager**

- **Core Concept**: Data access patterns, SOLID principles
- **What You'll Learn**: Repository pattern, unit of work, separation of concerns
- **Deliverable**: Asset loading and caching system

---

## Phase 4: Advanced .NET Features (Projects 13-16)

### Project 13: Reflection & Attributes

**Project**: Build a **Configuration System**

- **Core Concept**: Runtime type inspection, metadata
- **What You'll Learn**: `Type.GetType()`, custom attributes, attribute-driven configuration
- **Deliverable**: Flexible configuration library using attributes

### Project 14: Expression Trees & Dynamic Code

**Project**: Build a **Dynamic Query Builder**

- **Core Concept**: Expression trees, dynamic code generation
- **What You'll Learn**: `Expression<T>`, dynamic method creation, code as data
- **Deliverable**: Runtime query builder for game data

### Project 15: Memory Management & Spans

**Project**: Build a **High-Performance Game Buffer**

- **Core Concept**: Memory management, `Span<T>`, `Memory<T>`
- **What You'll Learn**: Stack allocation, memory pools, `Span<T>` for performance
- **Deliverable**: Zero-allocation buffer management system

### Project 16: Unsafe Code & Interop

**Project**: Build a **Native Library Wrapper**

- **Core Concept**: P/Invoke, unsafe code, interoperability
- **What You'll Learn**: `DllImport`, unsafe blocks, marshaling, calling native libraries
- **Deliverable**: C# wrapper for a native game physics library

---

## Phase 5: Tooling & Distribution (Projects 17-20)

### Project 17: NuGet Package Creation

**Project**: **Package Your Best Library**

- **Core Concept**: NuGet ecosystem, package management
- **What You'll Learn**: `.csproj` configuration, versioning, package metadata, publishing
- **Deliverable**: Published NuGet package

### Project 18: Unit Testing & Mocking

**Project**: **Test Suite for Game Logic**

- **Core Concept**: Unit testing, test-driven development
- **What You'll Learn**: xUnit, NUnit, Moq, test organization, mocking dependencies
- **Deliverable**: Comprehensive test suite

### Project 19: Source Generators

**Project**: **Code Generator for Game Entities**

- **Core Concept**: Compile-time code generation
- **What You'll Learn**: `ISourceGenerator`, Roslyn analyzers, compile-time metaprogramming
- **Deliverable**: Source generator that creates boilerplate code

### Project 20: Performance Profiling & Optimization

**Project**: **Optimize Your Game Libraries**

- **Core Concept**: Performance analysis, optimization techniques
- **What You'll Learn**: dotMemory, PerfView, BenchmarkDotNet, performance best practices
- **Deliverable**: Performance-optimized versions of previous projects

---

## Final Project Options (Projects 21-22)

Choose one capstone project that combines multiple concepts:

### Option A: **Complete 2D Game Engine**

Combine all learned concepts into a lightweight 2D game engine

### Option B: **Game Development Toolkit**

Build a comprehensive toolkit with generators, analyzers, and utilities

### Option C: **Real-time Strategy Game Framework**

Create a framework for building RTS games with all architectural patterns

---

## 📚 Project Guides

Access detailed guides for each project in the curriculum:

### Phase 1: .NET Fundamentals & Idioms

- **[Project 1: Dice Rolling Library](./project-01-dice-roller/dice-roller-project-guide.md)** - .NET Type System & Conventions
- **[Project 2: Card Deck Shuffler](./project-02-card-deck/card-deck-project-guide.md)** - Collections & Enumerables
- **[Project 3: String Processing Utilities](./project-03-string-utilities/string-utilities-project-guide.md)** - Extension Methods & Fluent APIs
- **[Project 4: Turn-Based Game State Machine](./project-04-state-machine/state-machine-project-guide.md)** - Pattern Matching & Switch Expressions

### Phase 2: Advanced Language Features

- **[Project 5: Game Statistics Analyzer](./project-05-statistics-analyzer/statistics-analyzer-project-guide.md)** - LINQ & Deferred Execution

_Additional project guides will be added as the curriculum expands._

---

## Success Metrics

- [ ] Complete 20 focused projects
- [ ] Publish at least 3 NuGet packages
- [ ] Build one substantial final project
- [ ] Understand and apply SOLID principles throughout
- [ ] Demonstrate proficiency with async/await patterns
- [ ] Use dependency injection effectively
- [ ] Write comprehensive unit tests
- [ ] Apply appropriate design patterns

## 📚 Essential Microsoft Documentation

**Reading the official documentation is a core skill for .NET developers.** These links provide authoritative guidance for the entire curriculum:

### Foundation Reading

- [.NET Documentation Hub](https://docs.microsoft.com/en-us/dotnet/) - Your starting point for all .NET learning
- [C# Programming Guide](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/) - Comprehensive C# language guide
- [.NET API Reference](https://docs.microsoft.com/en-us/dotnet/api/) - Complete API documentation

### Architecture and Design

- [.NET Application Architecture Guides](https://docs.microsoft.com/en-us/dotnet/architecture/) - Best practices and patterns
- [Framework Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/) - How to design .NET libraries
- [Dependency Injection](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection) - Built-in DI patterns

### Advanced Topics

- [Performance Best Practices](https://docs.microsoft.com/en-us/dotnet/core/deploying/optimization) - Writing efficient .NET code
- [Memory Management](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/) - Understanding the GC
- [Async Programming](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/) - Mastering async/await

**💡 Pro Tip**: Bookmark these pages and refer to them throughout each project. The curriculum examples are designed to lead you to explore the documentation for deeper understanding.

---

## Resources & Next Steps

Each project folder contains:

- 📋 Detailed project specification with **Microsoft docs links**
- 🎯 Learning objectives tied to official documentation
- 🛠️ Implementation guide with doc references
- 🚀 Extension challenges for deeper exploration
- 📖 Curated reading lists from Microsoft docs

---

## 📚 Essential Reference Guides

Before diving into the projects, familiarize yourself with these essential reference documents:

### 🚀 [Quick Reference Guide](./quick-reference.md)

**Keep this open while coding!** Comprehensive cheat sheet covering:

- .NET naming conventions and language translation guide
- Memory concepts (stack vs heap, value vs reference types)
- Collection patterns and LINQ essentials
- Async/await patterns and best practices
- Extension methods and fluent API design
- Direct links to Microsoft documentation for every topic

### 📈 [Progress Tracker](./progress-tracker.md)

**Track your learning journey** with detailed checklists for:

- Project completion status
- Concept mastery tracking
- Skills assessment per phase
- Portfolio building milestones
- NuGet package publication goals

### 🏗️ [Microsoft Design Guidelines](./dotnet-design-guidelines.md)

**Build professional-quality code** following Microsoft's official standards:

- Official naming conventions for all .NET elements
- Type design guidelines (classes vs structs, interfaces vs abstract classes)
- Member design patterns (properties, methods, events)
- Error handling and exception design
- Performance guidelines and async patterns
- Library design and API surface best practices
- Framework design patterns (Builder, Factory, etc.)
- Links to all official Microsoft design documentation

### 📖 [.NET Glossary](./dotnet-glossary.md)

**Master the terminology!** Comprehensive reference covering:

- .NET-specific terms and concepts (CLR, JIT, Assembly, etc.)
- C# language terminology (delegates, generics, LINQ)
- Object-oriented programming concepts in .NET context
- Memory management terms (stack, heap, boxing, GC)
- Advanced features (reflection, async/await, expression trees)
- Ecosystem terminology (NuGet, namespaces, attributes)
- Direct links to Microsoft documentation for each concept

### Quick Navigation

- 💡 **New to .NET?** Start with the [Quick Reference Guide](./quick-reference.md)
- 🎯 **Track Progress?** Use the [Progress Tracker](./progress-tracker.md)
- 🏗️ **Writing Library Code?** Follow the [Design Guidelines](./dotnet-design-guidelines.md)
- 📖 **Need Definitions?** Check the [.NET Glossary](./dotnet-glossary.md)
- 🎮 **Ready to Code?** Begin with [Project 1: Dice Rolling Library](./project-01-dice-roller/dice-roller-project-guide.md)

---

**Ready to start your .NET journey? Begin with [Project 1: Dice Rolling Library](./project-01-dice-roller/dice-roller-project-guide.md)!** 🎲
