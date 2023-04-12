using System.Collections.Generic;
using System.Linq;
public static class Poker {
    public static List<string> BestHands(string[] hands) {
        List<Hand> highHands = new List<Hand> {
            new Hand(hands[0])
        };
        foreach (var cards in hands.Skip(1)) {
            Hand newHand = new Hand(cards);
            Hand highHand = highHands.First();
            if (newHand.Rank >= highHand.Rank) {
                if (newHand.Rank > highHand.Rank) {
                    highHands.Clear();
                    highHands.Add(newHand);
                }
                else if (newHand.Rank == highHand.Rank) {
                    if (newHand.Equals(highHand)) {
                        highHands.Add(newHand);
                    }
                    else if (newHand.OutRanks(highHand)) {
                        highHands.Clear();
                        highHands.Add(newHand);
                    }
                }
            }
        }
        return highHands.Select(h => h.ToString()).ToList();
    }
}
public static class Extensions {
    public static bool HasDuplicates(this List<Card> cards, int groupSize, int groupCount) {
        return cards.GroupBy(c => c.Rank).Count(c => c.Count() == groupSize) == groupCount;
    }
    public static bool IsStraight(this List<Card> hand) {
        List<int> straightValues = hand.SetAceValue().Select(c => c.Rank).ToList();
        return straightValues.SequenceEqual(Enumerable.Range(straightValues.Min(), hand.Count));
    }
    public static bool IsFlush(this List<Card> hand) {
        return hand.Select(c => c.Suit).Distinct().Count() == 1;
    }
    public static List<Card> SetAceValue(this IEnumerable<Card> cards) {
        List<Card> hand = cards.ToList();
        int aceValue = hand.Select(c => c.Rank).Contains(14) && hand.Select(c => c.Rank).Contains(2) ? 1 : 14;
        return hand.Select(c => c.Rank == 14 ? new Card(c.Suit, aceValue) : c).OrderBy(c => c.Rank).ToList();
    }
}
public class CardRanking {
    public int Rank { get; set; }
    public int Count { get; set; }
}
public class Hand {
    private Ranks _rank;
    private readonly Card[] _cards = new Card[5];
    private readonly string _cardsInString;
    private readonly List<CardRanking> _cardRanking;
    public enum Ranks {
        HighCard = 1,
        OnePair = 2,
        TwoPair = 3,
        ThreeOfAKind = 4,
        Straight = 5,
        Flush = 6,
        FullHouse = 7,
        FourOfAKind = 8,
        StraightFlush = 9
    }
    public Ranks Rank {
        get {
            List<Card> hand = _cards.ToList();
            if (hand.IsStraight() && hand.IsFlush()) {
                _rank = Ranks.StraightFlush;
                return _rank;
            }
            if (hand.HasDuplicates(4, 1)) {
                _rank = Ranks.FourOfAKind;
                return _rank;
            }
            if (hand.HasDuplicates(3, 1) && hand.HasDuplicates(2, 1)) {
                _rank = Ranks.FullHouse;
                return _rank;
            }
            if (hand.IsFlush()) {
                _rank = Ranks.Flush;
                return _rank;
            }
            if (hand.IsStraight()) {
                _rank = Ranks.Straight;
                return _rank;
            }
            if (hand.HasDuplicates(3, 1)) {
                _rank = Ranks.ThreeOfAKind;
                return _rank;
            }
            if (hand.HasDuplicates(2, 2)) {
                _rank = Ranks.TwoPair;
                return _rank;
            }
            if (hand.HasDuplicates(2, 1)) {
                _rank = Ranks.OnePair;
                return _rank;
            }
            _rank = Ranks.HighCard;
            return _rank;
        }
    }
    public Card[] Cards {
        get {
            return _cards.OrderByDescending(c => c.Rank).ToArray();
        }
    }
    public List<CardRanking> CardRanking {
        get {
            return _cardRanking;
        }
    }
    public Hand(string stringOfCards) {
        _cardsInString = stringOfCards;
        List<string> cards = stringOfCards.Split(" ").OrderBy(c => c).ToList();
        for (int i = 0; i < cards.Count; i++) {
            string card = cards[i];
            _cards[i] = new Card(card);
        }
        _cards = _cards.ToList().SetAceValue().ToArray();
        _cardRanking = _cards.GroupBy(c => c.Rank).Select(group => new { Rank = group.Key, Count = group.Count() }).OrderByDescending(x => x.Count).ThenByDescending(x => x.Rank).Select(c => new CardRanking { Rank = c.Rank, Count = c.Count }).ToList();
    }
    public bool OutRanks(Hand otherHand) {
        CardRanking[] thisRanking = _cardRanking.ToArray();
        CardRanking[] otherRanking = otherHand.CardRanking.ToArray();
        bool outRanks = true;
        for (int i = 0; i < thisRanking.Length; i++) {
            if (thisRanking[i].Rank > otherRanking[i].Rank) {
                break;
            }
            else if (thisRanking[i].Rank < otherRanking[i].Rank) {
                outRanks = false;
                break;
            }
        }
        return outRanks;
    }
    public bool Equals(Hand otherHand) {
        List<Card> currentHand = _cards.OrderByDescending(c => c.Rank).ToList();
        List<Card> secondHand = otherHand.Cards.OrderByDescending(c => c.Rank).ToList();
        bool isEqual = true;
        for (int i = 0; i < currentHand.Count(); i++) {
            if (currentHand[i].Rank != secondHand[i].Rank) {
                isEqual = false;
                break;
            }
        }
        return isEqual;
    }
    public override string ToString() {
        return _cardsInString;
    }
}
public class Card {
    private readonly int _rank;
    private readonly Suits _suit;
    private readonly string _stringValue;
    public enum Suits {
        Clubs = 1,
        Diamonds = 2,
        Hearts = 3,
        Spades = 4
    }
    private static readonly Dictionary<string, int> _ranks = new Dictionary<string, int>() {
        { "1", 1 },
        { "2", 2 },
        { "3", 3 },
        { "4", 4 },
        { "5", 5 },
        { "6", 6 },
        { "7", 7 },
        { "8", 8 },
        { "9", 9 },
        { "10", 10 },
        { "J", 11 },
        { "Q", 12 },
        { "K", 13 },
        { "A", 14 }
    };
    private static readonly Dictionary<string, Suits> _suits = new Dictionary<string, Suits>() {
        { "S", Suits.Spades },
        { "C", Suits.Clubs },
        { "H", Suits.Hearts },
        { "D", Suits.Diamonds }
    };
    public int Rank {
        get {
            return _rank;
        }
    }
    public Suits Suit {
        get {
            return _suit;
        }
    }
    public Card(string card) {
        _stringValue = card;
        if (card.Length == 3) {
            _rank = _ranks[card.Substring(0, 2)];
            _suit = _suits[card.Substring(2, 1)];
        }
        else {
            _rank = _ranks[card.Substring(0, 1)];
            _suit = _suits[card.Substring(1, 1)];
        }
    }
    public Card(Suits suit, int rank) {
        _rank = rank;
        _suit = suit;
    }
    public override string ToString() {
        return _stringValue;
    }
}
