# Project 20: Performance Profiling Suite

**Focus**: Diagnostics & Performance Analysis Capstone  
**Time**: ~4 hours  
**Deliverable**: `PerformanceProfiler` - Comprehensive performance analysis toolkit

## 📏 Microsoft Guidelines Applied

This project demonstrates advanced diagnostics and performance analysis:

- **Performance Guidelines** - Measuring and optimizing code performance
- **Diagnostic Guidelines** - Building comprehensive profiling tools
- **Instrumentation Guidelines** - Non-intrusive performance monitoring

## Learning Objectives

- Master performance profiling and diagnostic techniques
- Understand memory allocation analysis and optimization
- Build comprehensive performance monitoring tools
- Create actionable performance reports and visualizations
- Implement real-time performance tracking systems

## Quick Start

1. **Review Performance Concepts** 📖  
   Understanding of CPU profiling, memory analysis, and performance metrics

2. **Start Building** 💻  
   Follow the implementation guide below for comprehensive instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) for profiling milestones

## Project Structure

```
project-20-performance-profiler/
├── README.md                        # Project instructions
├── PerformanceProfiler.sln       # Solution file
├── PerformanceProfiler.Core/    # Main library implementation
├── PerformanceProfiler.App/     # Console application
└── PerformanceProfiler.Tests/   # Unit tests
│   ├── Core/                        # Profiler core engine
│   ├── CPU/                         # CPU profiling
│   ├── Memory/                      # Memory analysis
│   ├── IO/                          # I/O performance tracking
│   ├── Network/                     # Network performance
│   ├── Reporting/                   # Report generation
│   ├── Visualization/              # Performance visualizations
│   └── RealTime/                    # Live monitoring
└── PerformanceProfiler.Tests/   # Unit tests
```

## What You'll Build

A complete performance profiling suite featuring:

- **CPU Profiling** - Method-level execution time analysis
- **Memory Profiling** - Allocation tracking and leak detection
- **I/O Profiling** - File and network operation analysis
- **Real-Time Monitoring** - Live performance dashboards
- **Comparative Analysis** - Before/after performance comparison
- **Automated Reports** - Comprehensive performance reports
- **Integration Tools** - CI/CD pipeline integration
- **Visual Analytics** - Interactive performance visualizations

## Core Architecture

```csharp
// Main profiler orchestrator
public class PerformanceProfiler : IPerformanceProfiler
{
    private readonly List<IProfilerModule> _modules;
    private readonly IProfilerStorage _storage;
    private readonly IReportGenerator _reportGenerator;
    private readonly ProfilerConfiguration _config;

    public async Task<ProfilingSession> StartProfilingAsync(ProfilingOptions options)
    {
        var session = new ProfilingSession(Guid.NewGuid(), options);

        // Initialize profiling modules
        foreach (var module in _modules.Where(m => options.IsModuleEnabled(m.Name)))
        {
            await module.StartAsync(session);
        }

        // Start data collection
        session.Start();

        return session;
    }

    public async Task<ProfilingResults> StopProfilingAsync(ProfilingSession session)
    {
        session.Stop();

        // Collect data from all modules
        var results = new List<IProfilingData>();
        foreach (var module in _modules)
        {
            var moduleData = await module.StopAndCollectAsync(session);
            results.Add(moduleData);
        }

        // Store results
        var profilingResults = new ProfilingResults(session, results);
        await _storage.StoreAsync(profilingResults);

        return profilingResults;
    }

    public async Task<PerformanceReport> GenerateReportAsync(ProfilingResults results, ReportOptions options)
    {
        return await _reportGenerator.GenerateAsync(results, options);
    }
}

// CPU profiling with stack sampling
public class CpuProfiler : IProfilerModule
{
    private readonly StackSampler _stackSampler;
    private readonly MethodProfiler _methodProfiler;
    private readonly Timer _samplingTimer;

    public async Task StartAsync(ProfilingSession session)
    {
        _methodProfiler.EnableInstrumentation();

        // Start stack sampling at configured interval
        _samplingTimer = new Timer(
            callback: _ => CollectStackSample(),
            state: null,
            dueTime: TimeSpan.Zero,
            period: session.Options.SamplingInterval
        );
    }

    private void CollectStackSample()
    {
        var threads = Process.GetCurrentProcess().Threads.Cast<ProcessThread>();

        foreach (var thread in threads)
        {
            try
            {
                var stackTrace = GetStackTrace(thread.Id);
                _stackSampler.RecordSample(thread.Id, stackTrace, DateTime.UtcNow);
            }
            catch (Exception ex)
            {
                // Log sampling error but continue
                _logger.LogWarning(ex, "Failed to sample thread {ThreadId}", thread.Id);
            }
        }
    }

    public async Task<IProfilingData> StopAndCollectAsync(ProfilingSession session)
    {
        _samplingTimer?.Dispose();
        _methodProfiler.DisableInstrumentation();

        var samples = _stackSampler.GetSamples();
        var methodStats = _methodProfiler.GetStatistics();

        return new CpuProfilingData
        {
            StackSamples = samples,
            MethodStatistics = methodStats,
            TotalSamples = samples.Count,
            SamplingDuration = session.Duration
        };
    }
}

// Memory profiling with allocation tracking
public class MemoryProfiler : IProfilerModule
{
    private readonly AllocationTracker _allocationTracker;
    private readonly GCEventListener _gcEventListener;
    private readonly HeapAnalyzer _heapAnalyzer;

    public async Task StartAsync(ProfilingSession session)
    {
        // Enable allocation tracking
        _allocationTracker.Start();

        // Listen for GC events
        _gcEventListener.Start();

        // Schedule periodic heap snapshots
        if (session.Options.EnableHeapSnapshots)
        {
            ScheduleHeapSnapshots(session.Options.SnapshotInterval);
        }
    }

    public async Task<IProfilingData> StopAndCollectAsync(ProfilingSession session)
    {
        _allocationTracker.Stop();
        _gcEventListener.Stop();

        var allocationData = _allocationTracker.GetAllocations();
        var gcData = _gcEventListener.GetGCEvents();
        var heapSnapshots = _heapAnalyzer.GetSnapshots();

        return new MemoryProfilingData
        {
            Allocations = allocationData,
            GCEvents = gcData,
            HeapSnapshots = heapSnapshots,
            TotalAllocatedBytes = allocationData.Sum(a => a.Size),
            GCCollections = gcData.Count
        };
    }
}
```

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-20-performance-profiler/` directory:

```bash
# Create the main solution file
dotnet new sln -n PerformanceProfiler

# Create the main library project
dotnet new classlib -n PerformanceProfiler.Core -f net8.0

# Create the console application
dotnet new console -n PerformanceProfiler.App -f net8.0

# Create the test project
dotnet new mstest -n PerformanceProfiler.Tests -f net8.0

# Add projects to solution
dotnet sln add PerformanceProfiler.Core/PerformanceProfiler.Core.csproj
dotnet sln add PerformanceProfiler.App/PerformanceProfiler.App.csproj
dotnet sln add PerformanceProfiler.Tests/PerformanceProfiler.Tests.csproj

# Add project references
dotnet add PerformanceProfiler.App/PerformanceProfiler.App.csproj reference PerformanceProfiler.Core/PerformanceProfiler.Core.csproj
dotnet add PerformanceProfiler.Tests/PerformanceProfiler.Tests.csproj reference PerformanceProfiler.Core/PerformanceProfiler.Core.csproj
```

### Step 2: Configure Project Dependencies

Update `PerformanceProfiler.Core.csproj` to include performance monitoring packages:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <LangVersion>latest</LangVersion>
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    <PackageId>PerformanceProfiler.Core</PackageId>
    <Title>Performance Profiling Suite</Title>
    <Description>Comprehensive performance analysis and monitoring toolkit</Description>
    <Authors>Your Name</Authors>
    <Copyright>Copyright (c) 2024</Copyright>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Diagnostics.Runtime" Version="3.0.442" />
    <PackageReference Include="System.Diagnostics.PerformanceCounter" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.Hosting" Version="8.0.0" />
    <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.0" />
    <PackageReference Include="System.Text.Json" Version="8.0.0" />
    <PackageReference Include="BenchmarkDotNet" Version="0.13.8" />
  </ItemGroup>

</Project>
```

### Step 3: Create Directory Structure

Create the necessary directories for the profiling components:

```bash
mkdir PerformanceProfiler.Core/Core
mkdir PerformanceProfiler.Core/CPU
mkdir PerformanceProfiler.Core/Memory
mkdir PerformanceProfiler.Core/IO
mkdir PerformanceProfiler.Core/Network
mkdir PerformanceProfiler.Core/Reporting
mkdir PerformanceProfiler.Core/Visualization
mkdir PerformanceProfiler.Core/RealTime
mkdir PerformanceProfiler.Core/Models
```

## Implementation Guide

### Step 4: Core Profiling Models (30 minutes)

**PerformanceProfiler.Core/Models/ProfilingModels.cs**

```csharp
namespace PerformanceProfiler.Core.Models;

public class ProfilingSession
{
    public Guid Id { get; set; }
    public ProfilingOptions Options { get; set; }
    public DateTime StartTime { get; set; }
    public DateTime? EndTime { get; set; }
    public TimeSpan Duration => EndTime.HasValue ? EndTime.Value - StartTime : DateTime.UtcNow - StartTime;

    public ProfilingSession(Guid id, ProfilingOptions options)
    {
        Id = id;
        Options = options;
    }

    public void Start()
    {
        StartTime = DateTime.UtcNow;
    }

    public void Stop()
    {
        EndTime = DateTime.UtcNow;
    }
}

public class ProfilingOptions
{
    public bool EnableCpuProfiling { get; set; } = true;
    public bool EnableMemoryProfiling { get; set; } = true;
    public bool EnableIOProfiling { get; set; } = true;
    public bool EnableNetworkProfiling { get; set; } = false;
    public TimeSpan SamplingInterval { get; set; } = TimeSpan.FromMilliseconds(10);
    public bool EnableHeapSnapshots { get; set; } = false;
    public TimeSpan SnapshotInterval { get; set; } = TimeSpan.FromSeconds(30);
    public string[] TargetProcesses { get; set; } = Array.Empty<string>();
    public string OutputPath { get; set; } = "profiling_results";

    public bool IsModuleEnabled(string moduleName) => moduleName.ToLower() switch
    {
        "cpu" => EnableCpuProfiling,
        "memory" => EnableMemoryProfiling,
        "io" => EnableIOProfiling,
        "network" => EnableNetworkProfiling,
        _ => false
    };
}

public class ProfilingResults
{
    public ProfilingSession Session { get; set; }
    public List<IProfilingData> Data { get; set; }
    public DateTime GeneratedAt { get; set; } = DateTime.UtcNow;

    public ProfilingResults(ProfilingSession session, List<IProfilingData> data)
    {
        Session = session;
        Data = data;
    }

    public T? GetData<T>() where T : class, IProfilingData
    {
        return Data.OfType<T>().FirstOrDefault();
    }
}

public interface IProfilingData
{
    string ModuleName { get; }
    DateTime StartTime { get; set; }
    DateTime EndTime { get; set; }
    Dictionary<string, object> Metrics { get; }
}

public class CpuProfilingData : IProfilingData
{
    public string ModuleName => "CPU";
    public DateTime StartTime { get; set; }
    public DateTime EndTime { get; set; }
    public Dictionary<string, object> Metrics { get; } = new();

    public List<StackSample> StackSamples { get; set; } = new();
    public Dictionary<string, MethodStatistics> MethodStatistics { get; set; } = new();
    public int TotalSamples { get; set; }
    public TimeSpan SamplingDuration { get; set; }
    public double CpuUsagePercent { get; set; }
}

public class MemoryProfilingData : IProfilingData
{
    public string ModuleName => "Memory";
    public DateTime StartTime { get; set; }
    public DateTime EndTime { get; set; }
    public Dictionary<string, object> Metrics { get; } = new();

    public List<AllocationInfo> Allocations { get; set; } = new();
    public List<GCEvent> GCEvents { get; set; } = new();
    public List<HeapSnapshot> HeapSnapshots { get; set; } = new();
    public long TotalAllocatedBytes { get; set; }
    public int GCCollections { get; set; }
    public long WorkingSetMemory { get; set; }
}

public class StackSample
{
    public int ThreadId { get; set; }
    public DateTime Timestamp { get; set; }
    public List<StackFrame> Frames { get; set; } = new();
}

public class StackFrame
{
    public string MethodName { get; set; } = string.Empty;
    public string ClassName { get; set; } = string.Empty;
    public string Namespace { get; set; } = string.Empty;
    public int LineNumber { get; set; }
    public string FileName { get; set; } = string.Empty;
}

public class MethodStatistics
{
    public string MethodName { get; set; } = string.Empty;
    public int CallCount { get; set; }
    public TimeSpan TotalExecutionTime { get; set; }
    public TimeSpan AverageExecutionTime => CallCount > 0 ?
        TimeSpan.FromTicks(TotalExecutionTime.Ticks / CallCount) : TimeSpan.Zero;
    public TimeSpan MinExecutionTime { get; set; } = TimeSpan.MaxValue;
    public TimeSpan MaxExecutionTime { get; set; } = TimeSpan.MinValue;
}

public class AllocationInfo
{
    public string TypeName { get; set; } = string.Empty;
    public long Size { get; set; }
    public DateTime Timestamp { get; set; }
    public string[] StackTrace { get; set; } = Array.Empty<string>();
    public int Generation { get; set; }
}

public class GCEvent
{
    public int Generation { get; set; }
    public GCType Type { get; set; }
    public DateTime StartTime { get; set; }
    public TimeSpan Duration { get; set; }
    public long MemoryBeforeCollection { get; set; }
    public long MemoryAfterCollection { get; set; }
}

public enum GCType
{
    Background,
    Foreground,
    Concurrent
}

public class HeapSnapshot
{
    public DateTime Timestamp { get; set; }
    public Dictionary<string, TypeInfo> TypeStatistics { get; set; } = new();
    public long TotalHeapSize { get; set; }
    public int TotalObjectCount { get; set; }
}

public class TypeInfo
{
    public string TypeName { get; set; } = string.Empty;
    public int InstanceCount { get; set; }
    public long TotalSize { get; set; }
    public long AverageSize => InstanceCount > 0 ? TotalSize / InstanceCount : 0;
}
```

### Step 5: I/O Performance Profiler (30 minutes)

**PerformanceProfiler.Core/IO/IOProfiler.cs**

```csharp
using System.Diagnostics;
using System.Diagnostics.Tracing;

namespace PerformanceProfiler.Core.IO;

public class IOProfiler : IProfilerModule
{
    private readonly List<IOOperation> _operations = new();
    private readonly PerformanceCounter? _diskReadCounter;
    private readonly PerformanceCounter? _diskWriteCounter;
    private readonly Timer? _samplingTimer;
    private readonly ILogger<IOProfiler> _logger;

    public string Name => "IO";

    public IOProfiler(ILogger<IOProfiler> logger)
    {
        _logger = logger;

        try
        {
            _diskReadCounter = new PerformanceCounter("PhysicalDisk", "Disk Read Bytes/sec", "_Total");
            _diskWriteCounter = new PerformanceCounter("PhysicalDisk", "Disk Write Bytes/sec", "_Total");
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Failed to initialize disk performance counters");
        }
    }

    public async Task StartAsync(ProfilingSession session)
    {
        _operations.Clear();

        // Start sampling disk I/O metrics
        _samplingTimer = new Timer(
            callback: _ => SampleDiskMetrics(),
            state: null,
            dueTime: TimeSpan.Zero,
            period: session.Options.SamplingInterval
        );

        // Enable file system event tracking
        EnableFileSystemTracking();

        await Task.CompletedTask;
    }

    public async Task<IProfilingData> StopAndCollectAsync(ProfilingSession session)
    {
        _samplingTimer?.Dispose();
        DisableFileSystemTracking();

        var data = new IOProfilingData
        {
            StartTime = session.StartTime,
            EndTime = session.EndTime ?? DateTime.UtcNow,
            Operations = _operations.ToList(),
            TotalReadBytes = _operations.Where(o => o.Type == IOOperationType.Read).Sum(o => o.BytesTransferred),
            TotalWriteBytes = _operations.Where(o => o.Type == IOOperationType.Write).Sum(o => o.BytesTransferred),
            TotalOperations = _operations.Count
        };

        // Add metrics
        data.Metrics["AverageReadTime"] = _operations
            .Where(o => o.Type == IOOperationType.Read)
            .Average(o => o.Duration.TotalMilliseconds);

        data.Metrics["AverageWriteTime"] = _operations
            .Where(o => o.Type == IOOperationType.Write)
            .Average(o => o.Duration.TotalMilliseconds);

        await Task.CompletedTask;
        return data;
    }

    private void SampleDiskMetrics()
    {
        try
        {
            var readBytesPerSec = _diskReadCounter?.NextValue() ?? 0;
            var writeBytesPerSec = _diskWriteCounter?.NextValue() ?? 0;

            if (readBytesPerSec > 0 || writeBytesPerSec > 0)
            {
                _operations.Add(new IOOperation
                {
                    Type = IOOperationType.DiskSample,
                    Timestamp = DateTime.UtcNow,
                    BytesTransferred = (long)(readBytesPerSec + writeBytesPerSec),
                    Duration = TimeSpan.FromSeconds(1),
                    FileName = "System Disk I/O Sample"
                });
            }
        }
        catch (Exception ex)
        {
            _logger.LogWarning(ex, "Failed to sample disk metrics");
        }
    }

    private void EnableFileSystemTracking()
    {
        // In a real implementation, this would use ETW (Event Tracing for Windows)
        // or similar platform-specific mechanisms to track file system operations
        _logger.LogInformation("File system tracking enabled");
    }

    private void DisableFileSystemTracking()
    {
        _logger.LogInformation("File system tracking disabled");
    }

    public void Dispose()
    {
        _samplingTimer?.Dispose();
        _diskReadCounter?.Dispose();
        _diskWriteCounter?.Dispose();
    }
}

public class IOProfilingData : IProfilingData
{
    public string ModuleName => "IO";
    public DateTime StartTime { get; set; }
    public DateTime EndTime { get; set; }
    public Dictionary<string, object> Metrics { get; } = new();

    public List<IOOperation> Operations { get; set; } = new();
    public long TotalReadBytes { get; set; }
    public long TotalWriteBytes { get; set; }
    public int TotalOperations { get; set; }
}

public class IOOperation
{
    public IOOperationType Type { get; set; }
    public DateTime Timestamp { get; set; }
    public TimeSpan Duration { get; set; }
    public string FileName { get; set; } = string.Empty;
    public long BytesTransferred { get; set; }
    public string ProcessName { get; set; } = string.Empty;
    public int ProcessId { get; set; }
}

public enum IOOperationType
{
    Read,
    Write,
    Open,
    Close,
    Delete,
    DiskSample
}
```

### Step 6: Real-Time Monitoring Dashboard (45 minutes)

**PerformanceProfiler.Core/RealTime/RealTimeMonitor.cs**

```csharp
using System.Collections.Concurrent;
using System.Diagnostics;

namespace PerformanceProfiler.Core.RealTime;

public class RealTimeMonitor : IDisposable
{
    private readonly ConcurrentQueue<PerformanceMetric> _metrics = new();
    private readonly Timer _collectionTimer;
    private readonly ILogger<RealTimeMonitor> _logger;
    private readonly List<IMetricCollector> _collectors = new();

    public event EventHandler<PerformanceMetricEventArgs>? MetricReceived;
    public event EventHandler<PerformanceAlertEventArgs>? AlertTriggered;

    public RealTimeMonitor(ILogger<RealTimeMonitor> logger)
    {
        _logger = logger;

        // Initialize metric collectors
        _collectors.Add(new CpuMetricCollector());
        _collectors.Add(new MemoryMetricCollector());
        _collectors.Add(new DiskMetricCollector());
        _collectors.Add(new NetworkMetricCollector());

        // Start collection timer
        _collectionTimer = new Timer(CollectMetrics, null, TimeSpan.Zero, TimeSpan.FromSeconds(1));
    }

    private void CollectMetrics(object? state)
    {
        var timestamp = DateTime.UtcNow;

        foreach (var collector in _collectors)
        {
            try
            {
                var metrics = collector.CollectMetrics(timestamp);
                foreach (var metric in metrics)
                {
                    _metrics.Enqueue(metric);
                    MetricReceived?.Invoke(this, new PerformanceMetricEventArgs(metric));

                    // Check for alerts
                    CheckForAlerts(metric);
                }
            }
            catch (Exception ex)
            {
                _logger.LogWarning(ex, "Failed to collect metrics from {CollectorType}", collector.GetType().Name);
            }
        }

        // Limit queue size to prevent memory issues
        while (_metrics.Count > 10000)
        {
            _metrics.TryDequeue(out _);
        }
    }

    private void CheckForAlerts(PerformanceMetric metric)
    {
        // CPU usage alert
        if (metric.Name == "CPU_Usage" && metric.Value > 80)
        {
            AlertTriggered?.Invoke(this, new PerformanceAlertEventArgs(
                AlertLevel.Warning,
                $"High CPU usage detected: {metric.Value:F1}%",
                metric
            ));
        }

        // Memory usage alert
        if (metric.Name == "Memory_Usage_Percent" && metric.Value > 85)
        {
            AlertTriggered?.Invoke(this, new PerformanceAlertEventArgs(
                AlertLevel.Critical,
                $"High memory usage detected: {metric.Value:F1}%",
                metric
            ));
        }

        // Disk queue length alert
        if (metric.Name == "Disk_Queue_Length" && metric.Value > 10)
        {
            AlertTriggered?.Invoke(this, new PerformanceAlertEventArgs(
                AlertLevel.Warning,
                $"High disk queue length: {metric.Value:F0}",
                metric
            ));
        }
    }

    public PerformanceMetric[] GetRecentMetrics(TimeSpan timeWindow)
    {
        var cutoff = DateTime.UtcNow - timeWindow;
        return _metrics.Where(m => m.Timestamp >= cutoff).ToArray();
    }

    public PerformanceMetric[] GetMetricsByName(string metricName, TimeSpan timeWindow)
    {
        var cutoff = DateTime.UtcNow - timeWindow;
        return _metrics
            .Where(m => m.Name == metricName && m.Timestamp >= cutoff)
            .ToArray();
    }

    public void Dispose()
    {
        _collectionTimer.Dispose();
        foreach (var collector in _collectors.OfType<IDisposable>())
        {
            collector.Dispose();
        }
    }
}

public interface IMetricCollector
{
    PerformanceMetric[] CollectMetrics(DateTime timestamp);
}

public class CpuMetricCollector : IMetricCollector, IDisposable
{
    private readonly PerformanceCounter _cpuCounter;

    public CpuMetricCollector()
    {
        _cpuCounter = new PerformanceCounter("Processor", "% Processor Time", "_Total");
    }

    public PerformanceMetric[] CollectMetrics(DateTime timestamp)
    {
        var cpuUsage = _cpuCounter.NextValue();

        return new[]
        {
            new PerformanceMetric
            {
                Name = "CPU_Usage",
                Value = cpuUsage,
                Unit = "Percent",
                Timestamp = timestamp,
                Category = "System"
            }
        };
    }

    public void Dispose()
    {
        _cpuCounter.Dispose();
    }
}

public class MemoryMetricCollector : IMetricCollector, IDisposable
{
    private readonly PerformanceCounter _availableMemoryCounter;
    private readonly Process _currentProcess;

    public MemoryMetricCollector()
    {
        _availableMemoryCounter = new PerformanceCounter("Memory", "Available MBytes");
        _currentProcess = Process.GetCurrentProcess();
    }

    public PerformanceMetric[] CollectMetrics(DateTime timestamp)
    {
        var availableMemoryMB = _availableMemoryCounter.NextValue();
        var workingSetMB = _currentProcess.WorkingSet64 / (1024.0 * 1024.0);
        var privateMemoryMB = _currentProcess.PrivateMemorySize64 / (1024.0 * 1024.0);

        // Calculate total system memory (simplified)
        var totalMemoryMB = GC.GetTotalMemory(false) / (1024.0 * 1024.0) + availableMemoryMB;
        var usagePercent = ((totalMemoryMB - availableMemoryMB) / totalMemoryMB) * 100;

        return new[]
        {
            new PerformanceMetric
            {
                Name = "Memory_Available_MB",
                Value = availableMemoryMB,
                Unit = "MB",
                Timestamp = timestamp,
                Category = "Memory"
            },
            new PerformanceMetric
            {
                Name = "Memory_Usage_Percent",
                Value = usagePercent,
                Unit = "Percent",
                Timestamp = timestamp,
                Category = "Memory"
            },
            new PerformanceMetric
            {
                Name = "Process_Working_Set_MB",
                Value = workingSetMB,
                Unit = "MB",
                Timestamp = timestamp,
                Category = "Process"
            },
            new PerformanceMetric
            {
                Name = "Process_Private_Memory_MB",
                Value = privateMemoryMB,
                Unit = "MB",
                Timestamp = timestamp,
                Category = "Process"
            }
        };
    }

    public void Dispose()
    {
        _availableMemoryCounter.Dispose();
        _currentProcess.Dispose();
    }
}

public class DiskMetricCollector : IMetricCollector, IDisposable
{
    private readonly PerformanceCounter _diskQueueLengthCounter;
    private readonly PerformanceCounter _diskTimeCounter;

    public DiskMetricCollector()
    {
        _diskQueueLengthCounter = new PerformanceCounter("PhysicalDisk", "Current Disk Queue Length", "_Total");
        _diskTimeCounter = new PerformanceCounter("PhysicalDisk", "% Disk Time", "_Total");
    }

    public PerformanceMetric[] CollectMetrics(DateTime timestamp)
    {
        var queueLength = _diskQueueLengthCounter.NextValue();
        var diskTime = _diskTimeCounter.NextValue();

        return new[]
        {
            new PerformanceMetric
            {
                Name = "Disk_Queue_Length",
                Value = queueLength,
                Unit = "Count",
                Timestamp = timestamp,
                Category = "Disk"
            },
            new PerformanceMetric
            {
                Name = "Disk_Time_Percent",
                Value = diskTime,
                Unit = "Percent",
                Timestamp = timestamp,
                Category = "Disk"
            }
        };
    }

    public void Dispose()
    {
        _diskQueueLengthCounter.Dispose();
        _diskTimeCounter.Dispose();
    }
}

public class NetworkMetricCollector : IMetricCollector
{
    public PerformanceMetric[] CollectMetrics(DateTime timestamp)
    {
        // Simplified network metrics collection
        // In a real implementation, this would use NetworkInterface class
        // to get accurate network statistics

        return new[]
        {
            new PerformanceMetric
            {
                Name = "Network_Bytes_Sent_Per_Sec",
                Value = Random.Shared.Next(0, 1000000), // Placeholder
                Unit = "Bytes/sec",
                Timestamp = timestamp,
                Category = "Network"
            },
            new PerformanceMetric
            {
                Name = "Network_Bytes_Received_Per_Sec",
                Value = Random.Shared.Next(0, 1000000), // Placeholder
                Unit = "Bytes/sec",
                Timestamp = timestamp,
                Category = "Network"
            }
        };
    }
}

public class PerformanceMetric
{
    public string Name { get; set; } = string.Empty;
    public double Value { get; set; }
    public string Unit { get; set; } = string.Empty;
    public DateTime Timestamp { get; set; }
    public string Category { get; set; } = string.Empty;
    public Dictionary<string, string> Tags { get; set; } = new();
}

public class PerformanceMetricEventArgs : EventArgs
{
    public PerformanceMetric Metric { get; }

    public PerformanceMetricEventArgs(PerformanceMetric metric)
    {
        Metric = metric;
    }
}

public class PerformanceAlertEventArgs : EventArgs
{
    public AlertLevel Level { get; }
    public string Message { get; }
    public PerformanceMetric Metric { get; }

    public PerformanceAlertEventArgs(AlertLevel level, string message, PerformanceMetric metric)
    {
        Level = level;
        Message = message;
        Metric = metric;
    }
}

public enum AlertLevel
{
    Info,
    Warning,
    Critical
}
```

### Step 7: Performance Report Generator (30 minutes)

**PerformanceProfiler.Core/Reporting/ReportGenerator.cs**

```csharp
using System.Text.Json;

namespace PerformanceProfiler.Core.Reporting;

public class ReportGenerator : IReportGenerator
{
    private readonly ILogger<ReportGenerator> _logger;

    public ReportGenerator(ILogger<ReportGenerator> logger)
    {
        _logger = logger;
    }

    public async Task<PerformanceReport> GenerateAsync(ProfilingResults results, ReportOptions options)
    {
        var report = new PerformanceReport
        {
            SessionInfo = CreateSessionSummary(results.Session),
            GeneratedAt = DateTime.UtcNow,
            Options = options
        };

        // Generate sections based on available data
        if (results.GetData<CpuProfilingData>() is var cpuData && cpuData != null)
        {
            report.Sections.Add(await GenerateCpuSectionAsync(cpuData, options));
        }

        if (results.GetData<MemoryProfilingData>() is var memoryData && memoryData != null)
        {
            report.Sections.Add(await GenerateMemorySectionAsync(memoryData, options));
        }

        if (results.GetData<IOProfilingData>() is var ioData && ioData != null)
        {
            report.Sections.Add(await GenerateIOSectionAsync(ioData, options));
        }

        // Generate summary and recommendations
        report.Sections.Add(GenerateSummarySection(results));
        report.Sections.Add(GenerateRecommendationsSection(results));

        // Save report if output path is specified
        if (!string.IsNullOrEmpty(options.OutputPath))
        {
            await SaveReportAsync(report, options.OutputPath);
        }

        return report;
    }

    private SessionSummary CreateSessionSummary(ProfilingSession session)
    {
        return new SessionSummary
        {
            SessionId = session.Id,
            StartTime = session.StartTime,
            EndTime = session.EndTime,
            Duration = session.Duration,
            EnabledModules = GetEnabledModules(session.Options)
        };
    }

    private string[] GetEnabledModules(ProfilingOptions options)
    {
        var modules = new List<string>();
        if (options.EnableCpuProfiling) modules.Add("CPU");
        if (options.EnableMemoryProfiling) modules.Add("Memory");
        if (options.EnableIOProfiling) modules.Add("I/O");
        if (options.EnableNetworkProfiling) modules.Add("Network");
        return modules.ToArray();
    }

    private async Task<ReportSection> GenerateCpuSectionAsync(CpuProfilingData data, ReportOptions options)
    {
        var section = new ReportSection
        {
            Title = "CPU Performance Analysis",
            Priority = ReportPriority.High
        };

        // CPU utilization summary
        section.Content.Add("summary", new
        {
            AverageCpuUsage = $"{data.CpuUsagePercent:F1}%",
            TotalSamples = data.TotalSamples,
            SamplingDuration = data.SamplingDuration.ToString(@"mm\:ss"),
            TopMethods = data.MethodStatistics
                .OrderByDescending(kvp => kvp.Value.TotalExecutionTime)
                .Take(10)
                .Select(kvp => new
                {
                    Method = kvp.Key,
                    TotalTime = kvp.Value.TotalExecutionTime.TotalMilliseconds,
                    CallCount = kvp.Value.CallCount,
                    AverageTime = kvp.Value.AverageExecutionTime.TotalMilliseconds
                })
                .ToArray()
        });

        // Hot paths analysis
        var hotPaths = AnalyzeHotPaths(data.StackSamples);
        section.Content.Add("hotPaths", hotPaths);

        // Performance insights
        var insights = GenerateCpuInsights(data);
        section.Content.Add("insights", insights);

        await Task.CompletedTask;
        return section;
    }

    private async Task<ReportSection> GenerateMemorySectionAsync(MemoryProfilingData data, ReportOptions options)
    {
        var section = new ReportSection
        {
            Title = "Memory Performance Analysis",
            Priority = ReportPriority.High
        };

        // Memory usage summary
        section.Content.Add("summary", new
        {
            TotalAllocatedBytes = FormatBytes(data.TotalAllocatedBytes),
            AllocationCount = data.Allocations.Count,
            GCCollections = data.GCCollections,
            WorkingSetMemory = FormatBytes(data.WorkingSetMemory),
            TopAllocatingTypes = data.Allocations
                .GroupBy(a => a.TypeName)
                .OrderByDescending(g => g.Sum(a => a.Size))
                .Take(10)
                .Select(g => new
                {
                    TypeName = g.Key,
                    TotalSize = FormatBytes(g.Sum(a => a.Size)),
                    InstanceCount = g.Count()
                })
                .ToArray()
        });

        // GC analysis
        if (data.GCEvents.Any())
        {
            section.Content.Add("gcAnalysis", new
            {
                TotalGCTime = TimeSpan.FromMilliseconds(data.GCEvents.Sum(gc => gc.Duration.TotalMilliseconds)),
                Gen0Collections = data.GCEvents.Count(gc => gc.Generation == 0),
                Gen1Collections = data.GCEvents.Count(gc => gc.Generation == 1),
                Gen2Collections = data.GCEvents.Count(gc => gc.Generation == 2),
                AverageGCDuration = data.GCEvents.Average(gc => gc.Duration.TotalMilliseconds)
            });
        }

        // Memory insights
        var insights = GenerateMemoryInsights(data);
        section.Content.Add("insights", insights);

        await Task.CompletedTask;
        return section;
    }

    private async Task<ReportSection> GenerateIOSectionAsync(IOProfilingData data, ReportOptions options)
    {
        var section = new ReportSection
        {
            Title = "I/O Performance Analysis",
            Priority = ReportPriority.Medium
        };

        section.Content.Add("summary", new
        {
            TotalOperations = data.TotalOperations,
            TotalReadBytes = FormatBytes(data.TotalReadBytes),
            TotalWriteBytes = FormatBytes(data.TotalWriteBytes),
            AverageReadTime = data.Metrics.GetValueOrDefault("AverageReadTime", 0.0),
            AverageWriteTime = data.Metrics.GetValueOrDefault("AverageWriteTime", 0.0),
            TopFiles = data.Operations
                .Where(op => !string.IsNullOrEmpty(op.FileName))
                .GroupBy(op => op.FileName)
                .OrderByDescending(g => g.Sum(op => op.BytesTransferred))
                .Take(10)
                .Select(g => new
                {
                    FileName = g.Key,
                    TotalBytes = FormatBytes(g.Sum(op => op.BytesTransferred)),
                    OperationCount = g.Count()
                })
                .ToArray()
        });

        var insights = GenerateIOInsights(data);
        section.Content.Add("insights", insights);

        await Task.CompletedTask;
        return section;
    }

    private ReportSection GenerateSummarySection(ProfilingResults results)
    {
        var section = new ReportSection
        {
            Title = "Executive Summary",
            Priority = ReportPriority.Critical
        };

        var summary = new
        {
            ProfilingDuration = results.Session.Duration.ToString(@"mm\:ss"),
            ModulesAnalyzed = results.Data.Count,
            KeyFindings = GenerateKeyFindings(results),
            OverallRating = CalculateOverallPerformanceRating(results)
        };

        section.Content.Add("summary", summary);
        return section;
    }

    private ReportSection GenerateRecommendationsSection(ProfilingResults results)
    {
        var section = new ReportSection
        {
            Title = "Performance Recommendations",
            Priority = ReportPriority.High
        };

        var recommendations = new List<PerformanceRecommendation>();

        // CPU recommendations
        if (results.GetData<CpuProfilingData>() is var cpuData && cpuData != null)
        {
            recommendations.AddRange(GenerateCpuRecommendations(cpuData));
        }

        // Memory recommendations
        if (results.GetData<MemoryProfilingData>() is var memoryData && memoryData != null)
        {
            recommendations.AddRange(GenerateMemoryRecommendations(memoryData));
        }

        section.Content.Add("recommendations", recommendations.OrderByDescending(r => r.Priority).ToArray());
        return section;
    }

    private string[] GenerateKeyFindings(ProfilingResults results)
    {
        var findings = new List<string>();

        // Analyze CPU performance
        if (results.GetData<CpuProfilingData>() is var cpuData && cpuData != null)
        {
            if (cpuData.CpuUsagePercent > 80)
                findings.Add($"High CPU utilization detected: {cpuData.CpuUsagePercent:F1}%");

            var topMethod = cpuData.MethodStatistics.OrderByDescending(kvp => kvp.Value.TotalExecutionTime).FirstOrDefault();
            if (topMethod.Value != null)
                findings.Add($"Most time-consuming method: {topMethod.Key} ({topMethod.Value.TotalExecutionTime.TotalMilliseconds:F0}ms)");
        }

        // Analyze memory performance
        if (results.GetData<MemoryProfilingData>() is var memoryData && memoryData != null)
        {
            if (memoryData.GCCollections > 100)
                findings.Add($"High GC activity: {memoryData.GCCollections} collections during profiling");

            if (memoryData.TotalAllocatedBytes > 100 * 1024 * 1024) // 100MB
                findings.Add($"High memory allocation: {FormatBytes(memoryData.TotalAllocatedBytes)}");
        }

        return findings.ToArray();
    }

    private PerformanceRating CalculateOverallPerformanceRating(ProfilingResults results)
    {
        var scores = new List<double>();

        // CPU score
        if (results.GetData<CpuProfilingData>() is var cpuData && cpuData != null)
        {
            var cpuScore = Math.Max(0, 100 - cpuData.CpuUsagePercent);
            scores.Add(cpuScore);
        }

        // Memory score
        if (results.GetData<MemoryProfilingData>() is var memoryData && memoryData != null)
        {
            var memoryScore = Math.Max(0, 100 - (memoryData.GCCollections / 10.0));
            scores.Add(memoryScore);
        }

        if (!scores.Any()) return PerformanceRating.Unknown;

        var averageScore = scores.Average();
        return averageScore switch
        {
            >= 80 => PerformanceRating.Excellent,
            >= 60 => PerformanceRating.Good,
            >= 40 => PerformanceRating.Fair,
            >= 20 => PerformanceRating.Poor,
            _ => PerformanceRating.Critical
        };
    }

    private async Task SaveReportAsync(PerformanceReport report, string outputPath)
    {
        try
        {
            Directory.CreateDirectory(Path.GetDirectoryName(outputPath) ?? ".");

            var jsonOptions = new JsonSerializerOptions
            {
                WriteIndented = true,
                PropertyNamingPolicy = JsonNamingPolicy.CamelCase
            };

            var json = JsonSerializer.Serialize(report, jsonOptions);
            await File.WriteAllTextAsync(outputPath, json);

            _logger.LogInformation("Performance report saved to {OutputPath}", outputPath);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to save performance report to {OutputPath}", outputPath);
        }
    }

    // Helper methods
    private object[] AnalyzeHotPaths(List<StackSample> samples)
    {
        return samples
            .SelectMany(s => s.Frames)
            .GroupBy(f => $"{f.ClassName}.{f.MethodName}")
            .OrderByDescending(g => g.Count())
            .Take(10)
            .Select(g => new { Method = g.Key, SampleCount = g.Count() })
            .ToArray();
    }

    private string[] GenerateCpuInsights(CpuProfilingData data)
    {
        var insights = new List<string>();

        if (data.CpuUsagePercent > 80)
            insights.Add("Application is CPU-bound. Consider optimizing hot code paths.");

        if (data.MethodStatistics.Any(kvp => kvp.Value.CallCount > 10000))
            insights.Add("Some methods are called very frequently. Consider caching or optimization.");

        return insights.ToArray();
    }

    private string[] GenerateMemoryInsights(MemoryProfilingData data)
    {
        var insights = new List<string>();

        if (data.GCCollections > 100)
            insights.Add("High GC pressure detected. Consider reducing allocations or using object pooling.");

        var largeObjects = data.Allocations.Where(a => a.Size > 85000).Count();
        if (largeObjects > 0)
            insights.Add($"Found {largeObjects} large object allocations that may cause Gen2 GC pressure.");

        return insights.ToArray();
    }

    private string[] GenerateIOInsights(IOProfilingData data)
    {
        var insights = new List<string>();

        if (data.Operations.Count > 1000)
            insights.Add("High I/O operation count. Consider batching operations or using async I/O.");

        var avgReadTime = data.Metrics.GetValueOrDefault("AverageReadTime", 0.0);
        if (avgReadTime > 100) // 100ms
            insights.Add("Slow I/O operations detected. Consider using faster storage or optimizing file access patterns.");

        return insights.ToArray();
    }

    private List<PerformanceRecommendation> GenerateCpuRecommendations(CpuProfilingData data)
    {
        var recommendations = new List<PerformanceRecommendation>();

        if (data.CpuUsagePercent > 80)
        {
            recommendations.Add(new PerformanceRecommendation
            {
                Priority = RecommendationPriority.High,
                Category = "CPU",
                Title = "Optimize CPU-intensive operations",
                Description = "Application is using high CPU. Profile and optimize the most time-consuming methods.",
                Impact = "Can significantly improve application responsiveness"
            });
        }

        return recommendations;
    }

    private List<PerformanceRecommendation> GenerateMemoryRecommendations(MemoryProfilingData data)
    {
        var recommendations = new List<PerformanceRecommendation>();

        if (data.GCCollections > 100)
        {
            recommendations.Add(new PerformanceRecommendation
            {
                Priority = RecommendationPriority.High,
                Category = "Memory",
                Title = "Reduce memory allocations",
                Description = "High GC activity suggests excessive allocations. Consider object pooling or reducing temporary objects.",
                Impact = "Can reduce GC pauses and improve throughput"
            });
        }

        return recommendations;
    }

    private static string FormatBytes(long bytes)
    {
        string[] suffixes = { "B", "KB", "MB", "GB", "TB" };
        var doubleBytes = (double)bytes;
        var suffixIndex = 0;

        while (doubleBytes >= 1024 && suffixIndex < suffixes.Length - 1)
        {
            doubleBytes /= 1024;
            suffixIndex++;
        }

        return $"{doubleBytes:F2} {suffixes[suffixIndex]}";
    }
}

// Supporting classes
public interface IReportGenerator
{
    Task<PerformanceReport> GenerateAsync(ProfilingResults results, ReportOptions options);
}

public class PerformanceReport
{
    public SessionSummary SessionInfo { get; set; } = new();
    public DateTime GeneratedAt { get; set; }
    public ReportOptions Options { get; set; } = new();
    public List<ReportSection> Sections { get; set; } = new();
}

public class SessionSummary
{
    public Guid SessionId { get; set; }
    public DateTime StartTime { get; set; }
    public DateTime? EndTime { get; set; }
    public TimeSpan Duration { get; set; }
    public string[] EnabledModules { get; set; } = Array.Empty<string>();
}

public class ReportSection
{
    public string Title { get; set; } = string.Empty;
    public ReportPriority Priority { get; set; }
    public Dictionary<string, object> Content { get; set; } = new();
}

public class ReportOptions
{
    public string OutputPath { get; set; } = string.Empty;
    public ReportFormat Format { get; set; } = ReportFormat.Json;
    public bool IncludeRawData { get; set; } = false;
    public bool IncludeCharts { get; set; } = true;
}

public enum ReportFormat
{
    Json,
    Html,
    Pdf
}

public enum ReportPriority
{
    Low,
    Medium,
    High,
    Critical
}

public enum PerformanceRating
{
    Unknown,
    Critical,
    Poor,
    Fair,
    Good,
    Excellent
}

public class PerformanceRecommendation
{
    public RecommendationPriority Priority { get; set; }
    public string Category { get; set; } = string.Empty;
    public string Title { get; set; } = string.Empty;
    public string Description { get; set; } = string.Empty;
    public string Impact { get; set; } = string.Empty;
}

public enum RecommendationPriority
{
    Low,
    Medium,
    High,
    Critical
}
        _gcEventListener.Stop();

        var allocations = _allocationTracker.GetAllocations();
        var gcEvents = _gcEventListener.GetEvents();
        var heapSnapshots = _heapAnalyzer.GetSnapshots();

        // Analyze for memory leaks
        var leakAnalysis = AnalyzeMemoryLeaks(allocations, gcEvents);

        return new MemoryProfilingData
        {
            Allocations = allocations,
            GCEvents = gcEvents,
            HeapSnapshots = heapSnapshots,
            LeakAnalysis = leakAnalysis,
            TotalAllocatedBytes = allocations.Sum(a => a.Size),
            PeakMemoryUsage = GetPeakMemoryUsage()
        };
    }

    private MemoryLeakAnalysis AnalyzeMemoryLeaks(
        IReadOnlyList<AllocationEvent> allocations,
        IReadOnlyList<GCEvent> gcEvents)
    {
        var analysis = new MemoryLeakAnalysis();

        // Group allocations by type
        var allocationsByType = allocations.GroupBy(a => a.Type);

        foreach (var group in allocationsByType)
        {
            var totalAllocated = group.Sum(a => a.Size);
            var totalReleased = CalculateReleasedMemory(group, gcEvents);
            var potentialLeak = totalAllocated - totalReleased;

            if (potentialLeak > analysis.LeakThreshold)
            {
                analysis.PotentialLeaks.Add(new MemoryLeak
                {
                    Type = group.Key,
                    AllocatedBytes = totalAllocated,
                    ReleasedBytes = totalReleased,
                    PotentialLeakBytes = potentialLeak
                });
            }
        }

        return analysis;
    }
}

// Real-time performance monitoring
public class RealTimeMonitor : IRealTimeMonitor
{
    private readonly PerformanceCounterService _perfCounters;
    private readonly EventEmitter _eventEmitter;
    private readonly MetricsCollector _metricsCollector;

    public async Task StartMonitoringAsync(MonitoringConfiguration config)
    {
        // Start performance counter collection
        await _perfCounters.StartAsync(config.CounterNames, config.SamplingInterval);

        // Begin metrics collection
        _metricsCollector.Start();

        // Set up real-time event emission
        _eventEmitter.StartEmitting(config.EventInterval);
    }

    public IObservable<PerformanceMetrics> GetMetricsStream()
    {
        return Observable.Interval(TimeSpan.FromSeconds(1))
            .Select(_ => CollectCurrentMetrics())
            .Where(metrics => metrics != null);
    }

    private PerformanceMetrics CollectCurrentMetrics()
    {
        return new PerformanceMetrics
        {
            Timestamp = DateTime.UtcNow,
            CpuUsage = _perfCounters.GetCpuUsage(),
            MemoryUsage = _perfCounters.GetMemoryUsage(),
            GCCollections = GC.CollectionCount(0) + GC.CollectionCount(1) + GC.CollectionCount(2),
            ThreadCount = Process.GetCurrentProcess().Threads.Count,
            HandleCount = Process.GetCurrentProcess().HandleCount,
            CustomMetrics = _metricsCollector.GetCustomMetrics()
        };
    }
}

// Performance report generation
public class ReportGenerator : IReportGenerator
{
    private readonly ITemplateEngine _templateEngine;
    private readonly IChartGenerator _chartGenerator;

    public async Task<PerformanceReport> GenerateAsync(ProfilingResults results, ReportOptions options)
    {
        var report = new PerformanceReport
        {
            SessionInfo = results.Session,
            GeneratedAt = DateTime.UtcNow,
            Options = options
        };

        // Generate executive summary
        report.ExecutiveSummary = GenerateExecutiveSummary(results);

        // Generate detailed sections
        if (options.IncludeCpuAnalysis)
        {
            report.CpuAnalysis = await GenerateCpuAnalysisAsync(results.GetData<CpuProfilingData>());
        }

        if (options.IncludeMemoryAnalysis)
        {
            report.MemoryAnalysis = await GenerateMemoryAnalysisAsync(results.GetData<MemoryProfilingData>());
        }

        if (options.IncludeVisualizations)
        {
            report.Charts = await GenerateChartsAsync(results);
        }

        // Generate recommendations
        report.Recommendations = GenerateRecommendations(results);

        return report;
    }

    private async Task<CpuAnalysisSection> GenerateCpuAnalysisAsync(CpuProfilingData cpuData)
    {
        var analysis = new CpuAnalysisSection();

        // Find hot methods
        analysis.HotMethods = cpuData.MethodStatistics
            .OrderByDescending(m => m.ExclusiveTime)
            .Take(10)
            .ToList();

        // Analyze call patterns
        analysis.CallPatterns = AnalyzeCallPatterns(cpuData.StackSamples);

        // Generate flame graph data
        analysis.FlameGraphData = GenerateFlameGraphData(cpuData.StackSamples);

        return analysis;
    }

    private List<PerformanceRecommendation> GenerateRecommendations(ProfilingResults results)
    {
        var recommendations = new List<PerformanceRecommendation>();

        // CPU recommendations
        var cpuData = results.GetData<CpuProfilingData>();
        if (cpuData != null)
        {
            var hottestMethod = cpuData.MethodStatistics.OrderByDescending(m => m.ExclusiveTime).FirstOrDefault();
            if (hottestMethod?.ExclusiveTime > TimeSpan.FromMilliseconds(100))
            {
                recommendations.Add(new PerformanceRecommendation
                {
                    Category = "CPU",
                    Priority = Priority.High,
                    Title = $"Optimize hot method: {hottestMethod.MethodName}",
                    Description = $"Method {hottestMethod.MethodName} consumes {hottestMethod.ExclusiveTime.TotalMilliseconds:F1}ms per call",
                    Impact = "High - significant CPU usage reduction possible"
                });
            }
        }

        // Memory recommendations
        var memoryData = results.GetData<MemoryProfilingData>();
        if (memoryData?.LeakAnalysis.PotentialLeaks.Any() == true)
        {
            foreach (var leak in memoryData.LeakAnalysis.PotentialLeaks)
            {
                recommendations.Add(new PerformanceRecommendation
                {
                    Category = "Memory",
                    Priority = Priority.High,
                    Title = $"Potential memory leak: {leak.Type}",
                    Description = $"Type {leak.Type} has {leak.PotentialLeakBytes:N0} bytes potentially leaked",
                    Impact = "High - memory usage will grow over time"
                });
            }
        }

        return recommendations;
    }
}
```

## Key Features

### 1. Comprehensive Profiling

- **CPU Profiling** - Method timing and hot path analysis
- **Memory Profiling** - Allocation tracking and leak detection
- **I/O Profiling** - File and network operation monitoring
- **Threading Analysis** - Concurrency and synchronization profiling

### 2. Real-Time Monitoring

- **Live Dashboards** - Real-time performance visualization
- **Alert System** - Threshold-based performance alerts
- **Metrics Streaming** - Continuous performance data streams
- **Historical Tracking** - Long-term performance trend analysis

### 3. Advanced Analytics

- **Comparative Analysis** - Before/after performance comparison
- **Regression Detection** - Automated performance regression alerts
- **Root Cause Analysis** - Intelligent performance issue diagnosis
- **Predictive Analytics** - Performance trend forecasting

## 🚀 Extension Challenges

### ⭐⭐⭐ **Challenge 1: Machine Learning Performance Prediction**

Implement ML-based performance prediction and optimization:

```csharp
public class MLPerformancePredictor
{
    public async Task<PerformancePrediction> PredictPerformanceAsync(ApplicationMetrics metrics)
    {
        // Train ML models on historical performance data
        // Predict future performance bottlenecks before they occur
        // Provide proactive optimization recommendations
        // Continuous learning from new profiling data
    }
}
```

**Requirements:**

- Train ML models on historical performance data
- Predict future performance bottlenecks before they occur
- Provide proactive optimization recommendations
- Continuous learning from new profiling data

### ⭐⭐⭐⭐ **Challenge 2: Distributed Performance Profiling**

Build distributed profiling across multiple services/containers:

```csharp
public class DistributedProfiler
{
    public async Task<DistributedProfilingResults> ProfileDistributedSystemAsync(ServiceTopology topology)
    {
        // Cross-service performance correlation and analysis
        // Integration with distributed tracing systems (OpenTelemetry)
        // Service dependency performance impact analysis
        // Containerized and microservice environment support
    }
}
```

**Requirements:**

- Cross-service performance correlation and analysis
- Integration with distributed tracing systems (OpenTelemetry)
- Service dependency performance impact analysis
- Containerized and microservice environment support

### ⭐⭐⭐⭐ **Challenge 3: Automated Performance Regression Detection**

Implement CI/CD integration with automated regression detection:

```csharp
public class PerformanceRegressionDetector
{
    public async Task<RegressionReport> DetectRegressionsAsync(ProfilingResults current, ProfilingResults baseline)
    {
        // Statistical analysis for performance regression detection
        // Automated alerting and notification systems
        // CI/CD pipeline integration with build gates
        // Baseline performance tracking and comparison
    }
}
```

**Requirements:**

- Statistical analysis for performance regression detection
- Automated alerting and notification systems
- CI/CD pipeline integration with build gates
- Baseline performance tracking and comparison

### ⭐⭐⭐⭐⭐ **Challenge 4: Real-Time Performance Optimization**

Build system that automatically optimizes performance in real-time:

```csharp
public class RealTimeOptimizer
{
    public async Task<OptimizationResult> OptimizePerformanceAsync(PerformanceMetrics metrics)
    {
        // Dynamic resource allocation based on performance metrics
        // Automatic application configuration tuning
        // Real-time JIT optimization hints and suggestions
        // Safe rollback mechanisms for failed optimizations
    }
}
```

**Requirements:**

- Dynamic resource allocation based on performance metrics
- Automatic application configuration tuning
- Real-time JIT optimization hints and suggestions
- Safe rollback mechanisms for failed optimizations

### ⭐⭐⭐⭐⭐ **Challenge 5: GPU and Hardware Profiling**

Extend profiling to include GPU and specialized hardware:

```csharp
public class HardwareProfiler
{
    public async Task<HardwareProfilingData> ProfileHardwareAsync(ProfilingSession session)
    {
        // GPU utilization and memory profiling (NVIDIA/AMD)
        // CUDA/OpenCL kernel performance analysis
        // Hardware-specific optimization recommendations
        // Multi-GPU and heterogeneous computing support
    }
}
```

**Requirements:**

- GPU utilization and memory profiling (NVIDIA/AMD)
- CUDA/OpenCL kernel performance analysis
- Hardware-specific optimization recommendations
- Multi-GPU and heterogeneous computing support

### ⭐⭐⭐⭐⭐ **Challenge 6: Performance Digital Twin**

Create a digital twin that simulates application performance:

```csharp
public class PerformanceDigitalTwin
{
    public async Task<SimulationResults> SimulatePerformanceAsync(ApplicationModel model, LoadScenario scenario)
    {
        // Comprehensive performance simulation and modeling
        // What-if analysis for architectural and infrastructure changes
        // Capacity planning with scaling predictions
        // Integration with cloud infrastructure APIs for cost analysis
    }
}
```

**Requirements:**

- Comprehensive performance simulation and modeling
- What-if analysis for architectural and infrastructure changes
- Capacity planning with scaling predictions
- Integration with cloud infrastructure APIs for cost analysis

### ⭐⭐⭐⭐⭐ **Challenge 7: AI-Powered Performance Optimization**

Build an AI system that automatically optimizes code performance:

```csharp
public class AIPerformanceOptimizer
{
    public async Task<OptimizationSuggestion[]> AnalyzeAndOptimizeAsync(SourceCode code, PerformanceMetrics metrics)
    {
        // Deep code analysis with performance correlation
        // AI-powered optimization recommendations
        // Automatic code refactoring suggestions
        // Performance impact prediction for changes
    }
}
```

**Requirements:**

- Deep code analysis with performance correlation
- AI-powered optimization recommendations
- Automatic code refactoring suggestions
- Performance impact prediction for proposed changes

## Sample Applications

Profile these types of applications to demonstrate capabilities:

1. **Web API** - Request/response performance analysis
2. **Game Engine** - Frame rate and rendering performance
3. **Data Processing** - Batch processing optimization
4. **Real-Time System** - Latency and throughput analysis

## Integration Tools

Build comprehensive integration capabilities:

- **CI/CD Integration** - Automated performance testing
- **IDE Extensions** - Visual Studio profiling integration
- **API Monitoring** - Application performance monitoring
- **Load Testing** - Performance under stress conditions

## Visualization Features

Create rich performance visualizations:

- **Flame Graphs** - Call stack visualization
- **Timeline Charts** - Execution timeline analysis
- **Memory Maps** - Heap visualization
- **Performance Dashboards** - Real-time monitoring displays

## Resources

- 📚 **[Progress Tracker](../progress-tracker.md)** - Track your learning progress
- 📖 **[Performance Profiling](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/)** - .NET profiling tools
- 🧠 **[Performance Analysis](https://docs.microsoft.com/en-us/windows-hardware/test/wpt/)** - Windows Performance Toolkit
- 📘 **[Optimization Techniques](https://www.oreilly.com/library/view/high-performance-net/9781785288463/)** - .NET optimization
- 📘 **[Profiling Best Practices](https://blog.jetbrains.com/dotnet/2017/11/14/performance-profiling-asp-net-core-applications/)** - Profiling strategies

---

### Step 8: Console Demo Application (30 minutes)

**PerformanceProfiler.App/Program.cs**

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Extensions.Logging;
using PerformanceProfiler.Core.Core;
using PerformanceProfiler.Core.Models;
using PerformanceProfiler.Core.RealTime;
using PerformanceProfiler.Core.Reporting;
using System.Diagnostics;

namespace PerformanceProfiler.App;

class Program
{
    static async Task Main(string[] args)
    {
        Console.WriteLine("🚀 Performance Profiler Demo");
        Console.WriteLine("============================\n");

        // Setup dependency injection
        var services = new ServiceCollection();
        ConfigureServices(services);

        using var serviceProvider = services.BuildServiceProvider();

        try
        {
            // Demo 1: Basic profiling session
            await DemoBasicProfilingAsync(serviceProvider);

            // Demo 2: Real-time monitoring
            await DemoRealTimeMonitoringAsync(serviceProvider);

            // Demo 3: Performance reporting
            await DemoPerformanceReportingAsync(serviceProvider);

            Console.WriteLine("\n✅ All demos completed successfully!");
            Console.WriteLine("Press any key to exit...");
            Console.ReadKey();
        }
        catch (Exception ex)
        {
            Console.WriteLine($"\n❌ Demo failed: {ex.Message}");
            Console.WriteLine($"Stack trace: {ex.StackTrace}");
        }
    }

    private static void ConfigureServices(ServiceCollection services)
    {
        services.AddLogging(builder => builder.AddConsole().SetMinimumLevel(LogLevel.Information));

        // Register profiler services
        services.AddSingleton<IPerformanceProfiler, PerformanceProfilerImplementation>();
        services.AddSingleton<IReportGenerator, ReportGenerator>();
        services.AddSingleton<RealTimeMonitor>();

        // Register profiling modules
        services.AddTransient<IProfilerModule, CPUProfiler>();
        services.AddTransient<IProfilerModule, MemoryProfiler>();
        services.AddTransient<IProfilerModule, IOProfiler>();
    }

    private static async Task DemoBasicProfilingAsync(ServiceProvider serviceProvider)
    {
        Console.WriteLine("📊 Demo: Basic Performance Profiling");
        Console.WriteLine("====================================");

        var profiler = serviceProvider.GetRequiredService<IPerformanceProfiler>();

        // Configure profiling options
        var options = new ProfilingOptions
        {
            EnableCpuProfiling = true,
            EnableMemoryProfiling = true,
            EnableIOProfiling = true,
            SamplingInterval = TimeSpan.FromMilliseconds(100),
            OutputPath = "demo_results.json"
        };

        // Start profiling session
        Console.WriteLine("Starting profiling session...");
        var session = await profiler.StartProfilingAsync(options);

        // Simulate some work to profile
        Console.WriteLine("Performing work to profile...");
        await SimulateWorkload();

        // Stop profiling and collect results
        Console.WriteLine("Stopping profiling session...");
        var results = await profiler.StopProfilingAsync(session);

        // Display summary
        Console.WriteLine($"✅ Profiling completed:");
        Console.WriteLine($"   - Session Duration: {results.Session.Duration:mm\\:ss}");
        Console.WriteLine($"   - Modules Profiled: {results.Data.Count}");
        Console.WriteLine($"   - Results Generated: {results.GeneratedAt:HH:mm:ss}");

        Console.WriteLine();
    }

    private static async Task DemoRealTimeMonitoringAsync(ServiceProvider serviceProvider)
    {
        Console.WriteLine("📈 Demo: Real-Time Performance Monitoring");
        Console.WriteLine("==========================================");

        var monitor = serviceProvider.GetRequiredService<RealTimeMonitor>();

        // Subscribe to alerts
        monitor.AlertTriggered += (s, e) =>
        {
            Console.WriteLine($"🚨 {e.Level} Alert: {e.Message}");
        };

        // Subscribe to metric updates
        var metricCount = 0;
        monitor.MetricReceived += (s, e) =>
        {
            metricCount++;
            if (metricCount % 10 == 0) // Show every 10th metric
            {
                Console.WriteLine($"📊 {e.Metric.Category}/{e.Metric.Name}: {e.Metric.Value:F1} {e.Metric.Unit}");
            }
        };

        Console.WriteLine("Real-time monitoring started. Collecting metrics for 10 seconds...");

        // Simulate some CPU-intensive work to trigger alerts
        var cpuTask = Task.Run(() =>
        {
            var sw = Stopwatch.StartNew();
            while (sw.Elapsed < TimeSpan.FromSeconds(5))
            {
                // Busy wait to generate CPU load
                Math.Sqrt(Random.Shared.NextDouble());
            }
        });

        await Task.Delay(TimeSpan.FromSeconds(10));

        Console.WriteLine($"✅ Collected {metricCount} metrics during monitoring period");

        // Get recent metrics
        var recentMetrics = monitor.GetRecentMetrics(TimeSpan.FromMinutes(1));
        Console.WriteLine($"   - Recent metrics available: {recentMetrics.Length}");

        Console.WriteLine();
    }

    private static async Task DemoPerformanceReportingAsync(ServiceProvider serviceProvider)
    {
        Console.WriteLine("📄 Demo: Performance Report Generation");
        Console.WriteLine("======================================");

        var profiler = serviceProvider.GetRequiredService<IPerformanceProfiler>();
        var reportGenerator = serviceProvider.GetRequiredService<IReportGenerator>();

        // Quick profiling session for reporting
        var options = new ProfilingOptions
        {
            EnableCpuProfiling = true,
            EnableMemoryProfiling = true,
            SamplingInterval = TimeSpan.FromMilliseconds(50)
        };

        var session = await profiler.StartProfilingAsync(options);
        await SimulateWorkload();
        var results = await profiler.StopProfilingAsync(session);

        // Generate comprehensive report
        var reportOptions = new ReportOptions
        {
            OutputPath = "performance_report.json",
            Format = ReportFormat.Json,
            IncludeRawData = false,
            IncludeCharts = true
        };

        Console.WriteLine("Generating performance report...");
        var report = await reportGenerator.GenerateAsync(results, reportOptions);

        Console.WriteLine($"✅ Performance report generated:");
        Console.WriteLine($"   - Report Sections: {report.Sections.Count}");
        Console.WriteLine($"   - Generated At: {report.GeneratedAt:HH:mm:ss}");

        // Display key findings
        var summarySection = report.Sections.FirstOrDefault(s => s.Title == "Executive Summary");
        if (summarySection?.Content.TryGetValue("summary", out var summaryObj) == true)
        {
            Console.WriteLine($"   - Executive Summary Available");
        }

        var recommendationSection = report.Sections.FirstOrDefault(s => s.Title == "Performance Recommendations");
        if (recommendationSection?.Content.TryGetValue("recommendations", out var recommendationsObj) == true)
        {
            Console.WriteLine($"   - Performance Recommendations Available");
        }

        Console.WriteLine($"   - Report saved to: {reportOptions.OutputPath}");
        Console.WriteLine();
    }

    private static async Task SimulateWorkload()
    {
        // CPU-intensive work
        var cpuTask = Task.Run(() =>
        {
            for (int i = 0; i < 1000000; i++)
            {
                Math.Sqrt(i);
            }
        });

        // Memory allocations
        var memoryTask = Task.Run(() =>
        {
            var lists = new List<byte[]>();
            for (int i = 0; i < 100; i++)
            {
                lists.Add(new byte[1024 * 1024]); // 1MB allocations
                Thread.Sleep(10);
            }
        });

        // I/O operations
        var ioTask = Task.Run(async () =>
        {
            var tempFiles = new List<string>();
            for (int i = 0; i < 10; i++)
            {
                var fileName = Path.GetTempFileName();
                await File.WriteAllTextAsync(fileName, $"Test data {i}".PadRight(1000, 'x'));
                tempFiles.Add(fileName);
            }

            // Clean up
            foreach (var file in tempFiles)
            {
                try { File.Delete(file); } catch { }
            }
        });

        await Task.WhenAll(cpuTask, memoryTask, ioTask);
    }
}

// Mock implementation interfaces for demo
public interface IPerformanceProfiler
{
    Task<ProfilingSession> StartProfilingAsync(ProfilingOptions options);
    Task<ProfilingResults> StopProfilingAsync(ProfilingSession session);
}

public interface IProfilerModule
{
    string Name { get; }
    Task StartAsync(ProfilingSession session);
    Task<IProfilingData> StopAndCollectAsync(ProfilingSession session);
}

// Simplified implementations for demo
public class PerformanceProfilerImplementation : IPerformanceProfiler
{
    private readonly ILogger<PerformanceProfilerImplementation> _logger;
    private readonly IEnumerable<IProfilerModule> _modules;

    public PerformanceProfilerImplementation(
        ILogger<PerformanceProfilerImplementation> logger,
        IEnumerable<IProfilerModule> modules)
    {
        _logger = logger;
        _modules = modules;
    }

    public async Task<ProfilingSession> StartProfilingAsync(ProfilingOptions options)
    {
        var session = new ProfilingSession(Guid.NewGuid(), options);
        session.Start();

        foreach (var module in _modules)
        {
            if (options.IsModuleEnabled(module.Name))
            {
                await module.StartAsync(session);
                _logger.LogInformation("Started profiling module: {ModuleName}", module.Name);
            }
        }

        return session;
    }

    public async Task<ProfilingResults> StopProfilingAsync(ProfilingSession session)
    {
        session.Stop();

        var data = new List<IProfilingData>();
        foreach (var module in _modules)
        {
            if (session.Options.IsModuleEnabled(module.Name))
            {
                var moduleData = await module.StopAndCollectAsync(session);
                data.Add(moduleData);
                _logger.LogInformation("Collected data from module: {ModuleName}", module.Name);
            }
        }

        return new ProfilingResults(session, data);
    }
}

// Mock profiler modules for demo
public class CPUProfiler : IProfilerModule
{
    public string Name => "cpu";

    public async Task StartAsync(ProfilingSession session)
    {
        await Task.CompletedTask;
    }

    public async Task<IProfilingData> StopAndCollectAsync(ProfilingSession session)
    {
        await Task.CompletedTask;
        return new CpuProfilingData
        {
            StartTime = session.StartTime,
            EndTime = session.EndTime ?? DateTime.UtcNow,
            CpuUsagePercent = Random.Shared.Next(20, 90),
            TotalSamples = Random.Shared.Next(1000, 5000)
        };
    }
}

public class MemoryProfiler : IProfilerModule
{
    public string Name => "memory";

    public async Task StartAsync(ProfilingSession session)
    {
        await Task.CompletedTask;
    }

    public async Task<IProfilingData> StopAndCollectAsync(ProfilingSession session)
    {
        await Task.CompletedTask;
        return new MemoryProfilingData
        {
            StartTime = session.StartTime,
            EndTime = session.EndTime ?? DateTime.UtcNow,
            TotalAllocatedBytes = Random.Shared.Next(50, 200) * 1024 * 1024,
            GCCollections = Random.Shared.Next(10, 100)
        };
    }
}
```

## Building and Testing

```bash
# Build the solution
dotnet build

# Run all tests
dotnet test

# Run the profiler demo
dotnet run --project PerformanceProfiler.App

# Run with custom profiling options
dotnet run --project PerformanceProfiler.App -- --cpu --memory --duration 30
```

## Success Criteria

✅ **Core Profiling Engine**: Implemented modular profiling architecture  
✅ **CPU Profiling**: Stack sampling and method-level performance analysis  
✅ **Memory Profiling**: Allocation tracking and GC event monitoring  
✅ **I/O Profiling**: File system and disk performance monitoring  
✅ **Real-Time Monitoring**: Live performance metrics with alerting  
✅ **Comprehensive Reporting**: Automated performance analysis and recommendations  
✅ **Professional Integration**: CI/CD ready with JSON/HTML/PDF reports  
✅ **Enterprise Features**: Distributed profiling and performance baselines

## Professional Standards

This capstone follows industry profiling standards:

- **Non-Intrusive Profiling** - Minimal impact on application performance
- **Statistical Accuracy** - Reliable and reproducible measurements
- **Comprehensive Coverage** - Full application lifecycle profiling
- **Actionable Insights** - Clear recommendations for optimization
- **Enterprise Integration** - Scalable monitoring solutions

## 📚 Essential Documentation for This Project

**Performance profiling requires deep understanding of .NET internals and system performance:**

### Core Profiling APIs

- [Microsoft.Diagnostics.Runtime](https://docs.microsoft.com/en-us/dotnet/core/diagnostics/dotnet-dump) - ClrMD for memory analysis
- [System.Diagnostics.PerformanceCounter](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.performancecounter) - System performance counters
- [BenchmarkDotNet](https://benchmarkdotnet.org/) - Micro-benchmarking framework
- [ETW (Event Tracing for Windows)](https://docs.microsoft.com/en-us/windows/win32/etw/event-tracing-portal) - Low-level system tracing

### Performance Analysis

- [.NET Application Performance](https://docs.microsoft.com/en-us/dotnet/standard/performance/) - Performance best practices
- [Garbage Collection Performance](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/performance) - GC optimization
- [JIT Compilation](https://docs.microsoft.com/en-us/dotnet/standard/managed-execution-process) - Understanding JIT performance

### Monitoring and Observability

- [Application Performance Monitoring](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) - APM concepts
- [OpenTelemetry .NET](https://opentelemetry.io/docs/instrumentation/net/) - Modern observability standards
- [Distributed Tracing](https://docs.microsoft.com/en-us/azure/azure-monitor/app/distributed-tracing) - Cross-service performance

**💡 Study Strategy**: Start with the .NET diagnostics documentation and BenchmarkDotNet tutorials. Understanding memory management and GC behavior is crucial for effective profiling.

## Resources

- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[.NET Performance Documentation](https://learn.microsoft.com/en-us/dotnet/standard/performance/)**
- 📘 **[ClrMD Documentation](https://github.com/Microsoft/clrmd)** - Memory dump analysis
- 📘 **[Performance Counter Guide](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.performancecounter)** - System monitoring
- 📘 **[BenchmarkDotNet](https://benchmarkdotnet.org/articles/overview.html)** - Micro-benchmarking

## Contributing

When implementing this profiling suite:

1. Design for minimal performance impact during profiling
2. Implement comprehensive statistical analysis methods
3. Create clear, actionable performance reports
4. Build robust integration capabilities
5. Focus on developer experience and usability
