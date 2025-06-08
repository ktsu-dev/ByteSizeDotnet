# Project 15: High-Performance Game Buffer

**Focus**: Memory Management & Spans  
**Time**: ~2 hours  
**Deliverable**: `GameBuffer` library with advanced memory management

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Memory Management Guidelines](../microsoft-design-guidelines.md#memory-management)** - Efficient memory usage patterns
- **[Span and Memory Guidelines](../microsoft-design-guidelines.md#span-memory-guidelines)** - Modern memory APIs
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - Zero-allocation programming

## Learning Objectives

- Master Span<T>, Memory<T>, and ReadOnlySpan<T>
- Understand stack vs heap allocation strategies
- Learn zero-allocation programming techniques
- Practice unsafe code and pointer arithmetic
- Build high-performance, memory-efficient systems

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Memory Management Guidelines](../microsoft-design-guidelines.md#memory-management) before starting

2. **Start Coding** 💻  
   Follow the implementation guide below for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-15-game-buffer/
├── README.md                    # This file - complete implementation guide
├── GameBuffer.sln               # Solution file
├── GameBuffer.Core/             # Main library implementation
├── GameBuffer.App/              # Console application
└── GameBuffer.Tests/            # Unit tests
├── tests/                       # Unit tests
└── docs/                        # Additional documentation
```

## What You'll Build

A high-performance buffer management system featuring:

- Span<T> and Memory<T> based buffer operations
- Stack allocation for small, temporary buffers
- Custom memory allocators and pooling
- Zero-allocation string and binary operations
- SIMD-optimized buffer processing
- Memory mapping for large data sets

## Key Concepts

- **Span<T> and Memory<T>**: Modern memory abstraction types
- **Stack Allocation**: Using stackalloc for performance
- **Unsafe Code**: Pointer arithmetic and unmanaged memory
- **Memory Pooling**: Reusing buffer allocations
- **SIMD**: Single Instruction, Multiple Data optimization
- **Memory Mapping**: Efficient large file handling

## Resources

- 📚 **This README** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Span and Memory Documentation](https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/)**
- 📘 **[Unsafe Code Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code)**

## Building on Previous Projects

This project continues Phase 4 advanced .NET features with system-level programming:

- **Projects 1-12**: All foundational skills with performance focus
- **Project 13**: Reflection for runtime buffer type handling
- **Project 14**: Expression trees for optimized buffer operations
- **New Skills**: Memory management, spans, and performance optimization

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-15-game-buffer/` directory:

```bash
# Create the main projects
dotnet new sln -n GameBuffer
dotnet new console -n GameBuffer.App -f net8.0
dotnet new classlib -n GameBuffer.Core -f net8.0
dotnet new mstest -n GameBuffer.Tests -f net8.0

# Add projects to solution
dotnet sln add GameBuffer.App/GameBuffer.App.csproj
dotnet sln add GameBuffer.Core/GameBuffer.Core.csproj
dotnet sln add GameBuffer.Tests/GameBuffer.Tests.csproj

# Add project references
dotnet add GameBuffer.App/GameBuffer.App.csproj reference GameBuffer.Core/GameBuffer.Core.csproj
dotnet add GameBuffer.Tests/GameBuffer.Tests.csproj reference GameBuffer.Core/GameBuffer.Core.csproj
```

### Step 2: Configure Project Files

Update the `GameBuffer.Core.csproj` file:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <AllowUnsafeBlocks>true</AllowUnsafeBlocks>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>GameBuffer.Core</PackageId>
    <Title>High-Performance Game Buffer</Title>
    <Description>Memory management and buffer utilities for high-performance games</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="System.Memory" Version="4.5.5" />
    <PackageReference Include="System.Numerics.Vectors" Version="4.5.0" />
    <PackageReference Include="System.Runtime.CompilerServices.Unsafe" Version="6.0.0" />
    <PackageReference Include="BenchmarkDotNet" Version="0.13.12" />
  </ItemGroup>

</Project>
```

### Step 3: Create Directory Structure

Organize your buffer management code:

```
GameBuffer.Core/
├── Buffers/           # Core buffer types
├── Pools/             # Buffer pooling
├── Spans/             # Span utilities
├── Unsafe/            # Unsafe operations
├── SIMD/              # SIMD optimizations
├── Memory/            # Memory management
└── Benchmarks/        # Performance tests
```

## Implementation Guide

### Step 4: Core Buffer Types (30 minutes)

**Create the base buffer infrastructure**

```csharp
// File: GameBuffer.Core/Buffers/GameBuffer.cs
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

namespace GameBuffer.Core.Buffers;

/// <summary>
/// A high-performance buffer wrapper that can work with both managed and unmanaged memory
/// </summary>
/// <typeparam name="T">The type of elements in the buffer</typeparam>
public readonly struct GameBuffer<T> : IDisposable where T : unmanaged
{
    private readonly T[] _managedArray;
    private readonly IntPtr _unmanagedPtr;
    private readonly int _length;
    private readonly bool _isUnmanaged;
    private readonly bool _shouldDispose;

    // Managed array constructor
    public GameBuffer(T[] array, bool shouldDispose = false)
    {
        _managedArray = array ?? throw new ArgumentNullException(nameof(array));
        _unmanagedPtr = IntPtr.Zero;
        _length = array.Length;
        _isUnmanaged = false;
        _shouldDispose = shouldDispose;
    }

    // Unmanaged memory constructor
    public unsafe GameBuffer(T* pointer, int length, bool shouldDispose = false)
    {
        _managedArray = null!;
        _unmanagedPtr = (IntPtr)pointer;
        _length = length;
        _isUnmanaged = true;
        _shouldDispose = shouldDispose;
    }

    public int Length => _length;
    public bool IsEmpty => _length == 0;
    public bool IsUnmanaged => _isUnmanaged;

    /// <summary>
    /// Get a span view of the buffer
    /// </summary>
    public Span<T> AsSpan()
    {
        if (_isUnmanaged)
        {
            unsafe
            {
                return new Span<T>((T*)_unmanagedPtr, _length);
            }
        }
        return _managedArray.AsSpan();
    }

    /// <summary>
    /// Get a read-only span view of the buffer
    /// </summary>
    public ReadOnlySpan<T> AsReadOnlySpan()
    {
        if (_isUnmanaged)
        {
            unsafe
            {
                return new ReadOnlySpan<T>((T*)_unmanagedPtr, _length);
            }
        }
        return _managedArray.AsSpan();
    }

    /// <summary>
    /// Copy data to another buffer with bounds checking
    /// </summary>
    public void CopyTo(GameBuffer<T> destination)
    {
        if (destination.Length < _length)
            throw new ArgumentException("Destination buffer too small");

        AsReadOnlySpan().CopyTo(destination.AsSpan());
    }

    /// <summary>
    /// Fill the buffer with a specific value
    /// </summary>
    public void Fill(T value)
    {
        AsSpan().Fill(value);
    }

    /// <summary>
    /// Clear the buffer (set all elements to default)
    /// </summary>
    public void Clear()
    {
        AsSpan().Clear();
    }

    public void Dispose()
    {
        if (_shouldDispose && _isUnmanaged && _unmanagedPtr != IntPtr.Zero)
        {
            Marshal.FreeHGlobal(_unmanagedPtr);
        }
    }

    // Indexer for convenient access
    public ref T this[int index]
    {
        get
        {
            if (index < 0 || index >= _length)
                throw new IndexOutOfRangeException();

            return ref AsSpan()[index];
        }
    }
}

/// <summary>
/// Static factory methods for creating GameBuffers
/// </summary>
public static class GameBuffer
{
    /// <summary>
    /// Create a managed buffer
    /// </summary>
    public static GameBuffer<T> Create<T>(int length) where T : unmanaged
    {
        return new GameBuffer<T>(new T[length]);
    }

    /// <summary>
    /// Create an unmanaged buffer
    /// </summary>
    public static unsafe GameBuffer<T> CreateUnmanaged<T>(int length) where T : unmanaged
    {
        var ptr = (T*)Marshal.AllocHGlobal(length * sizeof(T));
        return new GameBuffer<T>(ptr, length, shouldDispose: true);
    }

    /// <summary>
    /// Create a buffer from existing data
    /// </summary>
    public static GameBuffer<T> FromArray<T>(T[] data) where T : unmanaged
    {
        return new GameBuffer<T>(data);
    }

    /// <summary>
    /// Create a stack-allocated buffer (use with caution)
    /// </summary>
    public static unsafe GameBuffer<T> FromStackAlloc<T>(Span<T> stackSpan) where T : unmanaged
    {
        fixed (T* ptr = stackSpan)
        {
            return new GameBuffer<T>(ptr, stackSpan.Length, shouldDispose: false);
        }
    }
}
```

### Step 5: Buffer Pooling System (25 minutes)

**Create efficient buffer pooling for performance**

```csharp
// File: GameBuffer.Core/Pools/GameBufferPool.cs
using System.Buffers;
using System.Collections.Concurrent;

namespace GameBuffer.Core.Pools;

/// <summary>
/// High-performance buffer pool for managing frequently allocated buffers
/// </summary>
/// <typeparam name="T">The type of elements in the buffer</typeparam>
public class GameBufferPool<T> : IDisposable where T : unmanaged
{
    private readonly ArrayPool<T> _arrayPool;
    private readonly ConcurrentQueue<PooledBuffer<T>> _availableBuffers;
    private readonly ConcurrentDictionary<int, PooledBuffer<T>> _rentedBuffers;
    private readonly int _maxBufferSize;
    private readonly int _maxPoolSize;
    private volatile bool _disposed;

    public GameBufferPool(int maxBufferSize = 1024 * 1024, int maxPoolSize = 100)
    {
        _arrayPool = ArrayPool<T>.Create(maxBufferSize, maxPoolSize);
        _availableBuffers = new ConcurrentQueue<PooledBuffer<T>>();
        _rentedBuffers = new ConcurrentDictionary<int, PooledBuffer<T>>();
        _maxBufferSize = maxBufferSize;
        _maxPoolSize = maxPoolSize;
    }

    /// <summary>
    /// Rent a buffer from the pool
    /// </summary>
    public PooledBuffer<T> Rent(int minimumLength)
    {
        ThrowIfDisposed();

        if (minimumLength > _maxBufferSize)
        {
            // For very large buffers, don't pool them
            var largeArray = new T[minimumLength];
            return new PooledBuffer<T>(largeArray, this, isPooled: false);
        }

        // Try to get from pool first
        if (_availableBuffers.TryDequeue(out var pooledBuffer) &&
            pooledBuffer.Length >= minimumLength)
        {
            _rentedBuffers[pooledBuffer.Id] = pooledBuffer;
            return pooledBuffer;
        }

        // Rent from ArrayPool
        var array = _arrayPool.Rent(minimumLength);
        var buffer = new PooledBuffer<T>(array, this, isPooled: true);
        _rentedBuffers[buffer.Id] = buffer;
        return buffer;
    }

    /// <summary>
    /// Return a buffer to the pool
    /// </summary>
    internal void Return(PooledBuffer<T> buffer)
    {
        if (_disposed || !buffer.IsPooled)
            return;

        if (_rentedBuffers.TryRemove(buffer.Id, out _))
        {
            // Clear the buffer before returning to pool
            buffer.Clear();

            if (_availableBuffers.Count < _maxPoolSize)
            {
                _availableBuffers.Enqueue(buffer);
            }
            else
            {
                // Pool is full, return to ArrayPool
                _arrayPool.Return(buffer.Array, clearArray: true);
            }
        }
    }

    public void Dispose()
    {
        if (_disposed)
            return;

        _disposed = true;

        // Return all buffers
        while (_availableBuffers.TryDequeue(out var buffer))
        {
            _arrayPool.Return(buffer.Array, clearArray: true);
        }

        foreach (var rentedBuffer in _rentedBuffers.Values)
        {
            if (rentedBuffer.IsPooled)
                _arrayPool.Return(rentedBuffer.Array, clearArray: true);
        }

        _rentedBuffers.Clear();
    }

    private void ThrowIfDisposed()
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(GameBufferPool<T>));
    }
}

/// <summary>
/// A buffer that's managed by a pool
/// </summary>
public class PooledBuffer<T> : IDisposable where T : unmanaged
{
    private static int _nextId = 0;

    internal T[] Array { get; }
    internal int Id { get; }
    internal bool IsPooled { get; }

    private readonly GameBufferPool<T> _pool;
    private bool _disposed;

    internal PooledBuffer(T[] array, GameBufferPool<T> pool, bool isPooled)
    {
        Array = array;
        Id = Interlocked.Increment(ref _nextId);
        IsPooled = isPooled;
        _pool = pool;
    }

    public int Length => Array.Length;
    public Span<T> AsSpan() => Array.AsSpan();
    public ReadOnlySpan<T> AsReadOnlySpan() => Array.AsSpan();

    public void Clear() => Array.AsSpan().Clear();
    public void Fill(T value) => Array.AsSpan().Fill(value);

    public ref T this[int index] => ref Array[index];

    public void Dispose()
    {
        if (!_disposed)
        {
            _disposed = true;
            _pool.Return(this);
        }
    }
}

/// <summary>
/// Global buffer pool singleton for convenience
/// </summary>
public static class SharedBufferPool
{
    private static readonly ConcurrentDictionary<Type, object> _pools = new();

    public static GameBufferPool<T> GetPool<T>() where T : unmanaged
    {
        return (GameBufferPool<T>)_pools.GetOrAdd(typeof(T),
            _ => new GameBufferPool<T>());
    }

    public static PooledBuffer<T> Rent<T>(int minimumLength) where T : unmanaged
    {
        return GetPool<T>().Rent(minimumLength);
    }
}
```

### Step 6: SIMD-Optimized Operations (30 minutes)

**Add high-performance vectorized operations**

```csharp
// File: GameBuffer.Core/SIMD/VectorOperations.cs
using System.Numerics;
using System.Runtime.CompilerServices;
using System.Runtime.Intrinsics;
using System.Runtime.Intrinsics.X86;

namespace GameBuffer.Core.SIMD;

/// <summary>
/// SIMD-optimized operations for game buffers
/// </summary>
public static class VectorOperations
{
    /// <summary>
    /// Multiply two float vectors element-wise
    /// </summary>
    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public static void Multiply(ReadOnlySpan<float> a, ReadOnlySpan<float> b, Span<float> result)
    {
        if (a.Length != b.Length || result.Length < a.Length)
            throw new ArgumentException("Array lengths must match");

        if (Vector.IsHardwareAccelerated && a.Length >= Vector<float>.Count)
        {
            MultiplyVectorized(a, b, result);
        }
        else
        {
            MultiplyScalar(a, b, result);
        }
    }

    private static void MultiplyVectorized(ReadOnlySpan<float> a, ReadOnlySpan<float> b, Span<float> result)
    {
        int vectorSize = Vector<float>.Count;
        int vectorizedLength = a.Length - (a.Length % vectorSize);

        // Process vectorized portion
        for (int i = 0; i < vectorizedLength; i += vectorSize)
        {
            var va = new Vector<float>(a.Slice(i, vectorSize));
            var vb = new Vector<float>(b.Slice(i, vectorSize));
            var vr = va * vb;
            vr.CopyTo(result.Slice(i, vectorSize));
        }

        // Process remaining elements
        for (int i = vectorizedLength; i < a.Length; i++)
        {
            result[i] = a[i] * b[i];
        }
    }

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    private static void MultiplyScalar(ReadOnlySpan<float> a, ReadOnlySpan<float> b, Span<float> result)
    {
        for (int i = 0; i < a.Length; i++)
        {
            result[i] = a[i] * b[i];
        }
    }

    /// <summary>
    /// Add scalar value to all elements in vector
    /// </summary>
    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    public static void AddScalar(Span<float> vector, float scalar)
    {
        if (Vector.IsHardwareAccelerated && vector.Length >= Vector<float>.Count)
        {
            AddScalarVectorized(vector, scalar);
        }
        else
        {
            AddScalarNormal(vector, scalar);
        }
    }

    private static void AddScalarVectorized(Span<float> vector, float scalar)
    {
        int vectorSize = Vector<float>.Count;
        int vectorizedLength = vector.Length - (vector.Length % vectorSize);
        var scalarVector = new Vector<float>(scalar);

        for (int i = 0; i < vectorizedLength; i += vectorSize)
        {
            var v = new Vector<float>(vector.Slice(i, vectorSize));
            var result = v + scalarVector;
            result.CopyTo(vector.Slice(i, vectorSize));
        }

        // Process remaining elements
        for (int i = vectorizedLength; i < vector.Length; i++)
        {
            vector[i] += scalar;
        }
    }

    [MethodImpl(MethodImplOptions.AggressiveInlining)]
    private static void AddScalarNormal(Span<float> vector, float scalar)
    {
        for (int i = 0; i < vector.Length; i++)
        {
            vector[i] += scalar;
        }
    }

    /// <summary>
    /// Calculate dot product of two vectors
    /// </summary>
    public static float DotProduct(ReadOnlySpan<float> a, ReadOnlySpan<float> b)
    {
        if (a.Length != b.Length)
            throw new ArgumentException("Vector lengths must match");

        if (Vector.IsHardwareAccelerated && a.Length >= Vector<float>.Count)
        {
            return DotProductVectorized(a, b);
        }
        return DotProductScalar(a, b);
    }

    private static float DotProductVectorized(ReadOnlySpan<float> a, ReadOnlySpan<float> b)
    {
        int vectorSize = Vector<float>.Count;
        int vectorizedLength = a.Length - (a.Length % vectorSize);
        var accumulator = Vector<float>.Zero;

        // Vectorized portion
        for (int i = 0; i < vectorizedLength; i += vectorSize)
        {
            var va = new Vector<float>(a.Slice(i, vectorSize));
            var vb = new Vector<float>(b.Slice(i, vectorSize));
            accumulator += va * vb;
        }

        // Sum the vector components
        float result = 0f;
        for (int i = 0; i < vectorSize; i++)
        {
            result += accumulator[i];
        }

        // Add remaining scalar elements
        for (int i = vectorizedLength; i < a.Length; i++)
        {
            result += a[i] * b[i];
        }

        return result;
    }

    private static float DotProductScalar(ReadOnlySpan<float> a, ReadOnlySpan<float> b)
    {
        float result = 0f;
        for (int i = 0; i < a.Length; i++)
        {
            result += a[i] * b[i];
        }
        return result;
    }

    /// <summary>
    /// Normalize a vector in-place
    /// </summary>
    public static void Normalize(Span<float> vector)
    {
        float magnitude = MathF.Sqrt(DotProduct(vector, vector));
        if (magnitude > float.Epsilon)
        {
            float invMagnitude = 1.0f / magnitude;
            for (int i = 0; i < vector.Length; i++)
            {
                vector[i] *= invMagnitude;
            }
        }
    }

    /// <summary>
    /// Transform vertices using a 4x4 matrix (optimized for game rendering)
    /// </summary>
    public static unsafe void TransformVertices(ReadOnlySpan<Vector3> vertices, Span<Vector3> transformed, in Matrix4x4 matrix)
    {
        if (transformed.Length < vertices.Length)
            throw new ArgumentException("Output buffer too small");

        fixed (Vector3* vertexPtr = vertices)
        fixed (Vector3* transformedPtr = transformed)
        {
            if (Avx.IsSupported && vertices.Length >= 8)
            {
                TransformVerticesAvx(vertexPtr, transformedPtr, vertices.Length, matrix);
            }
            else if (Sse.IsSupported && vertices.Length >= 4)
            {
                TransformVerticesSse(vertexPtr, transformedPtr, vertices.Length, matrix);
            }
            else
            {
                TransformVerticesScalar(vertices, transformed, matrix);
            }
        }
    }

    private static unsafe void TransformVerticesAvx(Vector3* vertices, Vector3* transformed, int count, in Matrix4x4 matrix)
    {
        // AVX implementation for processing 8 vertices at once
        // This is a simplified example - real implementation would be more complex
        for (int i = 0; i < count; i++)
        {
            transformed[i] = Vector3.Transform(vertices[i], matrix);
        }
    }

    private static unsafe void TransformVerticesSse(Vector3* vertices, Vector3* transformed, int count, in Matrix4x4 matrix)
    {
        // SSE implementation for processing 4 vertices at once
        for (int i = 0; i < count; i++)
        {
            transformed[i] = Vector3.Transform(vertices[i], matrix);
        }
    }

    private static void TransformVerticesScalar(ReadOnlySpan<Vector3> vertices, Span<Vector3> transformed, in Matrix4x4 matrix)
    {
        for (int i = 0; i < vertices.Length; i++)
        {
            transformed[i] = Vector3.Transform(vertices[i], matrix);
        }
    }
}
```

## Testing Your Understanding

Create a complete example in `GameBuffer.App/Program.cs`:

```csharp
using GameBuffer.Core.Buffers;
using GameBuffer.Core.Pools;
using GameBuffer.Core.SIMD;
using System.Diagnostics;
using System.Numerics;

namespace GameBuffer.App;

class Program
{
    static async Task Main(string[] args)
    {
        Console.WriteLine("🎮 High-Performance Game Buffer Demo");
        Console.WriteLine("====================================\n");

        // Demo 1: Basic buffer operations
        DemoBasicBuffers();

        // Demo 2: Buffer pooling
        await DemoBufferPooling();

        // Demo 3: SIMD operations
        DemoSIMDOperations();

        // Demo 4: Performance comparison
        await DemoPerformanceComparison();

        Console.WriteLine("\n✅ All demos completed successfully!");
    }

    static void DemoBasicBuffers()
    {
        Console.WriteLine("📦 Demo 1: Basic Buffer Operations");
        Console.WriteLine("----------------------------------");

        // Create different types of buffers
        using var managedBuffer = GameBuffer.Create<float>(1000);
        using var unmanagedBuffer = GameBuffer.CreateUnmanaged<int>(500);

        // Fill with data
        managedBuffer.Fill(3.14f);
        unmanagedBuffer.Fill(42);

        Console.WriteLine($"Managed buffer length: {managedBuffer.Length}");
        Console.WriteLine($"Unmanaged buffer length: {unmanagedBuffer.Length}");
        Console.WriteLine($"First element of managed: {managedBuffer[0]}");
        Console.WriteLine($"First element of unmanaged: {unmanagedBuffer[0]}");

        // Demonstrate span operations
        var span = managedBuffer.AsSpan();
        span[0] = 2.71f; // Euler's number
        Console.WriteLine($"Modified first element: {managedBuffer[0]}");

        // Stack allocation example
        Span<byte> stackSpan = stackalloc byte[64];
        stackSpan.Fill(0xFF);
        Console.WriteLine($"Stack buffer filled with: 0x{stackSpan[0]:X2}");

        Console.WriteLine();
    }

    static async Task DemoBufferPooling()
    {
        Console.WriteLine("🏊‍♂️ Demo 2: Buffer Pooling");
        Console.WriteLine("---------------------------");

        using var pool = new GameBufferPool<float>(maxBufferSize: 1024, maxPoolSize: 10);

        // Rent multiple buffers
        var buffers = new List<PooledBuffer<float>>();
        for (int i = 0; i < 5; i++)
        {
            var buffer = pool.Rent(100);
            buffer.Fill(i * 1.5f);
            buffers.Add(buffer);
            Console.WriteLine($"Rented buffer {i}: length={buffer.Length}, first_element={buffer[0]}");
        }

        // Return all buffers
        foreach (var buffer in buffers)
        {
            buffer.Dispose();
        }

        Console.WriteLine("All buffers returned to pool");

        // Rent again to show reuse
        using var reusedBuffer = pool.Rent(100);
        Console.WriteLine($"Reused buffer first element (should be 0): {reusedBuffer[0]}");

        Console.WriteLine();
    }

    static void DemoSIMDOperations()
    {
        Console.WriteLine("⚡ Demo 3: SIMD Operations");
        Console.WriteLine("-------------------------");

        const int size = 1000;
        var a = new float[size];
        var b = new float[size];
        var result = new float[size];

        // Initialize with test data
        for (int i = 0; i < size; i++)
        {
            a[i] = i * 0.1f;
            b[i] = i * 0.2f;
        }

        // Demonstrate vectorized operations
        var sw = Stopwatch.StartNew();
        VectorOperations.Multiply(a, b, result);
        sw.Stop();

        Console.WriteLine($"Vector multiplication completed in {sw.ElapsedTicks} ticks");
        Console.WriteLine($"Vector.IsHardwareAccelerated: {Vector.IsHardwareAccelerated}");
        Console.WriteLine($"Vector<float>.Count: {Vector<float>.Count}");
        Console.WriteLine($"Sample results: {result[0]}, {result[10]}, {result[100]}");

        // Demonstrate dot product
        var dotProduct = VectorOperations.DotProduct(a, b);
        Console.WriteLine($"Dot product: {dotProduct:F2}");

        // Demonstrate normalization
        var vector = new float[] { 3.0f, 4.0f, 0.0f }; // Length = 5
        VectorOperations.Normalize(vector);
        Console.WriteLine($"Normalized vector: [{vector[0]:F3}, {vector[1]:F3}, {vector[2]:F3}]");

        Console.WriteLine();
    }

    static async Task DemoPerformanceComparison()
    {
        Console.WriteLine("📊 Demo 4: Performance Comparison");
        Console.WriteLine("---------------------------------");

        const int iterations = 1000;
        const int arraySize = 10000;

        // Test array allocation vs pooling
        var sw = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            var array = new float[arraySize];
            array[0] = i; // Use the array
        }
        sw.Stop();
        var allocTime = sw.ElapsedMilliseconds;

        // Test pooled allocation
        using var pool = new GameBufferPool<float>();
        sw.Restart();
        for (int i = 0; i < iterations; i++)
        {
            using var buffer = pool.Rent(arraySize);
            buffer[0] = i; // Use the buffer
        }
        sw.Stop();
        var poolTime = sw.ElapsedMilliseconds;

        Console.WriteLine($"Array allocation: {allocTime}ms");
        Console.WriteLine($"Pooled allocation: {poolTime}ms");
        Console.WriteLine($"Pool is {(double)allocTime / poolTime:F1}x faster");

        // Test SIMD vs scalar operations
        var a = Enumerable.Range(0, arraySize).Select(i => (float)i).ToArray();
        var b = Enumerable.Range(0, arraySize).Select(i => (float)i * 2).ToArray();
        var result = new float[arraySize];

        // SIMD version
        sw.Restart();
        for (int i = 0; i < 100; i++)
        {
            VectorOperations.Multiply(a, b, result);
        }
        sw.Stop();
        var simdTime = sw.ElapsedTicks;

        // Scalar version
        sw.Restart();
        for (int i = 0; i < 100; i++)
        {
            for (int j = 0; j < arraySize; j++)
            {
                result[j] = a[j] * b[j];
            }
        }
        sw.Stop();
        var scalarTime = sw.ElapsedTicks;

        Console.WriteLine($"SIMD operations: {simdTime} ticks");
        Console.WriteLine($"Scalar operations: {scalarTime} ticks");
        Console.WriteLine($"SIMD is {(double)scalarTime / simdTime:F1}x faster");

        Console.WriteLine();
    }
}
```

## Extension Challenges

Once you've completed the basic implementation, try these advanced features:

### Challenge 1: Memory-Mapped File Buffers

Create a `MemoryMappedBuffer<T>` class that can handle very large datasets efficiently:

```csharp
public class MemoryMappedBuffer<T> : IDisposable where T : unmanaged
{
    private readonly MemoryMappedFile _mmf;
    private readonly MemoryMappedViewAccessor _accessor;

    public unsafe ReadOnlySpan<T> GetSpan(long offset, int count)
    {
        // Implementation using memory-mapped files
    }
}
```

### Challenge 2: Lock-Free Buffer Queue

Implement a lock-free circular buffer for high-performance producer-consumer scenarios:

```csharp
public class LockFreeBufferQueue<T> where T : unmanaged
{
    public bool TryEnqueue(ReadOnlySpan<T> data);
    public bool TryDequeue(Span<T> buffer, out int count);
}
```

### Challenge 3: SIMD String Operations

Create optimized string processing functions using SIMD:

```csharp
public static class FastStringOps
{
    public static int CountChar(ReadOnlySpan<char> text, char target);
    public static bool Contains(ReadOnlySpan<char> haystack, ReadOnlySpan<char> needle);
}
```

### Challenge 4: GPU Buffer Interop

Add support for GPU buffer interop using modern .NET features:

```csharp
public class GpuBuffer<T> : IDisposable where T : unmanaged
{
    public void CopyToGpu(ReadOnlySpan<T> data);
    public void CopyFromGpu(Span<T> data);
}
```

### Challenge 5: Performance Profiling

Add comprehensive benchmarking with BenchmarkDotNet:

```csharp
[MemoryDiagnoser]
[SimpleJob(RuntimeMoniker.Net80)]
public class BufferBenchmarks
{
    [Benchmark]
    public void ArrayAllocation() { /* ... */ }

    [Benchmark]
    public void PooledAllocation() { /* ... */ }
}
```

### Challenge 6: Custom Memory Allocator

Implement a specialized memory allocator for game objects:

```csharp
public class GameObjectAllocator : IDisposable
{
    public GameBuffer<T> Allocate<T>(int count) where T : unmanaged;
    public void Deallocate<T>(GameBuffer<T> buffer);
}
```

### Challenge 7: Span-based JSON Serialization

Create zero-allocation JSON serialization using spans:

```csharp
public static class SpanJsonSerializer
{
    public static bool TrySerialize<T>(T value, Span<byte> destination, out int bytesWritten);
    public static bool TryDeserialize<T>(ReadOnlySpan<byte> json, out T value);
}
```

## Resources

### Microsoft Documentation

- 📘 **[Span<T> and Memory<T>](https://learn.microsoft.com/en-us/dotnet/standard/memory-and-spans/)** - Complete span documentation
- 📘 **[ArrayPool<T>](https://learn.microsoft.com/en-us/dotnet/api/system.buffers.arraypool-1)** - Array pooling documentation
- 📘 **[Unsafe Code](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/unsafe-code)** - Unsafe code guidelines
- 📘 **[SIMD Support](https://learn.microsoft.com/en-us/dotnet/api/system.numerics.vector)** - Vector and SIMD operations
- 📘 **[Memory-Mapped Files](https://learn.microsoft.com/en-us/dotnet/standard/io/memory-mapped-files)** - Large file handling

### Performance Resources

- 📚 **[BenchmarkDotNet](https://benchmarkdotnet.org/)** - .NET benchmarking library
- 📚 **[PerfView](https://github.com/Microsoft/perfview)** - Memory and CPU profiling
- 📚 **[High-Performance .NET](https://www.manning.com/books/high-performance-dotnet)** - Performance optimization book

## Building on Previous Projects

This project integrates concepts from:

- **Projects 1-5**: All foundational .NET skills applied to performance scenarios
- **Projects 6-8**: Async patterns and object pooling for buffer management
- **Projects 9-12**: Advanced patterns applied to memory management
- **Project 13**: Reflection for dynamic buffer type handling
- **Project 14**: Expression trees for optimized buffer operations
- **Next Projects**: These buffer techniques will be used in native interop and game engine projects

## Performance Considerations

### Memory Management Best Practices

- Use `ArrayPool<T>` for frequently allocated arrays
- Prefer `Span<T>` over arrays for temporary operations
- Use `stackalloc` for small, short-lived buffers
- Implement proper disposal patterns for unmanaged resources

### SIMD Optimization Guidelines

- Check `Vector.IsHardwareAccelerated` before using SIMD
- Align data on vector boundaries when possible
- Use appropriate vector sizes for your target platforms
- Provide scalar fallbacks for non-SIMD scenarios

### Safety with Unsafe Code

- Always validate bounds in unsafe contexts
- Use `fixed` statements properly with spans
- Implement defensive programming practices
- Test thoroughly on different platforms and architectures

## Advanced Memory Patterns

This project demonstrates cutting-edge memory management:

- **Zero-Copy Operations**: Minimizing memory copying in data pipelines
- **Memory Alignment**: Optimizing data layout for CPU cache efficiency
- **Ref Returns**: Returning references to avoid copying
- **Custom Allocators**: Implementing specialized allocation strategies
- **Memory Diagnostics**: Profiling and optimizing memory usage

The techniques learned here form the foundation for high-performance game programming and will be essential for the remaining advanced projects in this curriculum.
