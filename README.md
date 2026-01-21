AEMS: Asset-Entity-Manifestation-State
A Nostr-Native Protocol for Durable, Community-Defined Game Entities — Conceptual, 2026-01-21
🏠 Overview · 🔧 RUNS · ⚡ WOCS · ❓ FAQ

Motivation
Traditional digital game assets are fragile: they live on centralized servers that can shut down, leaving items, cards, or characters inaccessible forever. Blockchain-based approaches (e.g., NFTs) introduce friction—high fees, speculation, environmental concerns, and reliance on specific chains—while still failing to guarantee true interoperability or permanence.
AEMS addresses this with a paradigm shift: define game entities as plain, signed Nostr events that persist indefinitely on open relays, without fees, chains, or central authorities. Universal archetypes become community-owned infrastructure—importable by any client or game—while interpretations remain flexible across titles. This enables durable, permissionless ecosystems where assets outlive individual games.
Philosophy
AEMS is deliberately minimal:

It defines only the four event kinds, basic structure, and separation of concerns.
It intentionally excludes: rendering, composites, namespacing/registries, rich metadata schemas, validation logic, databases/indexing, and marketplaces.
Higher-layer conventions (e.g., visual properties, composition rules, domain-specific schemas) belong in separate, optional community profiles or application-layer implementations.

This restraint ensures AEMS remains neutral infrastructure, not a platform.
What AEMS Deliberately Excludes



































ExcludedWhyWhere It BelongsDatabases/indexingProtocol defines structure, not storageRelay operators, clientsMarketplacesProtocol enables transfers, not commerceThird-party platforms (e.g., WOCS)Validation logicProtocol defines events, not rulesGame-specific ManifestationsNamespacing/registriesProtocol is permissionlessCommunity convention, reputationRich metadata standardsProtocol is minimalApplication-layer schemas
Technical Structure: Four Layers as Nostr Events








































LayerKindPurposeReplaceable?Key TagsEntity30001Universal archetypeParameterized (d-tag)d (identifier), optional grouping tagsManifestation30002Game-specific rules and propertiesParameterizedd, mandatory entity (ref)State30078Mutable instance dataApp-specificd (instance_id), manifestationAsset30003Optional ownership transfer chainParameterizedd (instance_id), prev_owner
Entity (Kind 30001)
Immutable, neutral concept.
JSON{
  "kind": 30001,
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
Parameterized replacement (per pubkey) allows community evolution.
Manifestation (Kind 30002)
Game-specific interpretation. MUST include an entity tag referencing the Entity event ID (and SHOULD include the Entity's d value for readability).
JSON{
  "kind": 30002,
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
State (Kind 30078)
Mutable instance data.
JSON{
  "kind": 30078,
  "pubkey": "...",
  "tags": [
    ["d", "instance-7f3a9b2c"],
    ["manifestation", "<manifestation_id>", "rpg-adventure:potion"],
    ["property", "charges_remaining", "3", "integer"]
  ],
  "content": {}
}
Asset (Kind 30003)
Optional ownership chain.
JSON{
  "kind": 30003,
  "pubkey": "<new_owner>",
  "tags": [
    ["d", "instance-7f3a9b2c"],
    ["prev_owner", "<old_pubkey>"],
    ["e", "<previous_transfer_event_id>"]
  ],
  "content": ""
}
Integration

RUNS: Import Entities as Records, apply Manifestations for Processors, update State.
WOCS: Coordinate commissions or trades.

Entities persist on Nostr; different engines can interpret them variably.
Getting Started

Publish a kind 30001 Entity with any Nostr client.
Query relays for kind 30001 events.
Create Manifestations referencing existing Entities.
Track instances with State and optional Asset events.

MIT License — Open for use and implementation.