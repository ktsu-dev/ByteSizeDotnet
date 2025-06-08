# Project 6: File-Based Game Save System

**Focus**: Async/Await Programming  
**Time**: ~2 hours  
**Deliverable**: Game save system with asynchronous file operations

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Async Guidelines](../microsoft-design-guidelines.md#asynchronous-programming-guidelines)** - Proper async/await patterns
- **[Exception Handling](../microsoft-design-guidelines.md#exception-throwing)** - Managing I/O exceptions gracefully
- **[Disposal Patterns](../microsoft-design-guidelines.md#dispose-pattern)** - Proper resource management
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - Efficient I/O operations

## Learning Objectives

By completing this project, you'll understand:

- Async/await programming patterns and best practices
- Task-based Asynchronous Programming (TAP) model
- File I/O operations with proper exception handling
- Resource management with `using` statements and `IDisposable`
- Cancellation support with `CancellationToken`
- Performance considerations for async operations
- Understanding Task vs Task<T>
- ConfigureAwait usage and best practices

## What You'll Build

A comprehensive game save system featuring:

- Asynchronous save/load operations
- Multiple save slot management
- Save data compression and encryption
- Automatic backup creation
- Cloud save synchronization support
- Progress tracking with cancellation support
- Atomic save operations
- Concurrent save/load operation handling
- Data integrity checks and validation

## Core Concepts to Learn

### Async/Await Fundamentals

```csharp
// Task vs Task<T>: Understanding asynchronous return types
public async Task SaveAsync() { /* no return value */ }
public async Task<T> LoadAsync<T>() { /* returns T */ }

// async/await keywords: Proper usage
public async Task<GameData> LoadGameAsync(string slot)
{
    using var stream = new FileStream(path, FileMode.Open);
    return await JsonSerializer.DeserializeAsync<GameData>(stream);
}

// ConfigureAwait: When and why to use ConfigureAwait(false)
var data = await LoadFromFileAsync(path).ConfigureAwait(false);
```

### Exception Handling in Async Code

```csharp
try
{
    await SaveGameAsync("slot1", gameData);
}
catch (SaveSystemException ex)
{
    // Handle save-specific exceptions
}
catch (Exception ex)
{
    // Handle general exceptions
    throw new SaveSystemException("Save operation failed", ex);
}
```

## Project Setup Instructions

### Step 1: Project Setup (15 minutes)

Navigate to the `project-06-game-save-system/` directory and create the solution:

```bash
# Create the main solution file
dotnet new sln -n GameSave

# Create the main library project
dotnet new classlib -n GameSave.Core -f net8.0

# Create the console application
dotnet new console -n GameSave.App -f net8.0

# Create the test project
dotnet new mstest -n GameSave.Tests -f net8.0

# Add necessary packages
dotnet add GameSave.Core package System.Text.Json
dotnet add GameSave.Core package System.IO.Compression

# Add projects to solution
dotnet sln add GameSave.Core/GameSave.Core.csproj
dotnet sln add GameSave.App/GameSave.App.csproj
dotnet sln add GameSave.Tests/GameSave.Tests.csproj

# Add project references
dotnet add GameSave.App/GameSave.App.csproj reference GameSave.Core/GameSave.Core.csproj
dotnet add GameSave.Tests/GameSave.Tests.csproj reference GameSave.Core/GameSave.Core.csproj
```

## Implementation Guide

### Step 2: Core Save Data Types (20 minutes)

**Save Metadata**

```csharp
namespace GameSave.Core.Models;

// Immutable save metadata
public record SaveMetadata(
    string SlotName,
    DateTime CreatedAt,
    DateTime LastModified,
    long FileSizeBytes,
    string Checksum,
    int Version
);

public record GameData(
    string PlayerName,
    int Level,
    DateTime LastPlayed,
    List<string> Achievements,
    Dictionary<string, object> CustomData
)
{
    // Factory method for new games
    public static GameData CreateNew(string playerName) => new(
        playerName,
        1,
        DateTime.UtcNow,
        new List<string>(),
        new Dictionary<string, object>()
    );
}
```

**Save Slot Management**

```csharp
namespace GameSave.Core.Models;

public class SaveSlot
{
    public string Name { get; }
    public SaveMetadata? Metadata { get; private set; }
    public bool IsEmpty => Metadata == null;
    public bool IsCorrupted { get; private set; }

    public SaveSlot(string name)
    {
        Name = name ?? throw new ArgumentNullException(nameof(name));
    }

    internal void UpdateMetadata(SaveMetadata metadata)
    {
        Metadata = metadata;
        IsCorrupted = false;
    }

    internal void MarkAsCorrupted()
    {
        IsCorrupted = true;
    }
}
```

### Step 3: Async Save Operations (45 minutes)

**Core SaveSystem Class**

```csharp
namespace GameSave.Core;

using System.IO.Compression;
using System.Security.Cryptography;
using System.Text.Json;

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

    private async Task CreateBackupAsync(string sourcePath, string backupPath, CancellationToken cancellationToken)
    {
        using var sourceStream = new FileStream(sourcePath, FileMode.Open, FileAccess.Read, FileShare.Read, bufferSize: 4096, useAsync: true);
        using var backupStream = new FileStream(backupPath, FileMode.Create, FileAccess.Write, FileShare.None, bufferSize: 4096, useAsync: true);

        await sourceStream.CopyToAsync(backupStream, cancellationToken);
    }

    private string GetSavePath(string slotName) => Path.Combine(_saveDirectory, $"{slotName}.save");
    private string GetTempSavePath(string slotName) => Path.Combine(_saveDirectory, $"{slotName}.tmp");
    private string GetBackupPath(string slotName) => Path.Combine(_saveDirectory, $"{slotName}.bak");
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

public async Task DeleteSaveAsync(string slotName, CancellationToken cancellationToken = default)
{
    ArgumentException.ThrowIfNullOrWhiteSpace(slotName);

    await _saveLock.WaitAsync(cancellationToken);
    try
    {
        var savePath = GetSavePath(slotName);
        var backupPath = GetBackupPath(slotName);

        if (File.Exists(savePath))
        {
            await Task.Run(() => File.Delete(savePath), cancellationToken);
        }

        if (File.Exists(backupPath))
        {
            await Task.Run(() => File.Delete(backupPath), cancellationToken);
        }
    }
    finally
    {
        _saveLock.Release();
    }
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

// Concurrent save operations for multiple slots
public async Task SaveMultipleAsync<T>(IEnumerable<(string slotName, T data)> saves, CancellationToken cancellationToken = default)
{
    var saveTasks = saves.Select(save => SaveGameAsync(save.slotName, save.data, cancellationToken));
    await Task.WhenAll(saveTasks);
}

// Load with retry logic
public async Task<T> LoadGameWithRetryAsync<T>(string slotName, int maxRetries = 3, CancellationToken cancellationToken = default)
{
    for (int attempt = 1; attempt <= maxRetries; attempt++)
    {
        try
        {
            return await LoadGameAsync<T>(slotName, cancellationToken);
        }
        catch (CorruptSaveException) when (attempt < maxRetries)
        {
            // Try loading from backup
            try
            {
                return await LoadBackupAsync<T>(slotName, cancellationToken);
            }
            catch
            {
                // Continue to next attempt
            }
        }
    }

    throw new SaveSystemException($"Failed to load save '{slotName}' after {maxRetries} attempts");
}

private async Task<T> LoadBackupAsync<T>(string slotName, CancellationToken cancellationToken)
{
    var backupPath = GetBackupPath(slotName);
    if (!File.Exists(backupPath))
    {
        throw new SaveNotFoundException($"Backup for slot '{slotName}' does not exist.");
    }

    return await LoadFromFileAsync<T>(backupPath, cancellationToken);
}
```

### Step 6: Exception Types and Error Handling (20 minutes)

**Custom Exception Types**

```csharp
namespace GameSaveSystem.Core.Exceptions;

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

public class InsufficientStorageException : SaveSystemException
{
    public InsufficientStorageException(string message) : base(message) { }
}
```

**Resource Management and Disposal**

```csharp
public async ValueTask DisposeAsync()
{
    if (_saveLock != null)
    {
        await _saveLock.WaitAsync();
        try
        {
            _saveLock.Dispose();
        }
        catch
        {
            // Ignore disposal errors
        }
    }
}
```

## Key Async Patterns to Practice

### 1. Async Methods

```csharp
// ✅ Correct async method signature
public async Task<T> LoadAsync<T>(string slot, CancellationToken cancellationToken = default)

// ❌ Avoid async void (except for event handlers)
public async void SomeMethod()
```

### 2. ConfigureAwait Usage

```csharp
// ✅ In library code, use ConfigureAwait(false)
var data = await LoadFromFileAsync(path, cancellationToken).ConfigureAwait(false);

// ✅ In UI code, usually omit ConfigureAwait (defaults to true)
var data = await LoadFromFileAsync(path, cancellationToken);
```

### 3. Exception Handling

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

### 4. Resource Management

```csharp
// ✅ Using statement with async
using var stream = new FileStream(path, FileMode.Open);
await stream.WriteAsync(data, cancellationToken);
// Stream automatically disposed here
```

## Testing Your Understanding

Create a console app to test your save system:

```csharp
using GameSaveSystem.Core;
using GameSaveSystem.Core.Models;
using GameSaveSystem.Core.Exceptions;

class Program
{
    static async Task Main(string[] args)
    {
        var saveSystem = new GameSaveSystem("./saves");
        var cts = new CancellationTokenSource();

        var gameData = GameData.CreateNew("TestPlayer") with
        {
            Level = 42,
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

            // Test batch operations
            var allMetadata = await saveSystem.GetAllSaveMetadataAsync(cts.Token);
            Console.WriteLine($"Total saves: {allMetadata.Count}");

            // Test concurrent saves
            var multiSaves = new[]
            {
                ("slot1", gameData with { PlayerName = "Player1" }),
                ("slot2", gameData with { PlayerName = "Player2" }),
                ("slot3", gameData with { PlayerName = "Player3" })
            };

            await saveSystem.SaveMultipleAsync(multiSaves, cts.Token);
            Console.WriteLine("Multiple saves completed!");

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

## Example Usage

```csharp
// Async save operations
await saveSystem.SaveGameAsync("slot1", gameData, cancellationToken);

// Async load with error handling
try
{
    var gameData = await saveSystem.LoadGameAsync<GameData>("slot1");
    // Handle successful load
}
catch (SaveNotFoundException)
{
    // Handle missing save file
}
catch (CorruptSaveException)
{
    // Handle corrupted save data
}

// Fluent async operations with retry
var gameData = await saveSystem.LoadGameWithRetryAsync<GameData>("slot1", maxRetries: 3);
```

## Success Criteria

✅ Created async save/load operations  
✅ Implemented proper exception handling  
✅ Used Task-based programming patterns  
✅ Applied resource management with using statements  
✅ Implemented cancellation support  
✅ Built atomic save operations  
✅ Created batch operations with Task.WhenAll  
✅ Demonstrated proper async disposal patterns

## 📚 Essential Documentation

**Master async programming by studying Microsoft's guidance:**

### Async Programming Fundamentals

- [Async Programming Patterns](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/) - TAP model
- [Task-based Asynchronous Pattern](https://docs.microsoft.com/en-us/dotnet/standard/asynchronous-programming-patterns/task-based-asynchronous-pattern-tap) - Best practices
- [ConfigureAwait FAQ](https://devblogs.microsoft.com/dotnet/configureawait-faq/) - When and why to use

### File I/O and Streams

- [File I/O Best Practices](https://docs.microsoft.com/en-us/dotnet/standard/io/) - Efficient file operations
- [Stream Class](https://docs.microsoft.com/en-us/dotnet/api/system.io.stream) - Base stream operations
- [FileStream](https://docs.microsoft.com/en-us/dotnet/api/system.io.filestream) - File-specific operations

### Exception Handling

- [Exception Handling](https://docs.microsoft.com/en-us/dotnet/csharp/fundamentals/exceptions/) - Best practices
- [Creating Custom Exceptions](https://docs.microsoft.com/en-us/dotnet/standard/exceptions/how-to-create-user-defined-exceptions) - Domain-specific errors

## Resources

- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Async Programming Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/)**
- 📘 **[File I/O Documentation](https://learn.microsoft.com/en-us/dotnet/standard/io/)**

## Building on Previous Projects

This project advances concepts from all previous projects:

- **Project 1-2**: Type design for save data structures
- **Project 3**: Extension methods for data serialization
- **Project 4**: State machines for save system states
- **Project 5**: LINQ for save data querying
- **New Skills**: Asynchronous programming and I/O operations

---

**Ready to build async? Begin with Step 1!** ⚡
