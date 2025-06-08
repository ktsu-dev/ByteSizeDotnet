# Project 4: State Machine Framework

**Focus**: Pattern Matching & Switch Expressions  
**Time**: ~2 hours  
**Deliverable**: Game state management library

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Pattern Matching Guidelines](../microsoft-design-guidelines.md#modern-language-features)** - Using switch expressions and pattern matching
- **[Generic Design](../microsoft-design-guidelines.md#generic-design-guidelines)** - Creating reusable generic state machines
- **[Event Design](../microsoft-design-guidelines.md#event-design)** - Proper event handling patterns
- **[Immutability Guidelines](../microsoft-design-guidelines.md#immutability)** - Immutable state transitions

## Learning Objectives

By completing this project, you'll understand:

- Modern C# pattern matching syntax
- Switch expressions vs traditional switch statements
- Tuple deconstruction in pattern matching
- Property patterns and `when` clauses
- State machine design patterns
- Immutable state transitions
- Generic type constraints and variance
- Event-driven programming patterns

## What You'll Build

A flexible state machine framework featuring:

- Game states (PlayerTurn, EnemyTurn, Victory, Defeat, Paused)
- State transitions with validation
- Action processing based on current state
- Turn management and player switching
- Game event handling and state history
- Undo/redo functionality
- Generic state and trigger definitions
- Pattern matching for state transitions
- Event-driven state change notifications
- Conditional transitions with guards

## Core Concepts to Learn

### Pattern Matching Evolution

```csharp
// Old way: traditional switch
switch (gameState)
{
    case GameState.PlayerTurn:
        return HandlePlayerTurn();
    case GameState.EnemyTurn:
        return HandleEnemyTurn();
}

// New way: switch expressions
var result = gameState switch
{
    GameState.PlayerTurn => HandlePlayerTurn(),
    GameState.EnemyTurn => HandleEnemyTurn(),
    _ => throw new InvalidOperationException()
};
```

### Advanced Pattern Matching

```csharp
// Property patterns, when clauses, tuple deconstruction
var canMove = (player, gameState, timeRemaining) switch
{
    (Player p, GameState.PlayerTurn, > 0) when p.IsAlive => true,
    (_, GameState.Paused, _) => false,
    _ => false
};
```

## Project Setup Instructions

### Step 1: Project Setup (10 minutes)

Navigate to the `project-04-state-machine/src` directory and create the solution:

```bash
# Create the main solution file
dotnet new sln -n StateMachine

# Create the main library project
dotnet new classlib -n StateMachine.Core -f net8.0

# Create the console application
dotnet new console -n StateMachine.App -f net8.0

# Create the test project
dotnet new mstest -n StateMachine.Tests -f net8.0

# Add projects to solution
dotnet sln add StateMachine.Core/StateMachine.Core.csproj
dotnet sln add StateMachine.App/StateMachine.App.csproj
dotnet sln add StateMachine.Tests/StateMachine.Tests.csproj

# Add project references
dotnet add StateMachine.App/StateMachine.App.csproj reference StateMachine.Core/StateMachine.Core.csproj
dotnet add StateMachine.Tests/StateMachine.Tests.csproj reference StateMachine.Core/StateMachine.Core.csproj
```

## Implementation Guide

### Step 2: Core State Types (25 minutes)

**Game State Definitions**

```csharp
namespace StateMachine.Core.Models;

public enum GameState
{
    WaitingForPlayers,
    PlayerTurn,
    EnemyTurn,
    ProcessingAction,
    Victory,
    Defeat,
    Draw,
    Paused,
    GameOver
}

public enum ActionType
{
    Move,
    Attack,
    Defend,
    UseItem,
    EndTurn,
    Pause,
    Resume,
    Quit
}

// Immutable action record
public record GameAction(
    ActionType Type,
    string PlayerId,
    Dictionary<string, object>? Parameters = null,
    DateTime Timestamp = default)
{
    public DateTime Timestamp { get; init; } = Timestamp == default ? DateTime.UtcNow : Timestamp;
}
```

**Game Context (State Container)**

```csharp
namespace StateMachine.Core.Models;

public record GameContext(
    GameState CurrentState,
    string? CurrentPlayerId,
    List<string> PlayerIds,
    Dictionary<string, object> GameData,
    List<GameAction> ActionHistory,
    DateTime LastStateChange)
{
    // Factory method
    public static GameContext CreateNew(IEnumerable<string> playerIds)
    {
        return new GameContext(
            CurrentState: GameState.WaitingForPlayers,
            CurrentPlayerId: null,
            PlayerIds: playerIds.ToList(),
            GameData: new Dictionary<string, object>(),
            ActionHistory: new List<GameAction>(),
            LastStateChange: DateTime.UtcNow
        );
    }
}
```

### Step 3: Pattern Matching State Transitions (40 minutes)

**State Machine Core**

```csharp
namespace StateMachine.Core.Engine;

public class TurnBasedGameStateMachine
{
    public GameContext ProcessAction(GameContext context, GameAction action)
    {
        // Validate action using pattern matching
        var isValidAction = IsValidAction(context, action);
        if (!isValidAction)
        {
            throw new InvalidOperationException($"Action {action.Type} not valid in state {context.CurrentState}");
        }

        // Process action and determine new state
        var newState = DetermineNewState(context, action);
        var newContext = ExecuteStateTransition(context, action, newState);

        return newContext;
    }

    private bool IsValidAction(GameContext context, GameAction action)
    {
        return (context.CurrentState, action.Type, action.PlayerId) switch
        {
            // Player can only act during their turn
            (GameState.PlayerTurn, ActionType.Move or ActionType.Attack or ActionType.UseItem, var playerId)
                when playerId == context.CurrentPlayerId => true,

            // Anyone can pause
            (not GameState.Paused, ActionType.Pause, _) => true,

            // Only resume from paused state
            (GameState.Paused, ActionType.Resume, _) => true,

            // End turn only during player's turn
            (GameState.PlayerTurn, ActionType.EndTurn, var playerId)
                when playerId == context.CurrentPlayerId => true,

            // Quit anytime
            (_, ActionType.Quit, _) => true,

            // Everything else is invalid
            _ => false
        };
    }

    private GameState DetermineNewState(GameContext context, GameAction action)
    {
        return (context.CurrentState, action.Type) switch
        {
            // State transitions using switch expressions
            (GameState.WaitingForPlayers, ActionType.Move) when context.PlayerIds.Count >= 2
                => GameState.PlayerTurn,

            (GameState.PlayerTurn, ActionType.EndTurn)
                => GetNextPlayerState(context),

            (GameState.PlayerTurn, ActionType.Attack)
                => CheckForGameEnd(context, action),

            (not GameState.Paused, ActionType.Pause)
                => GameState.Paused,

            (GameState.Paused, ActionType.Resume)
                => GetPreviousState(context),

            (_, ActionType.Quit)
                => GameState.GameOver,

            // Stay in current state for processing actions
            (var currentState, ActionType.Move or ActionType.UseItem)
                => currentState,

            _ => context.CurrentState
        };
    }

    private GameState GetNextPlayerState(GameContext context)
    {
        var currentIndex = context.PlayerIds.IndexOf(context.CurrentPlayerId ?? "");
        var nextIndex = (currentIndex + 1) % context.PlayerIds.Count;
        return nextIndex == 0 ? GameState.PlayerTurn : GameState.EnemyTurn;
    }

    private GameState CheckForGameEnd(GameContext context, GameAction action)
    {
        // Simplified game end logic - extend as needed
        return context.GameData.ContainsKey("enemyHealth") &&
               (int)context.GameData["enemyHealth"] <= 0
            ? GameState.Victory
            : GameState.ProcessingAction;
    }

    private GameState GetPreviousState(GameContext context)
    {
        // Simplified - return to player turn
        return GameState.PlayerTurn;
    }

    private GameContext ExecuteStateTransition(GameContext context, GameAction action, GameState newState)
    {
        var newHistory = context.ActionHistory.ToList();
        newHistory.Add(action);

        return context with
        {
            CurrentState = newState,
            CurrentPlayerId = DetermineCurrentPlayer(context, newState),
            ActionHistory = newHistory,
            LastStateChange = DateTime.UtcNow
        };
    }

    private string? DetermineCurrentPlayer(GameContext context, GameState newState)
    {
        return newState switch
        {
            GameState.PlayerTurn => context.PlayerIds.FirstOrDefault(),
            GameState.EnemyTurn => context.PlayerIds.Skip(1).FirstOrDefault(),
            _ => context.CurrentPlayerId
        };
    }
}
```

### Step 4: Advanced Pattern Matching (35 minutes)

**Complex State Logic**

```csharp
namespace StateMachine.Core.Analysis;

public class GameStateAnalyzer
{
    // Property patterns and when clauses
    public string GetStateDescription(GameContext context)
    {
        return context switch
        {
            // Property patterns
            { CurrentState: GameState.PlayerTurn, CurrentPlayerId: var playerId }
                => $"It's {playerId}'s turn",

            { CurrentState: GameState.Victory, ActionHistory: var history }
                when history.LastOrDefault() is { Type: ActionType.Attack } lastAction
                => $"Game won by {lastAction.PlayerId} with an attack!",

            // Nested patterns
            { CurrentState: GameState.Paused, GameData: var data }
                when data.ContainsKey("pausedBy")
                => $"Game paused by {data["pausedBy"]}",

            // Relational patterns
            { ActionHistory.Count: > 50 }
                => "This is a long game!",

            { ActionHistory.Count: < 5 }
                => "Game just started",

            _ => $"Current state: {context.CurrentState}"
        };
    }

    // Tuple patterns for complex conditions
    public bool CanPerformAction(GameContext context, ActionType actionType, string playerId)
    {
        return (context.CurrentState, actionType, playerId, context.CurrentPlayerId) switch
        {
            // Tuple deconstruction with when clause
            (GameState.PlayerTurn, ActionType.Move, var player, var currentPlayer)
                when player == currentPlayer && HasMovesRemaining(context, player) => true,

            // Multiple conditions in tuple
            (GameState.PlayerTurn, ActionType.Attack, var player, var currentPlayer)
                when player == currentPlayer && HasTargetsAvailable(context) => true,

            // Range patterns (C# 11+)
            (GameState.PlayerTurn, ActionType.UseItem, var player, var currentPlayer)
                when player == currentPlayer && GetItemCount(context, player) is > 0 and < 10 => true,

            _ => false
        };
    }

    private bool HasMovesRemaining(GameContext context, string playerId)
    {
        // Simplified logic
        return context.GameData.GetValueOrDefault($"{playerId}_moves", 3) is > 0;
    }

    private bool HasTargetsAvailable(GameContext context)
    {
        // Simplified logic
        return context.GameData.ContainsKey("availableTargets");
    }

    private int GetItemCount(GameContext context, string playerId)
    {
        // Simplified logic
        return (int)context.GameData.GetValueOrDefault($"{playerId}_items", 0);
    }
}
```

**Switch Expression Patterns**

```csharp
namespace StateMachine.Core.Rules;

public static class StateTransitionRules
{
    // Complex switch expressions with multiple variables
    public static GameState GetNextState(GameState current, ActionType action, GameContext context)
    {
        return (current, action, context.PlayerIds.Count) switch
        {
            // Multi-variable patterns
            (GameState.WaitingForPlayers, ActionType.Move, >= 2) => GameState.PlayerTurn,
            (GameState.PlayerTurn, ActionType.EndTurn, _) => GameState.EnemyTurn,
            (GameState.EnemyTurn, ActionType.EndTurn, _) => GameState.PlayerTurn,

            // Nested switch expression
            (GameState.PlayerTurn, ActionType.Attack, _) => context.GameData.ContainsKey("enemyHealth") switch
            {
                true when (int)context.GameData["enemyHealth"] <= 0 => GameState.Victory,
                _ => GameState.ProcessingAction
            },

            // Property patterns in switch
            (var state, ActionType.Pause, _) when state != GameState.Paused => GameState.Paused,
            (GameState.Paused, ActionType.Resume, _) => GameState.PlayerTurn, // Simplified

            // Default case
            _ => current
        };
    }

    // Pattern matching with guards
    public static bool IsTerminalState(GameState state) => state switch
    {
        GameState.Victory or GameState.Defeat or GameState.Draw or GameState.GameOver => true,
        _ => false
    };

    // Logical patterns
    public static bool CanActInState(GameState state) => state switch
    {
        GameState.PlayerTurn or GameState.EnemyTurn => true,
        not (GameState.Paused or GameState.GameOver) => true,
        _ => false
    };
}
```

### Step 5: State History and Undo (30 minutes)

**Immutable State Management**

```csharp
namespace StateMachine.Core.History;

public class StateHistoryManager
{
    private readonly Stack<GameContext> _stateHistory = new();

    public GameContext SaveState(GameContext current)
    {
        _stateHistory.Push(current);
        return current;
    }

    public GameContext? UndoLastAction()
    {
        return _stateHistory.Count switch
        {
            0 => null,
            1 => _stateHistory.Peek(), // Don't remove the initial state
            _ => _stateHistory.Pop()
        };
    }

    // Pattern matching for state validation
    public bool CanUndo(GameContext current)
    {
        return (current.CurrentState, _stateHistory.Count) switch
        {
            // Can't undo from terminal states
            (GameState.Victory or GameState.Defeat or GameState.GameOver, _) => false,

            // Can't undo if no history
            (_, 0 or 1) => false,

            // Can undo during normal gameplay
            (GameState.PlayerTurn or GameState.EnemyTurn, > 1) => true,

            _ => false
        };
    }

    public IEnumerable<GameAction> GetActionHistory()
    {
        return _stateHistory.SelectMany(context => context.ActionHistory).Distinct();
    }
}
```

## Modern Pattern Matching Features

### 1. Property Patterns

```csharp
// Access properties in patterns
var message = context switch
{
    { CurrentState: GameState.Victory, CurrentPlayerId: var winner } => $"{winner} wins!",
    { ActionHistory.Count: > 100 } => "Epic battle!",
    _ => "Game in progress"
};
```

### 2. Relational Patterns (C# 9+)

```csharp
var priority = turnNumber switch
{
    < 5 => "Early game",
    >= 5 and < 20 => "Mid game",
    >= 20 => "Late game",
    _ => "Unknown"
};
```

### 3. Logical Patterns

```csharp
var canAct = state switch
{
    GameState.PlayerTurn or GameState.EnemyTurn => true,
    not (GameState.Paused or GameState.GameOver) => true,
    _ => false
};
```

## Testing Your Understanding

Create a console app to test your state machine:

```csharp
using StateMachine.Core.Models;
using StateMachine.Core.Engine;
using StateMachine.Core.Analysis;

var players = new[] { "Alice", "Bob" };
var context = GameContext.CreateNew(players);
var stateMachine = new TurnBasedGameStateMachine();

// Test state transitions
context = stateMachine.ProcessAction(context, new GameAction(ActionType.Move, "Alice"));
Console.WriteLine($"After first move: {context.CurrentState}");

context = stateMachine.ProcessAction(context, new GameAction(ActionType.EndTurn, "Alice"));
Console.WriteLine($"After end turn: {context.CurrentState}, Current player: {context.CurrentPlayerId}");

// Test pattern matching analysis
var analyzer = new GameStateAnalyzer();
Console.WriteLine(analyzer.GetStateDescription(context));
Console.WriteLine($"Alice can attack: {analyzer.CanPerformAction(context, ActionType.Attack, "Alice")}");

// Test switch expressions
var description = context.CurrentState switch
{
    GameState.PlayerTurn => "Player is making their move",
    GameState.EnemyTurn => "Enemy is thinking...",
    GameState.Victory => "Game over - someone won!",
    _ => "Game state unknown"
};
Console.WriteLine(description);
```

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

## Common Pattern Matching Pitfalls

1. **Exhaustive matching**: Always include a default case

   ```csharp
   // ❌ Compiler warning if new enum values are added
   var result = state switch
   {
       GameState.PlayerTurn => "Player's turn",
       GameState.EnemyTurn => "Enemy's turn"
       // Missing cases!
   };

   // ✅ Include default case
   var result = state switch
   {
       GameState.PlayerTurn => "Player's turn",
       GameState.EnemyTurn => "Enemy's turn",
       _ => "Other state"
   };
   ```

2. **Complex conditions**: Break down complex patterns

   ```csharp
   // ❌ Hard to read
   var result = (a, b, c) switch { (var x, var y, var z) when x > 0 && y < 10 && z.IsValid && SomeComplexMethod(x, y, z) => DoSomething(), _ => Default() };

   // ✅ More readable
   var result = (a, b, c) switch
   {
       (> 0, < 10, var z) when z.IsValid && SomeComplexMethod(a, b, z) => DoSomething(),
       _ => Default()
   };
   ```

## Success Criteria

✅ Created a working turn-based game state machine  
✅ Used switch expressions for state transitions  
✅ Implemented property patterns and when clauses  
✅ Applied tuple deconstruction in pattern matching  
✅ Built immutable state transition system  
✅ Created comprehensive action validation  
✅ Implemented state history and undo functionality  
✅ Demonstrated modern C# pattern matching features

## 📚 Essential Documentation

**Master pattern matching by studying Microsoft's guidance:**

### Pattern Matching Fundamentals

- [Pattern Matching Guide](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching) - Core concepts and syntax
- [Switch Expressions](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression) - Modern switch syntax
- [Pattern Types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns) - All available pattern types

### State Machine Design

- [State Pattern](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/state-machine-design) - Design guidelines
- [Records](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) - Immutable state containers
- [Generic Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/generic-design-guidelines) - Creating reusable components

## Resources

- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Pattern Matching Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)**

## Building on Previous Projects

This project advances concepts from earlier projects:

- **Project 1**: Applies type design with generic constraints
- **Project 2**: Uses advanced enumerable patterns
- **Project 3**: Leverages fluent API design patterns
- **New Skills**: Pattern matching and generic framework design

---

**Ready to master state transitions? Begin with Step 1!** 🎮
