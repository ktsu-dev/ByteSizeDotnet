# Project 2: Card Deck Shuffler

**Focus**: Collections & Enumerables  
**Time**: ~2 hours  
**Deliverable**: Card game foundation library

## Learning Objectives

By completing this project, you'll understand:

- Generic collections (`List<T>`, `IEnumerable<T>`, `ICollection<T>`)
- How `foreach` works under the hood
- Difference between `IEnumerable<T>` and concrete collections
- Collection initializers and modern syntax
- When to use arrays vs lists vs other collections
- Implementing `IEnumerable<T>` yourself

## Project Overview

Build a comprehensive card deck library that any card game can use. You'll learn .NET's collection system by modeling a real-world domain that's inherently collection-heavy.

## What You'll Build

A library that can:

- Create standard 52-card decks with suits and ranks
- Shuffle decks using different algorithms
- Deal cards from the deck
- Support different deck types (Poker, Blackjack, UNO, etc.)
- Track dealt vs remaining cards
- Restore dealt cards back to deck

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

## Implementation Guide

### Step 1: Project Setup (10 minutes)

```bash
# Create new class library project
dotnet new classlib -n CardDeck
cd CardDeck
```

### Step 2: Core Card Types (25 minutes)

**Card Enums and Structs**

```csharp
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

    // IEquatable, ToString, GetHashCode
    public override string ToString() => $"{Rank} of {Suit}";
}
```

### Step 3: Deck as IEnumerable (30 minutes)

**Understanding IEnumerable<T>**

```csharp
public class Deck : IEnumerable<Card>
{
    private readonly List<Card> _cards;
    private readonly List<Card> _dealtCards;

    public int Count => _cards.Count;
    public int DealtCount => _dealtCards.Count;
    public bool IsEmpty => _cards.Count == 0;

    // Factory methods
    public static Deck CreateStandard52() => /* ... */;
    public static Deck CreateBlackjackDeck() => /* ... */;

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
    private readonly Random _random;

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
}
```

### Step 5: Collection Initialization & LINQ Preview (20 minutes)

**Modern Collection Syntax**

```csharp
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

## Testing Your Understanding

Create a console app to test your library:

```csharp
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

## Success Criteria

✅ Created a working card deck library  
✅ Implemented `IEnumerable<Card>` correctly  
✅ Used appropriate collection types (`List<T>`, `IReadOnlyList<T>`)  
✅ Demonstrated understanding of `foreach` mechanics  
✅ Implemented multiple shuffling algorithms  
✅ Used modern C# collection syntax  
✅ Applied LINQ operations effectively  
✅ Created safe collection exposure patterns

## Next Project Preview

Project 3 will teach extension methods and fluent APIs by building string processing utilities. You'll learn how to "extend" existing types like `string` with your own methods, making APIs that feel natural and chainable.

## Further Reading

- [Collections in .NET](https://docs.microsoft.com/en-us/dotnet/standard/collections/)
- [IEnumerable Interface](https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1)
- [yield keyword](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/yield)

---

**Ready to shuffle some cards? Begin with Step 1!** 🃏
