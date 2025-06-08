# Project 1: Dice Rolling Library

**Focus**: .NET Type System & Conventions  
**Time**: ~2 hours  
**Deliverable**: `DiceRoller` NuGet package

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Naming Conventions](../microsoft-design-guidelines.md#naming-guidelines)** - PascalCase for types, camelCase for parameters
- **[Type Design](../microsoft-design-guidelines.md#type-design-guidelines)** - Value types vs reference types
- **[Member Design](../microsoft-design-guidelines.md#member-design-guidelines)** - Properties vs fields

## Learning Objectives

- Master .NET naming conventions
- Understand value types (`struct`) vs reference types (`class`)
- Learn proper property and field usage
- Practice basic .NET project structure
- Apply essential C# idioms

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Type Design Guidelines](../microsoft-design-guidelines.md#type-design-guidelines) before starting

2. **Start Coding** 💻  
   See [detailed project guide](dice-roller-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-01-dice-roller/
├── README.md                    # This file
├── dice-roller-project-guide.md # Detailed implementation guide
├── src/                         # Your implementation
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
```

## What You'll Build

A dice rolling library that supports:

- Various dice types (d4, d6, d8, d10, d12, d20, d100)
- Multiple dice rolls (`3d6`, `2d20`)
- Modifiers (`1d20+5`, `3d6-2`)
- Dice notation parsing (`"2d6+3"`)
- Roll statistics and history

## Key Concepts

- **Value Types vs Reference Types**: When to use `struct` vs `class`
- **Properties vs Fields**: Proper data exposure patterns
- **Naming Conventions**: Following .NET standards
- **Constructor Patterns**: Traditional and primary constructors

## Resources

- 📚 **[Detailed Guide](dice-roller-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Type System Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types)**

## Contributing

When implementing this project:

1. Follow the Microsoft design guidelines
2. Write clean, readable code with proper naming
3. Include XML documentation comments
4. Add unit tests for core functionality
5. Update the progress tracker with your learnings
