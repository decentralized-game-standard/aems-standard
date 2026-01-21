# AEMS: Asset-Entity-Manifestation-State

**A Nostr-Native Protocol for Durable, Community-Defined Game Entities — Conceptual, 2026-01-14**

🏠 **[Overview](https://github.com/decentralized-game-standard)** · 🔧 **[RUNS](https://github.com/decentralized-game-standard/runs-standard)** · ⚡ **[WOCS](https://github.com/decentralized-game-standard/wocs-standard)** · ❓ **[FAQ](https://github.com/decentralized-game-standard/.github/blob/main/profile/FAQ.md)**

---

Physical game pieces endure because they are independent objects, interpreted by whatever rules players agree upon. A chess rook can move under standard rules today, a variant tomorrow, or serve as a decoration indefinitely. The piece itself carries no embedded logic—it simply exists, owned and transferred through physical custody or social agreement.

Digital equivalents have been designed differently: items, characters, and worlds are database entries controlled by servers. When servers stop, the assets vanish. Attempts to fix this with blockchain-based NFTs or Ordinals introduced new problems—speculative volatility, broken metadata links, chain-specific lock-in—without delivering reliable persistence or meaningful interoperability.

AEMS approaches the problem by defining game entities as structured, signed Nostr events. The protocol separates the universal concept of an object from game-specific rules, tracks mutable state, and optionally records ownership through verifiable transfer chains—all without blockchains, fees, or centralized authorities.

## The Fragility of Current Digital Assets

Players invest thousands of hours and dollars, yet control nothing lasting.

- **Server-Dependent Erasure** — When publishers end support, entire inventories disappear. Ubisoft's *The Crew* (2014) became unplayable in 2024 after server shutdown, with purchased content revoked from libraries. *City of Heroes* (2004–2012) lost all character data officially, surviving only through legally precarious private servers. Similar fates hit *WildStar*, *Tabula Rasa*, *Hellgate: London*, and hundreds of mobile titles—progress and purchases reduced to zero.

- **Licensed, Not Owned** — In-game purchases are typically revocable licenses. Account bans, region restrictions, or policy changes can strip access overnight. Even single-player titles with online components (*Might & Magic X: Legacy*) lose functionality when authentication servers die.

- **Failed Blockchain Promises** — NFTs were marketed as "true ownership" for gaming assets. In practice, most store only metadata URLs that break when hosting fails. Legal scholars note NFTs convey no automatic copyright or property rights—just a token pointing to data elsewhere. Projects like *Axie Infinity* saw asset values collapse 99%+. Ordinals embed data on Bitcoin but inherit high fees, slow confirmation, and speculative pricing detached from utility. Interoperability remains minimal: an NFT minted on one chain rarely functions meaningfully in another ecosystem.

These failures share a root cause: over-reliance on centralized infrastructure or unnecessary consensus mechanisms for data that does not require financial-grade immutability.

## Why Nostr Fits This Problem Perfectly

Nostr is a minimal, open protocol for relaying signed JSON events. Anyone can publish events using a cryptographic keypair. Anyone can run a relay to store and forward them. There is no canonical ledger, no mining, no fees—just resilient data distribution through voluntary replication.

Philosophically, Nostr embodies restraint: it provides only the primitives needed for censorship-resistant communication (keys, signatures, relays) and trusts higher layers to add meaning. This mirrors Bitcoin's minimalism but applied to social and application data.

For game entities:

- **No Immutable Ledger Required** — Financial assets need tamper-proof history to prevent double-spend. Game items do not. Social agreement ("this signed chain says I own the sword") suffices, as with physical trading cards. Blockchain adds complexity, cost, and speculation without benefit.

- **True Permissionless Participation** — Publish an entity definition or transfer ownership instantly, from any client, to any relay. No gas fees, no chain congestion, no approval. Relays can filter spam but cannot prevent publication—clients simply connect elsewhere.

- **Practical Persistence** — Data lives as long as relays mirror it. Communities preserve valued entities by running dedicated relays or archiving events. This has proven effective for social content; the same applies to curated game concepts. If you are the only person that values your AEMS items, you can back up the Nostr events yourself and store them on a hard drive for 80 years like a Pokémon card in your closet. No reliance on external communities needed at all.

- **Censorship Resistance Without Hype** — Unlike marketplace-dependent NFTs, no single entity controls visibility or validity. An entity definition remains queryable across the relay network indefinitely.

Nostr already handles replaceable and parameterized events (NIP-33), enabling versioned, identifier-based data—ideal for evolving entity definitions and unique instances.

## What AEMS Deliberately Excludes

AEMS follows the discipline of minimal protocols. Like TCP/IP excludes content semantics and MIDI excludes audio, AEMS excludes functionality that belongs in higher layers:

| Excluded | Why | Where It Belongs |
|----------|-----|------------------|
| **Databases/indexing** | Protocol defines structure, not storage | Relay operators, game clients |
| **Marketplaces** | Protocol enables transfers, not commerce | Third-party platforms via WOCS |
| **Validation logic** | Protocol defines events, not business rules | Game-specific Manifestations |
| **Namespacing/registries** | Protocol is permissionless, not curated | Community convention, reputation |
| **Rich metadata standards** | Protocol is minimal, not comprehensive | Application-layer schemas |

If AEMS tried to be a complete asset management system, it would become a platform to capture rather than infrastructure to build on. The deliberate gaps are where your applications live.

## Technical Structure: Four Layers as Nostr Events

AEMS defines four interconnected event kinds, all standard signed Nostr events.

| Layer        | Kind     | Purpose                                                                 | Replaceable?          | Key Tags                              |
|--------------|----------|-------------------------------------------------------------------------|-----------------------|---------------------------------------|
| **Entity**   | 30001   | Universal archetype ("a consumable that restores vitality")             | Parameterized (d-tag) | `d` (identifier), `type`, `category` |
| **Manifestation** | 30002 | Game-specific rules and properties                                      | Parameterized         | `d` (game:identifier), `entity` (ref) |
| **State**    | 30078   | Mutable instance data (durability, charges, modifications)              | App-specific (NIP-78) | `d` (instance_id), `manifestation`   |
| **Asset**    | 30003   | Ownership transfer chain                                                | Parameterized         | `d` (instance_id), `prev_owner`      |

### Entity (Kind 30001)
The neutral concept, creatable by anyone.

```json
{
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
```

Parameterized replacement ensures the latest version for "health-potion" wins per pubkey.

### Manifestation (Kind 30002)
How a specific game interprets the Entity.

```json
{
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
```

Games define their own rules without affecting the universal Entity.

### State (Kind 30078)
Runtime instance data, mutable via NIP-78 app-specific storage.

```json
{
  "kind": 30078,
  "pubkey": "...",
  "tags": [
    ["d", "instance-7f3a9b2c"],
    ["manifestation", "<manifestation_id>", "rpg-adventure:potion"],
    ["property", "charges_remaining", "3", "integer"],
    ["property", "enchanted", "fire", "string"]
  ],
  "content": {}
}
```

### Asset (Kind 30003)
Ownership via signed transfer chain—no blockchain needed.

```json
{
  "kind": 30003,
  "pubkey": "<new_owner>",
  "tags": [
    ["d", "instance-7f3a9b2c"],
    ["prev_owner", "<old_pubkey>"],
    ["transfer", "trade"],
    ["e", "<previous_transfer_event_id>"]
  ],
  "content": ""
}
```

Validity proven by traversing signed events backward to genesis.

## Integration with the Ecosystem

- **RUNS Engines** — Import AEMS Entities as Records. Apply Manifestations to select Processors. Update State Fields during simulation.
- **WOCS Coordination** — Commission new Entities or Manifestations via offers settled in sats. Trade instances permissionlessly.

An item earned in one game persists on Nostr. Another RUNS-based engine can query, manifest differently, and continue the story.

## Prototype Path

1. **Post an Entity** — Use any Nostr client to publish a kind 30001 event.
2. **Query Entities** — Filter relays for kind 30001 with desired tags.
3. **Create a Manifestation** — Reference an existing Entity.
4. **Track an Instance** — Publish State and Asset events for a specific object.

## Why This Approach Now

Nostr's relay network is mature and growing. Tools for signing and querying events are abundant. The repeated failures of centralized and blockchain approaches have clarified what works: minimal primitives, voluntary replication, cryptographic verification without consensus overhead.

AEMS invites experimentation. Define an entity today. Interpret it differently tomorrow. Preserve it for decades.

**MIT License** — Open for use, critique, implementation.