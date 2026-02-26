# AEMS: Asset-Entity-Manifestation-State

**Durable, community-owned game entities — built on Nostr, designed to outlast their creators.**

Conceptual, 2026-01-21

[🏠 Overview](https://github.com/enduring-game-standard) · [🔧 RUNS](https://github.com/enduring-game-standard/runs-spec) · [⚡ WOCS](https://github.com/enduring-game-standard/wocs-protocol) · [🎼 MAPS](https://github.com/enduring-game-standard/maps-notation) · [❓ FAQ](https://github.com/enduring-game-standard/.github/blob/main/profile/FAQ.md)

---

## The Problem

Game entities die with the studios that create them. When a server shuts down, a player's inventory vanishes. When a publisher discontinues a title, the characters, weapons, and currencies that players invested hundreds of hours acquiring become inaccessible. Blockchain-based approaches introduced token-based ownership but brought their own fragilities: discretionary supply, platform-dependent liquidity, and reliance on specific chains that may themselves disappear.

AEMS takes a different approach. It defines game entities as plain, signed Nostr events that persist on open relays without fees, chains, or central authorities. Entities are structured in four layers that separate the universal concept from its game-specific interpretation, player ownership, and mutable state. The result is a minimal protocol for game entities that any client can read, any game can import, and any community can maintain — regardless of whether the original creator is still operating.

---

## Core Premise

A sword is not a product. It is a concept that predates any game containing it. Every RPG that features swords shares an archetype; what differs is the interpretation. The same is true of health potions, playing cards, chess pieces, and every other entity that appears across multiple games.

AEMS treats genre entities as community-defined types rather than author-locked tokens. A "Health Potion" is a universal archetype that anyone can publish and any game can reference. Dark Souls interprets it as an Estus Flask. Final Fantasy VII interprets it as a Potion. Minecraft interprets it as a Splash Potion. Each interpretation is a game-specific Manifestation of the same underlying Entity. The archetype belongs to no one and is available to everyone.

This is possible because AEMS entities are [Nostr](https://nostr.com) events — published to open relays, discoverable by anyone with a relay connection, forkable without permission, and persistent without any single relay's cooperation. Nostr is not an implementation detail. It is the commons that makes AEMS entities findable, composable, and durable. The same properties that make Nostr a credible social protocol — cryptographic identity, relay-distributed persistence, plain-text readability — make it a credible substrate for game entities that must outlast their creators.

---

## A Concrete Example

A community publishes a **Sword** Entity (kind 30050) to Nostr relays: a universal archetype with a name, category, and description. The Entity belongs to no studio. Anyone can discover it by querying relays for kind 30050 events.

Studio A builds an action RPG and publishes a **Manifestation** (kind 30051) of the Sword Entity: a "Flameblade" with 120 base damage, a fire enchantment, and custom visual properties specific to their game. Studio B builds a survival game and publishes a different Manifestation of the same Sword Entity: a "Rustic Blade" with durability mechanics and crafting requirements. Both Manifestations reference the same underlying Entity.

A player in Studio A's game earns a Flameblade by completing a quest. The game publishes an **Asset** event (kind 30052) signed by the player's Nostr keypair, recording ownership of that specific instance.

Studio A closes. Its servers go dark.

The player's Asset event is still on Nostr relays. Studio B's game can read it, recognize the Sword Entity reference, and offer the player an equivalent Rustic Blade, or a community-built client can display the full provenance chain: Entity → Manifestation → Asset. The player's item persists because it was never stored on Studio A's servers. It was published to a commons that no single party controls.

---

## The Four Layers

AEMS defines a four-layer hierarchy. Each layer is a Nostr addressable event (kind 30050–30053), identified by its publisher's public key and a `d`-tag.

```
Entity → Manifestation → Asset → State
  ↓           ↓            ↓        ↓
universal   game-spec    player    mutable
archetype   interpret    instance  condition
```

### Entity (Kind 30050)

A **universal archetype** — an immutable concept that exists independently of any game. Entities are maintained by communities, not studios. Think of an Entity as the genus in a taxonomy: "Health Potion," "Sword," "Playing Card." Any Nostr user can publish an Entity; the protocol imposes no registry and no gatekeeping.

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

Entities use parameterized replacement: for a given `pubkey` and `d`-tag, relays store only the latest version, allowing community evolution without losing identity.

### Manifestation (Kind 30051)

A **game-specific implementation** of an Entity. How a particular game interprets, expresses, and balances the universal archetype. Every Manifestation MUST include an `entity` tag referencing the parent Entity.

A Manifestation is the layer that a [RUNS](https://github.com/enduring-game-standard/runs-spec) Processor reads: it provides the concrete properties that the game engine acts on.

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

Different games create different Manifestations of the same Entity. A "Health Potion" Entity might become a "Flask" in Dark Souls, a "Potion" in Final Fantasy VII, or a "Splash Potion" in Minecraft. Each Manifestation inherits the archetype and adapts it.

### Asset (Kind 30052)

A **player-owned instance** of a Manifestation. When a player acquires an item, they publish an Asset event signed by their own Nostr keypair, claiming ownership of that specific instance. The Asset persists on relays regardless of whether the game's servers continue operating.

Every Asset MUST include a `manifestation` tag referencing the parent Manifestation.

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

**Ownership transfers** are accomplished by the new owner publishing an updated Asset event with the same `d`-tag, referencing the previous owner for provenance:

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

### State (Kind 30053)

**Mutable condition** of a specific Asset. State captures current hit points, remaining charges, enchantments, wear, or any other property that changes during play. State is deliberately separated from identity: the Asset records *what you own*; the State records *what condition it is in*.

Every State event MUST include an `asset` tag referencing the parent Asset.

```json
{
  "kind": 30053,
  "pubkey": "...",
  "tags": [
    ["d", "instance-7f3a9b2c"],
    ["asset", "<asset_event_id>", "instance-7f3a9b2c"],
    ["property", "charges_remaining", "3", "integer"],
    ["property", "condition", "0.75", "float"],
    ["property", "custom_paint", "daughter-art-2026", "string"]
  ],
  "content": {}
}
```

State is updated by publishing new events. The most recent event for a given `d`-tag represents the current condition.

---

## What AEMS Deliberately Excludes

AEMS is deliberately minimal. It defines four event kinds, their structural relationships, and nothing more. This restraint ensures AEMS remains neutral infrastructure, not a platform.

| Excluded | Why | Where It Belongs |
|----------|-----|------------------|
| Databases/indexing | Protocol defines structure, not storage | Relay operators, clients |
| Marketplaces | Protocol enables transfers, not commerce | Third-party platforms, coordination layers |
| Validation logic | Protocol defines events, not rules | Game-specific Manifestations |
| Rendering | Protocol describes entities, not how they look | Game clients, visual engines |
| Namespacing/registries | Protocol is permissionless | Community convention, reputation |
| Rich metadata standards | Protocol is minimal | [AEMS Conventions](https://github.com/enduring-game-standard/aems-conventions), application-layer schemas |
| Composites/hierarchies | Protocol defines atoms, not molecules | Application-layer composition |

Higher-layer conventions — visual properties, composition rules, domain-specific schemas — belong in separate, optional community profiles. The [AEMS Conventions](https://github.com/enduring-game-standard/aems-conventions) companion provides recommended community conventions for interoperable implementations, including universal tag conventions (mandatory `entity` tags, namespacing via `d`-tags, consistent property naming) and domain-specific examples for playing cards, weapons, and enemies.

The boundary: the **standard** defines the four event kinds and their structural relationships. **Conventions** recommend how to use them interoperably. If a detail concerns *what games should agree on* rather than *what the protocol requires*, it belongs in conventions.

---

## Ecosystem Connections

AEMS is one protocol in a four-protocol architecture. Each protocol handles a different structural concern:

### AEMS → RUNS

AEMS defines the *things*. [RUNS](https://github.com/enduring-game-standard/runs-spec) defines the *engine* that processes them. A RUNS Processor reads AEMS Manifestations as input Records and transforms their State. The relationship is data to execution: AEMS provides the entities, RUNS provides the composable pipeline that makes them behave.

### AEMS → WOCS

[WOCS](https://github.com/enduring-game-standard/wocs-protocol) coordinates the *services* that maintain AEMS entities in multiplayer contexts — server hosting, state synchronization, anti-cheat verification. AEMS answers "what are the things?" WOCS answers "who runs the infrastructure that acts on them?"

### AEMS → MAPS

[MAPS Notation](https://github.com/enduring-game-standard/maps-notation) describes the *design grammar* of game mechanics. Where AEMS entities are concrete instances (a sword, a potion), MAPS patterns describe the structural relationships those instances participate in (a resource-acquire pattern, a locked-transition arc). A designer can notate the mechanical role of an Entity in MAPS and instantiate it in AEMS. Because Entities are open and discoverable, designers can study, fork, and extend each other's work across generations — the structural basis for cumulative craft.

### Why Nostr

Nostr is not an interchangeable transport layer. It is the commons infrastructure that gives AEMS its structural properties. Entities published to Nostr relays are discoverable by anyone with a relay connection, forkable by anyone with a signing key, and persistent across relay redundancy without requiring any single operator's cooperation. These properties — permissionless discovery, cryptographic ownership, distributed persistence — are what make AEMS entities durable community infrastructure rather than another proprietary database.

---

## Getting Started

1. **Define an Entity** — Publish a kind 30050 event to any Nostr relay. Use a descriptive `d`-tag as the identifier.
2. **Discover Entities** — Query relays for kind 30050 events to find universal archetypes published by the community.
3. **Create a Manifestation** — Publish a kind 30051 event referencing an Entity for your game, with game-specific properties.
4. **Claim an Asset** — When a player acquires an item, publish a kind 30052 event signed by the player's keypair.
5. **Track State** — Update mutable properties via kind 30053 events referencing the Asset.

For tag conventions, property naming patterns, and domain-specific examples, see [AEMS Conventions](https://github.com/enduring-game-standard/aems-conventions).

---

## Migration from Previous Versions

> **Note**: Kinds 30001–30003 were previously used but conflict with NIP-51 (Lists). Kinds 30050–30053 are the current AEMS allocations.

If migrating from earlier implementations:
- 30001 → 30050 (Entity)
- 30002 → 30051 (Manifestation)
- 30003 → 30052 (Asset)
- 30078 → 30053 (State)

---

## The Larger Vision

AEMS exists because game entities should outlast the studios that create them. When a player earns a sword, that sword should persist the way a physical chess piece persists — not because a server keeps running, but because the entity was published to a commons that anyone can read and no one controls.

This is one piece of a larger structural argument. Patient capital makes open protocols economically rational. Durable substrate ensures the artifacts survive. Cumulative craft transmits the knowledge of how to build them well. AEMS contributes to durable substrate: entities stored as open, readable events on a distributed commons, composable across games, maintainable by communities. For the full argument, see the [Enduring Game Standard overview](https://github.com/enduring-game-standard).

---

**MIT License** — Open for use and implementation.