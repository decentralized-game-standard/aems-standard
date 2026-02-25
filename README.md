# AEMS: Asset-Entity-Manifestation-State

**A Nostr-Native Protocol for Durable, Community-Defined Game Entities** — Conceptual, 2026-01-21

[🏠 Overview](https://github.com/enduring-game-standard) · [🔧 RUNS](https://github.com/enduring-game-standard/runs-spec) · [⚡ WOCS](https://github.com/enduring-game-standard/wocs-protocol) · [🎭 MAPS](https://github.com/enduring-game-standard/maps-notation) · [❓ FAQ](https://github.com/enduring-game-standard/.github/blob/main/profile/FAQ.md)

---

## Motivation

Traditional digital game assets are fragile: they live on centralized servers that can shut down, leaving items, cards, or characters inaccessible forever. Blockchain-based approaches (e.g., NFTs) introduce friction—high fees, speculation, environmental concerns, and reliance on specific chains—while still failing to guarantee true interoperability or permanence.

AEMS addresses this with a paradigm shift: define game entities as plain, signed Nostr events that persist indefinitely on open relays, without fees, chains, or central authorities. Universal archetypes become community-owned infrastructure—importable by any client or game—while interpretations remain flexible across titles. This enables durable, permissionless ecosystems where assets outlive individual games.

---

## Philosophy

AEMS is deliberately minimal:

- It defines only the four event kinds, basic structure, and separation of concerns.
- It intentionally excludes: rendering, composites, namespacing/registries, rich metadata schemas, validation logic, databases/indexing, and marketplaces.
- Higher-layer conventions (e.g., visual properties, composition rules, domain-specific schemas) belong in separate, optional community profiles or application-layer implementations.

This restraint ensures AEMS remains neutral infrastructure, not a platform.

### What AEMS Deliberately Excludes

| Excluded | Why | Where It Belongs |
|----------|-----|------------------|
| Databases/indexing | Protocol defines structure, not storage | Relay operators, clients |
| Marketplaces | Protocol enables transfers, not commerce | Third-party platforms and coordination layers |
| Validation logic | Protocol defines events, not rules | Game-specific Manifestations |
| Namespacing/registries | Protocol is permissionless | Community convention, reputation |
| Rich metadata standards | Protocol is minimal | Application-layer schemas |

---

## Technical Structure: Four Layers as Nostr Events

AEMS defines a four-layer hierarchy where each layer builds upon the previous:

```
Entity → Manifestation → Asset → State
  ↓           ↓            ↓        ↓
concept   game-impl    player's   instance's
                       instance   mutable stats
```

**Analogy (Chess Knight):**
- **Entity**: Chess Knight (the universal concept)
- **Manifestation**: Carved wooden Knight (a specific design/implementation)
- **Asset**: *My* carved wooden Knight (I own it; it's in my hand)
- **State**: My Knight has been hand-painted by my daughter (unique condition)

This structure enables full cross-game composability. A player can sell their half-damaged (State) sword (Asset) from FF7 (Manifestation) to another player who imports it into Minecraft. Minecraft can either import it wholesale (if AEMS-compliant) or re-interpret the hierarchical elements into its own umbrella—recognizing that the FF7 sword derives from the universal sword Entity and assigning an analogous Minecraft sword with similar stats.

### Layer Summary

| Layer | Kind | Purpose | Replaceable? | Key Tags |
|-------|------|---------|--------------|----------|
| Entity | 30050 | Universal archetype (concept) | Parameterized (d-tag) | `d` (identifier), optional grouping tags |
| Manifestation | 30051 | Game-specific implementation | Parameterized | `d`, mandatory `entity` (ref) |
| Asset | 30052 | Player's instance of a Manifestation | Parameterized | `d` (instance_id), mandatory `manifestation` (ref) |
| State | 30053 | Mutable stats of a specific Asset | App-specific | `d` (instance_id), mandatory `asset` (ref) |

---

## Entity (Kind 30050)

**Universal archetype.** An immutable, neutral concept that exists independently of any game.

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

Parameterized replacement (per pubkey) allows community evolution.

---

## Manifestation (Kind 30051)

**Game-specific implementation.** How a particular game interprets and expresses an Entity. MUST include an `entity` tag referencing the Entity event ID (and SHOULD include the Entity's `d` value for readability).

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

Different games can create different Manifestations of the same Entity. A "Health Potion" Entity might become a "Flask" in Dark Souls, a "Potion" in FF7, or a "Splash Potion" in Minecraft.

---

## Asset (Kind 30052)

**Player's instance of a Manifestation.** When a player acquires an item, they create an Asset event claiming ownership of a specific instance. Multiple players can own instances of the same Manifestation, each with a unique Asset ID.

MUST include a `manifestation` tag referencing the Manifestation event ID.

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

**Ownership transfers** are accomplished by the new owner publishing an updated Asset event with the same `d` tag (instance ID), referencing the previous owner for provenance:

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

---

## State (Kind 30053)

**Mutable stats of a specific Asset.** State captures the current condition, wear, enchantments, or any other mutable properties of a player's Asset instance.

MUST include an `asset` tag referencing the Asset event ID.

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

State is updated by publishing new State events. The most recent event for a given `d` tag represents the current state.

---

## Integration

- **RUNS**: Import Entities as Records, apply Manifestations for Processors, update State.
- **WOCS**: Coordinate ongoing work—server hosting, asset creation, provenance audits.

Entities persist on Nostr; different engines can interpret them variably.

---

## Getting Started

1. **Define an Entity** — Publish a kind 30050 Entity with any Nostr client.
2. **Query Entities** — Search relays for kind 30050 events to discover universal archetypes.
3. **Create Manifestations** — Publish kind 30051 events referencing existing Entities for your game.
4. **Claim Assets** — When players acquire items, publish kind 30052 events as their owned instances.
5. **Track State** — Update mutable properties via kind 30053 events referencing the Asset.

---

## Migration from Previous Versions

> **Note**: Kinds 30001-30003 were previously used but conflict with NIP-51 (Lists). Kinds 30050-30053 are now the official AEMS allocations.

If migrating from earlier AEMS implementations:
- 30001 → 30050 (Entity)
- 30002 → 30051 (Manifestation)
- 30003 → 30052 (Asset)
- 30078 → 30053 (State)

---

**MIT License** — Open for use and implementation.