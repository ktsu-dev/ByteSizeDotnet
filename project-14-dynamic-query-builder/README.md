# Project 14: Dynamic Query Builder

**Focus**: Expression Trees & Dynamic Code  
**Time**: ~2 hours  
**Deliverable**: `QueryBuilder` library with expression tree compilation

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Expression Tree Guidelines](../microsoft-design-guidelines.md#expression-tree-guidelines)** - Building and compiling expressions
- **[Dynamic Programming Guidelines](../microsoft-design-guidelines.md#dynamic-programming)** - Safe dynamic code generation
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - Compiled expression optimization

## Learning Objectives

- Master expression trees and runtime code generation
- Understand dynamic programming and code compilation
- Learn query composition and optimization techniques
- Practice functional programming with expressions
- Build high-performance query systems

## Quick Start

1. **Read the Guidelines** 📖  
   Review [Expression Tree Guidelines](../microsoft-design-guidelines.md#expression-tree-guidelines) before starting

2. **Start Coding** 💻  
   See [detailed project guide](dynamic-query-builder-project-guide.md) for step-by-step instructions

3. **Track Progress** 🎯  
   Update your [progress tracker](../progress-tracker.md) as you complete milestones

## Project Structure

```
project-14-dynamic-query-builder/
├── README.md                           # This file
├── dynamic-query-builder-project-guide.md # Detailed implementation guide
├── QueryBuilder.sln             # Solution file
├── QueryBuilder.Core/           # Main library implementation
├── QueryBuilder.App/            # Console application
└── QueryBuilder.Tests/          # Unit tests
├── tests/                              # Unit tests
└── docs/                               # Additional documentation
```

## What You'll Build

A sophisticated query builder system featuring:

- Expression tree-based query composition
- Dynamic predicate compilation and caching
- Fluent query API with strong typing
- Query optimization and transformation
- Support for complex filtering and sorting
- Integration with various data sources

## Key Concepts

- **Expression Trees**: Building code as data structures
- **Dynamic Compilation**: Converting expressions to executable code
- **Predicate Composition**: Combining query conditions
- **Query Optimization**: Transforming expressions for performance
- **Visitor Pattern**: Traversing and modifying expression trees
- **Caching**: Storing compiled expressions for reuse

## Resources

- 📚 **[Detailed Guide](dynamic-query-builder-project-guide.md)** - Complete implementation instructions
- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Expression Trees Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/)**
- 📘 **[Dynamic Programming Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/interop/)**

## Building on Previous Projects

This project continues Phase 4 advanced .NET features:

- **Projects 1-12**: All foundational skills in advanced runtime context
- **Project 13**: Reflection for type inspection in query building
- **New Skills**: Expression trees, dynamic compilation, and query optimization

## Example Usage

```csharp
// Fluent query builder with expression trees
var queryBuilder = new QueryBuilder<Player>();

var query = queryBuilder
    .Where(player => player.Level > 10)
    .And(player => player.Experience >= 1000)
    .Or(player => player.HasAchievement("Legendary"))
    .OrderBy(player => player.Level)
    .ThenBy(player => player.Name)
    .Select(player => new
    {
        player.Name,
        player.Level,
        player.Guild.Name
    });

// Compile to optimized delegate
var compiledQuery = query.Compile();
var results = compiledQuery(playerList);

// Dynamic query building from user input
var dynamicQuery = new DynamicQueryBuilder<Item>()
    .AddCondition("Type", ComparisonOperator.Equal, "Weapon")
    .AddCondition("Level", ComparisonOperator.GreaterThan, 5)
    .AddCondition("Rarity", ComparisonOperator.In, new[] { "Rare", "Epic", "Legendary" })
    .OrderBy("Level", SortDirection.Descending);

var items = itemRepository.Query(dynamicQuery);

// Expression tree manipulation
var originalExpression = (Expression<Func<Player, bool>>)(p => p.Level > 5);
var modifiedExpression = ExpressionRewriter.Rewrite(
    originalExpression,
    rewriter => rewriter.Replace(
        expr => expr.NodeType == ExpressionType.Constant && (int)((ConstantExpression)expr).Value == 5,
        Expression.Constant(10)
    )
);

// Query optimization
var optimizer = new QueryOptimizer<Player>()
    .AddRule(new ConstantFoldingRule())
    .AddRule(new RedundantConditionRule())
    .AddRule(new IndexHintRule());

var optimizedQuery = optimizer.Optimize(query);
```

## Advanced Expression Patterns

This project explores sophisticated expression tree techniques:

- **Expression Visitors**: Custom tree traversal and modification
- **Query Translation**: Converting expressions to different target languages
- **Partial Evaluation**: Compile-time constant folding
- **Expression Caching**: Memoizing compiled expressions
- **Type Safety**: Maintaining strong typing in dynamic scenarios

## Performance Focus

This project emphasizes high-performance query execution:

- Expression compilation and caching strategies
- Query plan optimization
- Memory-efficient expression tree construction
- Minimizing dynamic allocations
- Benchmark-driven optimization

## Project Setup Instructions

### Step 1: Create the Solution Structure (15 minutes)

Navigate to the `project-14-dynamic-query-builder/` directory:

```bash
dotnet new sln -n QueryBuilder
dotnet new classlib -n QueryBuilder.Core -f net8.0
dotnet new console -n QueryBuilder.App -f net8.0
dotnet new mstest -n QueryBuilder.Tests -f net8.0

dotnet sln add QueryBuilder.Core/QueryBuilder.Core.csproj
dotnet sln add QueryBuilder.App/QueryBuilder.App.csproj
dotnet sln add QueryBuilder.Tests/QueryBuilder.Tests.csproj

dotnet add QueryBuilder.App/QueryBuilder.App.csproj reference QueryBuilder.Core/QueryBuilder.Core.csproj
dotnet add QueryBuilder.Tests/QueryBuilder.Tests.csproj reference QueryBuilder.Core/QueryBuilder.Core.csproj
```

### Step 2: Directory Structure

```bash
mkdir QueryBuilder.Core/Expressions
mkdir QueryBuilder.Core/Compilation
mkdir QueryBuilder.Core/Optimization
mkdir QueryBuilder.Core/Visitors
mkdir QueryBuilder.Core/Models
mkdir QueryBuilder.Core/Fluent
mkdir QueryBuilder.Core/Dynamic
```

### Step 3: Core Expression Models (30 minutes)

Create `QueryBuilder.Core/Models/QueryModels.cs`:

```csharp
using System.Linq.Expressions;

namespace QueryBuilder.Core.Models;

public enum ComparisonOperator
{
    Equal,
    NotEqual,
    GreaterThan,
    GreaterThanOrEqual,
    LessThan,
    LessThanOrEqual,
    Contains,
    StartsWith,
    EndsWith,
    In,
    NotIn
}

public enum SortDirection
{
    Ascending,
    Descending
}

public class QueryCondition
{
    public string PropertyName { get; set; } = string.Empty;
    public ComparisonOperator Operator { get; set; }
    public object? Value { get; set; }
    public LogicalOperator LogicalOperator { get; set; } = LogicalOperator.And;
}

public enum LogicalOperator
{
    And,
    Or
}

public class SortExpression
{
    public string PropertyName { get; set; } = string.Empty;
    public SortDirection Direction { get; set; } = SortDirection.Ascending;
}

public class CompiledQuery<T>
{
    public Func<IEnumerable<T>, IEnumerable<T>>? FilterFunc { get; set; }
    public Func<IEnumerable<T>, IOrderedEnumerable<T>>? SortFunc { get; set; }
    public Expression<Func<T, bool>>? FilterExpression { get; set; }
    public DateTime CompiledAt { get; set; } = DateTime.UtcNow;
    public TimeSpan CompilationTime { get; set; }
}
```

### Step 4: Expression Builder Foundation (45 minutes)

Create `QueryBuilder.Core/Expressions/ExpressionBuilder.cs`:

```csharp
using System.Linq.Expressions;
using System.Reflection;
using QueryBuilder.Core.Models;

namespace QueryBuilder.Core.Expressions;

public static class ExpressionBuilder
{
    public static Expression<Func<T, bool>> BuildPredicate<T>(QueryCondition condition)
    {
        var parameter = Expression.Parameter(typeof(T), "x");
        var property = GetPropertyExpression(parameter, condition.PropertyName);
        var value = Expression.Constant(condition.Value, property.Type);

        var comparison = condition.Operator switch
        {
            ComparisonOperator.Equal => Expression.Equal(property, value),
            ComparisonOperator.NotEqual => Expression.NotEqual(property, value),
            ComparisonOperator.GreaterThan => Expression.GreaterThan(property, value),
            ComparisonOperator.GreaterThanOrEqual => Expression.GreaterThanOrEqual(property, value),
            ComparisonOperator.LessThan => Expression.LessThan(property, value),
            ComparisonOperator.LessThanOrEqual => Expression.LessThanOrEqual(property, value),
            ComparisonOperator.Contains => BuildContainsExpression(property, value),
            ComparisonOperator.StartsWith => BuildStringMethodExpression(property, value, "StartsWith"),
            ComparisonOperator.EndsWith => BuildStringMethodExpression(property, value, "EndsWith"),
            ComparisonOperator.In => BuildInExpression(property, condition.Value),
            ComparisonOperator.NotIn => Expression.Not(BuildInExpression(property, condition.Value)),
            _ => throw new ArgumentException($"Unsupported operator: {condition.Operator}")
        };

        return Expression.Lambda<Func<T, bool>>(comparison, parameter);
    }

    public static Expression<Func<T, bool>> CombinePredicates<T>(
        Expression<Func<T, bool>> left,
        Expression<Func<T, bool>> right,
        LogicalOperator logicalOperator)
    {
        var parameter = Expression.Parameter(typeof(T), "x");

        var leftBody = ReplaceParameter(left.Body, left.Parameters[0], parameter);
        var rightBody = ReplaceParameter(right.Body, right.Parameters[0], parameter);

        var combined = logicalOperator == LogicalOperator.And
            ? Expression.AndAlso(leftBody, rightBody)
            : Expression.OrElse(leftBody, rightBody);

        return Expression.Lambda<Func<T, bool>>(combined, parameter);
    }

    public static Expression<Func<T, TKey>> BuildKeySelector<T, TKey>(string propertyName)
    {
        var parameter = Expression.Parameter(typeof(T), "x");
        var property = GetPropertyExpression(parameter, propertyName);

        // Handle type conversion if necessary
        Expression convertedProperty = property;
        if (property.Type != typeof(TKey))
        {
            convertedProperty = Expression.Convert(property, typeof(TKey));
        }

        return Expression.Lambda<Func<T, TKey>>(convertedProperty, parameter);
    }

    private static Expression GetPropertyExpression(Expression parameter, string propertyName)
    {
        // Support nested properties like "Player.Guild.Name"
        var properties = propertyName.Split('.');
        Expression current = parameter;

        foreach (var prop in properties)
        {
            var propertyInfo = current.Type.GetProperty(prop, BindingFlags.Public | BindingFlags.Instance);
            if (propertyInfo == null)
            {
                throw new ArgumentException($"Property '{prop}' not found on type '{current.Type.Name}'");
            }
            current = Expression.Property(current, propertyInfo);
        }

        return current;
    }

    private static Expression BuildContainsExpression(Expression property, Expression value)
    {
        if (property.Type != typeof(string))
        {
            throw new ArgumentException("Contains operator is only supported for string properties");
        }

        var containsMethod = typeof(string).GetMethod("Contains", new[] { typeof(string) })!;
        return Expression.Call(property, containsMethod, value);
    }

    private static Expression BuildStringMethodExpression(Expression property, Expression value, string methodName)
    {
        if (property.Type != typeof(string))
        {
            throw new ArgumentException($"{methodName} operator is only supported for string properties");
        }

        var method = typeof(string).GetMethod(methodName, new[] { typeof(string) })!;
        return Expression.Call(property, method, value);
    }

    private static Expression BuildInExpression(Expression property, object? values)
    {
        if (values is not IEnumerable<object> enumerable)
        {
            throw new ArgumentException("In operator requires an enumerable value");
        }

        var valuesList = enumerable.ToList();
        if (valuesList.Count == 0)
        {
            return Expression.Constant(false);
        }

        var comparisons = valuesList.Select(val =>
            Expression.Equal(property, Expression.Constant(val, property.Type)));

        return comparisons.Aggregate(Expression.OrElse);
    }

    private static Expression ReplaceParameter(Expression expression, ParameterExpression oldParameter, ParameterExpression newParameter)
    {
        return new ParameterReplacer(oldParameter, newParameter).Visit(expression)!;
    }

    private class ParameterReplacer : ExpressionVisitor
    {
        private readonly ParameterExpression _oldParameter;
        private readonly ParameterExpression _newParameter;

        public ParameterReplacer(ParameterExpression oldParameter, ParameterExpression newParameter)
        {
            _oldParameter = oldParameter;
            _newParameter = newParameter;
        }

        protected override Expression VisitParameter(ParameterExpression node)
        {
            return node == _oldParameter ? _newParameter : node;
        }
    }
}
```

### Step 5: Fluent Query Builder (45 minutes)

Create `QueryBuilder.Core/Fluent/FluentQueryBuilder.cs`:

```csharp
using System.Linq.Expressions;
using QueryBuilder.Core.Expressions;
using QueryBuilder.Core.Models;

namespace QueryBuilder.Core.Fluent;

public class FluentQueryBuilder<T>
{
    private Expression<Func<T, bool>>? _whereExpression;
    private readonly List<(LambdaExpression KeySelector, SortDirection Direction)> _orderExpressions = new();

    public FluentQueryBuilder<T> Where(Expression<Func<T, bool>> predicate)
    {
        _whereExpression = _whereExpression == null
            ? predicate
            : ExpressionBuilder.CombinePredicates(_whereExpression, predicate, LogicalOperator.And);
        return this;
    }

    public FluentQueryBuilder<T> And(Expression<Func<T, bool>> predicate)
    {
        return Where(predicate);
    }

    public FluentQueryBuilder<T> Or(Expression<Func<T, bool>> predicate)
    {
        _whereExpression = _whereExpression == null
            ? predicate
            : ExpressionBuilder.CombinePredicates(_whereExpression, predicate, LogicalOperator.Or);
        return this;
    }

    public FluentQueryBuilder<T> OrderBy<TKey>(Expression<Func<T, TKey>> keySelector)
    {
        _orderExpressions.Clear();
        _orderExpressions.Add((keySelector, SortDirection.Ascending));
        return this;
    }

    public FluentQueryBuilder<T> OrderByDescending<TKey>(Expression<Func<T, TKey>> keySelector)
    {
        _orderExpressions.Clear();
        _orderExpressions.Add((keySelector, SortDirection.Descending));
        return this;
    }

    public FluentQueryBuilder<T> ThenBy<TKey>(Expression<Func<T, TKey>> keySelector)
    {
        _orderExpressions.Add((keySelector, SortDirection.Ascending));
        return this;
    }

    public FluentQueryBuilder<T> ThenByDescending<TKey>(Expression<Func<T, TKey>> keySelector)
    {
        _orderExpressions.Add((keySelector, SortDirection.Descending));
        return this;
    }

    public QueryBuilder<T, TResult> Select<TResult>(Expression<Func<T, TResult>> selector)
    {
        return new QueryBuilder<T, TResult>(_whereExpression, _orderExpressions, selector);
    }

    public CompiledQuery<T> Compile()
    {
        var stopwatch = System.Diagnostics.Stopwatch.StartNew();

        var filterFunc = _whereExpression?.Compile();
        var sortFunc = BuildSortFunction();

        stopwatch.Stop();

        return new CompiledQuery<T>
        {
            FilterFunc = filterFunc != null
                ? source => source.Where(filterFunc)
                : null,
            SortFunc = sortFunc,
            FilterExpression = _whereExpression,
            CompilationTime = stopwatch.Elapsed
        };
    }

    public IEnumerable<T> Execute(IEnumerable<T> source)
    {
        var compiled = Compile();
        var result = source.AsEnumerable();

        if (compiled.FilterFunc != null)
        {
            result = compiled.FilterFunc(result);
        }

        if (compiled.SortFunc != null)
        {
            result = compiled.SortFunc(result);
        }

        return result;
    }

    private Func<IEnumerable<T>, IOrderedEnumerable<T>>? BuildSortFunction()
    {
        if (_orderExpressions.Count == 0) return null;

        return source =>
        {
            var ordered = ApplyFirstSort(source, _orderExpressions[0]);

            for (int i = 1; i < _orderExpressions.Count; i++)
            {
                ordered = ApplyThenSort(ordered, _orderExpressions[i]);
            }

            return ordered;
        };
    }

    private static IOrderedEnumerable<T> ApplyFirstSort(
        IEnumerable<T> source,
        (LambdaExpression KeySelector, SortDirection Direction) sortExpression)
    {
        var compiled = sortExpression.KeySelector.Compile();
        return sortExpression.Direction == SortDirection.Ascending
            ? source.OrderBy(x => compiled.DynamicInvoke(x))
            : source.OrderByDescending(x => compiled.DynamicInvoke(x));
    }

    private static IOrderedEnumerable<T> ApplyThenSort(
        IOrderedEnumerable<T> source,
        (LambdaExpression KeySelector, SortDirection Direction) sortExpression)
    {
        var compiled = sortExpression.KeySelector.Compile();
        return sortExpression.Direction == SortDirection.Ascending
            ? source.ThenBy(x => compiled.DynamicInvoke(x))
            : source.ThenByDescending(x => compiled.DynamicInvoke(x));
    }
}

public class QueryBuilder<T, TResult>
{
    private readonly Expression<Func<T, bool>>? _whereExpression;
    private readonly List<(LambdaExpression KeySelector, SortDirection Direction)> _orderExpressions;
    private readonly Expression<Func<T, TResult>> _selectExpression;

    internal QueryBuilder(
        Expression<Func<T, bool>>? whereExpression,
        List<(LambdaExpression KeySelector, SortDirection Direction)> orderExpressions,
        Expression<Func<T, TResult>> selectExpression)
    {
        _whereExpression = whereExpression;
        _orderExpressions = orderExpressions;
        _selectExpression = selectExpression;
    }

    public IEnumerable<TResult> Execute(IEnumerable<T> source)
    {
        var query = source.AsQueryable();

        if (_whereExpression != null)
        {
            query = query.Where(_whereExpression);
        }

        // Apply sorting
        IOrderedQueryable<T>? orderedQuery = null;
        for (int i = 0; i < _orderExpressions.Count; i++)
        {
            var (keySelector, direction) = _orderExpressions[i];

            if (i == 0)
            {
                orderedQuery = direction == SortDirection.Ascending
                    ? query.OrderBy((dynamic)keySelector)
                    : query.OrderByDescending((dynamic)keySelector);
            }
            else
            {
                orderedQuery = direction == SortDirection.Ascending
                    ? orderedQuery!.ThenBy((dynamic)keySelector)
                    : orderedQuery!.ThenByDescending((dynamic)keySelector);
            }
        }

        var finalQuery = (IQueryable<T>)(orderedQuery ?? query);
        return finalQuery.Select(_selectExpression);
    }
}
```

## Testing Your Understanding

Create `QueryBuilder.App/Program.cs`:

```csharp
using QueryBuilder.Core.Fluent;
using QueryBuilder.Core.Models;

// Example domain models
public class Player
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public int Level { get; set; }
    public long Experience { get; set; }
    public Guild Guild { get; set; } = new();
    public List<string> Achievements { get; set; } = new();
}

public class Guild
{
    public string Name { get; set; } = string.Empty;
    public int MemberCount { get; set; }
}

static void Main()
{
    var players = new List<Player>
    {
        new() { Id = 1, Name = "Alice", Level = 15, Experience = 2000,
                Guild = new Guild { Name = "DragonSlayers", MemberCount = 50 },
                Achievements = new() { "FirstKill", "Legendary" } },
        new() { Id = 2, Name = "Bob", Level = 8, Experience = 500,
                Guild = new Guild { Name = "Novices", MemberCount = 20 },
                Achievements = new() { "FirstLogin" } },
        new() { Id = 3, Name = "Charlie", Level = 25, Experience = 5000,
                Guild = new Guild { Name = "DragonSlayers", MemberCount = 50 },
                Achievements = new() { "FirstKill", "Legendary", "GrandMaster" } },
    };

    // Fluent query building
    var queryBuilder = new FluentQueryBuilder<Player>();

    Console.WriteLine("=== High Level Players ===");
    var highLevelQuery = queryBuilder
        .Where(p => p.Level > 10)
        .And(p => p.Experience >= 1000)
        .OrderByDescending(p => p.Level)
        .ThenBy(p => p.Name);

    var highLevelPlayers = highLevelQuery.Execute(players);
    foreach (var player in highLevelPlayers)
    {
        Console.WriteLine($"{player.Name} - Level {player.Level}, XP: {player.Experience}");
    }

    Console.WriteLine("\n=== DragonSlayer Members ===");
    var guildQuery = new FluentQueryBuilder<Player>()
        .Where(p => p.Guild.Name == "DragonSlayers")
        .OrderBy(p => p.Name);

    var guildMembers = guildQuery.Execute(players);
    foreach (var player in guildMembers)
    {
        Console.WriteLine($"{player.Name} - {player.Guild.Name}");
    }

    Console.WriteLine("\n=== Players with Projections ===");
    var projectionQuery = new FluentQueryBuilder<Player>()
        .Where(p => p.Level > 5)
        .OrderBy(p => p.Level)
        .Select(p => new
        {
            PlayerName = p.Name,
            PlayerLevel = p.Level,
            GuildName = p.Guild.Name,
            AchievementCount = p.Achievements.Count
        });

    var projectedResults = projectionQuery.Execute(players);
    foreach (var result in projectedResults)
    {
        Console.WriteLine($"{result.PlayerName} (Level {result.PlayerLevel}) " +
                         $"in {result.GuildName} with {result.AchievementCount} achievements");
    }

    // Performance testing
    Console.WriteLine("\n=== Performance Test ===");
    var compiledQuery = queryBuilder.Compile();
    Console.WriteLine($"Query compilation took: {compiledQuery.CompilationTime.TotalMilliseconds}ms");
}
```

## Extension Challenges

1. **Dynamic Query Builder**: Build queries from string conditions
2. **Query Optimization**: Implement expression tree optimizations
3. **SQL Translation**: Convert expressions to SQL queries
4. **Caching System**: Cache compiled expressions for reuse
5. **Expression Visitors**: Custom expression tree transformations
6. **Performance Benchmarks**: Compare compiled vs interpreted execution
7. **Provider Pattern**: Support different data sources (SQL, NoSQL, etc.)

## Resources

- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 📘 **[Expression Trees Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/advanced-topics/expression-trees/)**
- 📘 **[LINQ Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)**

## Contributing

When implementing this project:

1. Design type-safe APIs that leverage expression trees
2. Implement efficient expression compilation and caching
3. Create comprehensive expression visitor patterns
4. Write performance benchmarks for query scenarios
5. Document expression tree patterns and optimizations
