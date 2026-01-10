# AEMS: Asset-Entity-Manifestation-State

**A Standard for Decentralized Game Entities — Version 0.2 (Conceptual) — January 2026**

AEMS defines how game entities—items, characters, worlds—can exist independently of any single game. It's the digital analogue of physical game tokens.

*Status: Conceptual. This document defines principles and schemas. Prototype implementations welcome.*

---

## Philosophy: Games Don't Own Tokens

A chess piece doesn't contain movement rules. The rules of chess interpret the piece. You can use the same piece in a variant, a different game, or as a paperweight. The piece exists independently.

A Pokémon card in your drawer doesn't vanish when the company folds. You can still battle your grandson in 80 years with house rules.

Digital games trap entities inside servers. When the server dies, so does your sword, your character, your world. AEMS breaks that trap.

**AEMS standardizes the token. Games interpret it.**

---

## The Four Layers

| Layer | What It Is | Analogy |
|-------|-----------|---------|
| **Entity** | Universal concept | "A chess Rook" |
| **Manifestation** | Game-specific interpretation | "Rook in Fischer Random" |
| **State** | Instance current values | "This Rook at A1" |
| **Asset** | Ownership record | "This piece is mine" |

### Entity

A universal, high-level concept. What something *is*.

```json
{
  "kind": 30001,
  "pubkey": "<creator>",
  "content": {
    "name": "Health Potion",
    "type": "item",
    "purpose": "Restores health when consumed"
  },
  "tags": [
    ["d", "health-potion"],
    ["type", "item"],
    ["category", "consumable"]
  ]
}
```

Entities are created by anyone—individuals, communities, or game developers. They define concepts that games *can* use, not concepts games *must* use.

### Manifestation

A game-specific interpretation of an Entity. What something *does here*.

```json
{
  "kind": 30002,
  "pubkey": "<game_developer>",
  "content": {
    "game": "FinalFantasy7",
    "name": "Potion",
    "description": "A soothing elixir that restores 50 HP."
  },
  "tags": [
    ["d", "ff7:potion"],
    ["entity", "<entity_id>", "health-potion"],
    ["game", "<game_id>"],
    ["property", "hp_restore", "50", "number"],
    ["property", "max_stack", "99", "number"],
    ["property", "uses", "1", "number"]
  ]
}
```

Each game creates its own Manifestation. FF7's "Potion" restores 50 HP. Dark Souls' "Estus Flask" restores 50% HP and has 3 uses. Same Entity, different interpretations.

**Games own their Manifestations.** They define the rules. AEMS just provides the structure.

### State

Current values for a specific instance. What's true *right now*.

```json
{
  "kind": 30078,
  "pubkey": "<player>",
  "content": {},
  "tags": [
    ["d", "<instance_id>"],
    ["manifestation", "<manifestation_id>", "ff7:potion"],
    ["property", "uses_remaining", "0", "number"],
    ["property", "acquired_at", "1712690000", "timestamp"]
  ]
}
```

State is mutable. It tracks what's happened to this specific instance—uses, damage, modifications.

### Asset (Ownership)

Chain of custody tracked via Nostr events. Who owns this instance.

```json
{
  "kind": 30003,
  "pubkey": "<current_owner>",
  "content": {},
  "tags": [
    ["d", "<instance_id>"],
    ["entity", "<entity_id>"],
    ["prev_owner", "<previous_owner_pubkey>"],
    ["transfer", "gift"],
    ["signature", "<prev_owner_signature>"]
  ]
}
```

Ownership is established by the chain of signed transfer events. No external blockchain required—just Nostr.

---

## What AEMS Does (and Doesn't Do)

| AEMS Does | AEMS Does NOT |
|-----------|---------------|
| Standardize entity structure | Define game logic |
| Enable entities to exist independently | Force interoperability |
| Allow optional ownership tracking | Mandate ownership |
| Provide structure games CAN use | Force games to use it |

**Composability is possible, not required.** Games choose whether to interpret AEMS entities, and how. A game can:
- Ignore AEMS entirely
- Use only Entities it recognizes
- Create custom Manifestations for shared Entities
- Invent proprietary Entities for internal use

---

## Tech Stack

**Nostr** — AEMS lives here. Entities, Manifestations, States, and ownership are all Nostr events.

**Lightning** (optional) — Games can enable buying, selling, or trading entities via sats. Handled through [WOSS](../woss/README.md), not AEMS itself.

---

## NOSTR Implementation

AEMS uses parameterized replaceable events (NIP-33) with these kinds:

| AEMS Element | Nostr Kind | Why |
|--------------|------------|-----|
| Entity | `30001` | Addressable, versioned |
| Manifestation | `30002` | Game can update interpretation |
| Asset (Ownership) | `30003` | Chain of custody |
| State | `30078` (NIP-78) | App-specific instance data |

Standard tags:
- `["d", "<identifier>"]` — Addressable event identifier
- `["entity", "<event_id>", "<d_tag>"]` — Reference to Entity
- `["game", "<event_id>"]` — Reference to game Entity
- `["property", "<name>", "<value>", "<type>"]` — Typed property
- `["type", "<category>"]` — Entity type for filtering

---

## Example: Health Potion Across Games

### 1. Entity (Universal)

Someone creates the universal concept of a health potion:

```json
{
  "kind": 30001,
  "pubkey": "community_curator",
  "content": {
    "name": "Health Potion",
    "type": "item",
    "purpose": "Restores health when consumed"
  },
  "tags": [["d", "health-potion"], ["type", "item"]]
}
```

### 2. Manifestation (Final Fantasy 7)

FF7's developers interpret it for their game:

```json
{
  "kind": 30002,
  "pubkey": "ff7_dev",
  "content": {
    "game": "FinalFantasy7",
    "name": "Potion",
    "description": "A soothing elixir that restores 50 HP."
  },
  "tags": [
    ["d", "ff7:potion"],
    ["entity", "<entity_id>", "health-potion"],
    ["property", "hp_restore", "50", "number"]
  ]
}
```

### 3. Manifestation (Minecraft)

Minecraft interprets it differently:

```json
{
  "kind": 30002,
  "pubkey": "mc_dev",
  "content": {
    "game": "Minecraft",
    "name": "Healing Potion",
    "description": "Splash potion that heals nearby entities."
  },
  "tags": [
    ["d", "mc:healing-potion"],
    ["entity", "<entity_id>", "health-potion"],
    ["property", "heal_hearts", "4", "number"],
    ["property", "splash", "true", "boolean"]
  ]
}
```

### 4. State (Player's Instance)

A player owns a specific potion in FF7:

```json
{
  "kind": 30078,
  "pubkey": "player123",
  "content": {},
  "tags": [
    ["d", "potion_instance_001"],
    ["manifestation", "<ff7_potion_id>", "ff7:potion"],
    ["property", "uses_remaining", "1", "number"]
  ]
}
```

### How It Fits

- The **Entity** is universal—any game can reference it
- Each game's **Manifestation** defines local rules
- The **State** tracks this specific instance
- **Ownership** travels with the player, not the game

---

## Prototype Path

| Tier | Action | You Learn |
|------|--------|-----------|
| 1 | Post an Entity to Nostr | Decentralized data exists |
| 2 | Read Entities in a game | Games can query shared data |
| 3 | Create a Manifestation | Games interpret entities locally |
| 4 | Track State and Ownership | Instance-level data management |

### Tier 1: Create an Entity (No Code)

Open any Nostr client (e.g., [Primal](https://primal.net)) and post:

```json
{ "name": "Iron Sword", "type": "item", "purpose": "A basic melee weapon" }
```

That's a decentralized entity. Anyone can read it. Any game can interpret it.

### Tier 2: Read Entities in a Game

Query Nostr for `kind:30001` events with `["type", "item"]`. Display them in your inventory. You're reading decentralized data.

### Tier 3: Add Your Manifestation

Create a `kind:30002` event that references the Entity and defines your game's interpretation. Now you control how it works in your game.

### Tier 4: Track Instances

Use `kind:30078` for State and `kind:30003` for ownership transfers. Players own their instances across game sessions.

---

## Related Standards

AEMS is part of the [Decentralized Game Standard](../../.github/profile/README.md) ecosystem:

| Standard | How It Relates |
|----------|----------------|
| [GERS](../gers/README.md) | AEMS entities become GERS Records processed by game engines |
| [WOSS](../woss/README.md) | Incentivize entity creation, curation, and trading with sats |

---

## Contributing

AEMS is FOSS. Shape it:
- **Discuss**: Open an issue with questions or ideas
- **Prototype**: Build something that uses AEMS
- **Curate**: Create useful Entity definitions

---

## License

[MIT License](LICENSE) — Use it, extend it, share it.
