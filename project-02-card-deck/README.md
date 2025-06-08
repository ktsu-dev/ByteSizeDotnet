# Project 2: Card Deck Shuffler

**Focus**: Collections & Enumerables  
**Time**: ~2 hours  
**Deliverable**: Card game foundation library

## 📏 Microsoft Guidelines Applied

This project emphasizes:

- **[Collection Guidelines](../microsoft-design-guidelines.md#collection-guidelines)** - Proper collection usage and interfaces
- **[Enum Design](../microsoft-design-guidelines.md#enum-design)** - Representing card suits and ranks
- **[Interface Design](../microsoft-design-guidelines.md#interface-design)** - Implementing `IEnumerable<T>`
- **[Naming Conventions](../microsoft-design-guidelines.md#naming-guidelines)** - Collection naming and method conventions
- **[Performance Guidelines](../microsoft-design-guidelines.md#performance-guidelines)** - Efficient enumeration patterns
- **[API Design](../microsoft-design-guidelines.md#api-design-guidelines)** - Fluent interface design

## Learning Objectives

By completing this project, you'll understand:

- Master .NET collection types and interfaces
- Understand `IEnumerable<T>`, `IList<T>`, and `ICollection<T>`
- Learn proper enum design and usage
- Practice LINQ fundamentals
- Implement collection interfaces correctly
- Generic collections (`List<T>`, `IEnumerable<T>`, `ICollection<T>`)
- How `foreach` works under the hood
- Difference between `IEnumerable<T>` and concrete collections
- Collection initializers and modern syntax
- When to use arrays vs lists vs other collections
- Implementing `IEnumerable<T>` yourself

## What You'll Build

A card deck library featuring:

- Standard 52-card deck representation
- Multiple shuffling algorithms (Fisher-Yates, Riffle, etc.)
- Card dealing and hand management
- Deck manipulation (cut, bridge, etc.)
- Statistics and shuffle quality metrics

## Core Concepts to Learn

### Collections Hierarchy

```
IEnumerable<T>        // Can iterate (foreach)
└── ICollection<T>    // Can add/remove/count
    └── IList<T>      // Can index and insert
        └── List<T>   // Concrete implementation
```

### Generic Collections

- `List<T>` - Most common, dynamic array
- `IEnumerable<T>` - For iteration and LINQ
- `IReadOnlyList<T>` - Expose lists safely
- `Queue<T>` and `Stack<T>` - Specialized ordering

## Project Setup Instructions

### Step 1: Project Setup (10 minutes)

Navigate to the `project-02-card-deck/src` directory and create the solution:

```bash
# Create the main solution file
dotnet new sln -n CardDeck

# Create the main library project
dotnet new classlib -n CardDeck.Core -f net8.0

# Create the console application
dotnet new console -n CardDeck.App -f net8.0

# Create the test project
dotnet new mstest -n CardDeck.Tests -f net8.0

# Add projects to solution
dotnet sln add CardDeck.Core/CardDeck.Core.csproj
dotnet sln add CardDeck.App/CardDeck.App.csproj
dotnet sln add CardDeck.Tests/CardDeck.Tests.csproj

# Add project references
dotnet add CardDeck.App/CardDeck.App.csproj reference CardDeck.Core/CardDeck.Core.csproj
dotnet add CardDeck.Tests/CardDeck.Tests.csproj reference CardDeck.Core/CardDeck.Core.csproj
```

## Implementation Guide

### Step 2: Core Card Types (25 minutes)

**Card Enums and Structs**

```csharp
namespace CardDeck.Core.Models;

public enum Suit
{
    Hearts,   // ♥
    Diamonds, // ♦
    Clubs,    // ♣
    Spades    // ♠
}

public enum Rank
{
    Two = 2, Three, Four, Five, Six, Seven, Eight,
    Nine, Ten, Jack, Queen, King, Ace
}

public readonly struct Card : IEquatable<Card>
{
    public Suit Suit { get; }
    public Rank Rank { get; }

    public Card(Suit suit, Rank rank)
    {
        Suit = suit;
        Rank = rank;
    }

    public bool Equals(Card other) => Suit == other.Suit && Rank == other.Rank;
    public override bool Equals(object? obj) => obj is Card other && Equals(other);
    public override int GetHashCode() => HashCode.Combine(Suit, Rank);

    public static bool operator ==(Card left, Card right) => left.Equals(right);
    public static bool operator !=(Card left, Card right) => !(left == right);

    public override string ToString() => $"{Rank} of {Suit}";
}
```

### Step 3: Deck as IEnumerable (30 minutes)

**Understanding IEnumerable<T>**

```csharp
namespace CardDeck.Core.Models;

public enum ShuffleAlgorithm
{
    FisherYates,
    Riffle
}

public class Deck : IEnumerable<Card>
{
    private readonly List<Card> _cards;
    private readonly List<Card> _dealtCards;
    private readonly Random _random;

    public int Count => _cards.Count;
    public int DealtCount => _dealtCards.Count;
    public bool IsEmpty => _cards.Count == 0;

    public Deck() : this(new Random()) { }

    public Deck(Random random)
    {
        _random = random ?? throw new ArgumentNullException(nameof(random));
        _cards = new List<Card>();
        _dealtCards = new List<Card>();
    }

    public Deck(IEnumerable<Card> cards) : this(new Random())
    {
        ArgumentNullException.ThrowIfNull(cards);
        _cards.AddRange(cards);
    }

    // Factory methods
    public static Deck CreateStandard52()
    {
        var deck = new Deck();

        foreach (Suit suit in Enum.GetValues<Suit>())
        {
            foreach (Rank rank in Enum.GetValues<Rank>())
            {
                deck._cards.Add(new Card(suit, rank));
            }
        }

        return deck;
    }

    public static Deck CreateBlackjackDeck()
    {
        // Could have different rules or multiple standard decks
        return CreateStandard52();
    }

    // IEnumerable implementation
    public IEnumerator<Card> GetEnumerator() => _cards.GetEnumerator();
    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

**Key Learning: Why implement IEnumerable?**

- Enables `foreach` loops: `foreach (var card in deck)`
- Enables LINQ: `deck.Where(c => c.Suit == Suit.Hearts)`
- Makes your type feel "native" to .NET

### Step 4: Collection Operations (35 minutes)

**Shuffling with Different Algorithms**

```csharp
public class Deck : IEnumerable<Card>
{
    // ... previous code ...

    public void Shuffle(ShuffleAlgorithm algorithm = ShuffleAlgorithm.FisherYates)
    {
        switch (algorithm)
        {
            case ShuffleAlgorithm.FisherYates:
                FisherYatesShuffle();
                break;
            case ShuffleAlgorithm.Riffle:
                RiffleShuffle();
                break;
            default:
                throw new ArgumentOutOfRangeException(nameof(algorithm));
        }
    }

    private void FisherYatesShuffle()
    {
        // Learn: Modern shuffle algorithm
        for (int i = _cards.Count - 1; i > 0; i--)
        {
            int j = _random.Next(i + 1);
            (_cards[i], _cards[j]) = (_cards[j], _cards[i]); // Tuple swap!
        }
    }

    private void RiffleShuffle()
    {
        // Simulate a riffle shuffle by splitting deck and interleaving
        var halfPoint = _cards.Count / 2;
        var leftHalf = _cards.Take(halfPoint).ToList();
        var rightHalf = _cards.Skip(halfPoint).ToList();

        _cards.Clear();

        int leftIndex = 0, rightIndex = 0;
        while (leftIndex < leftHalf.Count || rightIndex < rightHalf.Count)
        {
            // Randomly choose which half to take from
            if (leftIndex >= leftHalf.Count)
            {
                _cards.Add(rightHalf[rightIndex++]);
            }
            else if (rightIndex >= rightHalf.Count)
            {
                _cards.Add(leftHalf[leftIndex++]);
            }
            else if (_random.Next(2) == 0)
            {
                _cards.Add(leftHalf[leftIndex++]);
            }
            else
            {
                _cards.Add(rightHalf[rightIndex++]);
            }
        }
    }

    // Deal methods
    public Card Deal()
    {
        if (IsEmpty) throw new InvalidOperationException("Cannot deal from empty deck");

        var card = _cards[^1]; // Index from end operator
        _cards.RemoveAt(_cards.Count - 1);
        _dealtCards.Add(card);
        return card;
    }

    public IEnumerable<Card> Deal(int count)
    {
        // Learn: yield return for custom iterators
        for (int i = 0; i < count; i++)
        {
            yield return Deal();
        }
    }

    public void Reset()
    {
        // Return all dealt cards to the deck
        _cards.AddRange(_dealtCards);
        _dealtCards.Clear();
    }
}
```

### Step 5: Collection Initialization & LINQ Preview (20 minutes)

**Modern Collection Syntax**

```csharp
namespace CardDeck.Core.Services;

public static class DeckBuilder
{
    // Collection expressions (C# 12+)
    public static Deck CreateCustom(IEnumerable<Card> cards) => new(cards);

    // Target-typed new
    public static Dictionary<Suit, List<Card>> GroupBySuit(Deck deck)
    {
        return new()
        {
            [Suit.Hearts] = deck.Where(c => c.Suit == Suit.Hearts).ToList(),
            [Suit.Diamonds] = deck.Where(c => c.Suit == Suit.Diamonds).ToList(),
            [Suit.Clubs] = deck.Where(c => c.Suit == Suit.Clubs).ToList(),
            [Suit.Spades] = deck.Where(c => c.Suit == Suit.Spades).ToList(),
        };
    }

    // Extension method preview (more in Week 3)
    public static Deck Shuffle(this Deck deck)
    {
        deck.Shuffle();
        return deck; // Fluent interface
    }
}
```

## Key .NET Collection Patterns

### 1. Choosing the Right Collection

```csharp
// ✅ Use List<T> for most scenarios
private readonly List<Card> _cards = new();

// ✅ Expose as IReadOnlyList<T> for safety
public IReadOnlyList<Card> Cards => _cards;

// ✅ Use IEnumerable<T> for parameters (most flexible)
public void AddCards(IEnumerable<Card> cards) => _cards.AddRange(cards);

// ❌ Don't expose List<T> directly
public List<Card> Cards => _cards; // Allows external modification!
```

### 2. Collection Initialization

```csharp
// Modern way
var suits = new List<Suit> { Suit.Hearts, Suit.Diamonds, Suit.Clubs, Suit.Spades };

// Dictionary initialization
var cardValues = new Dictionary<Rank, int>
{
    [Rank.Ace] = 1,
    [Rank.King] = 13,
    [Rank.Queen] = 12,
    [Rank.Jack] = 11
};
```

### 3. Safe Iteration

```csharp
// ✅ Safe: Won't break if collection is modified
public IEnumerable<Card> GetRedCards()
{
    return _cards.Where(c => c.Suit is Suit.Hearts or Suit.Diamonds);
}

// ❌ Dangerous: Direct exposure
public List<Card> GetRedCards()
{
    return _cards.Where(c => c.Suit is Suit.Hearts or Suit.Diamonds).ToList();
}
```

## Implementation Tasks

### Phase 1: Core Models

- [ ] Implement `Card` record with proper equality comparison
- [ ] Create `Deck` class implementing `IEnumerable<Card>`
- [ ] Add standard 52-card deck creation
- [ ] Implement `Deal()` method with yield return

### Phase 2: Shuffling Algorithms

- [ ] Fisher-Yates shuffle implementation
- [ ] Riffle shuffle simulation
- [ ] Custom shuffle algorithms
- [ ] Shuffle quality metrics

### Phase 3: Fluent API

- [ ] Method chaining for deck operations
- [ ] Extension methods for card filtering
- [ ] LINQ integration
- [ ] Performance optimization

### Phase 4: Testing & Demo

- [ ] Unit tests for all shuffle algorithms
- [ ] Statistical testing for randomness
- [ ] Console application demonstration
- [ ] Performance benchmarks

## Testing Your Understanding

Create a console app to test your library:

```csharp
using CardDeck.Core.Models;
using CardDeck.Core.Services;

var deck = Deck.CreateStandard52();
Console.WriteLine($"Created deck with {deck.Count} cards");

// Test enumeration
foreach (var card in deck.Take(5))
{
    Console.WriteLine(card);
}

// Test shuffling and dealing
deck.Shuffle();
var hand = deck.Deal(5).ToList();
Console.WriteLine($"Dealt hand: {string.Join(", ", hand)}");
Console.WriteLine($"Remaining cards: {deck.Count}");

// Test LINQ operations
var redCards = deck.Where(c => c.Suit is Suit.Hearts or Suit.Diamonds);
Console.WriteLine($"Red cards remaining: {redCards.Count()}");

// Test grouping
var groupedBySuit = DeckBuilder.GroupBySuit(deck);
foreach (var (suit, cards) in groupedBySuit)
{
    Console.WriteLine($"{suit}: {cards.Count} cards");
}
```

## Example Usage

```csharp
// Create and shuffle a deck
var deck = new Deck();
var shuffled = deck.Shuffle(ShuffleType.FisherYates);

// Deal cards with LINQ
var hand = shuffled
    .Deal(5)
    .BySuit(Suit.Hearts)
    .OrderBy(card => card.Rank)
    .ToList();

// Analyze shuffle quality
var stats = ShuffleStatistics.Analyze(deck, 1000);
Console.WriteLine($"Randomness score: {stats.RandomnessScore:F2}");
```

## Extension Challenges

1. **Multiple Deck Types**: Support UNO cards, Tarot decks, custom card sets
2. **Hand Evaluation**: Add poker hand ranking (pair, flush, etc.)
3. **Deck Statistics**: Track shuffling patterns, deal history
4. **Custom Iterators**: Implement different dealing patterns (alternating, by suit)
5. **Performance**: Benchmark different shuffling algorithms

## Common Collection Pitfalls

1. **Modifying during iteration**:

   ```csharp
   // ❌ Will throw exception
   foreach (var card in deck)
   {
       if (card.Rank == Rank.Ace)
           deck.RemoveCard(card); // Modifies collection!
   }

   // ✅ Use ToList() to create snapshot
   foreach (var card in deck.ToList())
   {
       if (card.Rank == Rank.Ace)
           deck.RemoveCard(card);
   }
   ```

2. **Multiple enumeration**:

   ```csharp
   var redCards = deck.Where(c => c.Suit is Suit.Hearts or Suit.Diamonds);
   Console.WriteLine(redCards.Count()); // Enumerates once
   Console.WriteLine(redCards.First()); // Enumerates again!

   // ✅ Materialize once
   var redCardsList = deck.Where(c => c.Suit is Suit.Hearts or Suit.Diamonds).ToList();
   ```

3. **Null in collections**:
   ```csharp
   // ✅ Prevent nulls at the boundary
   public void AddCards(IEnumerable<Card> cards)
   {
       ArgumentNullException.ThrowIfNull(cards);
       foreach (var card in cards)
       {
           ArgumentNullException.ThrowIfNull(card);
           _cards.Add(card);
       }
   }
   ```

## Building and Testing

```bash
# Build the solution
dotnet build

# Run all tests
dotnet test

# Run the demo application
dotnet run --project CardDeck.App
```

## Success Criteria

- [ ] All shuffle algorithms implement correct randomization
- [ ] Deck implements IEnumerable<T> properly
- [ ] Extension methods provide fluent card operations
- [ ] Unit tests achieve >90% code coverage
- [ ] Console demo showcases all features
- [ ] Performance meets benchmark requirements

✅ Created a working card deck library  
✅ Implemented `IEnumerable<Card>` correctly  
✅ Used appropriate collection types (`List<T>`, `IReadOnlyList<T>`)  
✅ Demonstrated understanding of `foreach` mechanics  
✅ Implemented multiple shuffling algorithms  
✅ Used modern C# collection syntax  
✅ Applied LINQ operations effectively  
✅ Created safe collection exposure patterns

## Resources

- 📖 **[Microsoft Guidelines](../microsoft-design-guidelines.md)** - Coding standards reference
- 🧠 **[Language Essentials](../language-essentials.md)** - Core .NET concepts
- 📘 **[Collections Documentation](https://learn.microsoft.com/en-us/dotnet/standard/collections/)**
- 📘 **[LINQ Documentation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)**
- [Collections in .NET](https://docs.microsoft.com/en-us/dotnet/standard/collections/)
- [IEnumerable Interface](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1)
- [yield keyword](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/yield)

## Building on Project 1

This project builds on the foundational concepts from the Dice Roller:

- Applies naming conventions learned in Project 1
- Extends type design knowledge to enums and interfaces
- Introduces collection-specific design patterns

## Contributing

When implementing this project:

1. Follow collection interface guidelines
2. Use appropriate enum design patterns
3. Implement proper `IEnumerable<T>` support
4. Write comprehensive unit tests
5. Document collection behavior clearly

This project follows Microsoft's [C# Coding Conventions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions) and [.NET Design Guidelines](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/).

---

**Ready to shuffle some cards? Begin with Step 1!** 🃏
