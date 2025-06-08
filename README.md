# .NET Byte-Size Curriculum

_A progressive learning path for .NET development focused on libraries, tools, and games_

## Overview

This curriculum is designed for 2-hour evening sessions with ADHD-friendly, focused projects. Each project teaches one core concept while building practical applications.

## Learning Philosophy

- **Single Focus**: Each project teaches one main concept
- **Independent Projects**: No project dependencies to maintain focus
- **Practical Applications**: Libraries, tools, and games over web services
- **Progressive Complexity**: Concepts build upon each other logically
- **Template-Driven**: Pre-configured project templates (Projects 2-20) streamline setup and let you focus on learning

## Essential Resources Before You Start

1. 📏 **[Microsoft Design Guidelines](microsoft-design-guidelines.md)** - The foundation for all .NET code
2. 📚 **[Curriculum Details](curriculum-details.md)** - What you'll build (detailed project breakdown)
3. 🧠 **[Language Essentials](language-essentials.md)** - Core .NET concepts you need to know
4. 🎯 **[Progress Tracker](progress-tracker.md)** - Track your journey through the curriculum

## Quick Reference Resources

5. 📖 **[Reference Directory](reference/)** - Deep-dive references when you need them
   - **[Glossary](reference/glossary.md)** - .NET terminology explained
   - **[Language Comparison](reference/language-comparison.md)** - How .NET maps to other languages
   - **[Quick Syntax](reference/quick-syntax.md)** - Handy syntax reference

---

## Project Structure

**Template Information**: All projects except Project 1 include pre-configured .NET solution templates:

```
project-xx-name/
├── ProjectName.sln              # Solution file with references
├── ProjectName.Core/            # Main library (.NET 8.0)
├── ProjectName.App/             # Console app for testing
├── ProjectName.Tests/           # Unit tests (MSTest)
└── README.md                    # Detailed instructions
```

**Project 1** requires manual setup to teach foundational .NET project creation skills. **Projects 2-20** include templates so you can focus on implementation and learning concepts rather than project configuration.

---

## Curriculum Overview

### Phase 1: .NET Fundamentals & Idioms (Projects 1-4)

**Focus**: Core language features and conventions

1. **Dice Rolling Library** - Type system & naming conventions
2. **Card Deck Shuffler** - Collections & enumerables
3. **String Processing Utilities** - Extension methods & fluent APIs
4. **State Machine** - Pattern matching & switch expressions

### Phase 2: Advanced Language Features (Projects 5-8)

**Focus**: Modern C# capabilities

5. **Game Statistics Analyzer** - LINQ & deferred execution
6. **File-Based Game Save System** - Async/await programming
7. **Event-Driven Audio Manager** - Delegates, events & functional programming
8. **Generic Object Pool** - Generics & constraints

### Phase 3: Architecture & Design Patterns (Projects 9-12)

**Focus**: Software design principles

9. **Modular Game Plugin System** - Dependency injection & IoC
10. **Procedural Dungeon Generator** - Builder & factory patterns
11. **Game Input Manager** - Observer & command patterns
12. **Game Asset Manager** - Repository pattern & unit of work

### Phase 4: Advanced .NET Features (Projects 13-16)

**Focus**: Runtime and performance features

13. **Configuration System** - Reflection & attributes
14. **Dynamic Query Builder** - Expression trees & dynamic code
15. **High-Performance Game Buffer** - Memory management & spans
16. **Native Library Wrapper** - Unsafe code & interop

### Phase 5: Integration & Advanced Systems (Projects 17-20)

**Focus**: System integration and advanced topics

17. **Mini Game Engine** - Integration capstone combining all concepts
18. **Real-Time Multiplayer System** - Networking & real-time communication
19. **Extensible Modding Framework** - Plugin architecture & extensibility
20. **Performance Profiling Suite** - Diagnostics & performance analysis

---

## Getting Started

### 1. **Read the Foundation** 📏

Start with [Microsoft Design Guidelines](microsoft-design-guidelines.md) to understand .NET conventions and best practices.

### 2. **Plan Your Journey** 📚

Review [Curriculum Details](curriculum-details.md) to understand what you'll build and learn in each project.

### 3. **Set Up Tracking** 🎯

Use the [Progress Tracker](progress-tracker.md) to monitor your advancement and reflect on learnings.

### 4. **Reference as Needed** 📖

Bookmark the [Reference Directory](reference/) for quick lookups during development.

---

## Final Projects (Choose One)

After completing the 20 core projects, choose a capstone project:

- **Game Engine Architecture** - Combine multiple patterns into a cohesive game engine
- **Development Toolchain** - Build a complete development workflow with CI/CD
- **Performance-Critical Library** - Create a highly optimized library using advanced .NET features

---

## Contributing

This curriculum is open for contributions! See individual project folders for specific contribution guidelines.

---

## Learning Support

- **Microsoft Guidelines Integration** - Each project explicitly references relevant design guidelines
- **Cross-References** - Documents link to related concepts across the curriculum
- **Progressive Disclosure** - Start simple, reference deeper material as needed
- **Practical Focus** - Every concept is taught through building real libraries and tools
