# AEMS: A Decentralized Standard for Game Entity Data

**AEMS** (Asset-Entity-Manifestation-State) is a decentralized, genre-agnostic pattern for standardizing game entity data across all types of games. It extends the Entity-Component-System (ECS) model to support decentralized ownership, cross-game interoperability, and game-specific customization while keeping the structure simple and extensible.

## Intent

The AEMS pattern aims to:
- **Standardize** game entity data for **all game genres** (e.g., RPGs, shooters, puzzlers, simulations).
- **Decentralize** entity management, ensuring entities exist independently of any single game or platform.
- **Enable optional ownership** of entities (e.g., items, characters, locations) through decentralized tokens.
- **Support optional interoperability**, allowing games to share or adapt entities without mandating it.
- **Remain simple** with a predictable core and extensible edges for game-specific needs.

## Problems Solved

Game development faces challenges in entity management:
- **Game-Specific Rules**: Each game’s unique mechanics and lore make sharing entities (e.g., items, characters) across titles difficult.
- **Centralized Control**: Traditional ECS patterns are tied to a game’s engine, limiting decentralization and cross-game use.
- **Ownership**: Implementing decentralized ownership (e.g., via tokens) is complex and often game-specific.
- **Interoperability**: Enabling entities to work across games (e.g., a potion moving from one RPG to another) requires a universal standard.

AEMS addresses these by:
- Separating the universal **Entity** (e.g., "Health Potion") from its game-specific **Manifestation** (e.g., FF7’s "Potion").
- Using **Assets** for optional, decentralized ownership, independent of game logic.
- Tracking instance-specific changes with **State**, isolating dynamic data from the core Entity and Manifestation.

## Tech Stack

AEMS is designed with a lightweight, decentralized tech stack to maximize flexibility and avoid bloat:
- **Liquid**: A sidechain secured by energy (Bitcoin’s proof-of-work) rather than speculative tokens. Liquid is used for **Assets**, providing decentralized ownership without the grift of traditional token systems. It ensures entities can be uniquely owned and traded securely.
- **Nostr**: A decentralized protocol for storing and sharing information via event-based notes. Nostr hosts **Entity**, **Manifestation**, and **State** data, offering a lightweight alternative to bloated blockchains. Its simplicity and focus on data (not money) make it ideal for AEMS’s decentralized storage.
- **Game (Client/Server)**: The game itself (whether client-side, server-side, or hybrid) handles all logic and replaces traditional internal databases with queries to Nostr for entity data. This keeps game mechanics flexible and decoupled from the data layer.

While AEMS is tech-agnostic (Assets could use other token systems, and Nostr could be swapped for similar protocols), this stack provides a robust foundation for decentralization without unnecessary complexity.

## Schema

AEMS consists of four core elements: **Asset**, **Entity**, **Manifestation**, and **State**. Each is lightweight, decentralized, and extensible.

### 1. Asset
- **What It Is**: An optional, external token representing ownership of a unique Entity instance (e.g., a specific "Health Potion" owned by a player).
- **Why It Matters**: Ties an Entity instance to a player for decentralized ownership. Games can ignore Assets if ownership isn’t relevant (e.g., for NPCs).
- **Structure**:
  ```json
  {
    "asset_id": "unique_hash",       // Unique identifier (e.g., liquid_abc123)
    "metadata": {
      "entity_id": "entity_hash"     // Links to an Entity’s unique identifier
    }
  }
  ```

### 2. Entity
- **What It Is**: A universal, high-level concept for any game object (e.g., item, character, location, or game itself).
- **Why It Matters**: Serves as the root for all Manifestations, providing a standardized, decentralized anchor for entities.
- **Structure**:
  ```json
  {
    "id": "entity_hash",             // Unique identifier (e.g., Nostr note hash)
    "content": {
      "name": "string",              // Human-readable name (e.g., "Health Potion")
      "type": "string",              // Category (e.g., "item", "character", "location", "game")
      "purpose": "string"            // Optional intent (e.g., "small health regeneration item")
    },
    "links": [                       // References to related entities
      ["game", "game_hash"]          // Optional link to a game Entity’s hash
    ],
    "tags": [                        // Metadata for filtering/categorization
      ["type", "string"],            // Matches content.type (e.g., ["type", "item"])
      ["custom_tag", "value"]        // Game-specific or community tags
    ],
    "created_at": "timestamp"        // Creation time for versioning
  }
  ```

### 3. Manifestation
- **What It Is**: A game-specific realization of an Entity, defining its properties and behavior within a particular game’s context.
- **Why It Matters**: Adapts the universal Entity to a game’s unique rules and lore, acting as a static template.
- **Structure**:
  ```json
  {
    "id": "manifest_hash",           // Unique identifier (e.g., Nostr note hash)
    "content": {
      "game": "string",              // Game identifier (e.g., "FinalFantasy7")
      "description": "string",       // Optional flavor text (e.g., "Restores vitality")
      "schema": {                    // Game-specific properties
        "property_name": "value"     // e.g., "HP_Restore": 50, "Uses": 1
      }
    },
    "links": [
      ["entity_parent", "entity_hash"],  // Links to parent Entity
      ["game", "game_hash"]              // Optional link to a game Entity’s hash
    ],
    "tags": [
      ["category", "string"],        // e.g., ["category", "consumable"]
      ["custom_tag", "value"]        // Game-specific or community tags
    ],
    "created_at": "timestamp"
  }
  ```

### 4. State
- **What It Is**: A mutable snapshot of a specific Entity instance’s current data in a game, tied to a Manifestation and optionally an Asset.
- **Why It Matters**: Tracks dynamic changes (e.g., usage, damage) for an owned or active Entity instance.
- **Structure**:
  ```json
  {
    "id": "state_hash",              // Unique identifier (e.g., Nostr note hash)
    "content": {
      "property_name": "value"       // Matches Manifestation schema (e.g., "Uses": 0)
    },
    "links": [
      ["manifestation", "manifest_hash"],  // Links to parent Manifestation
      ["asset", "asset_id"]                // Optional, ties to owning Asset
    ],
    "tags": [
      ["custom_tag", "value"]        // Optional game-specific tags
    ],
    "created_at": "timestamp"        // Timestamp for versioning/latest state
  }
  ```

## Example: Health Potion Across Games

Let’s use the iconic **Health Potion** to illustrate AEMS in action, showing how it spans games like Final Fantasy 7 and Minecraft.

### 1. Asset (Ownership)
- A player owns a specific Health Potion instance.
  ```json
  {
    "asset_id": "liquid_hp_123",
    "metadata": {
      "entity_id": "entity_hp"
    }
  }
  ```

### 2. Entity (Universal Concept)
- The universal "Health Potion" definition.
  ```json
  {
    "id": "entity_hp",
    "content": {
      "name": "Health Potion",
      "type": "item",
      "purpose": "Intended to be used as a small, one-time-use health regeneration item"
    },
    "links": [],
    "tags": [["type", "item"], ["category", "consumable"]],
    "created_at": 1712689100
  }
  ```

### 3. Entity (Game: Final Fantasy 7)
- The game itself as an Entity, providing context for Manifestations.
  ```json
  {
    "id": "entity_ff7",
    "content": {
      "name": "Final Fantasy 7",
      "type": "game",
      "purpose": "A classic JRPG with a rich narrative and turn-based combat"
    },
    "links": [],
    "tags": [["type", "game"]],
    "created_at": 1712689000
  }
  ```

### 4. Manifestation (Final Fantasy 7)
- The Health Potion in Final Fantasy 7, without a redundant name field.
  ```json
  {
    "id": "manifest_ff7_hp",
    "content": {
      "game": "FinalFantasy7",
      "description": "A soothing elixir that restores a small amount of HP.",
      "schema": {
        "HP_Restore": 50,
        "Uses": 1
      }
    },
    "links": [["entity_parent", "entity_hp"], ["game", "entity_ff7"]],
    "tags": [["category", "consumable"]],
    "created_at": 1712689200
  }
  ```

### 5. Manifestation (Minecraft)
- The Health Potion in Minecraft.
  ```json
  {
    "id": "manifest_mc_hp",
    "content": {
      "game": "Minecraft",
      "description": "A throwable potion that heals nearby players.",
      "schema": {
        "Heal_Amount": 4,
        "Area_Effect": true
      }
    },
    "links": [["entity_parent", "entity_hp"], ["game", "entity_mc"]],
    "tags": [["category", "consumable"]],
    "created_at": 1712689300
  }
  ```

### 6. State (Final Fantasy 7 Instance)
- The current state of a specific Health Potion in Final Fantasy 7.
  ```json
  {
    "id": "state_hp_ff7_001",
    "content": {
      "HP_Restore": 50,
      "Uses": 0  // Potion has been used
    },
    "links": [["manifestation", "manifest_ff7_hp"], ["asset", "liquid_hp_123"]],
    "tags": [],
    "created_at": 1712690000
  }
  ```

### How It Fits Together
- The **Entity** ("Health Potion") is a universal concept, linked via `links.entity_parent` in Manifestations.
- Each game creates a **Manifestation** tailored to its rules, referencing both the Entity and a game Entity (e.g., "FinalFantasy7").
- The **State** tracks changes for a specific instance, tied to an **Asset** for ownership.
- **Links** ensure relationships (e.g., Entity to Manifestation), while **tags** handle categorization (e.g., "consumable").

This example shows how AEMS supports diverse games while keeping the schema DRY and decentralized.

## Summary

AEMS provides a standardized, decentralized framework for game entities across all genres. By splitting entities into **Asset**, **Entity**, **Manifestation**, and **State**, and leveraging Liquid, Nostr, and game logic, it solves challenges around customization, ownership, and interoperability. Its simplicity and flexibility make it a powerful tool for the future of gaming.

## Prototype Path

Start small, build up. Here's how to get hands-on with AEMS:

### Tier 1: Create an Entity (No Code)
Open any Nostr client (e.g., [Primal](https://primal.net), [Damus](https://damus.io)) and post a note with this content:
```json
{ "name": "Iron Sword", "type": "item", "purpose": "A basic melee weapon" }
```
That's it—you've created a decentralized game entity. Anyone can read it, any game can interpret it.

### Tier 2: Read Entities in a Game
Build a simple game or script that fetches entities from a Nostr relay:
- Query for notes with `["type", "item"]` tags
- Display them in your game's inventory
- You're now reading decentralized data

### Tier 3: Add State and Ownership
- Create a **Manifestation** for your game (your interpretation of the entity)
- Track **State** for entity instances (e.g., durability, uses)
- Optionally link to **Assets** on Liquid for true ownership

## Related Standards

AEMS is part of the [Decentralized Game Standard](../../.github/profile/README.md) ecosystem:

| Standard | How It Relates |
|----------|----------------|
| [Forge-Engine](../forge-engine/README.md) | Entities flow through Forge-Engine processors as data |
| [Stream Protocol](../stream-layer/README.md) | Incentivize entity creation, curation, and hosting with sats |

## Contributing

As a FOSS standard, AEMS invites community contributions. Fork, star, or submit a PR to help shape decentralized game development!
