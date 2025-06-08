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
├── AudioManager.sln                    # Solution file
├── AudioManager.Core/                  # Main library
│   ├── AudioManager.Core.csproj       # Core library project
│   ├── Events/                         # Audio event definitions
│   ├── Managers/                       # Audio management classes
│   └── Delegates/                      # Callback delegate definitions
├── AudioManager.App/                   # Console application
│   ├── AudioManager.App.csproj        # Console app project
│   └── Program.cs                      # Demo application code
└── AudioManager.Tests/                 # Unit tests
    ├── AudioManager.Tests.csproj       # Test project with MSTest
    └── UnitTest1.cs                    # Test implementations
```

## Learning Objectives

1. **Master Delegates**: Implement custom delegate types and patterns
2. **Design Events**: Create type-safe event systems with proper patterns
3. **Apply Functional Programming**: Use Action, Func, and Predicate delegates
4. **Implement Callbacks**: Asynchronous and synchronous callback patterns
5. **Understand Event Lifecycle**: Subscription, unsubscription, and memory management

## Quick Start

1. **Open the template**: Navigate to project directory
2. **Build solution**: `dotnet build` (should compile successfully)
3. **Run tests**: `dotnet test` (empty tests will pass)
4. **Implement features**: Follow the implementation guide below

## Project Structure

```
project-07-audio-manager/
├── README.md                    # This file
├── AudioManager.sln             # Solution file
├── AudioManager.Core/           # Main library implementation
├── AudioManager.App/            # Console application
└── AudioManager.Tests/          # Unit tests
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

## Project Setup Instructions

### Step 1: Create the Solution Structure (10 minutes)

Navigate to the `project-07-audio-manager/` directory:

```bash
# Create the main solution file
dotnet new sln -n AudioManager

# Create the main library project
dotnet new classlib -n AudioManager.Core -f net8.0

# Create the console application
dotnet new console -n AudioManager.App -f net8.0

# Create the test project
dotnet new mstest -n AudioManager.Tests -f net8.0

# Add projects to solution
dotnet sln add AudioManager.Core/AudioManager.Core.csproj
dotnet sln add AudioManager.App/AudioManager.App.csproj
dotnet sln add AudioManager.Tests/AudioManager.Tests.csproj

# Add project references
dotnet add AudioManager.App/AudioManager.App.csproj reference AudioManager.Core/AudioManager.Core.csproj
dotnet add AudioManager.Tests/AudioManager.Tests.csproj reference AudioManager.Core/AudioManager.Core.csproj
```

### Step 2: Configure Project Files

Update the `AudioManager.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>AudioManager.Core</PackageId>
    <Title>Event-Driven Audio Manager</Title>
    <Description>A library for managing audio with delegates, events and functional programming</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your audio management code:

```
AudioManager.Core/
├── Events/            # Event definitions and args
├── Delegates/         # Delegate type definitions
├── Models/           # Audio data models
├── Managers/         # Audio manager implementations
├── Effects/          # Audio effect processors
└── Extensions/       # Fluent API extensions
```

Create these directories:

```bash
mkdir AudioManager.Core/Events
mkdir AudioManager.Core/Delegates
mkdir AudioManager.Core/Models
mkdir AudioManager.Core/Managers
mkdir AudioManager.Core/Effects
mkdir AudioManager.Core/Extensions
```

## Implementation Guide

### Step 4: Core Audio Models (20 minutes)

**Audio Data Structures**

```csharp
// Core audio models
namespace AudioManager.Core.Models;

public enum AudioState
{
    Stopped,
    Playing,
    Paused,
    Loading,
    Error
}

public enum AudioFormat
{
    WAV,
    MP3,
    OGG,
    FLAC
}

public class AudioClip
{
    public string Name { get; set; } = string.Empty;
    public string FilePath { get; set; } = string.Empty;
    public TimeSpan Duration { get; set; }
    public AudioFormat Format { get; set; }
    public float Volume { get; set; } = 1.0f;
    public bool IsLooping { get; set; }
    public DateTime LastPlayed { get; set; }
    public int PlayCount { get; set; }

    public AudioClip(string name, string filePath, TimeSpan duration, AudioFormat format)
    {
        Name = name;
        FilePath = filePath;
        Duration = duration;
        Format = format;
    }

    public override string ToString() => $"{Name} ({Duration:mm\\:ss})";
}

public class AudioChannel
{
    public string Name { get; set; }
    public float Volume { get; set; } = 1.0f;
    public bool IsMuted { get; set; }
    public List<AudioClip> PlayingClips { get; set; } = new();
    public Dictionary<string, float> Effects { get; set; } = new();

    public AudioChannel(string name)
    {
        Name = name;
    }

    public float GetEffectiveVolume() => IsMuted ? 0f : Volume;
}

public class PlaybackInfo
{
    public AudioClip Clip { get; set; }
    public TimeSpan Position { get; set; }
    public AudioState State { get; set; }
    public float CurrentVolume { get; set; }
    public DateTime StartTime { get; set; }

    public PlaybackInfo(AudioClip clip)
    {
        Clip = clip;
        State = AudioState.Stopped;
        CurrentVolume = clip.Volume;
    }

    public TimeSpan RemainingTime => Clip.Duration - Position;
    public float ProgressPercentage => Position.TotalSeconds / Clip.Duration.TotalSeconds * 100f;
}
```

### Step 5: Event System Design (30 minutes)

**Custom Events and EventArgs**

```csharp
// Event definitions
namespace AudioManager.Core.Events;

// Custom EventArgs classes
public class AudioEventArgs : EventArgs
{
    public AudioClip Clip { get; }
    public DateTime Timestamp { get; }

    public AudioEventArgs(AudioClip clip)
    {
        Clip = clip;
        Timestamp = DateTime.UtcNow;
    }
}

public class AudioStartedEventArgs : AudioEventArgs
{
    public string Channel { get; }
    public float Volume { get; }

    public AudioStartedEventArgs(AudioClip clip, string channel, float volume) : base(clip)
    {
        Channel = channel;
        Volume = volume;
    }
}

public class AudioFinishedEventArgs : AudioEventArgs
{
    public bool CompletedNaturally { get; }
    public TimeSpan PlayDuration { get; }

    public AudioFinishedEventArgs(AudioClip clip, bool completedNaturally, TimeSpan playDuration) : base(clip)
    {
        CompletedNaturally = completedNaturally;
        PlayDuration = playDuration;
    }
}

public class AudioErrorEventArgs : AudioEventArgs
{
    public Exception Exception { get; }
    public string ErrorMessage { get; }

    public AudioErrorEventArgs(AudioClip clip, Exception exception) : base(clip)
    {
        Exception = exception;
        ErrorMessage = exception.Message;
    }
}

public class VolumeChangedEventArgs : EventArgs
{
    public string Target { get; } // Channel name or "Master"
    public float OldVolume { get; }
    public float NewVolume { get; }
    public DateTime Timestamp { get; }

    public VolumeChangedEventArgs(string target, float oldVolume, float newVolume)
    {
        Target = target;
        OldVolume = oldVolume;
        NewVolume = newVolume;
        Timestamp = DateTime.UtcNow;
    }
}

public class EffectAppliedEventArgs : EventArgs
{
    public AudioClip Clip { get; }
    public string EffectName { get; }
    public Dictionary<string, object> Parameters { get; }

    public EffectAppliedEventArgs(AudioClip clip, string effectName, Dictionary<string, object> parameters)
    {
        Clip = clip;
        EffectName = effectName;
        Parameters = parameters;
    }
}
```

### Step 6: Delegate Types and Functional Programming (25 minutes)

**Custom Delegate Definitions**

```csharp
// Delegate definitions
namespace AudioManager.Core.Delegates;

// Audio processing delegates
public delegate void AudioProcessorDelegate(AudioClip clip, float[] audioData);
public delegate bool AudioFilterDelegate(AudioClip clip);
public delegate float AudioEffectDelegate(float sample, float time, Dictionary<string, object> parameters);
public delegate void AudioCompletionCallback(AudioClip clip, bool success);

// Higher-order function delegates
public delegate T AudioTransformDelegate<T>(T input);
public delegate AudioClip AudioClipModifier(AudioClip clip);
public delegate bool AudioValidationPredicate(AudioClip clip);

// Event handler delegates (using standard EventHandler pattern)
public delegate void AudioEventHandler(object sender, AudioEventArgs e);
public delegate void VolumeEventHandler(object sender, VolumeChangedEventArgs e);

// Functional composition helpers
public static class AudioDelegateExtensions
{
    // Compose audio effect functions
    public static AudioEffectDelegate Compose(this AudioEffectDelegate first, AudioEffectDelegate second)
    {
        return (sample, time, parameters) =>
        {
            var intermediate = first(sample, time, parameters);
            return second(intermediate, time, parameters);
        };
    }

    // Chain audio processors
    public static AudioProcessorDelegate Chain(this AudioProcessorDelegate first, AudioProcessorDelegate second)
    {
        return (clip, data) =>
        {
            first(clip, data);
            second(clip, data);
        };
    }

    // Combine audio filters with AND logic
    public static AudioFilterDelegate And(this AudioFilterDelegate first, AudioFilterDelegate second)
    {
        return clip => first(clip) && second(clip);
    }

    // Combine audio filters with OR logic
    public static AudioFilterDelegate Or(this AudioFilterDelegate first, AudioFilterDelegate second)
    {
        return clip => first(clip) || second(clip);
    }
}

// Pre-built effect delegates
public static class AudioEffects
{
    public static AudioEffectDelegate VolumeEffect(float volume) =>
        (sample, time, parameters) => sample * volume;

    public static AudioEffectDelegate FadeInEffect(float duration) =>
        (sample, time, parameters) =>
        {
            var fadeMultiplier = Math.Min(time / duration, 1.0f);
            return sample * fadeMultiplier;
        };

    public static AudioEffectDelegate FadeOutEffect(float duration, float totalDuration) =>
        (sample, time, parameters) =>
        {
            var fadeStart = totalDuration - duration;
            if (time < fadeStart) return sample;
            var fadeMultiplier = 1.0f - ((time - fadeStart) / duration);
            return sample * Math.Max(fadeMultiplier, 0.0f);
        };

    public static AudioEffectDelegate LowPassFilter(float cutoffFrequency) =>
        (sample, time, parameters) =>
        {
            // Simplified low-pass filter implementation
            var alpha = cutoffFrequency / (cutoffFrequency + 1.0f);
            return sample * alpha;
        };
}
```

### Step 7: Event-Driven Audio Manager (35 minutes)

**Main Audio Manager with Events**

```csharp
using AudioManager.Core.Events;
using AudioManager.Core.Models;
using AudioManager.Core.Delegates;

namespace AudioManager.Core.Managers;

public class EventDrivenAudioManager : IDisposable
{
    private readonly Dictionary<string, AudioChannel> _channels;
    private readonly Dictionary<AudioClip, PlaybackInfo> _playbackInfos;
    private readonly List<AudioProcessorDelegate> _globalProcessors;
    private readonly Timer _updateTimer;

    private float _masterVolume = 1.0f;
    private bool _isDisposed;

    // Events following Microsoft guidelines
    public event EventHandler<AudioStartedEventArgs>? AudioStarted;
    public event EventHandler<AudioFinishedEventArgs>? AudioFinished;
    public event EventHandler<AudioErrorEventArgs>? AudioError;
    public event EventHandler<VolumeChangedEventArgs>? VolumeChanged;
    public event EventHandler<EffectAppliedEventArgs>? EffectApplied;

    // Functional event aggregation
    public event Action<AudioClip>? OnAnyAudioEvent;

    public EventDrivenAudioManager()
    {
        _channels = new Dictionary<string, AudioChannel>
        {
            ["Master"] = new AudioChannel("Master"),
            ["Music"] = new AudioChannel("Music"),
            ["SFX"] = new AudioChannel("SFX"),
            ["Voice"] = new AudioChannel("Voice")
        };

        _playbackInfos = new Dictionary<AudioClip, PlaybackInfo>();
        _globalProcessors = new List<AudioProcessorDelegate>();

        // Update timer for state management
        _updateTimer = new Timer(UpdatePlaybackStates, null, TimeSpan.Zero, TimeSpan.FromMilliseconds(100));
    }

    public async Task<bool> PlayAsync(AudioClip clip, string channelName = "SFX", AudioCompletionCallback? callback = null)
    {
        ArgumentNullException.ThrowIfNull(clip);

        try
        {
            if (!_channels.TryGetValue(channelName, out var channel))
            {
                throw new ArgumentException($"Channel '{channelName}' not found", nameof(channelName));
            }

            // Create playback info
            var playbackInfo = new PlaybackInfo(clip)
            {
                State = AudioState.Loading,
                StartTime = DateTime.UtcNow
            };

            _playbackInfos[clip] = playbackInfo;

            // Simulate async audio loading
            await LoadAudioAsync(clip);

            // Start playback
            playbackInfo.State = AudioState.Playing;
            playbackInfo.CurrentVolume = clip.Volume * channel.GetEffectiveVolume() * _masterVolume;

            channel.PlayingClips.Add(clip);
            clip.PlayCount++;
            clip.LastPlayed = DateTime.UtcNow;

            // Raise events
            var startedArgs = new AudioStartedEventArgs(clip, channelName, playbackInfo.CurrentVolume);
            AudioStarted?.Invoke(this, startedArgs);
            OnAnyAudioEvent?.Invoke(clip);

            // Set up completion callback
            if (callback != null)
            {
                SetupCompletionCallback(clip, callback);
            }

            return true;
        }
        catch (Exception ex)
        {
            var errorArgs = new AudioErrorEventArgs(clip, ex);
            AudioError?.Invoke(this, errorArgs);
            return false;
        }
    }

    public void Stop(AudioClip clip)
    {
        ArgumentNullException.ThrowIfNull(clip);

        if (_playbackInfos.TryGetValue(clip, out var playbackInfo))
        {
            var wasPlaying = playbackInfo.State == AudioState.Playing;
            playbackInfo.State = AudioState.Stopped;

            // Remove from all channels
            foreach (var channel in _channels.Values)
            {
                channel.PlayingClips.Remove(clip);
            }

            if (wasPlaying)
            {
                var finishedArgs = new AudioFinishedEventArgs(clip, false, DateTime.UtcNow - playbackInfo.StartTime);
                AudioFinished?.Invoke(this, finishedArgs);
                OnAnyAudioEvent?.Invoke(clip);
            }

            _playbackInfos.Remove(clip);
        }
    }

    public void SetChannelVolume(string channelName, float volume)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(channelName);
        ArgumentOutOfRangeException.ThrowIfNegative(volume);

        if (_channels.TryGetValue(channelName, out var channel))
        {
            var oldVolume = channel.Volume;
            channel.Volume = Math.Clamp(volume, 0f, 1f);

            var volumeChangedArgs = new VolumeChangedEventArgs(channelName, oldVolume, channel.Volume);
            VolumeChanged?.Invoke(this, volumeChangedArgs);

            // Update volume for all playing clips in this channel
            UpdateChannelPlayingVolumes(channel);
        }
    }

    public void SetMasterVolume(float volume)
    {
        ArgumentOutOfRangeException.ThrowIfNegative(volume);

        var oldVolume = _masterVolume;
        _masterVolume = Math.Clamp(volume, 0f, 1f);

        var volumeChangedArgs = new VolumeChangedEventArgs("Master", oldVolume, _masterVolume);
        VolumeChanged?.Invoke(this, volumeChangedArgs);

        // Update volume for all playing clips
        foreach (var channel in _channels.Values)
        {
            UpdateChannelPlayingVolumes(channel);
        }
    }

    // Functional programming: Apply effects using delegates
    public void ApplyEffect(AudioClip clip, string effectName, AudioEffectDelegate effect, Dictionary<string, object>? parameters = null)
    {
        ArgumentNullException.ThrowIfNull(clip);
        ArgumentException.ThrowIfNullOrWhiteSpace(effectName);
        ArgumentNullException.ThrowIfNull(effect);

        parameters ??= new Dictionary<string, object>();

        if (_playbackInfos.ContainsKey(clip))
        {
            // Apply effect to audio data (simplified)
            ProcessAudioWithEffect(clip, effect, parameters);

            var effectArgs = new EffectAppliedEventArgs(clip, effectName, parameters);
            EffectApplied?.Invoke(this, effectArgs);
        }
    }

    // Add global audio processor using delegates
    public void AddGlobalProcessor(AudioProcessorDelegate processor)
    {
        ArgumentNullException.ThrowIfNull(processor);
        _globalProcessors.Add(processor);
    }

    public void RemoveGlobalProcessor(AudioProcessorDelegate processor)
    {
        _globalProcessors.Remove(processor);
    }

    // Get currently playing clips using functional filtering
    public IEnumerable<AudioClip> GetPlayingClips(AudioFilterDelegate? filter = null)
    {
        var playingClips = _playbackInfos
            .Where(kvp => kvp.Value.State == AudioState.Playing)
            .Select(kvp => kvp.Key);

        return filter != null ? playingClips.Where(filter) : playingClips;
    }

    private async Task LoadAudioAsync(AudioClip clip)
    {
        // Simulate async audio file loading
        await Task.Delay(Random.Shared.Next(50, 200));

        // In real implementation, load audio file data here
        Console.WriteLine($"Loaded audio: {clip.Name}");
    }

    private void SetupCompletionCallback(AudioClip clip, AudioCompletionCallback callback)
    {
        // Monitor clip completion and invoke callback
        Task.Run(async () =>
        {
            while (_playbackInfos.ContainsKey(clip) && _playbackInfos[clip].State == AudioState.Playing)
            {
                await Task.Delay(100);

                if (_playbackInfos.TryGetValue(clip, out var info))
                {
                    info.Position = DateTime.UtcNow - info.StartTime;

                    if (info.Position >= clip.Duration)
                    {
                        // Natural completion
                        info.State = AudioState.Stopped;
                        callback(clip, true);

                        var finishedArgs = new AudioFinishedEventArgs(clip, true, info.Position);
                        AudioFinished?.Invoke(this, finishedArgs);

                        break;
                    }
                }
            }
        });
    }

    private void UpdatePlaybackStates(object? state)
    {
        var completedClips = new List<AudioClip>();

        foreach (var (clip, info) in _playbackInfos.ToList())
        {
            if (info.State == AudioState.Playing)
            {
                info.Position = DateTime.UtcNow - info.StartTime;

                if (info.Position >= clip.Duration)
                {
                    completedClips.Add(clip);
                }
            }
        }

        // Handle completed clips
        foreach (var clip in completedClips)
        {
            if (!clip.IsLooping)
            {
                Stop(clip);
            }
            else
            {
                // Restart for looping clips
                _playbackInfos[clip].StartTime = DateTime.UtcNow;
                _playbackInfos[clip].Position = TimeSpan.Zero;
            }
        }
    }

    private void UpdateChannelPlayingVolumes(AudioChannel channel)
    {
        foreach (var clip in channel.PlayingClips)
        {
            if (_playbackInfos.TryGetValue(clip, out var info))
            {
                info.CurrentVolume = clip.Volume * channel.GetEffectiveVolume() * _masterVolume;
            }
        }
    }

    private void ProcessAudioWithEffect(AudioClip clip, AudioEffectDelegate effect, Dictionary<string, object> parameters)
    {
        // Simplified audio processing
        // In real implementation, process actual audio samples
        var audioData = new float[1024]; // Simulated audio buffer

        for (int i = 0; i < audioData.Length; i++)
        {
            var time = i / 44100f; // Assume 44.1kHz sample rate
            audioData[i] = effect(audioData[i], time, parameters);
        }

        // Apply global processors
        foreach (var processor in _globalProcessors)
        {
            processor(clip, audioData);
        }
    }

    public void Dispose()
    {
        if (_isDisposed) return;

        _updateTimer?.Dispose();

        // Stop all playing audio
        foreach (var clip in _playbackInfos.Keys.ToList())
        {
            Stop(clip);
        }

        _playbackInfos.Clear();
        _channels.Clear();
        _globalProcessors.Clear();

        _isDisposed = true;
    }
}
```

## Key .NET Event and Delegate Patterns

### 1. Proper Event Design

```csharp
// ✅ Follow Microsoft event guidelines
public event EventHandler<AudioStartedEventArgs>? AudioStarted;

// ✅ Use custom EventArgs for rich event data
public class AudioStartedEventArgs : EventArgs
{
    public AudioClip Clip { get; }
    public string Channel { get; }
    public float Volume { get; }
    // ... constructor and additional properties
}

// ❌ Don't use raw delegates for events
public Action<AudioClip>? OnAudioStarted; // Not recommended for public events
```

### 2. Functional Composition with Delegates

```csharp
// ✅ Compose audio effects functionally
var reverbEffect = AudioEffects.LowPassFilter(1000f)
    .Compose(AudioEffects.VolumeEffect(0.8f))
    .Compose(AudioEffects.FadeInEffect(2.0f));

// ✅ Chain processors
var masterProcessor = normalizeProcessor
    .Chain(dynamicRangeProcessor)
    .Chain(outputLimiter);
```

### 3. Memory Management with Events

```csharp
// ✅ Proper event subscription/unsubscription
public void SubscribeToAudioEvents(EventDrivenAudioManager manager)
{
    manager.AudioStarted += OnAudioStarted;
    manager.AudioFinished += OnAudioFinished;
}

public void UnsubscribeFromAudioEvents(EventDrivenAudioManager manager)
{
    manager.AudioStarted -= OnAudioStarted;
    manager.AudioFinished -= OnAudioFinished;
}

// ✅ Use weak references for long-lived subscriptions if needed
```

## Testing Your Understanding

Create a complete audio management demo:

```csharp
using AudioManager.Core.Managers;
using AudioManager.Core.Models;
using AudioManager.Core.Delegates;
using AudioManager.Core.Events;

// Create audio manager and clips
var audioManager = new EventDrivenAudioManager();

var bgMusic = new AudioClip("Background Music", "bgmusic.mp3", TimeSpan.FromMinutes(3), AudioFormat.MP3)
{
    IsLooping = true
};

var jumpSound = new AudioClip("Jump", "jump.wav", TimeSpan.FromMilliseconds(500), AudioFormat.WAV);
var explosionSound = new AudioClip("Explosion", "explosion.wav", TimeSpan.FromSeconds(2), AudioFormat.WAV);

// Subscribe to events
audioManager.AudioStarted += (sender, e) =>
    Console.WriteLine($"🎵 Started: {e.Clip.Name} on {e.Channel} at volume {e.Volume:F2}");

audioManager.AudioFinished += (sender, e) =>
    Console.WriteLine($"🔇 Finished: {e.Clip.Name} (completed naturally: {e.CompletedNaturally})");

audioManager.VolumeChanged += (sender, e) =>
    Console.WriteLine($"🔊 Volume changed on {e.Target}: {e.OldVolume:F2} → {e.NewVolume:F2}");

audioManager.EffectApplied += (sender, e) =>
    Console.WriteLine($"✨ Effect applied: {e.EffectName} to {e.Clip.Name}");

// Play audio with callbacks
await audioManager.PlayAsync(bgMusic, "Music", (clip, success) =>
    Console.WriteLine($"Background music {'started' + (success ? "" : " failed")}"));

await audioManager.PlayAsync(jumpSound, "SFX");

// Apply effects using functional composition
var fadeEffect = AudioEffects.FadeInEffect(1.0f).Compose(AudioEffects.VolumeEffect(0.7f));
audioManager.ApplyEffect(explosionSound, "FadeInVolume", fadeEffect);

await audioManager.PlayAsync(explosionSound, "SFX");

// Demonstrate volume control
audioManager.SetChannelVolume("Music", 0.3f);
audioManager.SetMasterVolume(0.8f);

// Functional filtering
var musicFilter = new AudioFilterDelegate(clip => clip.Name.Contains("Music"));
var sfxFilter = new AudioFilterDelegate(clip => clip.Duration < TimeSpan.FromSeconds(5));

var playingMusic = audioManager.GetPlayingClips(musicFilter);
var playingShortSounds = audioManager.GetPlayingClips(sfxFilter);

Console.WriteLine($"Playing music tracks: {playingMusic.Count()}");
Console.WriteLine($"Playing short sounds: {playingShortSounds.Count()}");

// Clean up
audioManager.Dispose();
```

## Extension Challenges

1. **Audio Visualization**: Create delegates for real-time audio spectrum analysis
2. **Dynamic Effects**: Build a system for chaining and modifying effects in real-time
3. **Spatial Audio**: Implement 3D positional audio with distance-based volume
4. **Audio Scripting**: Create a domain-specific language for audio sequences
5. **Performance Monitoring**: Track audio processing performance with delegates
6. **Plugin System**: Design a plugin architecture using delegates and events
7. **Audio Synthesis**: Build procedural audio generation using functional composition

## Common Pitfalls & Solutions

1. **Memory Leaks with Events**:

   ```csharp
   // ❌ Forgetting to unsubscribe
   manager.AudioStarted += OnAudioStarted;
   // ... later, manager goes out of scope but subscriber remains

   // ✅ Always unsubscribe or use weak references
   try
   {
       manager.AudioStarted += OnAudioStarted;
       // Use the manager...
   }
   finally
   {
       manager.AudioStarted -= OnAudioStarted;
   }
   ```

2. **Thread Safety with Events**:

   ```csharp
   // ✅ Safe event invocation
   private void OnAudioEvent(AudioEventArgs args)
   {
       AudioStarted?.Invoke(this, args); // Null-conditional operator is thread-safe
   }

   // ❌ Not thread-safe
   if (AudioStarted != null)
       AudioStarted(this, args); // AudioStarted could become null between check and call
   ```

3. **Exception Handling in Delegates**:

   ```csharp
   // ✅ Handle exceptions in delegate chains
   public void InvokeProcessors(AudioClip clip, float[] data)
   {
       foreach (var processor in _processors)
       {
           try
           {
               processor(clip, data);
           }
           catch (Exception ex)
           {
               // Log error but continue with other processors
               Console.WriteLine($"Processor error: {ex.Message}");
           }
       }
   }
   ```

4. **Delegate Performance**:

   ```csharp
   // ✅ Cache compiled delegates for performance
   private static readonly AudioEffectDelegate CachedVolumeEffect = AudioEffects.VolumeEffect(0.8f);

   // ❌ Creating delegates in hot paths
   for (int i = 0; i < 1000000; i++)
   {
       var effect = AudioEffects.VolumeEffect(0.8f); // Creates new delegate each time
   }
   ```

## Resources

- 📚 **[Progress Tracker](../progress-tracker.md)** - Track your learning progress
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
