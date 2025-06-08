# Project 6: File-Based Game Save System

**Focus**: Async/Await Programming  
**Time**: ~2 hours  
**Deliverable**: `GameSaveSystem` NuGet package

## Learning Objectives

By completing this project, you'll understand:

- Async/await programming patterns and best practices
- Task-based Asynchronous Programming (TAP) model
- File I/O operations with proper exception handling
- Resource management with `using` statements and `IDisposable`
- Cancellation support with `CancellationToken`
- Performance considerations for async operations

## Project Overview

Build a comprehensive game save system that handles save data asynchronously. This project introduces you to async programming in .NET, which is essential for responsive applications and games.

## What You'll Build

A save system that can:

- Save and load game data asynchronously without blocking the UI
- Manage multiple save slots with atomic operations
- Compress and encrypt save data for smaller file sizes and security
- Create automatic backups before overwriting saves
- Handle concurrent save/load operations safely
- Support cancellation for long-running operations
- Provide cloud save synchronization hooks

## Core Concepts to Learn

### Async/Await Fundamentals

- **Task vs Task<T>**: Understanding asynchronous return types
- **async/await keywords**: Proper usage and common pitfalls
- **ConfigureAwait**: When and why to use `ConfigureAwait(false)`
- **Async void**: Why to avoid it (except for event handlers)

### Exception Handling in Async Code

- **Try/catch with async**: Handling exceptions in async methods
- **AggregateException**: Understanding exception wrapping
- **Custom exceptions**: Creating domain-specific exception types

### Resource Management

- **using statements**: Automatic resource disposal
- **IAsyncDisposable**: Async disposal patterns (C# 8+)
- **Stream management**: Proper handling of file streams

## Implementation Guide

### Step 1: Project Setup (15 minutes)

```bash
# Create new class library project
dotnet new classlib -n GameSaveSystem
cd GameSaveSystem

# Add necessary packages
dotnet add package System.Text.Json
dotnet add package System.IO.Compression
```

### Step 2: Core Save Data Types (20 minutes)

**`SaveMetadata` (record)**

```csharp
// Immutable save metadata
public record SaveMetadata(
    string SlotName,
    DateTime CreatedAt,
    DateTime LastModified,
    long FileSizeBytes,
    string Checksum,
    int Version
);
```

**`SaveSlot` (class)**

```csharp
public class SaveSlot
{
    public string Name { get; }
    public SaveMetadata? Metadata { get; private set; }
    public bool IsEmpty => Metadata == null;

    // Constructor and async methods
}
```

### Step 3: Async Save Operations (45 minutes)

**Core SaveSystem Class**

```csharp
public class GameSaveSystem : IAsyncDisposable
{
    private readonly string _saveDirectory;
    private readonly SemaphoreSlim _saveLock;
    private readonly JsonSerializerOptions _jsonOptions;

    public GameSaveSystem(string saveDirectory)
    {
        _saveDirectory = saveDirectory ?? throw new ArgumentNullException(nameof(saveDirectory));
        _saveLock = new SemaphoreSlim(1, 1); // Only one save operation at a time
        _jsonOptions = new JsonSerializerOptions
        {
            WriteIndented = true,
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase
        };

        Directory.CreateDirectory(_saveDirectory);
    }

    public async Task SaveGameAsync<T>(string slotName, T gameData, CancellationToken cancellationToken = default)
    {
        ArgumentException.ThrowIfNullOrWhiteSpace(slotName);
        ArgumentNullException.ThrowIfNull(gameData);

        await _saveLock.WaitAsync(cancellationToken);
        try
        {
            await SaveGameInternalAsync(slotName, gameData, cancellationToken);
        }
        finally
        {
            _saveLock.Release();
        }
    }

    private async Task SaveGameInternalAsync<T>(string slotName, T gameData, CancellationToken cancellationToken)
    {
        var tempPath = GetTempSavePath(slotName);
        var finalPath = GetSavePath(slotName);
        var backupPath = GetBackupPath(slotName);

        try
        {
            // Create backup if save exists
            if (File.Exists(finalPath))
            {
                await CreateBackupAsync(finalPath, backupPath, cancellationToken);
            }

            // Save to temporary file first (atomic operation)
            await SaveToFileAsync(tempPath, gameData, cancellationToken);

            // Move temp file to final location
            File.Move(tempPath, finalPath, overwrite: true);
        }
        catch
        {
            // Clean up temp file on error
            if (File.Exists(tempPath))
            {
                File.Delete(tempPath);
            }
            throw;
        }
    }

    private async Task SaveToFileAsync<T>(string path, T data, CancellationToken cancellationToken)
    {
        using var fileStream = new FileStream(path, FileMode.Create, FileAccess.Write, FileShare.None, bufferSize: 4096, useAsync: true);
        using var gzipStream = new GZipStream(fileStream, CompressionLevel.Optimal);

        await JsonSerializer.SerializeAsync(gzipStream, data, _jsonOptions, cancellationToken);
    }
}
```

### Step 4: Async Load Operations (30 minutes)

**Load Methods with Exception Handling**

```csharp
public async Task<T> LoadGameAsync<T>(string slotName, CancellationToken cancellationToken = default)
{
    ArgumentException.ThrowIfNullOrWhiteSpace(slotName);

    var savePath = GetSavePath(slotName);

    if (!File.Exists(savePath))
    {
        throw new SaveNotFoundException($"Save slot '{slotName}' does not exist.");
    }

    try
    {
        return await LoadFromFileAsync<T>(savePath, cancellationToken);
    }
    catch (JsonException ex)
    {
        throw new CorruptSaveException($"Save slot '{slotName}' contains invalid data.", ex);
    }
    catch (InvalidDataException ex)
    {
        throw new CorruptSaveException($"Save slot '{slotName}' is corrupted.", ex);
    }
}

private async Task<T> LoadFromFileAsync<T>(string path, CancellationToken cancellationToken)
{
    using var fileStream = new FileStream(path, FileMode.Open, FileAccess.Read, FileShare.Read, bufferSize: 4096, useAsync: true);
    using var gzipStream = new GZipStream(fileStream, CompressionMode.Decompress);

    var result = await JsonSerializer.DeserializeAsync<T>(gzipStream, _jsonOptions, cancellationToken);
    return result ?? throw new CorruptSaveException("Deserialized data is null.");
}

public async Task<bool> SaveExistsAsync(string slotName, CancellationToken cancellationToken = default)
{
    ArgumentException.ThrowIfNullOrWhiteSpace(slotName);

    var savePath = GetSavePath(slotName);

    // Use Task.Run for CPU-bound File.Exists check if needed
    return await Task.Run(() => File.Exists(savePath), cancellationToken);
}
```

### Step 5: Advanced Async Patterns (30 minutes)

**Batch Operations with Task.WhenAll**

```csharp
public async Task<IReadOnlyList<SaveMetadata>> GetAllSaveMetadataAsync(CancellationToken cancellationToken = default)
{
    var saveFiles = Directory.GetFiles(_saveDirectory, "*.save");

    var metadataTasks = saveFiles.Select(async file =>
    {
        try
        {
            return await GetSaveMetadataAsync(Path.GetFileNameWithoutExtension(file), cancellationToken);
        }
        catch
        {
            return null; // Skip corrupted saves
        }
    });

    var results = await Task.WhenAll(metadataTasks);
    return results.Where(m => m != null).ToList()!;
}

public async Task<SaveMetadata> GetSaveMetadataAsync(string slotName, CancellationToken cancellationToken = default)
{
    ArgumentException.ThrowIfNullOrWhiteSpace(slotName);

    var savePath = GetSavePath(slotName);
    var fileInfo = new FileInfo(savePath);

    if (!fileInfo.Exists)
    {
        throw new SaveNotFoundException($"Save slot '{slotName}' does not exist.");
    }

    // Calculate checksum asynchronously
    var checksum = await CalculateChecksumAsync(savePath, cancellationToken);

    return new SaveMetadata(
        SlotName: slotName,
        CreatedAt: fileInfo.CreationTime,
        LastModified: fileInfo.LastWriteTime,
        FileSizeBytes: fileInfo.Length,
        Checksum: checksum,
        Version: 1 // Could be read from save data
    );
}

private async Task<string> CalculateChecksumAsync(string filePath, CancellationToken cancellationToken)
{
    using var sha256 = SHA256.Create();
    using var fileStream = new FileStream(filePath, FileMode.Open, FileAccess.Read, FileShare.Read, bufferSize: 4096, useAsync: true);

    var hashBytes = await sha256.ComputeHashAsync(fileStream, cancellationToken);
    return Convert.ToHexString(hashBytes);
}
```

### Step 6: Exception Types and Error Handling (20 minutes)

**Custom Exception Types**

```csharp
public class SaveSystemException : Exception
{
    public SaveSystemException(string message) : base(message) { }
    public SaveSystemException(string message, Exception innerException) : base(message, innerException) { }
}

public class SaveNotFoundException : SaveSystemException
{
    public SaveNotFoundException(string message) : base(message) { }
}

public class CorruptSaveException : SaveSystemException
{
    public CorruptSaveException(string message) : base(message) { }
    public CorruptSaveException(string message, Exception innerException) : base(message, innerException) { }
}

public class SaveSlotInUseException : SaveSystemException
{
    public SaveSlotInUseException(string message) : base(message) { }
}
```

## Key Async Patterns to Practice

1. **Async Methods**:

   ```csharp
   // Correct async method signature
   public async Task<T> LoadAsync<T>(string slot, CancellationToken cancellationToken = default)

   // Not async void (except for event handlers)
   public async void SomeMethod() // ❌ Avoid this
   ```

2. **ConfigureAwait Usage**:

   ```csharp
   // In library code, use ConfigureAwait(false)
   var data = await LoadFromFileAsync(path, cancellationToken).ConfigureAwait(false);

   // In UI code, usually omit ConfigureAwait (defaults to true)
   var data = await LoadFromFileAsync(path, cancellationToken);
   ```

3. **Exception Handling**:

   ```csharp
   try
   {
       await SomeAsyncOperation();
   }
   catch (SpecificException ex)
   {
       // Handle specific exception
   }
   catch (Exception ex)
   {
       // Handle general exception
       throw new CustomException("Context-specific message", ex);
   }
   ```

4. **Resource Management**:
   ```csharp
   // Using statement with async
   using var stream = new FileStream(path, FileMode.Open);
   await stream.WriteAsync(data, cancellationToken);
   // Stream automatically disposed here
   ```

## Testing Your Understanding

Create a console app to test your save system:

```csharp
public class GameData
{
    public string PlayerName { get; set; } = "";
    public int Level { get; set; }
    public DateTime LastPlayed { get; set; }
    public List<string> Achievements { get; set; } = new();
}

class Program
{
    static async Task Main(string[] args)
    {
        var saveSystem = new GameSaveSystem("./saves");
        var cts = new CancellationTokenSource();

        var gameData = new GameData
        {
            PlayerName = "TestPlayer",
            Level = 42,
            LastPlayed = DateTime.Now,
            Achievements = ["First Steps", "Level 10", "Boss Defeated"]
        };

        try
        {
            // Test async save
            Console.WriteLine("Saving game...");
            await saveSystem.SaveGameAsync("test-slot", gameData, cts.Token);
            Console.WriteLine("Game saved successfully!");

            // Test async load
            Console.WriteLine("Loading game...");
            var loadedData = await saveSystem.LoadGameAsync<GameData>("test-slot", cts.Token);
            Console.WriteLine($"Loaded: {loadedData.PlayerName}, Level {loadedData.Level}");

            // Test metadata
            var metadata = await saveSystem.GetSaveMetadataAsync("test-slot", cts.Token);
            Console.WriteLine($"Save created: {metadata.CreatedAt}, Size: {metadata.FileSizeBytes} bytes");
        }
        catch (SaveSystemException ex)
        {
            Console.WriteLine($"Save system error: {ex.Message}");
        }
        finally
        {
            await saveSystem.DisposeAsync();
        }
    }
}
```

## Performance Considerations

1. **Avoid Blocking Calls**: Never use `.Result` or `.Wait()` on async operations
2. **Use Appropriate Async Methods**: Prefer `ReadAllTextAsync` over `ReadAllText` for I/O
3. **Limit Concurrency**: Use `SemaphoreSlim` to control concurrent operations
4. **Stream Efficiently**: Use buffered streams for large files
5. **Cancel Appropriately**: Always support `CancellationToken` for long operations

## Advanced Features (Optional)

1. **Cloud Save Sync**: Add methods for uploading/downloading saves
2. **Save Versioning**: Handle save format migrations
3. **Save Validation**: Add data integrity checks
4. **Incremental Saves**: Only save changed data
5. **Save Compression**: Different compression levels based on data size

## Common Async Pitfalls to Avoid

1. **Async Void**: Only use for event handlers
2. **Sync over Async**: Don't use `.Result` or `.Wait()`
3. **Forgetting ConfigureAwait**: Use `ConfigureAwait(false)` in libraries
4. **Not Handling Cancellation**: Always check `CancellationToken`
5. **Improper Exception Handling**: Async exceptions need special care

## Documentation Links

- [Async Programming Patterns](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/)
- [Task-based Asynchronous Pattern](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap)
- [File I/O Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/io/)
- [ConfigureAwait FAQ](https://devblogs.microsoft.com/dotnet/configureawait-faq/)
