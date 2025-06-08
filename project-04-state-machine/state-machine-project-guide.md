# Project 4: State Machine for Turn-Based Games

**Focus**: Pattern Matching & Switch Expressions  
**Time**: ~2 hours  
**Deliverable**: Game state management library

## Learning Objectives

By completing this project, you'll understand:

- Modern C# pattern matching syntax
- Switch expressions vs traditional switch statements
- Tuple deconstruction in pattern matching
- Property patterns and `when` clauses
- State machine design patterns
- Immutable state transitions

## Project Overview

Build a state machine library for turn-based games like chess, checkers, or RPG battles. You'll use modern C# pattern matching to handle complex state transitions elegantly, making game logic both readable and maintainable.

## What You'll Build

A state machine library that handles:

- Game states (PlayerTurn, EnemyTurn, Victory, Defeat, Paused)
- State transitions with validation
- Action processing based on current state
- Turn management and player switching
- Game event handling and state history
- Undo/redo functionality

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

## Implementation Guide

### Step 1: Project Setup (10 minutes)

```bash
# Create new class library project
dotnet new classlib -n GameStateMachine
cd GameStateMachine
```

### Step 2: Core State Types (25 minutes)

**Game State Definitions**

```csharp
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
}
```

### Step 4: Advanced Pattern Matching (35 minutes)

**Complex State Logic**

```csharp
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
}
```

**Switch Expression Patterns**

```csharp
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
}
```

### Step 5: State History and Undo (30 minutes)

**Immutable State Management**

```csharp
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
```

## Extension Challenges

1. **Complex Game Rules**: Add chess-like piece movement validation
2. **AI State Machine**: Create an AI player that responds to game states
3. **Multiplayer Support**: Extend to handle more than 2 players
4. **Event System**: Add state change notifications and observers
5. **Save/Load**: Serialize game state to JSON with pattern matching

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

## Next Project Preview

Project 5 introduces LINQ and deferred execution by building a game statistics analyzer. You'll learn how LINQ queries work, the difference between immediate and deferred execution, and how to efficiently process large datasets.

## Further Reading

- [Pattern Matching in C#](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching)
- [Switch Expressions](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/switch-expression)
- [State Machine Pattern](https://en.wikipedia.org/wiki/State_pattern)

---

**Ready to master state transitions? Begin with Step 1!** 🎮
