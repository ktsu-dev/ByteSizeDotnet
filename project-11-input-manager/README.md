# Project 11: Game Input Manager

**Focus**: Observer & Command Patterns  
**Time**: ~2 hours  
**Deliverable**: `InputManager` library with behavioral design patterns

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Observer Pattern Guidelines](../microsoft-design-guidelines.md#observer-pattern)** - Event notification systems
- **[Command Pattern Guidelines](../microsoft-design-guidelines.md#command-pattern)** - Action encapsulation
- **[Event Design](../microsoft-design-guidelines.md#event-design)** - Proper event handling patterns

## Learning Objectives

- Master observer and command design patterns
- Understand decoupled communication through events
- Learn action queuing and undo/redo functionality
- Practice behavioral pattern implementation
- Build responsive and extensible input systems

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Observer Pattern Guidelines](../microsoft-design-guidelines.md#observer-pattern) before starting

2. **Start Coding** 💻  
   See [detailed project guide](input-manager-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-11-input-manager/
├── README.md                    # This file
├── input-manager-project-guide.md # Detailed implementation guide
├── src/                         # Your implementation
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
```

## What You'll Build

A comprehensive input management system featuring:

- Observer pattern for input event distribution
- Command pattern for action encapsulation
- Input binding and remapping system
- Undo/redo functionality for actions
- Macro recording and playback
- Context-sensitive input handling

## Key Concepts

- **Observer Pattern**: Decoupled notification of input events
- **Command Pattern**: Encapsulating actions as objects
- **Chain of Responsibility**: Input handling hierarchy
- **State Pattern**: Context-dependent input behavior
- **Memento Pattern**: Capturing action state for undo/redo
- **Interpreter Pattern**: Input sequence parsing

## Resources

- 📚 **[Detailed Guide](input-manager-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Observer Pattern Documentation](https://learn.microsoft.com/en-us/dotnet/standard/events/)**
- 📘 **[Command Pattern Examples](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/)**

## Building on Previous Projects

This project continues Phase 3 architectural patterns:

- **Projects 1-8**: All foundational skills in pattern context
- **Project 9**: Dependency injection for input system components
- **Project 10**: Factory patterns for command creation
- **New Skills**: Behavioral design patterns and event-driven systems

## Example Usage

```csharp
// Observer pattern for input events
var inputManager = new InputManager();

inputManager.KeyPressed += (sender, e) =>
    Console.WriteLine($"Key pressed: {e.Key}");

inputManager.MouseMoved += (sender, e) =>
    UpdateCursor(e.Position);

// Command pattern for actions
var moveCommand = new MoveCommand(player, direction: Vector2.Up);
var attackCommand = new AttackCommand(player, target: enemy);

inputManager.BindKey(Keys.W, moveCommand);
inputManager.BindKey(Keys.Space, attackCommand);

// Input context switching
var gameplayContext = new InputContext("gameplay")
    .Bind(Keys.W, new MoveCommand(Direction.Forward))
    .Bind(Keys.S, new MoveCommand(Direction.Backward))
    .Bind(Keys.A, new MoveCommand(Direction.Left))
    .Bind(Keys.D, new MoveCommand(Direction.Right));

var menuContext = new InputContext("menu")
    .Bind(Keys.Up, new NavigateMenuCommand(Direction.Up))
    .Bind(Keys.Down, new NavigateMenuCommand(Direction.Down))
    .Bind(Keys.Enter, new SelectMenuItemCommand());

inputManager.PushContext(gameplayContext);

// Undo/redo functionality
var commandHistory = new CommandHistory();
commandHistory.Execute(moveCommand);
commandHistory.Execute(attackCommand);

commandHistory.Undo(); // Undoes attack
commandHistory.Redo(); // Redoes attack
```

## Pattern Implementation Focus

This project demonstrates multiple behavioral patterns:

- **Observer**: Input event notification
- **Command**: Action encapsulation and queuing
- **Chain of Responsibility**: Input handler hierarchy
- **State**: Context-dependent behavior
- **Memento**: Action state capture for undo/redo

## Advanced Features

- **Input Buffering**: Queuing rapid input sequences
- **Gesture Recognition**: Complex input pattern detection
- **Input Recording**: Macro creation and playback
- **Context Stacking**: Hierarchical input contexts
- **Input Validation**: Ensuring valid command sequences

## Contributing

When implementing this project:

1. Use observer pattern for loose coupling between input and actions
2. Implement command pattern with proper encapsulation
3. Design for extensibility through behavioral interfaces
4. Create comprehensive tests for input scenarios
5. Document pattern usage and interaction flows
