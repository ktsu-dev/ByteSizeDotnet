# Project 7: Event-Driven Audio Manager

**Focus**: Delegates, Events & Functional Programming  
**Template Provided**: ✅ Ready-to-build solution structure

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Event Design Guidelines](../microsoft-design-guidelines.md#event-design-guidelines)** - Proper event implementation patterns
- **[Delegate Guidelines](../microsoft-design-guidelines.md#delegate-guidelines)** - Delegate usage and naming conventions
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - Efficient event handling
- **[API Design](../microsoft-design-guidelines.md#api-design-guidelines)** - Callback and functional interfaces

## Template Structure Provided

This project includes a complete solution template:

```
src/
├── AudioManager.sln                    # Solution file
├── AudioManager.Core/                  # Main library
│   ├── AudioManager.Core.csproj       # Core library project
│   ├── Events/                         # TODO: Implement audio events
│   ├── Managers/                       # TODO: Implement audio management
│   └── Delegates/                      # TODO: Implement callback delegates
├── AudioManager.App/                   # Console application
│   ├── AudioManager.App.csproj        # Console app project
│   └── Program.cs                      # TODO: Implement demo application
└── AudioManager.Tests/                 # Unit tests
    ├── AudioManager.Tests.csproj       # Test project with MSTest
    └── Tests/                          # TODO: Implement test classes
```

## Learning Objectives

1. **Master Delegates**: Implement custom delegate types and patterns
2. **Design Events**: Create type-safe event systems with proper patterns
3. **Apply Functional Programming**: Use Action, Func, and Predicate delegates
4. **Implement Callbacks**: Asynchronous and synchronous callback patterns
5. **Understand Event Lifecycle**: Subscription, unsubscription, and memory management

## Quick Start

1. **Open the template**: Navigate to `src/` directory
2. **Build solution**: `dotnet build` (should compile successfully)
3. **Run tests**: `dotnet test` (empty tests will pass)
4. **Implement features**: Follow the implementation guide below

## Project Structure

```
project-07-audio-manager/
├── README.md                    # This file
├── audio-manager-project-guide.md # Detailed implementation guide
├── src/                         # Your implementation
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
```

## What You'll Build

A sophisticated audio management system featuring:

- Event-driven audio playback control
- Dynamic volume and effects management
- Audio source tracking and mixing
- Callback-based audio completion handling
- Functional composition of audio effects
- Reactive audio state management

## Key Concepts

- **Delegates**: First-class function types and method references
- **Events**: Publisher-subscriber pattern implementation
- **Action and Func**: Built-in delegate types
- **Lambda Expressions**: Anonymous functions and closures
- **Observer Pattern**: Decoupled event notification
- **Functional Programming**: Composing operations with functions

## Resources

- 📚 **[Detailed Guide](audio-manager-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Events and Delegates Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/)**
- 📘 **[Functional Programming in C#](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/functional-programming-vs-imperative-programming)**

## Building on Previous Projects

This project advances concepts from all previous projects:

- **Project 1-2**: Type design for audio data structures
- **Project 3**: Extension methods for fluent audio API
- **Project 4**: State machines for audio player states
- **Project 5**: LINQ for audio collection processing
- **Project 6**: Async operations for audio loading
- **New Skills**: Event-driven programming and functional patterns

## Example Usage

```csharp
// Event-driven audio control
audioManager.AudioStarted += (sender, e) =>
    Console.WriteLine($"Started playing: {e.AudioClip.Name}");

audioManager.AudioFinished += (sender, e) =>
    Console.WriteLine($"Finished playing: {e.AudioClip.Name}");

// Functional audio effects composition
var processedAudio = audioClip
    .WithVolume(0.8f)
    .WithFadeIn(2.0f)
    .WithReverb(0.3f)
    .WithCallback(clip => Logger.Log($"Processing: {clip.Name}"));

// Delegate-based audio mixing
var mixer = new AudioMixer()
    .AddChannel("music", volume: 0.7f)
    .AddChannel("sfx", volume: 1.0f)
    .OnMixComplete(result => SaveToFile(result));
```

## Functional Programming Focus

This project emphasizes functional programming concepts:

- Using delegates as first-class values
- Composing operations with function chaining
- Implementing higher-order functions
- Creating immutable audio processing pipelines

## Contributing

When implementing this project:

1. Follow .NET event conventions (`EventHandler<T>`)
2. Use appropriate delegate types (`Action`, `Func`)
3. Implement proper event subscription/unsubscription
4. Create composable functional interfaces
5. Write comprehensive tests for event handling

## Implementation Tasks

### Phase 1: Event Foundation

- [ ] Design core audio event types
- [ ] Implement EventHandler patterns
- [ ] Create custom EventArgs classes
- [ ] Build event aggregation system

### Phase 2: Delegate System

- [ ] Define audio processing delegates
- [ ] Implement callback patterns
- [ ] Create functional pipeline architecture
- [ ] Add predicate-based filtering

### Phase 3: Audio Management

- [ ] Build event-driven audio manager
- [ ] Implement playlist with notifications
- [ ] Create volume control with events
- [ ] Add audio state management

### Phase 4: Advanced Features

- [ ] Asynchronous event handling
- [ ] Weak event patterns for memory safety
- [ ] Event subscription management
- [ ] Performance monitoring and metrics

## Building and Testing

```bash
# Build the solution
dotnet build

# Run all tests
dotnet test

# Run the demo application
dotnet run --project AudioManager.App
```

## Success Criteria

- [ ] Events follow Microsoft guidelines for type safety
- [ ] Delegates implement proper functional patterns
- [ ] Memory leaks prevented through proper event management
- [ ] Unit tests achieve >90% code coverage
- [ ] Console demo showcases all audio events
- [ ] Performance meets real-time audio requirements

## Resources

- [Event Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/event)
- [Delegates and Events](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)
- [Func and Action Delegates](https://docs.microsoft.com/en-us/dotnet/api/system.func-1)
- [Weak Event Pattern](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/advanced/weak-event-patterns)

## Building on Previous Projects

- Applies project structure from Project 1
- Uses functional concepts from previous projects
- Introduces event-driven programming
- Prepares for advanced async patterns in future projects

## Contributing

This project follows Microsoft's [C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions) and [.NET Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/).
