# .NET Curriculum - Detailed Project Guide

_Comprehensive breakdown of all 20 projects with learning objectives and Microsoft guideline references_

---

## Project Template Information

**Important Note**: All projects except Project 1 include pre-configured project templates to streamline development. Each project directory contains:

- **Solution file** (`.sln`) - Pre-configured with correct project references
- **Core library project** (`ProjectName.Core/`) - Main implementation code
- **Console application** (`ProjectName.App/`) - Demonstration and testing
- **Unit test project** (`ProjectName.Tests/`) - Comprehensive test coverage
- **README.md** - Detailed instructions and implementation guidance

The templates follow Microsoft's recommended project structure:

```
project-xx-name/
├── ProjectName.sln              # Solution file
├── ProjectName.Core/            # Main library (.NET 8.0 class library)
│   ├── Models/                  # Data models and value types
│   ├── Services/                # Business logic and services
│   ├── Interfaces/              # Contracts and abstractions
│   └── Extensions/              # Extension methods
├── ProjectName.App/             # Console application (.NET 8.0)
│   └── Program.cs               # Demonstration code
├── ProjectName.Tests/           # Unit tests (MSTest framework)
│   └── UnitTest1.cs             # Test implementations
└── README.md                    # Project instructions
```

**For instructors/advanced users**: Each project README includes complete manual setup instructions for creating the solution structure from scratch, should you need to recreate projects or understand the underlying dotnet CLI commands.

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

**⚠️ Note**: This project requires manual setup (no template provided) to teach foundational .NET project creation skills.

**Directory Structure**: `project-01-dice-roller/`

```
project-01-dice-roller/
├── README.md                    # Complete setup and implementation guide
├── DiceRoller.sln               # Your implementation (to be created)
├── DiceRoller.Core/
├── DiceRoller.App/
└── DiceRoller.Tests/
```

### Project 2: Collections & Enumerables

**Project**: Build a **Card Deck Shuffler**

- **Core Concept**: `IEnumerable<T>`, `List<T>`, `Array`, generic collections
- **What You'll Learn**: Generic collections, `foreach` mechanics, `IEnumerable` interface
- **Deliverable**: Card game foundation library
- **Microsoft Guidelines Focus**:
  - [Collection Guidelines](microsoft-design-guidelines.md#collection-guidelines)
  - [Generic Type Parameters](microsoft-design-guidelines.md#generic-type-parameters)

**✅ Template Available**: Pre-configured solution with `CardDeck.Core`, `CardDeck.App`, and `CardDeck.Tests` projects.

**Directory Structure**: `project-02-card-deck/`

### Project 3: Extension Methods & Fluent APIs

**Project**: Build **String Processing Utilities**

- **Core Concept**: Extension methods and method chaining
- **What You'll Learn**: `this` parameter, fluent interface design, extending existing types
- **Deliverable**: Text manipulation library with fluent API
- **Microsoft Guidelines Focus**:
  - [Method Design](microsoft-design-guidelines.md#method-design)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

**✅ Template Available**: Pre-configured solution with `StringUtilities.Core`, `StringUtilities.App`, and `StringUtilities.Tests` projects.

**Directory Structure**: `project-03-string-utilities/`

### Project 4: Pattern Matching & Switch Expressions

**Project**: Build a **State Machine for Turn-Based Games**

- **Core Concept**: Modern C# pattern matching, switch expressions
- **What You'll Learn**: Pattern matching, `switch` expressions, `when` clauses, tuple deconstruction
- **Deliverable**: Game state management library
- **Microsoft Guidelines Focus**:
  - [Immutability Guidelines](microsoft-design-guidelines.md#immutability-guidelines)

**✅ Template Available**: Pre-configured solution with `StateMachine.Core`, `StateMachine.App`, and `StateMachine.Tests` projects.

**Directory Structure**: `project-04-state-machine/`

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

**✅ Template Available**: Pre-configured solution with `GameStatistics.Core`, `GameStatistics.App`, and `GameStatistics.Tests` projects.

**Directory Structure**: `project-05-statistics-analyzer/`

### Project 6: Async/Await & Task-Based Programming

**Project**: Build a **File-Based Game Save System**

- **Core Concept**: Asynchronous programming, `Task<T>`
- **What You'll Learn**: `async`/`await`, `Task.Run`, `ConfigureAwait`, async best practices
- **Deliverable**: Async file I/O library for games
- **Microsoft Guidelines Focus**:
  - [Async Guidelines](microsoft-design-guidelines.md#async-guidelines)
  - [Exception Design](microsoft-design-guidelines.md#exception-design)

**✅ Template Available**: Pre-configured solution with `GameSave.Core`, `GameSave.App`, and `GameSave.Tests` projects.

**Directory Structure**: `project-06-game-save-system/`

### Project 7: Delegates, Events & Functional Programming

**Project**: Build an **Event-Driven Audio Manager**

- **Core Concept**: `Action<T>`, `Func<T>`, events, functional concepts
- **What You'll Learn**: Delegates, events, lambda expressions, functional programming in C#
- **Deliverable**: Audio system with event-driven architecture
- **Microsoft Guidelines Focus**:
  - [Event Design](microsoft-design-guidelines.md#event-design)

**✅ Template Available**: Pre-configured solution with `AudioManager.Core`, `AudioManager.App`, and `AudioManager.Tests` projects.

**Directory Structure**: `project-07-audio-manager/`

### Project 8: Generics & Constraints

**Project**: Build a **Generic Object Pool**

- **Core Concept**: Generic types, constraints, covariance/contravariance
- **What You'll Learn**: Generic constraints (`where T : class`), generic collections, type safety
- **Deliverable**: Performance-optimized object pooling library
- **Microsoft Guidelines Focus**:
  - [Generic Type Parameters](microsoft-design-guidelines.md#generic-type-parameters)

**✅ Template Available**: Pre-configured solution with `ObjectPool.Core`, `ObjectPool.App`, and `ObjectPool.Tests` projects.

**Directory Structure**: `project-08-object-pool/`

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

**✅ Template Available**: Pre-configured solution with `PluginSystem.Core`, `PluginSystem.App`, and `PluginSystem.Tests` projects.

**Directory Structure**: `project-09-plugin-system/`

### Project 10: Builder Pattern & Factory Pattern

**Project**: Build a **Procedural Dungeon Generator**

- **Core Concept**: Creational design patterns
- **What You'll Learn**: Builder pattern, factory pattern, fluent configuration
- **Deliverable**: Configurable dungeon generation library
- **Microsoft Guidelines Focus**:
  - [Builder Pattern](microsoft-design-guidelines.md#builder-pattern)
  - [Factory Pattern](microsoft-design-guidelines.md#factory-pattern)

**✅ Template Available**: Pre-configured solution with `DungeonGenerator.Core`, `DungeonGenerator.App`, and `DungeonGenerator.Tests` projects.

**Directory Structure**: `project-10-dungeon-generator/`

### Project 11: Observer Pattern & Command Pattern

**Project**: Build a **Game Input Manager**

- **Core Concept**: Behavioral design patterns
- **What You'll Learn**: Observer pattern, command pattern, undo/redo functionality
- **Deliverable**: Flexible input handling system
- **Microsoft Guidelines Focus**:
  - [Event Design](microsoft-design-guidelines.md#event-design)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

**✅ Template Available**: Pre-configured solution with `InputManager.Core`, `InputManager.App`, and `InputManager.Tests` projects.

**Directory Structure**: `project-11-input-manager/`

### Project 12: Repository Pattern & Unit of Work

**Project**: Build a **Game Asset Manager**

- **Core Concept**: Data access patterns, SOLID principles
- **What You'll Learn**: Repository pattern, unit of work, separation of concerns
- **Deliverable**: Asset loading and caching system
- **Microsoft Guidelines Focus**:
  - [Abstract Classes vs Interfaces](microsoft-design-guidelines.md#abstract-classes-vs-interfaces)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

**✅ Template Available**: Pre-configured solution with `AssetManager.Core`, `AssetManager.App`, and `AssetManager.Tests` projects.

**Directory Structure**: `project-12-asset-manager/`

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

**✅ Template Available**: Pre-configured solution with `ConfigurationSystem.Core`, `ConfigurationSystem.App`, and `ConfigurationSystem.Tests` projects.

**Directory Structure**: `project-13-configuration-system/`

### Project 14: Expression Trees & Dynamic Code

**Project**: Build a **Dynamic Query Builder**

- **Core Concept**: Expression trees, dynamic code generation
- **What You'll Learn**: `Expression<T>`, dynamic method creation, code as data
- **Deliverable**: Runtime query builder for game data
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Exception Design](microsoft-design-guidelines.md#exception-design)

**✅ Template Available**: Pre-configured solution with `QueryBuilder.Core`, `QueryBuilder.App`, and `QueryBuilder.Tests` projects.

**Directory Structure**: `project-14-dynamic-query-builder/`

### Project 15: Memory Management & Spans

**Project**: Build a **High-Performance Game Buffer**

- **Core Concept**: Memory management, `Span<T>`, `Memory<T>`
- **What You'll Learn**: Stack allocation, memory pools, `Span<T>` for performance
- **Deliverable**: Zero-allocation buffer management system
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Collection Guidelines](microsoft-design-guidelines.md#collection-guidelines)

**✅ Template Available**: Pre-configured solution with `GameBuffer.Core`, `GameBuffer.App`, and `GameBuffer.Tests` projects.

**Directory Structure**: `project-15-game-buffer/`

### Project 16: Unsafe Code & Interop

**Project**: Build a **Native Library Wrapper**

- **Core Concept**: P/Invoke, unsafe code, interoperability
- **What You'll Learn**: `DllImport`, unsafe blocks, marshaling, calling native libraries
- **Deliverable**: C# wrapper for a native game physics library
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Exception Design](microsoft-design-guidelines.md#exception-design)

**✅ Template Available**: Pre-configured solution with `NativeWrapper.Core`, `NativeWrapper.App`, and `NativeWrapper.Tests` projects.

**Directory Structure**: `project-16-native-wrapper/`

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

**✅ Template Available**: Pre-configured solution with `GameEngine.Core`, `GameEngine.App`, and `GameEngine.Tests` projects.

**Directory Structure**: `project-17-game-engine/`

### Project 18: Unit Testing & Mocking

**Project**: **Test Suite for Game Logic**

- **Core Concept**: Unit testing, test-driven development
- **What You'll Learn**: MSTest, Moq, test organization, mocking dependencies, test attributes
- **Deliverable**: Comprehensive test suite
- **Microsoft Guidelines Focus**:
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)
  - [Exception Design](microsoft-design-guidelines.md#exception-design)

**✅ Template Available**: Pre-configured solution with `MultiplayerSystem.Core`, `MultiplayerSystem.App`, and `MultiplayerSystem.Tests` projects.

**Directory Structure**: `project-18-multiplayer-system/`

### Project 19: Source Generators

**Project**: **Code Generator for Game Entities**

- **Core Concept**: Compile-time code generation
- **What You'll Learn**: `ISourceGenerator`, Roslyn analyzers, compile-time metaprogramming
- **Deliverable**: Source generator that creates boilerplate code
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [API Surface Design](microsoft-design-guidelines.md#api-surface-design)

**✅ Template Available**: Pre-configured solution with `ModdingFramework.Core`, `ModdingFramework.App`, and `ModdingFramework.Tests` projects.

**Directory Structure**: `project-19-modding-framework/`

### Project 20: Performance Profiling & Optimization

**Project**: **Optimize Your Game Libraries**

- **Core Concept**: Performance analysis, optimization techniques
- **What You'll Learn**: dotMemory, PerfView, BenchmarkDotNet, performance best practices
- **Deliverable**: Performance-optimized versions of previous projects
- **Microsoft Guidelines Focus**:
  - [Performance Guidelines](microsoft-design-guidelines.md#performance-guidelines)
  - [Collection Guidelines](microsoft-design-guidelines.md#collection-guidelines)

**✅ Template Available**: Pre-configured solution with `PerformanceProfiler.Core`, `PerformanceProfiler.App`, and `PerformanceProfiler.Tests` projects.

**Directory Structure**: `project-20-performance-profiler/`

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
