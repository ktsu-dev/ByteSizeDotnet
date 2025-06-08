# Project 4: State Machine Framework

**Focus**: Pattern Matching & Switch Expressions  
**Time**: ~2 hours  
**Deliverable**: `StateMachine` framework for game logic

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Pattern Matching Guidelines](../microsoft-design-guidelines.md#modern-language-features)** - Using switch expressions and pattern matching
- **[Generic Design](../microsoft-design-guidelines.md#generic-design-guidelines)** - Creating reusable generic state machines
- **[Event Design](../microsoft-design-guidelines.md#event-design)** - Proper event handling patterns

## Learning Objectives

- Master modern C# pattern matching features
- Understand switch expressions and pattern-based programming
- Learn generic type constraints and variance
- Practice event-driven programming patterns
- Build reusable framework components

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Generic Design Guidelines](../microsoft-design-guidelines.md#generic-design-guidelines) before starting

2. **Start Coding** 💻  
   See [detailed project guide](state-machine-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-04-state-machine/
├── README.md                      # This file
├── state-machine-project-guide.md # Detailed implementation guide
├── src/                           # Your implementation
├── tests/                         # Unit tests
└── docs/                          # Additional documentation
```

## What You'll Build

A flexible state machine framework featuring:

- Generic state and trigger definitions
- Pattern matching for state transitions
- Event-driven state change notifications
- Conditional transitions with guards
- State entry/exit actions
- Hierarchical state support

## Key Concepts

- **Switch Expressions**: Modern pattern matching syntax
- **Pattern Matching**: Using `is`, `when`, and destructuring
- **Generic Constraints**: Restricting type parameters appropriately
- **Delegates and Events**: Implementing observer patterns
- **Functional Programming**: Using functions as first-class values

## Resources

- 📚 **[Detailed Guide](state-machine-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Pattern Matching Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)**
- 📘 **[Generic Types Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/)**

## Building on Previous Projects

This project advances concepts from earlier projects:

- **Project 1**: Applies type design with generic constraints
- **Project 2**: Uses advanced enumerable patterns
- **Project 3**: Leverages fluent API design patterns
- **New Skills**: Pattern matching and generic framework design

## Example Usage

```csharp
// Define game states
public enum GameState { Menu, Playing, Paused, GameOver }
public enum GameTrigger { Start, Pause, Resume, Die, Restart }

// Create state machine
var gameStateMachine = new StateMachine<GameState, GameTrigger>(GameState.Menu)
    .Configure(GameState.Menu)
        .Permit(GameTrigger.Start, GameState.Playing)
    .Configure(GameState.Playing)
        .Permit(GameTrigger.Pause, GameState.Paused)
        .Permit(GameTrigger.Die, GameState.GameOver)
    .Configure(GameState.Paused)
        .Permit(GameTrigger.Resume, GameState.Playing)
        .Permit(GameTrigger.Restart, GameState.Menu);

// Use pattern matching for reactions
var action = gameStateMachine.CurrentState switch
{
    GameState.Playing => UpdateGameLogic(),
    GameState.Paused => ShowPauseMenu(),
    GameState.GameOver => HandleGameOver(),
    _ => DoNothing()
};
```

## Contributing

When implementing this project:

1. Use modern pattern matching features appropriately
2. Design generic interfaces with proper constraints
3. Implement events following .NET conventions
4. Create comprehensive unit tests for all transitions
5. Document state machine behavior clearly
