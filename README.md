# AEMS: Asset-Entity-Manifestation-State

**A Nostr-Native Protocol for Durable, Community-Defined Game Entities — Conceptual, 2026-01-21**

[🏠 Overview](https://github.com/decentralized-game-standard) · [🔧 RUNS](https://github.com/decentralized-game-standard/runs-standard) · [⚡ WOCS](https://github.com/decentralized-game-standard/wocs-standard) · [🎭 MAPS](https://github.com/decentralized-game-standard/ludic-notation-standard) · [❓ FAQ](https://github.com/decentralized-game-standard/.github/blob/main/profile/FAQ.md)

A standard deck of playing cards is fifty-two objects that participate in hundreds of games. The ace of spades in your poker hand is the same ace of spades that anchors a game of spades, bridges a hand of blackjack, or sits in a solitaire column. The card's *identity* — ace, spade suit — is universal. Its *role* changes with every game: high card here, low card there, trump in one ruleset, irrelevant in another. Your physical deck sits in a kitchen drawer. It has a coffee ring on the seven of hearts and a crease on the jack of diamonds from when your kid bent it. That deck is yours. No company granted you permission to keep it. It works in every card game ever invented and every card game that will ever be invented, because the interface — face, suit, value — is an open standard older than any living corporation.

A video game item works differently. You earned a legendary sword over forty hours of play. You upgraded it, enchanted it, named it. But the sword exists as a row in a database you cannot read, on a server you do not control, under terms that let the operator delete it whenever they choose. When the servers shut down, the sword is gone. You never owned it. You rented access to a state in someone else's system.

What if game items worked like playing cards?

AEMS defines a four-layer structure for game entities, published as signed Nostr events on open relays. No server required for persistence. No chain required for ownership. No corporation required for existence.

## Four Layers

The structure maps directly to what you already understand about a deck of cards.

**Entity** is the universal identity. "Health potion" as an archetype. "Sword that deals damage." The way "ace of spades" exists as a concept independent of any particular card game or any physical deck. Entities are published as Nostr events (kind 30050) and persist on relays indefinitely. Anyone can publish an Entity. Community curation determines which Entities gain adoption, the way a new card game gains adoption through play rather than through a licensing authority.

**Manifestation** is a specific game's interpretation of that Entity. The health potion Entity becomes a "Vitality Elixir" in one game (restores 100 HP, stacks to 50) and a "Flask of Crimson Tears" in another (restores 40% of max HP, limited charges). The way the ace of spades is worth 1 point in one game, 11 in another, and serves as the highest trump in a third. Manifestations reference their parent Entity and are published as kind 30051 events.

**Asset** is a player's specific instance of a Manifestation. Your potion, in your inventory, acquired at a specific time. The coffee-stained seven of hearts in your kitchen drawer. You own it because your cryptographic key signed it. No intermediary confirmed or can revoke that ownership. Kind 30052.

**State** is the mutable condition of that specific Asset. Three charges remaining. 75% durability. A custom skin your friend designed. State is updated by publishing new events. The most recent event is the current State. Kind 30053.

```
Entity → Manifestation → Asset → State
  ↓           ↓            ↓        ↓
concept   game-rules    yours    its condition
```

## Cross-Game Composability

This layered structure produces a consequence that has no equivalent in the current model of digital games.

A player owns a sword Asset from Game A. The sword's Manifestation specifies +50 attack and a fire enchantment within Game A's rules. The sword's Entity is the universal archetype "sword that deals damage." Game B recognizes the same Entity and has its own Manifestation of that archetype, with different stats appropriate to Game B's balance. The player imports the sword. Game B reads the Entity, applies its own Manifestation, and the sword exists in both games simultaneously, with game-appropriate stats in each and continuous ownership throughout.

The Asset's State carries forward: upgrades, enchantments, wear. When Game A shuts down, the sword remains. The Entity is on Nostr relays. The Asset is signed to the player's key. The State is a sequence of signed events. Nothing was stored on Game A's servers that was required for the sword to persist.

The same mechanism that lets an ace of spades move between poker, blackjack, and a game invented next year lets an AEMS entity move between digital games that may not exist yet.

## Technical Structure

### Entity (Kind 30050)

```json
{
  "kind": 30050,
  "pubkey": "...",
  "tags": [
    ["d", "health-potion"],
    ["type", "consumable"],
    ["category", "healing"]
  ],
  "content": {
    "name": "Health Potion",
    "description": "Restores vitality when used"
  }
}
```

Parameterized replacement (per pubkey) allows community evolution. Anyone can publish an Entity. Reputation and community curation determine which Entities gain adoption, the way open-source libraries gain adoption through use rather than through gatekeepers.

### Manifestation (Kind 30051)

```json
{
  "kind": 30051,
  "pubkey": "...",
  "tags": [
    ["d", "rpg-adventure:potion"],
    ["entity", "<entity_event_id>", "health-potion"],
    ["property", "restore_amount", "100", "integer"],
    ["property", "stack_limit", "50", "integer"]
  ],
  "content": {
    "game": "RPG Adventure",
    "name": "Vitality Elixir"
  }
}
```

Different games create different Manifestations of the same Entity. The Entity provides interoperability. The Manifestation provides game-specific balance. Neither depends on the other's server to exist.

### Asset (Kind 30052)

```json
{
  "kind": 30052,
  "pubkey": "<owner_pubkey>",
  "tags": [
    ["d", "instance-7f3a9b2c"],
    ["manifestation", "<manifestation_id>", "rpg-adventure:potion"],
    ["acquired", "1706886400"]
  ],
  "content": {}
}
```

Ownership transfers require the new owner to publish an updated Asset event with the same instance ID, referencing the previous owner for provenance:

```json
{
  "kind": 30052,
  "pubkey": "<new_owner_pubkey>",
  "tags": [
    ["d", "instance-7f3a9b2c"],
    ["manifestation", "<manifestation_id>", "rpg-adventure:potion"],
    ["prev_owner", "<old_owner_pubkey>"],
    ["e", "<previous_asset_event_id>"]
  ],
  "content": {}
}
```

An unbroken chain of signed events serves as provenance. No marketplace or escrow required at the protocol level. Those can exist as services above.

### State (Kind 30053)

```json
{
  "kind": 30053,
  "pubkey": "...",
  "tags": [
    ["d", "instance-7f3a9b2c"],
    ["asset", "<asset_event_id>", "instance-7f3a9b2c"],
    ["property", "charges_remaining", "3", "integer"],
    ["property", "condition", "0.75", "float"],
    ["property", "custom_skin", "friend-design-2026", "string"]
  ],
  "content": {}
}
```

The most recent State event for a given instance ID represents the current condition. History is the sequence of prior events, readable by anyone.

## Integration

RUNS runtimes import Entities as Records, apply selected Manifestations to configure Processors, and update State through gameplay. WOCS coordinates ongoing ecosystem work: server hosting, asset creation, provenance audits. MAPS Scores reference Entities as the nouns of interactive grammar.

Entities persist on Nostr relays. Any engine that reads the event format can interpret them. The protocol defines structure. Databases, indexing, marketplaces, validation logic, namespacing, and rich metadata schemas belong to the layers above, built by whoever finds them useful.

## Status

AEMS is a conceptual specification. No production implementations exist. The event kinds (30050-30053) are allocated to avoid conflicts with NIP-51 Lists. Previous implementations using kinds 30001-30003 should migrate to the current allocations.

**MIT License** — Open for use and implementation.