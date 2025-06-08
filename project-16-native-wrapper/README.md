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
   See [detailed project guide](native-wrapper-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-16-native-wrapper/
├── README.md                    # This file
├── native-wrapper-project-guide.md # Detailed implementation guide
├── src/                         # Your implementation
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
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

- 📚 **[Detailed Guide](native-wrapper-project-guide.md)** - Complete implementation instructions
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

## Example Usage

```csharp
// P/Invoke declarations for native audio library
public static class NativeAudio
{
    [DllImport("gameaudio", CallingConvention = CallingConvention.Cdecl)]
    public static extern IntPtr CreateAudioEngine(AudioConfig config);

    [DllImport("gameaudio", CallingConvention = CallingConvention.Cdecl)]
    public static extern AudioResult PlaySound(IntPtr engine,
        [MarshalAs(UnmanagedType.LPStr)] string filename, float volume);

    [DllImport("gameaudio", CallingConvention = CallingConvention.Cdecl)]
    public static extern void DestroyAudioEngine(IntPtr engine);
}

// Safe managed wrapper
public class AudioEngine : IDisposable
{
    private IntPtr _nativeHandle;
    private bool _disposed;

    public AudioEngine(AudioConfiguration config)
    {
        var nativeConfig = MarshalConfig(config);
        _nativeHandle = NativeAudio.CreateAudioEngine(nativeConfig);

        if (_nativeHandle == IntPtr.Zero)
            throw new AudioEngineException("Failed to create native audio engine");
    }

    public async Task<AudioSource> LoadSoundAsync(string filename)
    {
        ThrowIfDisposed();

        var result = await Task.Run(() =>
            NativeAudio.LoadSound(_nativeHandle, filename));

        return result.IsSuccess
            ? new AudioSource(result.Handle, this)
            : throw new AudioLoadException($"Failed to load {filename}: {result.Error}");
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed && _nativeHandle != IntPtr.Zero)
        {
            NativeAudio.DestroyAudioEngine(_nativeHandle);
            _nativeHandle = IntPtr.Zero;
        }
        _disposed = true;
    }
}

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
            _ => throw new PlatformNotSupportedException()
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
    private readonly Dictionary<IntPtr, GCHandle> _callbacks = new();

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
```

## Advanced Interop Patterns

This project explores sophisticated native interop scenarios:

- **Custom Marshaling**: Implementing ICustomMarshaler for complex types
- **Function Pointers**: Using delegate types for native callbacks
- **COM Interop**: Working with COM objects and interfaces
- **Memory-Mapped Files**: Sharing data between native and managed code
- **Platform-Specific Code**: Conditional compilation for different platforms

## Safety and Performance

Critical considerations for native interop:

- **Memory Safety**: Preventing buffer overruns and use-after-free
- **Exception Safety**: Handling exceptions across native boundaries
- **Resource Management**: Ensuring proper cleanup of native resources
- **Performance**: Minimizing marshaling overhead
- **Thread Safety**: Managing concurrency across interop boundaries

## Contributing

When implementing this project:

1. Use safe patterns for all native interop operations
2. Implement proper resource management with disposable patterns
3. Handle platform differences gracefully
4. Write comprehensive tests including error scenarios
5. Document native dependencies and platform requirements
