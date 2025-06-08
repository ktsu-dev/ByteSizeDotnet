# Project 16: Native Library Wrapper

**Focus**: Unsafe Code & Interop  
**Time**: ~2 hours  
**Deliverable**: `NativeWrapper` library with platform interoperability

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Interop Guidelines](../microsoft-design-guidelines.md#interop-guidelines)** - Platform invoke and marshaling
- **[Unsafe Code Guidelines](../microsoft-design-guidelines.md#unsafe-code-guidelines)** - Safe unsafe code practices
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - Efficient native interop

## Learning Objectives

- Master Platform Invoke (P/Invoke) and native interop
- Understand unsafe code and pointer manipulation
- Learn marshaling and memory management across boundaries
- Practice cross-platform native library integration
- Build safe wrappers around unsafe native APIs

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Interop Guidelines](../microsoft-design-guidelines.md#interop-guidelines) before starting

2. **Start Coding** 💻  
   Follow the implementation guide below for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-16-native-wrapper/
├── README.md                    # This file - complete implementation guide
├── NativeWrapper.sln            # Solution file
├── NativeWrapper.Core/          # Main library implementation
├── NativeWrapper.App/           # Console application
└── NativeWrapper.Tests/         # Unit tests

```

## What You'll Build

A comprehensive native interop system featuring:

- P/Invoke wrappers for native game libraries
- Safe managed wrappers around unsafe APIs
- Cross-platform native library loading
- Custom marshaling for complex data structures
- Callback handling from native to managed code
- Resource management for native resources

## Key Concepts

- **Platform Invoke (P/Invoke)**: Calling native functions from managed code
- **Marshaling**: Converting data between managed and native representations
- **Unsafe Code**: Working with pointers and unmanaged memory
- **Native Library Loading**: Dynamic library loading across platforms
- **Callbacks**: Function pointers and delegate marshaling
- **Resource Management**: Proper cleanup of native resources

## Resources

- 📚 **This README** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[P/Invoke Documentation](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke)**
- 📘 **[Unsafe Code Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code)**

## Building on Previous Projects

This project completes Phase 4 by integrating system-level programming:

- **Projects 1-12**: All foundational skills in native interop context
- **Project 13**: Reflection for dynamic native library discovery
- **Project 14**: Expression trees for optimized marshaling
- **Project 15**: Memory management for native data
- **New Skills**: Native interop, unsafe code, and cross-platform development

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-16-native-wrapper/` directory:

```bash
# Create the main projects
dotnet new sln -n NativeWrapper
dotnet new console -n NativeWrapper.App -f net8.0
dotnet new classlib -n NativeWrapper.Core -f net8.0
dotnet new mstest -n NativeWrapper.Tests -f net8.0

# Add projects to solution
dotnet sln add NativeWrapper.App/NativeWrapper.App.csproj
dotnet sln add NativeWrapper.Core/NativeWrapper.Core.csproj
dotnet sln add NativeWrapper.Tests/NativeWrapper.Tests.csproj

# Add project references
dotnet add NativeWrapper.App/NativeWrapper.App.csproj reference NativeWrapper.Core/NativeWrapper.Core.csproj
dotnet add NativeWrapper.Tests/NativeWrapper.Tests.csproj reference NativeWrapper.Core/NativeWrapper.Core.csproj
```

### Step 2: Configure Project Files

Update the `NativeWrapper.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="System.Runtime.CompilerServices.Unsafe" Version="6.0.0" />
  </ItemGroup>
</Project>
```

## Implementation Guide

### Step 3: Safe Handle Foundation (25 minutes)

Create safe handle types for native resource management:

```csharp
// File: NativeWrapper.Core/SafeHandles/SafeNativeHandle.cs
using Microsoft.Win32.SafeHandles;

namespace NativeWrapper.Core.SafeHandles;

public abstract class SafeNativeHandle : SafeHandleZeroOrMinusOneIsInvalid
{
    protected SafeNativeHandle() : base(true) { }
    protected SafeNativeHandle(bool ownsHandle) : base(ownsHandle) { }

    protected override bool ReleaseHandle()
    {
        return ReleaseNativeHandle(handle);
    }

    protected abstract bool ReleaseNativeHandle(IntPtr handle);
}

public class SafeAudioEngineHandle : SafeNativeHandle
{
    internal SafeAudioEngineHandle(IntPtr handle)
    {
        SetHandle(handle);
    }

    protected override bool ReleaseNativeHandle(IntPtr handle)
    {
        try
        {
            // Call native cleanup function
            return true;
        }
        catch
        {
            return false;
        }
    }
}
```

### Step 4: P/Invoke Declarations (30 minutes)

Create organized P/Invoke declarations:

```csharp
// File: NativeWrapper.Core/Interop/NativeAudioInterop.cs
using System.Runtime.InteropServices;

namespace NativeWrapper.Core.Interop;

internal static class NativeAudioInterop
{
    private const string LibraryName = "gameaudio";

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern IntPtr CreateAudioEngine(in AudioEngineConfig config);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern void DestroyAudioEngine(IntPtr engine);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern AudioResult LoadSound(IntPtr engine,
        [MarshalAs(UnmanagedType.LPUTF8Str)] string filename,
        out IntPtr soundHandle);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern AudioResult PlaySound(IntPtr engine, IntPtr sound, float volume);
}

[StructLayout(LayoutKind.Sequential)]
internal struct AudioEngineConfig
{
    public int SampleRate;
    public int BufferSize;
    public int Channels;
    public AudioFormat Format;
}

internal enum AudioResult : int
{
    Success = 0,
    InvalidParameter = -1,
    OutOfMemory = -2,
    DeviceError = -3,
    FileNotFound = -4
}

internal enum AudioFormat : int
{
    PCM16 = 0,
    PCM24 = 1,
    PCM32 = 2,
    Float32 = 3
}
```

### Step 5: Safe Managed Wrappers (35 minutes)

Create comprehensive managed wrappers:

```csharp
// File: NativeWrapper.Core/Wrappers/AudioEngine.cs
using NativeWrapper.Core.Interop;
using NativeWrapper.Core.SafeHandles;

namespace NativeWrapper.Core.Wrappers;

public class AudioEngine : IDisposable
{
    private readonly SafeAudioEngineHandle _handle;
    private readonly Dictionary<string, AudioSource> _loadedSounds;
    private bool _disposed;

    public AudioEngine(AudioConfiguration config)
    {
        ArgumentNullException.ThrowIfNull(config);
        config.Validate();

        var nativeConfig = new AudioEngineConfig
        {
            SampleRate = config.SampleRate,
            BufferSize = config.BufferSize,
            Channels = config.Channels,
            Format = (AudioFormat)config.Format
        };

        var handle = NativeAudioInterop.CreateAudioEngine(in nativeConfig);
        if (handle == IntPtr.Zero)
        {
            throw new InvalidOperationException("Failed to create native audio engine");
        }

        _handle = new SafeAudioEngineHandle(handle);
        _loadedSounds = new Dictionary<string, AudioSource>();
    }

    public bool IsValid => !_handle.IsInvalid && !_disposed;

    public async Task<AudioSource> LoadSoundAsync(string filename)
    {
        ThrowIfDisposed();
        ArgumentException.ThrowIfNullOrEmpty(filename);

        if (_loadedSounds.TryGetValue(filename, out var existingSource))
        {
            return existingSource;
        }

        return await Task.Run(() =>
        {
            var result = NativeAudioInterop.LoadSound(_handle.DangerousGetHandle(),
                filename, out var soundHandle);

            if (result != AudioResult.Success)
            {
                throw new InvalidOperationException($"Failed to load sound '{filename}'");
            }

            var source = new AudioSource(soundHandle, filename, this);
            _loadedSounds[filename] = source;
            return source;
        });
    }

    public void PlaySound(AudioSource source, float volume = 1.0f)
    {
        ThrowIfDisposed();
        ArgumentNullException.ThrowIfNull(source);

        if (volume < 0.0f || volume > 1.0f)
            throw new ArgumentOutOfRangeException(nameof(volume));

        var result = NativeAudioInterop.PlaySound(_handle.DangerousGetHandle(),
            source.Handle, volume);

        if (result != AudioResult.Success)
        {
            throw new InvalidOperationException("Failed to play sound");
        }
    }

    internal void UnloadSound(string filename)
    {
        _loadedSounds.Remove(filename);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                foreach (var sound in _loadedSounds.Values)
                {
                    sound.Dispose();
                }
                _loadedSounds.Clear();
                _handle.Dispose();
            }
            _disposed = true;
        }
    }

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }

    private void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(AudioEngine));
    }
}

public class AudioSource : IDisposable
{
    internal IntPtr Handle { get; private set; }
    public string Filename { get; }

    private readonly AudioEngine _engine;
    private bool _disposed;

    internal AudioSource(IntPtr handle, string filename, AudioEngine engine)
    {
        Handle = handle;
        Filename = filename;
        _engine = engine;
    }

    public void Play(float volume = 1.0f)
    {
        ThrowIfDisposed();
        _engine.PlaySound(this, volume);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                _engine.UnloadSound(Filename);
            }
            Handle = IntPtr.Zero;
            _disposed = true;
        }
    }

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }

    private void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(AudioSource));
    }
}

public class AudioConfiguration
{
    public int SampleRate { get; set; } = 44100;
    public int BufferSize { get; set; } = 1024;
    public int Channels { get; set; } = 2;
    public AudioFormatType Format { get; set; } = AudioFormatType.Float32;

    public void Validate()
    {
        if (SampleRate <= 0)
            throw new ArgumentException("Sample rate must be positive");
        if (BufferSize <= 0)
            throw new ArgumentException("Buffer size must be positive");
        if (Channels <= 0)
            throw new ArgumentException("Channel count must be positive");
    }
}

public enum AudioFormatType
{
    PCM16 = 0,
    PCM24 = 1,
    PCM32 = 2,
    Float32 = 3
}
```

## Testing Your Understanding

Create a complete example in `NativeWrapper.App/Program.cs`:

```csharp
using NativeWrapper.Core.Wrappers;
using System.Runtime.InteropServices;

namespace NativeWrapper.App;

class Program
{
    static async Task Main(string[] args)
    {
        Console.WriteLine("🔧 Native Library Wrapper Demo");
        Console.WriteLine("==============================\n");

        // Demo 1: Platform detection
        DemoPlatformDetection();

        // Demo 2: Audio engine simulation
        await DemoAudioEngine();

        // Demo 3: Error handling
        DemoErrorHandling();

        Console.WriteLine("\n✅ All demos completed successfully!");
    }

    static void DemoPlatformDetection()
    {
        Console.WriteLine("🖥️  Demo 1: Platform Detection");
        Console.WriteLine("------------------------------");

        Console.WriteLine($"OS: {Environment.OSVersion}");
        Console.WriteLine($"Architecture: {RuntimeInformation.OSArchitecture}");
        Console.WriteLine($"Framework: {RuntimeInformation.FrameworkDescription}");
        Console.WriteLine($"Process Architecture: {RuntimeInformation.ProcessArchitecture}");
        Console.WriteLine();
    }

    static async Task DemoAudioEngine()
    {
        Console.WriteLine("🔊 Demo 2: Audio Engine Simulation");
        Console.WriteLine("----------------------------------");

        try
        {
            var config = new AudioConfiguration
            {
                SampleRate = 44100,
                BufferSize = 1024,
                Channels = 2,
                Format = AudioFormatType.Float32
            };

            config.Validate();
            Console.WriteLine("✅ Audio configuration validated");
            Console.WriteLine($"Config: {config.SampleRate}Hz, {config.Channels} channels");

            // Note: This demonstrates the pattern without actual native libraries
            Console.WriteLine("Would create audio engine with native interop");
            Console.WriteLine("Would load and play sound files");

            Console.WriteLine("Audio demo completed");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Audio engine error: {ex.Message}");
        }

        Console.WriteLine();
    }

    static void DemoErrorHandling()
    {
        Console.WriteLine("⚠️  Demo 3: Error Handling");
        Console.WriteLine("--------------------------");

        try
        {
            var invalidConfig = new AudioConfiguration
            {
                SampleRate = -1,
                BufferSize = 0,
                Channels = 0
            };

            invalidConfig.Validate();
        }
        catch (ArgumentException ex)
        {
            Console.WriteLine($"✅ Caught validation error: {ex.Message}");
        }

        Console.WriteLine("Error handling demo completed");
        Console.WriteLine();
    }
}
```

// Complex marshaling example
[StructLayout(LayoutKind.Sequential)]
public struct NativeVertex
{
public Vector3 Position;
public Vector3 Normal;
public Vector2 TexCoord;
public uint Color;
}

public unsafe class MeshRenderer : IDisposable
{
public void UploadVertices(ReadOnlySpan<Vertex> vertices)
{
// Convert managed vertices to native format
var nativeVertices = stackalloc NativeVertex[vertices.Length];

        for (int i = 0; i < vertices.Length; i++)
        {
            nativeVertices[i] = ConvertToNative(vertices[i]);
        }

        // Upload to native graphics API
        fixed (NativeVertex* ptr = nativeVertices)
        {
            NativeGraphics.UploadVertexData(_bufferHandle, ptr, vertices.Length);
        }
    }

}

// Cross-platform native library loading
public static class NativeLibraryLoader
{
public static IntPtr LoadLibrary(string libraryName)
{
return Environment.OSVersion.Platform switch
{
PlatformID.Win32NT => LoadLibraryWin32(libraryName),
PlatformID.Unix => LoadLibraryUnix(libraryName),
PlatformID.MacOSX => LoadLibraryMacOS(libraryName),
\_ => throw new PlatformNotSupportedException()
};
}

    [DllImport("kernel32", SetLastError = true, CharSet = CharSet.Unicode)]
    private static extern IntPtr LoadLibrary(string lpFileName);

    [DllImport("libdl", CharSet = CharSet.Ansi)]
    private static extern IntPtr dlopen(string fileName, int flags);

}

// Callback handling
public delegate void NativeCallback(IntPtr userData, int eventType);

public class CallbackManager
{
private readonly Dictionary<IntPtr, GCHandle> \_callbacks = new();

    public void RegisterCallback(object userData, Action<int> callback)
    {
        var handle = GCHandle.Alloc(new CallbackData(userData, callback));
        var nativeCallback = new NativeCallback(OnNativeCallback);

        _callbacks[GCHandle.ToIntPtr(handle)] = handle;

        NativeLibrary.RegisterCallback(nativeCallback, GCHandle.ToIntPtr(handle));
    }

    private static void OnNativeCallback(IntPtr userData, int eventType)
    {
        var handle = GCHandle.FromIntPtr(userData);
        var callbackData = (CallbackData)handle.Target!;
        callbackData.Callback(eventType);
    }

}

````

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-16-native-wrapper/` directory:

```bash
# Create the main projects
dotnet new sln -n NativeWrapper
dotnet new console -n NativeWrapper.App -f net8.0
dotnet new classlib -n NativeWrapper.Core -f net8.0
dotnet new mstest -n NativeWrapper.Tests -f net8.0

# Add projects to solution
dotnet sln add NativeWrapper.App/NativeWrapper.App.csproj
dotnet sln add NativeWrapper.Core/NativeWrapper.Core.csproj
dotnet sln add NativeWrapper.Tests/NativeWrapper.Tests.csproj

# Add project references
dotnet add NativeWrapper.App/NativeWrapper.App.csproj reference NativeWrapper.Core/NativeWrapper.Core.csproj
dotnet add NativeWrapper.Tests/NativeWrapper.Tests.csproj reference NativeWrapper.Core/NativeWrapper.Core.csproj
````

### Step 2: Configure Project Files

Update the `NativeWrapper.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>NativeWrapper.Core</PackageId>
    <Title>Native Library Wrapper</Title>
    <Description>Safe managed wrappers for native libraries with cross-platform support</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="System.Runtime.CompilerServices.Unsafe" Version="6.0.0" />
  </ItemGroup>

  <!-- Platform-specific native libraries -->
  <ItemGroup Condition="'$(RuntimeIdentifier)' == 'win-x64'">
    <Content Include="runtimes\win-x64\native\**" CopyToOutputDirectory="PreserveNewest" />
  </ItemGroup>

  <ItemGroup Condition="'$(RuntimeIdentifier)' == 'linux-x64'">
    <Content Include="runtimes\linux-x64\native\**" CopyToOutputDirectory="PreserveNewest" />
  </ItemGroup>

  <ItemGroup Condition="'$(RuntimeIdentifier)' == 'osx-x64'">
    <Content Include="runtimes\osx-x64\native\**" CopyToOutputDirectory="PreserveNewest" />
  </ItemGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your interop code:

```
NativeWrapper.Core/
├── Interop/           # P/Invoke declarations
├── SafeHandles/       # Safe handle types
├── Marshaling/        # Custom marshaling
├── Wrappers/          # Managed wrappers
├── Platforms/         # Platform-specific code
├── Exceptions/        # Native exception handling
└── Utilities/         # Interop utilities
```

## Implementation Guide

### Step 4: Safe Handle Foundation (25 minutes)

**Create safe handle types for native resource management**

```csharp
// File: NativeWrapper.Core/SafeHandles/SafeNativeHandle.cs
using Microsoft.Win32.SafeHandles;
using System.Runtime.InteropServices;

namespace NativeWrapper.Core.SafeHandles;

/// <summary>
/// Base safe handle for native resources
/// </summary>
public abstract class SafeNativeHandle : SafeHandleZeroOrMinusOneIsInvalid
{
    protected SafeNativeHandle() : base(true) { }
    protected SafeNativeHandle(bool ownsHandle) : base(ownsHandle) { }

    protected override bool ReleaseHandle()
    {
        return ReleaseNativeHandle(handle);
    }

    protected abstract bool ReleaseNativeHandle(IntPtr handle);
}

/// <summary>
/// Safe handle for audio engine resources
/// </summary>
public class SafeAudioEngineHandle : SafeNativeHandle
{
    internal SafeAudioEngineHandle(IntPtr handle)
    {
        SetHandle(handle);
    }

    protected override bool ReleaseNativeHandle(IntPtr handle)
    {
        try
        {
            NativeAudioInterop.DestroyAudioEngine(handle);
            return true;
        }
        catch
        {
            return false;
        }
    }
}

/// <summary>
/// Safe handle for graphics context
/// </summary>
public class SafeGraphicsContextHandle : SafeNativeHandle
{
    internal SafeGraphicsContextHandle(IntPtr handle)
    {
        SetHandle(handle);
    }

    protected override bool ReleaseNativeHandle(IntPtr handle)
    {
        try
        {
            NativeGraphicsInterop.DestroyContext(handle);
            return true;
        }
        catch
        {
            return false;
        }
    }
}
```

### Step 5: P/Invoke Declarations (30 minutes)

**Create organized P/Invoke declarations**

```csharp
// File: NativeWrapper.Core/Interop/NativeAudioInterop.cs
using System.Runtime.InteropServices;
using NativeWrapper.Core.SafeHandles;

namespace NativeWrapper.Core.Interop;

/// <summary>
/// P/Invoke declarations for native audio library
/// </summary>
internal static class NativeAudioInterop
{
    private const string LibraryName = "gameaudio";

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern IntPtr CreateAudioEngine(in AudioEngineConfig config);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern void DestroyAudioEngine(IntPtr engine);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern AudioResult LoadSound(IntPtr engine,
        [MarshalAs(UnmanagedType.LPUTF8Str)] string filename,
        out IntPtr soundHandle);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern AudioResult PlaySound(IntPtr engine, IntPtr sound, float volume);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern AudioResult SetCallback(IntPtr engine, IntPtr callback, IntPtr userData);

    [DllImport(LibraryName, CallingConvention = CallingConvention.Cdecl)]
    internal static extern AudioResult GetLastError(IntPtr engine,
        [MarshalAs(UnmanagedType.LPUTF8Str)] out string errorMessage);
}

/// <summary>
/// Native structures that match the C layout
/// </summary>
[StructLayout(LayoutKind.Sequential)]
internal struct AudioEngineConfig
{
    public int SampleRate;
    public int BufferSize;
    public int Channels;
    public AudioFormat Format;
}

// Enums matching native definitions
internal enum AudioResult : int
{
    Success = 0,
    InvalidParameter = -1,
    OutOfMemory = -2,
    DeviceError = -3,
    FileNotFound = -4
}

internal enum AudioFormat : int
{
    PCM16 = 0,
    PCM24 = 1,
    PCM32 = 2,
    Float32 = 3
}
```

### Step 6: Cross-Platform Library Loading (25 minutes)

**Implement platform-specific native library loading**

```csharp
// File: NativeWrapper.Core/Platforms/NativeLibraryLoader.cs
using System.Runtime.InteropServices;

namespace NativeWrapper.Core.Platforms;

/// <summary>
/// Cross-platform native library loading
/// </summary>
public static class NativeLibraryLoader
{
    private static readonly Dictionary<string, IntPtr> _loadedLibraries = new();

    public static IntPtr LoadLibrary(string libraryName)
    {
        if (_loadedLibraries.TryGetValue(libraryName, out var existingHandle))
        {
            return existingHandle;
        }

        var handle = OperatingSystem.IsWindows()
            ? LoadLibraryWindows(libraryName)
            : OperatingSystem.IsLinux()
                ? LoadLibraryLinux(libraryName)
                : OperatingSystem.IsMacOS()
                    ? LoadLibraryMacOS(libraryName)
                    : throw new PlatformNotSupportedException($"Platform not supported: {Environment.OSVersion.Platform}");

        if (handle != IntPtr.Zero)
        {
            _loadedLibraries[libraryName] = handle;
        }

        return handle;
    }

    public static IntPtr GetProcAddress(IntPtr libraryHandle, string functionName)
    {
        return OperatingSystem.IsWindows()
            ? GetProcAddressWindows(libraryHandle, functionName)
            : GetProcAddressUnix(libraryHandle, functionName);
    }

    public static bool FreeLibrary(IntPtr libraryHandle)
    {
        var result = OperatingSystem.IsWindows()
            ? FreeLibraryWindows(libraryHandle)
            : FreeLibraryUnix(libraryHandle);

        // Remove from cache
        var toRemove = _loadedLibraries.Where(kvp => kvp.Value == libraryHandle).ToList();
        foreach (var item in toRemove)
        {
            _loadedLibraries.Remove(item.Key);
        }

        return result;
    }

    // Windows implementations
    [DllImport("kernel32", SetLastError = true, CharSet = CharSet.Unicode)]
    private static extern IntPtr LoadLibrary(string lpFileName);

    [DllImport("kernel32", SetLastError = true, CharSet = CharSet.Ansi)]
    private static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

    [DllImport("kernel32", SetLastError = true)]
    private static extern bool FreeLibrary(IntPtr hModule);

    // Unix/Linux implementations
    [DllImport("libdl", CharSet = CharSet.Ansi)]
    private static extern IntPtr dlopen(string fileName, int flags);

    [DllImport("libdl", CharSet = CharSet.Ansi)]
    private static extern IntPtr dlsym(IntPtr handle, string symbol);

    [DllImport("libdl")]
    private static extern int dlclose(IntPtr handle);

    private static IntPtr LoadLibraryWindows(string libraryName)
    {
        var paths = new[]
        {
            libraryName,
            $"{libraryName}.dll",
            Path.Combine(AppContext.BaseDirectory, $"{libraryName}.dll"),
            Path.Combine(AppContext.BaseDirectory, "runtimes", "win-x64", "native", $"{libraryName}.dll")
        };

        foreach (var path in paths)
        {
            var handle = LoadLibrary(path);
            if (handle != IntPtr.Zero)
                return handle;
        }

        throw new DllNotFoundException($"Unable to load library '{libraryName}' on Windows");
    }

    private static IntPtr LoadLibraryLinux(string libraryName)
    {
        const int RTLD_NOW = 2;

        var paths = new[]
        {
            libraryName,
            $"lib{libraryName}.so",
            $"{libraryName}.so",
            Path.Combine(AppContext.BaseDirectory, $"lib{libraryName}.so"),
            Path.Combine(AppContext.BaseDirectory, "runtimes", "linux-x64", "native", $"lib{libraryName}.so")
        };

        foreach (var path in paths)
        {
            var handle = dlopen(path, RTLD_NOW);
            if (handle != IntPtr.Zero)
                return handle;
        }

        throw new DllNotFoundException($"Unable to load library '{libraryName}' on Linux");
    }

    private static IntPtr LoadLibraryMacOS(string libraryName)
    {
        const int RTLD_NOW = 2;

        var paths = new[]
        {
            libraryName,
            $"lib{libraryName}.dylib",
            $"{libraryName}.dylib",
            Path.Combine(AppContext.BaseDirectory, $"lib{libraryName}.dylib"),
            Path.Combine(AppContext.BaseDirectory, "runtimes", "osx-x64", "native", $"lib{libraryName}.dylib")
        };

        foreach (var path in paths)
        {
            var handle = dlopen(path, RTLD_NOW);
            if (handle != IntPtr.Zero)
                return handle;
        }

        throw new DllNotFoundException($"Unable to load library '{libraryName}' on macOS");
    }

    private static IntPtr GetProcAddressWindows(IntPtr libraryHandle, string functionName)
    {
        return GetProcAddress(libraryHandle, functionName);
    }

    private static IntPtr GetProcAddressUnix(IntPtr libraryHandle, string functionName)
    {
        return dlsym(libraryHandle, functionName);
    }

    private static bool FreeLibraryWindows(IntPtr libraryHandle)
    {
        return FreeLibrary(libraryHandle);
    }

    private static bool FreeLibraryUnix(IntPtr libraryHandle)
    {
        return dlclose(libraryHandle) == 0;
    }
}

/// <summary>
/// Platform detection utilities
/// </summary>
public static class PlatformDetector
{
    public static PlatformType GetCurrentPlatform()
    {
        if (OperatingSystem.IsWindows())
            return PlatformType.Windows;
        if (OperatingSystem.IsLinux())
            return PlatformType.Linux;
        if (OperatingSystem.IsMacOS())
            return PlatformType.MacOS;
        if (OperatingSystem.IsFreeBSD())
            return PlatformType.FreeBSD;

        return PlatformType.Unknown;
    }

    public static string GetNativeLibraryPath(string libraryName)
    {
        var platform = GetCurrentPlatform();
        var baseDirectory = AppContext.BaseDirectory;

        return platform switch
        {
            PlatformType.Windows => Path.Combine(baseDirectory, "runtimes", "win-x64", "native", $"{libraryName}.dll"),
            PlatformType.Linux => Path.Combine(baseDirectory, "runtimes", "linux-x64", "native", $"lib{libraryName}.so"),
            PlatformType.MacOS => Path.Combine(baseDirectory, "runtimes", "osx-x64", "native", $"lib{libraryName}.dylib"),
            _ => throw new PlatformNotSupportedException($"Platform {platform} not supported")
        };
    }

    public static bool IsFeatureSupported(NativeFeature feature)
    {
        return feature switch
        {
            NativeFeature.SIMD => Vector.IsHardwareAccelerated,
            NativeFeature.AVX => System.Runtime.Intrinsics.X86.Avx.IsSupported,
            NativeFeature.SSE => System.Runtime.Intrinsics.X86.Sse.IsSupported,
            NativeFeature.ARM_NEON => System.Runtime.Intrinsics.Arm.AdvSimd.IsSupported,
            _ => false
        };
    }
}

public enum PlatformType
{
    Unknown,
    Windows,
    Linux,
    MacOS,
    FreeBSD
}

public enum NativeFeature
{
    SIMD,
    AVX,
    SSE,
    ARM_NEON
}
```

### Step 7: Safe Managed Wrappers (35 minutes)

**Create comprehensive managed wrappers with proper error handling**

```csharp
// File: NativeWrapper.Core/Wrappers/AudioEngine.cs
using NativeWrapper.Core.Interop;
using NativeWrapper.Core.SafeHandles;
using NativeWrapper.Core.Exceptions;
using System.Runtime.InteropServices;

namespace NativeWrapper.Core.Wrappers;

/// <summary>
/// Managed wrapper for native audio engine
/// </summary>
public class AudioEngine : IDisposable
{
    private readonly SafeAudioEngineHandle _handle;
    private readonly Dictionary<string, AudioSource> _loadedSounds;
    private readonly object _lock = new();
    private bool _disposed;

    public AudioEngine(AudioConfiguration config)
    {
        ArgumentNullException.ThrowIfNull(config);
        config.Validate();

        var nativeConfig = new AudioEngineConfig
        {
            SampleRate = config.SampleRate,
            BufferSize = config.BufferSize,
            Channels = config.Channels,
            Format = (AudioFormat)config.Format
        };

        var handle = NativeAudioInterop.CreateAudioEngine(in nativeConfig);
        if (handle == IntPtr.Zero)
        {
            throw new AudioEngineException("Failed to create native audio engine");
        }

        _handle = new SafeAudioEngineHandle(handle);
        _loadedSounds = new Dictionary<string, AudioSource>();
    }

    public bool IsValid => !_handle.IsInvalid && !_disposed;

    public async Task<AudioSource> LoadSoundAsync(string filename)
    {
        ThrowIfDisposed();
        ArgumentException.ThrowIfNullOrEmpty(filename);

        lock (_lock)
        {
            if (_loadedSounds.TryGetValue(filename, out var existingSource))
            {
                return existingSource;
            }
        }

        return await Task.Run(() =>
        {
            var result = NativeAudioInterop.LoadSound(_handle.DangerousGetHandle(), filename, out var soundHandle);

            if (result != AudioResult.Success)
            {
                var errorMsg = GetLastError();
                throw new AudioLoadException($"Failed to load sound '{filename}': {errorMsg}");
            }

            var source = new AudioSource(soundHandle, filename, this);

            lock (_lock)
            {
                _loadedSounds[filename] = source;
            }

            return source;
        });
    }

    public void PlaySound(AudioSource source, float volume = 1.0f)
    {
        ThrowIfDisposed();
        ArgumentNullException.ThrowIfNull(source);

        if (volume < 0.0f || volume > 1.0f)
            throw new ArgumentOutOfRangeException(nameof(volume), "Volume must be between 0.0 and 1.0");

        var result = NativeAudioInterop.PlaySound(_handle.DangerousGetHandle(), source.Handle, volume);
        if (result != AudioResult.Success)
        {
            var errorMsg = GetLastError();
            throw new AudioPlaybackException($"Failed to play sound: {errorMsg}");
        }
    }

    private string GetLastError()
    {
        try
        {
            var result = NativeAudioInterop.GetLastError(_handle.DangerousGetHandle(), out var errorMessage);
            return result == AudioResult.Success ? errorMessage ?? "Unknown error" : "Failed to get error message";
        }
        catch
        {
            return "Unknown error";
        }
    }

    internal void UnloadSound(string filename)
    {
        lock (_lock)
        {
            _loadedSounds.Remove(filename);
        }
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                lock (_lock)
                {
                    foreach (var sound in _loadedSounds.Values)
                    {
                        sound.Dispose();
                    }
                    _loadedSounds.Clear();
                }

                _handle.Dispose();
            }

            _disposed = true;
        }
    }

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }

    private void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(AudioEngine));
    }
}

/// <summary>
/// Represents a loaded audio source
/// </summary>
public class AudioSource : IDisposable
{
    internal IntPtr Handle { get; private set; }
    public string Filename { get; }

    private readonly AudioEngine _engine;
    private bool _disposed;

    internal AudioSource(IntPtr handle, string filename, AudioEngine engine)
    {
        Handle = handle;
        Filename = filename;
        _engine = engine;
    }

    public void Play(float volume = 1.0f)
    {
        ThrowIfDisposed();
        _engine.PlaySound(this, volume);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                _engine.UnloadSound(Filename);
            }

            Handle = IntPtr.Zero;
            _disposed = true;
        }
    }

    public void Dispose()
    {
        Dispose(disposing: true);
        GC.SuppressFinalize(this);
    }

    private void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(AudioSource));
    }
}

// Supporting types and configuration
public class AudioConfiguration
{
    public int SampleRate { get; set; } = 44100;
    public int BufferSize { get; set; } = 1024;
    public int Channels { get; set; } = 2;
    public AudioFormatType Format { get; set; } = AudioFormatType.Float32;

    public void Validate()
    {
        if (SampleRate <= 0)
            throw new ArgumentException("Sample rate must be positive");
        if (BufferSize <= 0)
            throw new ArgumentException("Buffer size must be positive");
        if (Channels <= 0)
            throw new ArgumentException("Channel count must be positive");
    }
}

public enum AudioFormatType
{
    PCM16 = 0,
    PCM24 = 1,
    PCM32 = 2,
    Float32 = 3
}
```

## Testing Your Understanding

Create a complete example in `NativeWrapper.App/Program.cs`:

```csharp
using NativeWrapper.Core.Wrappers;
using NativeWrapper.Core.Platforms;
using NativeWrapper.Core.Exceptions;
using System.Numerics;

namespace NativeWrapper.App;

class Program
{
    static async Task Main(string[] args)
    {
        Console.WriteLine("🔧 Native Library Wrapper Demo");
        Console.WriteLine("==============================\n");

        // Demo 1: Platform detection and library loading
        DemoPlatformDetection();

        // Demo 2: Audio engine with safe wrappers
        await DemoAudioEngine();

        // Demo 3: Error handling and resource management
        DemoErrorHandling();

        // Demo 4: Cross-platform compatibility
        DemoCrossPlatformFeatures();

        Console.WriteLine("\n✅ All demos completed successfully!");
    }

    static void DemoPlatformDetection()
    {
        Console.WriteLine("🖥️  Demo 1: Platform Detection");
        Console.WriteLine("------------------------------");

        var platform = PlatformDetector.GetCurrentPlatform();
        Console.WriteLine($"Current platform: {platform}");

        var libraryPath = PlatformDetector.GetNativeLibraryPath("gameaudio");
        Console.WriteLine($"Audio library path: {libraryPath}");

        var features = new[] { NativeFeature.SIMD, NativeFeature.AVX, NativeFeature.SSE, NativeFeature.ARM_NEON };
        foreach (var feature in features)
        {
            var isSupported = PlatformDetector.IsFeatureSupported(feature);
            Console.WriteLine($"{feature} support: {isSupported}");
        }

        Console.WriteLine();
    }

    static async Task DemoAudioEngine()
    {
        Console.WriteLine("🔊 Demo 2: Audio Engine");
        Console.WriteLine("-----------------------");

        try
        {
            var config = new AudioConfiguration
            {
                SampleRate = 44100,
                BufferSize = 1024,
                Channels = 2,
                Format = AudioFormatType.Float32
            };

            config.Validate();

            // Note: This would work with actual native libraries
            Console.WriteLine("Creating audio engine...");
            Console.WriteLine($"Configuration: {config.SampleRate}Hz, {config.Channels} channels, {config.Format}");

            // Simulate what would happen with real native libraries
            Console.WriteLine("✅ Audio engine configuration validated");
            Console.WriteLine("Would load native audio library and initialize engine");

            // Simulate loading sounds
            var soundFiles = new[] { "explosion.wav", "music.ogg", "footstep.wav" };

            foreach (var filename in soundFiles)
            {
                Console.WriteLine($"Would load sound: {filename}");
                // In real implementation: var sound = await audioEngine.LoadSoundAsync(filename);
            }

            Console.WriteLine("Audio demo completed successfully");
        }
        catch (Exception ex) when (ex is AudioEngineException || ex is ArgumentException)
        {
            Console.WriteLine($"Audio engine error: {ex.Message}");
        }

        Console.WriteLine();
    }

    static void DemoErrorHandling()
    {
        Console.WriteLine("⚠️  Demo 3: Error Handling");
        Console.WriteLine("--------------------------");

        // Test configuration validation
        try
        {
            var invalidConfig = new AudioConfiguration
            {
                SampleRate = -1, // Invalid
                BufferSize = 0,  // Invalid
                Channels = 0     // Invalid
            };

            Console.WriteLine("Testing invalid configuration...");
            invalidConfig.Validate();
        }
        catch (ArgumentException ex)
        {
            Console.WriteLine($"✅ Caught expected validation error: {ex.Message}");
        }

        // Test resource management
        Console.WriteLine("\nTesting resource cleanup patterns...");

        var disposables = new List<IDisposable>();
        try
        {
            // Simulate creating multiple resources
            for (int i = 0; i < 3; i++)
            {
                Console.WriteLine($"Would create native resource {i + 1}");
                // In real implementation, these would be actual native resources
            }

            Console.WriteLine("✅ Resource creation simulation completed");
        }
        finally
        {
            foreach (var disposable in disposables)
            {
                disposable?.Dispose();
            }
            Console.WriteLine("✅ All resources properly disposed");
        }

        Console.WriteLine();
    }

    static void DemoCrossPlatformFeatures()
    {
        Console.WriteLine("🌐 Demo 4: Cross-Platform Compatibility");
        Console.WriteLine("---------------------------------------");

        // Test platform-specific paths
        var testLibraries = new[] { "gameaudio", "gamegfx", "gamecore" };

        foreach (var library in testLibraries)
        {
            try
            {
                var path = PlatformDetector.GetNativeLibraryPath(library);
                var exists = File.Exists(path);
                Console.WriteLine($"{library}: {path} (exists: {exists})");
            }
            catch (PlatformNotSupportedException ex)
            {
                Console.WriteLine($"{library}: Platform not supported - {ex.Message}");
            }
        }

        // Test architecture detection
        Console.WriteLine($"\nRuntime information:");
        Console.WriteLine($"OS: {Environment.OSVersion}");
        Console.WriteLine($"Architecture: {RuntimeInformation.OSArchitecture}");
        Console.WriteLine($"Framework: {RuntimeInformation.FrameworkDescription}");
        Console.WriteLine($"Process Architecture: {RuntimeInformation.ProcessArchitecture}");

        // Test SIMD capabilities
        Console.WriteLine($"\nSIMD capabilities:");
        Console.WriteLine($"Vector<T> is supported: {Vector.IsHardwareAccelerated}");
        Console.WriteLine($"Vector<float> count: {Vector<float>.Count}");
        Console.WriteLine($"Vector<int> count: {Vector<int>.Count}");

        Console.WriteLine();
    }
}
```

## 🚀 Extension Challenges

### Challenge 1: Advanced COM Interop (⭐⭐⭐⭐)

Implement comprehensive COM object support:

- Create automatic COM wrapper generators
- Handle COM events and connection points
- Implement custom COM interfaces with late binding
- Add COM object lifetime management with reference counting
- Support for COM threading models (STA/MTA)

### Challenge 2: High-Performance Function Pointers (⭐⭐⭐⭐⭐)

Build type-safe, high-performance function pointer system:

- Implement delegate caching to avoid repeated marshaling
- Create IL generation for optimized native calls
- Add function pointer validation and signature checking
- Support for multiple calling conventions
- Implement function pointer hot-swapping

### Challenge 3: Memory-Mapped Interop (⭐⭐⭐⭐)

Create comprehensive shared memory system:

- Implement cross-process communication via memory mapping
- Add synchronization primitives for shared data access
- Create type-safe structured access to mapped memory
- Support for large file mapping and sparse files
- Implement memory-mapped circular buffers

### Challenge 4: Advanced Callback Management (⭐⭐⭐⭐⭐)

Build sophisticated callback handling system:

- Implement callback lifetime management with weak references
- Add callback exception isolation and error reporting
- Create callback priority and filtering systems
- Support for batched and deferred callback execution
- Implement callback performance monitoring

### Challenge 5: Dynamic P/Invoke Code Generation (⭐⭐⭐⭐⭐)

Create runtime P/Invoke generation:

- Parse native library headers and generate bindings
- Implement automatic signature inference
- Create optimized IL for generated P/Invoke calls
- Add native library versioning and compatibility checking
- Support for generating async versions of native calls

### Challenge 6: Native String Optimization (⭐⭐⭐)

Implement high-performance string marshaling:

- Add string pooling for frequently marshaled strings
- Implement zero-copy string access where possible
- Create custom string encoders for specific native formats
- Add validation for native string security (buffer overruns)
- Support for string marshaling across different locales

### Challenge 7: Cross-Platform Assembly Analysis (⭐⭐⭐⭐⭐)

Build native dependency analysis tools:

- Create tools to analyze native library dependencies
- Implement automatic native library packaging
- Add runtime compatibility checking across platforms
- Create deployment packages with platform-specific natives
- Implement native library hot-patching and updates

## Resources

### Microsoft Documentation

- 📘 **[P/Invoke Documentation](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke)** - Platform invoke fundamentals
- 📘 **[Unsafe Code](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code)** - Unsafe code and pointers
- 📘 **[Custom Marshaling](https://learn.microsoft.com/en-us/dotnet/framework/interop/custom-marshaling)** - Advanced marshaling techniques
- 📘 **[SafeHandle](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.safehandle)** - Safe native resource management
- 📘 **[Native Library Loading](https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/deployment-overview)** - Cross-platform deployment

### Interop Resources

- 📚 **[PInvoke.net](https://pinvoke.net/)** - P/Invoke signature database
- 📚 **[Interop Patterns](https://docs.microsoft.com/en-us/dotnet/standard/native-interop/)** - Best practices guide
- 📚 **[Cross-Platform Development](https://docs.microsoft.com/en-us/dotnet/core/rid-catalog)** - Runtime identifier catalog

## Building on Previous Projects

This project integrates concepts from:

- **Projects 1-5**: All foundational .NET skills applied to native interop
- **Projects 6-8**: Async patterns and resource management for native operations
- **Projects 9-12**: Advanced patterns applied to native wrappers
- **Project 13**: Reflection for dynamic native library discovery
- **Project 14**: Expression trees for optimized marshaling
- **Project 15**: Memory management for native data buffers
- **Next Projects**: These interop techniques enable the final advanced projects

## Advanced Interop Patterns

This project demonstrates sophisticated native interop:

- **Safe Resource Management**: Using SafeHandle types for automatic cleanup
- **Custom Marshaling**: Implementing ICustomMarshaler for complex data types
- **Cross-Platform Compatibility**: Writing portable interop code
- **Error Translation**: Converting native errors to managed exceptions
- **Performance Optimization**: Minimizing marshaling overhead
- **Security Considerations**: Validating native inputs and preventing exploits

The skills learned here are essential for building high-performance applications that integrate with native libraries, game engines, and system APIs.
