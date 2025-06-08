# Project 2: Card Deck Shuffler

**Focus**: Collections & Enumerables  
**Time**: ~2 hours  
**Deliverable**: `CardDeck` library with advanced shuffling capabilities

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Collection Guidelines](../microsoft-design-guidelines.md#collection-guidelines)** - Proper collection usage and interfaces
- **[Enum Design](../microsoft-design-guidelines.md#enum-design)** - Representing card suits and ranks
- **[Interface Design](../microsoft-design-guidelines.md#interface-design)** - Implementing `IEnumerable<T>`

## Learning Objectives

- Master .NET collection types and interfaces
- Understand `IEnumerable<T>`, `IList<T>`, and `ICollection<T>`
- Learn proper enum design and usage
- Practice LINQ fundamentals
- Implement collection interfaces correctly

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Collection Guidelines](../microsoft-design-guidelines.md#collection-guidelines) before starting

2. **Start Coding** 💻  
   See [detailed project guide](card-deck-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-02-card-deck/
├── README.md                    # This file
├── card-deck-project-guide.md  # Detailed implementation guide
├── src/                         # Your implementation
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
```

## What You'll Build

A card deck library featuring:

- Standard 52-card deck representation
- Multiple shuffling algorithms (Fisher-Yates, Riffle, etc.)
- Card dealing and hand management
- Deck manipulation (cut, bridge, etc.)
- Statistics and shuffle quality metrics

## Key Concepts

- **Collection Interfaces**: `IEnumerable<T>`, `IList<T>`, `ICollection<T>`
- **Enum Design**: Representing card properties with proper enums
- **LINQ Basics**: Filtering, mapping, and aggregating collections
- **Immutability Patterns**: Creating immutable collections
- **Iterator Methods**: Using `yield return` for lazy evaluation

## Resources

- 📚 **[Detailed Guide](card-deck-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Collections Documentation](https://learn.microsoft.com/en-us/dotnet/standard/collections/)**
- 📘 **[LINQ Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)**

## Building on Project 1

This project builds on the foundational concepts from the Dice Roller:

- Applies naming conventions learned in Project 1
- Extends type design knowledge to enums and interfaces
- Introduces collection-specific design patterns

## Contributing

When implementing this project:

1. Follow collection interface guidelines
2. Use appropriate enum design patterns
3. Implement proper `IEnumerable<T>` support
4. Write comprehensive unit tests
5. Document collection behavior clearly
