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
   Follow the implementation guide below for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-11-input-manager/
├── README.md                    # This file - complete implementation guide
├── InputManager.sln             # Solution file
├── InputManager.Core/           # Main library implementation
├── InputManager.App/            # Console application
└── InputManager.Tests/          # Unit tests
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
```

## What You'll Build

A comprehensive input management system featuring:

- **Observer Pattern** - Decoupled input event distribution and notification
- **Command Pattern** - Encapsulating player actions as executable commands
- **Input Binding System** - Flexible key mapping and runtime reconfiguration
- **Command History** - Undo/redo functionality with memento pattern
- **Input Contexts** - Switching between different input modes (game, menu, dialog)
- **Macro Recording** - Recording and replaying complex input sequences
- **Input Validation** - Chain of responsibility for input filtering
- **Real-Time Processing** - Buffered input handling for responsive gameplay

## Key Concepts

### Observer Pattern

```csharp
// Input events notify multiple listeners without tight coupling
public event EventHandler<KeyPressedEventArgs>? KeyPressed;
public event EventHandler<MouseMovedEventArgs>? MouseMoved;

// Multiple systems can observe the same input
inputManager.KeyPressed += soundSystem.PlayKeySound;
inputManager.KeyPressed += uiSystem.HandleKeyInput;
inputManager.KeyPressed += gameSystem.ProcessGameInput;
```

### Command Pattern

```csharp
// Encapsulate actions as objects for flexibility
public interface ICommand
{
    void Execute();
    void Undo();
    bool CanExecute();
}

// Different actions become interchangeable commands
ICommand moveCommand = new MovePlayerCommand(player, Vector2.Up);
ICommand attackCommand = new AttackCommand(player, target);
inputManager.BindKey(Keys.W, moveCommand);
```

### Chain of Responsibility

```csharp
// Input flows through a chain of handlers
public abstract class InputHandler
{
    protected InputHandler? _nextHandler;
    public abstract bool Handle(InputEvent inputEvent);
}

// UI -> Game -> System level handling
var chain = new UIInputHandler()
    .SetNext(new GameInputHandler())
    .SetNext(new SystemInputHandler());
```

### Core Behavioral Patterns Applied

- **Observer** - Event-driven input distribution
- **Command** - Action encapsulation and queuing
- **Chain of Responsibility** - Hierarchical input processing
- **State** - Context-dependent input behavior
- **Memento** - Action state capture for undo/redo
- **Strategy** - Pluggable input processing algorithms

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-11-input-manager/` directory:

```bash
# Create the main solution file
dotnet new sln -n InputManager

# Create the main library project
dotnet new classlib -n InputManager.Core -f net8.0

# Create the console application
dotnet new console -n InputManager.App -f net8.0

# Create the test project
dotnet new mstest -n InputManager.Tests -f net8.0

# Add projects to solution
dotnet sln add InputManager.Core/InputManager.Core.csproj
dotnet sln add InputManager.App/InputManager.App.csproj
dotnet sln add InputManager.Tests/InputManager.Tests.csproj

# Add project references
dotnet add InputManager.App/InputManager.App.csproj reference InputManager.Core/InputManager.Core.csproj
dotnet add InputManager.Tests/InputManager.Tests.csproj reference InputManager.Core/InputManager.Core.csproj
```

### Step 2: Configure Project Files

Update the `InputManager.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>InputManager.Core</PackageId>
    <Title>Game Input Manager</Title>
    <Description>A library for game input management using behavioral design patterns</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your input management code:

```
InputManager.Core/
├── Commands/         # Command pattern implementations
├── Events/           # Event argument types and handlers
├── Contexts/         # Input context management
├── Bindings/         # Key binding and mapping
├── History/          # Command history and undo/redo
├── Handlers/         # Chain of responsibility handlers
├── Devices/          # Input device abstractions
└── Validation/       # Input validation and filtering
```

Create these directories:

```bash
mkdir InputManager.Core/Commands
mkdir InputManager.Core/Events
mkdir InputManager.Core/Contexts
mkdir InputManager.Core/Bindings
mkdir InputManager.Core/History
mkdir InputManager.Core/Handlers
mkdir InputManager.Core/Devices
mkdir InputManager.Core/Validation
```

## Implementation Guide

### Step 4: Core Input Types and Events (25 minutes)

**Foundation Types and Event Arguments**

```csharp
namespace InputManager.Core.Events;

public enum InputDeviceType
{
    Keyboard,
    Mouse,
    Gamepad,
    Touch
}

public enum KeyState
{
    Released,
    Pressed,
    Held
}

public enum MouseButton
{
    Left,
    Right,
    Middle,
    X1,
    X2
}

public readonly struct Vector2
{
    public float X { get; }
    public float Y { get; }

    public Vector2(float x, float y)
    {
        X = x;
        Y = y;
    }

    public static Vector2 Zero => new(0, 0);
    public static Vector2 Up => new(0, 1);
    public static Vector2 Down => new(0, -1);
    public static Vector2 Left => new(-1, 0);
    public static Vector2 Right => new(1, 0);

    public static Vector2 operator +(Vector2 a, Vector2 b) => new(a.X + b.X, a.Y + b.Y);
    public static Vector2 operator -(Vector2 a, Vector2 b) => new(a.X - b.X, a.Y - b.Y);
    public static Vector2 operator *(Vector2 a, float scalar) => new(a.X * scalar, a.Y * scalar);

    public override string ToString() => $"({X:F2}, {Y:F2})";
}

// Event argument classes for Observer pattern
public abstract class InputEventArgs : EventArgs
{
    public DateTime Timestamp { get; }
    public InputDeviceType DeviceType { get; }
    public bool Handled { get; set; }

    protected InputEventArgs(InputDeviceType deviceType)
    {
        Timestamp = DateTime.UtcNow;
        DeviceType = deviceType;
    }
}

public class KeyEventArgs : InputEventArgs
{
    public Keys Key { get; }
    public KeyState State { get; }
    public bool IsRepeat { get; }
    public ModifierKeys Modifiers { get; }

    public KeyEventArgs(Keys key, KeyState state, ModifierKeys modifiers = ModifierKeys.None, bool isRepeat = false)
        : base(InputDeviceType.Keyboard)
    {
        Key = key;
        State = state;
        Modifiers = modifiers;
        IsRepeat = isRepeat;
    }
}

public class MouseEventArgs : InputEventArgs
{
    public Vector2 Position { get; }
    public Vector2 Delta { get; }
    public MouseButton? Button { get; }
    public KeyState ButtonState { get; }
    public float ScrollDelta { get; }

    public MouseEventArgs(Vector2 position, Vector2 delta = default, MouseButton? button = null,
        KeyState buttonState = KeyState.Released, float scrollDelta = 0)
        : base(InputDeviceType.Mouse)
    {
        Position = position;
        Delta = delta;
        Button = button;
        ButtonState = buttonState;
        ScrollDelta = scrollDelta;
    }
}

[Flags]
public enum ModifierKeys
{
    None = 0,
    Control = 1,
    Alt = 2,
    Shift = 4,
    Windows = 8
}

// Simplified Keys enum (in real implementation, use system-provided enum)
public enum Keys
{
    None = 0,
    A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z,
    D0, D1, D2, D3, D4, D5, D6, D7, D8, D9,
    F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, F11, F12,
    Space, Enter, Escape, Tab, Backspace, Delete,
    Up, Down, Left, Right,
    LeftShift, RightShift, LeftControl, RightControl, LeftAlt, RightAlt
}
```

### Step 5: Command Pattern Implementation (35 minutes)

**Core Command Interface and Base Classes**

```csharp
namespace InputManager.Core.Commands;

// Command Pattern - Encapsulate actions as objects
public interface ICommand
{
    string Name { get; }
    bool CanExecute();
    void Execute();
    void Undo();
    ICommand Clone();
}

public interface IUndoableCommand : ICommand
{
    bool CanUndo { get; }
    object? Memento { get; }
}

public abstract class Command : IUndoableCommand
{
    protected Command(string name)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
        Id = Guid.NewGuid();
        Timestamp = DateTime.UtcNow;
    }

    public string Name { get; }
    public Guid Id { get; }
    public DateTime Timestamp { get; }
    public object? Memento { get; protected set; }

    public virtual bool CanExecute() => true;
    public virtual bool CanUndo => Memento != null;

    public abstract void Execute();
    public abstract void Undo();
    public abstract ICommand Clone();

    public override string ToString() => $"{Name} ({Id:N})";
}

// Null Object Pattern for commands
public class NullCommand : ICommand
{
    public static readonly NullCommand Instance = new();

    private NullCommand() { }

    public string Name => "No Operation";
    public bool CanExecute() => false;
    public void Execute() { }
    public void Undo() { }
    public ICommand Clone() => Instance;
}

// Composite command for macro support
public class MacroCommand : Command
{
    private readonly List<ICommand> _commands = new();

    public MacroCommand(string name) : base(name) { }

    public IReadOnlyList<ICommand> Commands => _commands;

    public void AddCommand(ICommand command)
    {
        ArgumentNullException.ThrowIfNull(command);
        _commands.Add(command);
    }

    public override bool CanExecute() => _commands.All(c => c.CanExecute());

    public override void Execute()
    {
        foreach (var command in _commands)
        {
            if (command.CanExecute())
            {
                command.Execute();
            }
        }
    }

    public override void Undo()
    {
        // Undo in reverse order
        for (int i = _commands.Count - 1; i >= 0; i--)
        {
            var command = _commands[i];
            if (command is IUndoableCommand undoable && undoable.CanUndo)
            {
                command.Undo();
            }
        }
    }

    public override ICommand Clone()
    {
        var clone = new MacroCommand(Name);
        foreach (var command in _commands)
        {
            clone.AddCommand(command.Clone());
        }
        return clone;
    }
}

// Game-specific command examples
public class MovePlayerCommand : Command
{
    private readonly IPlayer _player;
    private readonly Vector2 _direction;
    private Vector2 _previousPosition;

    public MovePlayerCommand(IPlayer player, Vector2 direction)
        : base($"Move {direction}")
    {
        _player = player ?? throw new ArgumentNullException(nameof(player));
        _direction = direction;
    }

    public override bool CanExecute() => _player.CanMove(_direction);

    public override void Execute()
    {
        _previousPosition = _player.Position;
        _player.Move(_direction);
        Memento = _previousPosition; // Store state for undo
    }

    public override void Undo()
    {
        if (Memento is Vector2 position)
        {
            _player.SetPosition(position);
        }
    }

    public override ICommand Clone() => new MovePlayerCommand(_player, _direction);
}

public class AttackCommand : Command
{
    private readonly IPlayer _player;
    private readonly ITarget? _target;
    private AttackMemento? _attackMemento;

    public AttackCommand(IPlayer player, ITarget? target = null)
        : base("Attack")
    {
        _player = player ?? throw new ArgumentNullException(nameof(player));
        _target = target;
    }

    public override bool CanExecute() => _player.CanAttack() && (_target?.IsValid ?? true);

    public override void Execute()
    {
        var damage = _player.AttackDamage;
        var criticalHit = _player.RollCritical();

        _attackMemento = new AttackMemento
        {
            Target = _target,
            Damage = damage,
            CriticalHit = criticalHit,
            PlayerStamina = _player.Stamina
        };

        _player.ConsumeStamina(10); // Attack costs stamina
        _target?.TakeDamage(damage, criticalHit);

        Memento = _attackMemento;
    }

    public override void Undo()
    {
        if (_attackMemento != null)
        {
            _player.RestoreStamina(_attackMemento.PlayerStamina);
            _attackMemento.Target?.RestoreHealth(_attackMemento.Damage);
        }
    }

    public override ICommand Clone() => new AttackCommand(_player, _target);

    private class AttackMemento
    {
        public ITarget? Target { get; set; }
        public int Damage { get; set; }
        public bool CriticalHit { get; set; }
        public int PlayerStamina { get; set; }
    }
}

// Command factory for creating commands from input
public interface ICommandFactory
{
    ICommand CreateCommand(string commandName, params object[] parameters);
    bool CanCreateCommand(string commandName);
    IEnumerable<string> SupportedCommands { get; }
}

public class GameCommandFactory : ICommandFactory
{
    private readonly Dictionary<string, Func<object[], ICommand>> _commandCreators = new();
    private readonly IPlayer _player;

    public GameCommandFactory(IPlayer player)
    {
        _player = player ?? throw new ArgumentNullException(nameof(player));
        RegisterDefaultCommands();
    }

    public IEnumerable<string> SupportedCommands => _commandCreators.Keys;

    public ICommand CreateCommand(string commandName, params object[] parameters)
    {
        if (_commandCreators.TryGetValue(commandName.ToLower(), out var creator))
        {
            return creator(parameters);
        }

        return NullCommand.Instance;
    }

    public bool CanCreateCommand(string commandName) =>
        _commandCreators.ContainsKey(commandName.ToLower());

    private void RegisterDefaultCommands()
    {
        _commandCreators["move"] = args =>
        {
            var direction = args.Length > 0 ? (Vector2)args[0] : Vector2.Zero;
            return new MovePlayerCommand(_player, direction);
        };

        _commandCreators["attack"] = args =>
        {
            var target = args.Length > 0 ? args[0] as ITarget : null;
            return new AttackCommand(_player, target);
        };

        _commandCreators["jump"] = _ => new JumpCommand(_player);
        _commandCreators["interact"] = _ => new InteractCommand(_player);
    }
}

// Additional game commands
public class JumpCommand : Command
{
    private readonly IPlayer _player;

    public JumpCommand(IPlayer player) : base("Jump")
    {
        _player = player ?? throw new ArgumentNullException(nameof(player));
    }

    public override bool CanExecute() => _player.IsGrounded && _player.Stamina >= 5;

    public override void Execute()
    {
        Memento = new { Stamina = _player.Stamina, WasGrounded = _player.IsGrounded };
        _player.Jump();
        _player.ConsumeStamina(5);
    }

    public override void Undo()
    {
        if (Memento is { Stamina: int stamina })
        {
            _player.RestoreStamina(stamina);
            _player.Land(); // Force land
        }
    }

    public override ICommand Clone() => new JumpCommand(_player);
}
```

### Step 6: Observer Pattern for Input Events (30 minutes)

**Input Manager with Event Distribution**

```csharp
namespace InputManager.Core;

// Observer Pattern - Input event distribution
public interface IInputManager
{
    // Keyboard events
    event EventHandler<KeyEventArgs>? KeyPressed;
    event EventHandler<KeyEventArgs>? KeyReleased;
    event EventHandler<KeyEventArgs>? KeyHeld;

    // Mouse events
    event EventHandler<MouseEventArgs>? MouseMoved;
    event EventHandler<MouseEventArgs>? MouseButtonPressed;
    event EventHandler<MouseEventArgs>? MouseButtonReleased;
    event EventHandler<MouseEventArgs>? MouseScrolled;

    // Input processing
    void ProcessInput();
    void Update(TimeSpan deltaTime);

    // Input state queries
    bool IsKeyPressed(Keys key);
    bool IsKeyHeld(Keys key);
    bool IsMouseButtonPressed(MouseButton button);
    Vector2 MousePosition { get; }
}

public class InputManager : IInputManager
{
    private readonly Dictionary<Keys, KeyState> _keyStates = new();
    private readonly Dictionary<MouseButton, KeyState> _mouseButtonStates = new();
    private readonly Dictionary<Keys, DateTime> _keyPressTimestamps = new();
    private readonly Queue<InputEventArgs> _inputQueue = new();
    private readonly object _inputLock = new();

    private Vector2 _mousePosition = Vector2.Zero;
    private Vector2 _previousMousePosition = Vector2.Zero;

    // Keyboard events
    public event EventHandler<KeyEventArgs>? KeyPressed;
    public event EventHandler<KeyEventArgs>? KeyReleased;
    public event EventHandler<KeyEventArgs>? KeyHeld;

    // Mouse events
    public event EventHandler<MouseEventArgs>? MouseMoved;
    public event EventHandler<MouseEventArgs>? MouseButtonPressed;
    public event EventHandler<MouseEventArgs>? MouseButtonReleased;
    public event EventHandler<MouseEventArgs>? MouseScrolled;

    public Vector2 MousePosition => _mousePosition;

    public void ProcessInput()
    {
        lock (_inputLock)
        {
            while (_inputQueue.Count > 0)
            {
                var inputEvent = _inputQueue.Dequeue();
                ProcessInputEvent(inputEvent);
            }
        }
    }

    public void Update(TimeSpan deltaTime)
    {
        ProcessHeldKeys(deltaTime);
        UpdateMouseDelta();
    }

    // Input state queries
    public bool IsKeyPressed(Keys key) =>
        _keyStates.TryGetValue(key, out var state) && state == KeyState.Pressed;

    public bool IsKeyHeld(Keys key) =>
        _keyStates.TryGetValue(key, out var state) && state == KeyState.Held;

    public bool IsMouseButtonPressed(MouseButton button) =>
        _mouseButtonStates.TryGetValue(button, out var state) && state == KeyState.Pressed;

    // Simulate input injection (in real implementation, this would come from OS)
    public void InjectKeyInput(Keys key, KeyState state, ModifierKeys modifiers = ModifierKeys.None)
    {
        var eventArgs = new KeyEventArgs(key, state, modifiers);
        lock (_inputLock)
        {
            _inputQueue.Enqueue(eventArgs);
        }
    }

    public void InjectMouseInput(Vector2 position, MouseButton? button = null,
        KeyState buttonState = KeyState.Released, float scrollDelta = 0)
    {
        var delta = position - _mousePosition;
        var eventArgs = new MouseEventArgs(position, delta, button, buttonState, scrollDelta);

        lock (_inputLock)
        {
            _inputQueue.Enqueue(eventArgs);
        }
    }

    private void ProcessInputEvent(InputEventArgs inputEvent)
    {
        switch (inputEvent)
        {
            case KeyEventArgs keyEvent:
                ProcessKeyEvent(keyEvent);
                break;
            case MouseEventArgs mouseEvent:
                ProcessMouseEvent(mouseEvent);
                break;
        }
    }

    private void ProcessKeyEvent(KeyEventArgs keyEvent)
    {
        var previousState = _keyStates.GetValueOrDefault(keyEvent.Key, KeyState.Released);
        _keyStates[keyEvent.Key] = keyEvent.State;

        switch (keyEvent.State)
        {
            case KeyState.Pressed when previousState == KeyState.Released:
                _keyPressTimestamps[keyEvent.Key] = DateTime.UtcNow;
                OnKeyPressed(keyEvent);
                break;
            case KeyState.Released when previousState != KeyState.Released:
                _keyPressTimestamps.Remove(keyEvent.Key);
                OnKeyReleased(keyEvent);
                break;
        }
    }

    private void ProcessMouseEvent(MouseEventArgs mouseEvent)
    {
        if (mouseEvent.Position != _mousePosition)
        {
            _previousMousePosition = _mousePosition;
            _mousePosition = mouseEvent.Position;
            OnMouseMoved(mouseEvent);
        }

        if (mouseEvent.Button.HasValue)
        {
            var button = mouseEvent.Button.Value;
            var previousState = _mouseButtonStates.GetValueOrDefault(button, KeyState.Released);
            _mouseButtonStates[button] = mouseEvent.ButtonState;

            switch (mouseEvent.ButtonState)
            {
                case KeyState.Pressed when previousState == KeyState.Released:
                    OnMouseButtonPressed(mouseEvent);
                    break;
                case KeyState.Released when previousState != KeyState.Released:
                    OnMouseButtonReleased(mouseEvent);
                    break;
            }
        }

        if (mouseEvent.ScrollDelta != 0)
        {
            OnMouseScrolled(mouseEvent);
        }
    }

    private void ProcessHeldKeys(TimeSpan deltaTime)
    {
        var currentTime = DateTime.UtcNow;
        var heldThreshold = TimeSpan.FromMilliseconds(500); // 0.5 seconds

        foreach (var (key, timestamp) in _keyPressTimestamps.ToList())
        {
            if (currentTime - timestamp > heldThreshold && _keyStates[key] == KeyState.Pressed)
            {
                _keyStates[key] = KeyState.Held;
                var heldEvent = new KeyEventArgs(key, KeyState.Held);
                OnKeyHeld(heldEvent);
            }
        }
    }

    private void UpdateMouseDelta()
    {
        if (_mousePosition != _previousMousePosition)
        {
            var delta = _mousePosition - _previousMousePosition;
            var deltaEvent = new MouseEventArgs(_mousePosition, delta);
            OnMouseMoved(deltaEvent);
        }
    }

    // Event firing methods (Observer Pattern)
    protected virtual void OnKeyPressed(KeyEventArgs e) => KeyPressed?.Invoke(this, e);
    protected virtual void OnKeyReleased(KeyEventArgs e) => KeyReleased?.Invoke(this, e);
    protected virtual void OnKeyHeld(KeyEventArgs e) => KeyHeld?.Invoke(this, e);
    protected virtual void OnMouseMoved(MouseEventArgs e) => MouseMoved?.Invoke(this, e);
    protected virtual void OnMouseButtonPressed(MouseEventArgs e) => MouseButtonPressed?.Invoke(this, e);
    protected virtual void OnMouseButtonReleased(MouseEventArgs e) => MouseButtonReleased?.Invoke(this, e);
    protected virtual void OnMouseScrolled(MouseEventArgs e) => MouseScrolled?.Invoke(this, e);
}

// Observable input processor with filtering
public class ObservableInputProcessor : IDisposable
{
    private readonly IInputManager _inputManager;
    private readonly List<IInputObserver> _observers = new();
    private readonly List<IInputFilter> _filters = new();

    public ObservableInputProcessor(IInputManager inputManager)
    {
        _inputManager = inputManager ?? throw new ArgumentNullException(nameof(inputManager));
        SubscribeToInputEvents();
    }

    public void Subscribe(IInputObserver observer)
    {
        ArgumentNullException.ThrowIfNull(observer);
        if (!_observers.Contains(observer))
        {
            _observers.Add(observer);
        }
    }

    public void Unsubscribe(IInputObserver observer)
    {
        _observers.Remove(observer);
    }

    public void AddFilter(IInputFilter filter)
    {
        ArgumentNullException.ThrowIfNull(filter);
        _filters.Add(filter);
    }

    private void SubscribeToInputEvents()
    {
        _inputManager.KeyPressed += OnKeyPressed;
        _inputManager.KeyReleased += OnKeyReleased;
        _inputManager.MouseButtonPressed += OnMouseButtonPressed;
        _inputManager.MouseMoved += OnMouseMoved;
    }

    private void OnKeyPressed(object? sender, KeyEventArgs e)
    {
        if (ApplyFilters(e))
        {
            NotifyObservers(observer => observer.OnKeyPressed(e));
        }
    }

    private void OnKeyReleased(object? sender, KeyEventArgs e)
    {
        if (ApplyFilters(e))
        {
            NotifyObservers(observer => observer.OnKeyReleased(e));
        }
    }

    private void OnMouseButtonPressed(object? sender, MouseEventArgs e)
    {
        if (ApplyFilters(e))
        {
            NotifyObservers(observer => observer.OnMouseButtonPressed(e));
        }
    }

    private void OnMouseMoved(object? sender, MouseEventArgs e)
    {
        if (ApplyFilters(e))
        {
            NotifyObservers(observer => observer.OnMouseMoved(e));
        }
    }

    private bool ApplyFilters(InputEventArgs inputEvent)
    {
        return _filters.All(filter => filter.ShouldProcess(inputEvent));
    }

    private void NotifyObservers(Action<IInputObserver> action)
    {
        foreach (var observer in _observers.ToList()) // ToList to avoid modification during iteration
        {
            try
            {
                action(observer);
            }
            catch (Exception ex)
            {
                // Log exception but continue notifying other observers
                Console.WriteLine($"Error notifying observer: {ex.Message}");
            }
        }
    }

    public void Dispose()
    {
        _inputManager.KeyPressed -= OnKeyPressed;
        _inputManager.KeyReleased -= OnKeyReleased;
        _inputManager.MouseButtonPressed -= OnMouseButtonPressed;
        _inputManager.MouseMoved -= OnMouseMoved;
        _observers.Clear();
        _filters.Clear();
    }
}

// Observer interface for input events
public interface IInputObserver
{
    void OnKeyPressed(KeyEventArgs e);
    void OnKeyReleased(KeyEventArgs e);
    void OnMouseButtonPressed(MouseEventArgs e);
    void OnMouseMoved(MouseEventArgs e);
}

// Filter interface for input processing
public interface IInputFilter
{
    bool ShouldProcess(InputEventArgs inputEvent);
}

// Concrete observer implementation
public class GameInputObserver : IInputObserver
{
    private readonly ICommandFactory _commandFactory;

    public GameInputObserver(ICommandFactory commandFactory)
    {
        _commandFactory = commandFactory ?? throw new ArgumentNullException(nameof(commandFactory));
    }

    public void OnKeyPressed(KeyEventArgs e)
    {
        var command = MapKeyToCommand(e.Key);
        if (command.CanExecute())
        {
            command.Execute();
        }
    }

    public void OnKeyReleased(KeyEventArgs e)
    {
        // Handle key release actions if needed
    }

    public void OnMouseButtonPressed(MouseEventArgs e)
    {
        if (e.Button == MouseButton.Left)
        {
            var attackCommand = _commandFactory.CreateCommand("attack");
            if (attackCommand.CanExecute())
            {
                attackCommand.Execute();
            }
        }
    }

    public void OnMouseMoved(MouseEventArgs e)
    {
        // Handle mouse movement (camera, cursor, etc.)
    }

    private ICommand MapKeyToCommand(Keys key)
    {
        return key switch
        {
            Keys.W => _commandFactory.CreateCommand("move", Vector2.Up),
            Keys.S => _commandFactory.CreateCommand("move", Vector2.Down),
            Keys.A => _commandFactory.CreateCommand("move", Vector2.Left),
            Keys.D => _commandFactory.CreateCommand("move", Vector2.Right),
            Keys.Space => _commandFactory.CreateCommand("jump"),
            Keys.E => _commandFactory.CreateCommand("interact"),
            _ => NullCommand.Instance
        };
    }
}
```

## Testing Your Understanding

Create a console app to test your input manager:

```csharp
using InputManager.Core;
using InputManager.Core.Commands;
using InputManager.Core.Events;

// Create a simple player implementation
public class SimplePlayer : IPlayer
{
    public Vector2 Position { get; private set; } = Vector2.Zero;
    public int Stamina { get; private set; } = 100;
    public bool IsGrounded { get; private set; } = true;
    public int AttackDamage => 10;

    public bool CanMove(Vector2 direction) => true;
    public bool CanAttack() => Stamina >= 10;

    public void Move(Vector2 direction)
    {
        Position += direction;
        Console.WriteLine($"Player moved to {Position}");
    }

    public void SetPosition(Vector2 position)
    {
        Position = position;
        Console.WriteLine($"Player position set to {Position}");
    }

    public void Jump()
    {
        IsGrounded = false;
        Console.WriteLine("Player jumped!");
    }

    public void Land()
    {
        IsGrounded = true;
        Console.WriteLine("Player landed.");
    }

    public void ConsumeStamina(int amount)
    {
        Stamina = Math.Max(0, Stamina - amount);
        Console.WriteLine($"Stamina: {Stamina}");
    }

    public void RestoreStamina(int amount)
    {
        Stamina = amount;
        Console.WriteLine($"Stamina restored to: {Stamina}");
    }

    public bool RollCritical() => false; // Simplified
}

// Test the input system
static void Main(string[] args)
{
    var inputManager = new InputManager();
    var player = new SimplePlayer();
    var commandFactory = new GameCommandFactory(player);
    var inputObserver = new GameInputObserver(commandFactory);

    using var inputProcessor = new ObservableInputProcessor(inputManager);
    inputProcessor.Subscribe(inputObserver);

    Console.WriteLine("Input Manager Test - Press keys to trigger commands:");
    Console.WriteLine("W/A/S/D - Move, Space - Jump, E - Interact, Q - Quit");

    // Simulate input events
    inputManager.InjectKeyInput(Keys.W, KeyState.Pressed);
    inputManager.ProcessInput();

    inputManager.InjectKeyInput(Keys.Space, KeyState.Pressed);
    inputManager.ProcessInput();

    inputManager.InjectKeyInput(Keys.A, KeyState.Pressed);
    inputManager.ProcessInput();

    Console.WriteLine("\nInput processing complete!");
}
```

## Extension Challenges

1. **Input Recording & Playback**: Implement macro recording system
2. **Context Stack Management**: Hierarchical input contexts with push/pop
3. **Gesture Recognition**: Complex input pattern detection
4. **Input Buffering**: Advanced input timing and buffering
5. **Customizable Key Bindings**: Runtime key remapping with persistence
6. **Multi-Device Support**: Gamepad and touch input integration
7. **Input Analytics**: Track input patterns and player behavior

## Resources

- 📚 **This README** - Complete implementation instructions
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
