# Bridge Plugin Specification (draft)

> Status: local owner draft, not committed or published.
> Source: owner voice-discussion notes, 2026-09-28.

## Purpose

Bridge Plugin is a server-side Clear Sky Multiplayer plugin connecting the game server to an external REST backend. It stays small and predictable: reads safe server state, emits events, receives strictly typed commands, executes a small allow-list of actions, and safely messages players.

The product space includes persistent-player identification; linking a game identity to Telegram, Discord, or a player cabinet; private linked-player communication; approximate companion location; short tactical signals; friend presence and invitations; off-process reputation/social graphs; and later a low-pop Director or other server mechanics.

## Owner decisions (2026-09-28)

- **Current live baseline:** `_spawn_v6` is the current spawn plugin. It remains the Bridge
  baseline; its documented armor/bonus behavior must not be described as a random weapon
  system.
- **Future loadout generation:** `random_guns_armor` is a planned separate plugin. Bridge MVP
  must not depend on it; later it may consume only a narrow, verified allowed-variation
  contract that preserves the server's balance rules.
- **Inbound command chat:** this is a long-standing desired feature, but no supported capture
  point is confirmed yet. Proving a safe inbound-message hook is a discovery gate before
  command-driven social functions enter the MVP.
- **Identity:** the operator will define an authentication/linking path for own community
  players, including regulars and Telegram-group members. The plugin does not infer identity
  from a nickname or group membership. Routine gameplay needs no new login. For special
  events only, after a nickname is already linked to a Telegram owner, backend sends that
  owner a bot confirmation request; the player confirms participation in Telegram. This stays
  feature-flagged until a working website and sufficient community activity justify it.
- **Backend boundary:** REST/backend work runs in a separate service, not in the injected
  plugin DLL.
- **First landmark maps:** vanilla Army Warehouses (`Склады`) and the custom Neptune Factory
  (`Завод Нептун`). Landmark maps must be versioned independently for each map.

## 1. Architecture and authority

The plugin reads game state, handles events, transports them to the backend, receives typed actions, performs only pre-authorized effects, and sends player messages. The backend owns accounts, friends, squads, reputation, cooldowns, permissions, Telegram/Discord, privacy, history, and complex game logic. It is the source of truth for social data; the game process stores no long-lived social state.

```text
game callback -> bounded local queue -> worker/bridge -> REST backend
REST response -> validated local action queue -> game callback -> allow-listed effect
```

Backend unavailability is an optional-dependency failure: the core server and Hardmatch continue while social features temporarily stop.

## 2. Player, identity, and trust model

For a session, transmit server player ID, nickname, connection ID, connection time, alive/dead state, safe position, stable outfit state if available, and any other genuinely stable identifiers. The backend separately links an account to Telegram, Discord, web, and possibly several game nicknames. The bridge never decides that two nicknames belong to one person.

The backend may classify a player as unknown, known, regular, trusted, administrator, or tester. The bridge does not interpret that level. Backend policy may use it to limit social features, expose experiments to trusted users, or scope test commands.

## 3. Friends and temporary squads

A friend/companion link is mutually confirmed. Per-link permissions independently grant private messages, online status, join/leave notifications, approximate-location requests, Telegram/Discord calls, and a temporary squad. Every participant can revoke each permission.

A temporary squad ends with session end, manual removal, or expiry. It is a social overlay only: the engine still treats members as enemies, damage and friendly fire remain, and there are no automatic HUD markers or team rules.

## 4. Private channel and commands

The existing command-chat channel is proposed as the Bridge UI. It carries either a private message to a linked companion or a compact Bridge command. Commands have no nested parameters or arbitrary long text. The first set is help, status, find companion, here, danger, help signal, call, and privacy. Signals include nickname and a landmark zone, such as `Merc: danger, central building`; later signals can add retreating, coming to you, clear, and wait.

## 5. Location and presence

The bridge may periodically send safe positions for active players. User-facing results are landmarks, not precise coordinates: central building, north warehouse, east yard, tunnel, roof, railway, base, or industrial zone. Landmark polygons/boxes may be resolved in the bridge or backend per map.

Location must not become a continuous radar. It requires mutual permissions, cooldowns, coarse location, opt-out, an upper bound on tracked companions, and initially one active companion. The backend tracks join, leave, map change, and new session; game or platform notifications are optional and revocable.

## 6. Invitations and reputation

An in-game call creates a backend request. After permission and cooldown checks, a Telegram/Discord bot may send caller, server, current map, and approximate online count.

Reputation stays in the backend and can record regularity, community tenure, sessions, event participation, activity, trust, violations, and tester status. It is for social access, experiments, community roles, external achievements, and at most small cosmetic variation. It must not automatically become combat power.

## 7. Outfit and inventory boundary

Current balance rules always win. Backend may return an allowed outfit class or modifier, but reputation can only select among already allowed variants, never grant a stronger combat outfit merely for a high score.

MVP does not read or alter inventory. Existing verified item spawning may be optional, but no feature requires inventory enumeration, exact counts, deletion, transfer, or full restoration. Quest/event state is virtual and lives in the backend. Physical rewards are rare optional effects because they are difficult to balance, verify, and remove.

Current `_spawn_v6` is the armor/bonus baseline and does not provide the planned random weapon system. A future `random_guns_armor` plugin may define its own verified balance contract; Bridge does not control individual weapon probabilities. Frequent extra weapon grants are risky because they may cause duplicates, inventory/switching errors, or unfair advantage.

## 8. String, command, and REST safety

Hard limits apply to command, nickname, user message, backend reply, and game-chat message lengths. Backend text is validated before entering a game API: terminate correctly, reject unsupported control characters, use one explicit encoding policy, split long text safely, and route through a dedicated safe-send function.

There is no generic `execute command` endpoint. Typed actions are separately validated: send a player message, private message, squad signal, social-state update, state query, or pre-existing safe action. Every request has authentication, authorization, expiry, idempotency, audit record, payload limits, cooldowns, and rate limits.

HTTP/REST work has short timeouts and never blocks a game callback. Position updates and all events are rate limited; position does not stream each frame. The bridge logs lifecycle, backend errors, rejected commands, rate-limit events, conversion errors, sensitive actions, and unknown player IDs without dumping failures into game chat.

## 9. Feature flags and test group

Private chat, location sharing, squad signals, Telegram notifications, Discord notifications, reputation integration, and experimental commands each have their own feature flag. Backend selects trusted testers first. Unknown hooks, new messages, teleport, non-standard item issue, and all memory-structure work are isolated, test-only, and easy to roll back.

## 10. Explicit non-goals and risks

The MVP excludes full inventory inspection/deletion/transfer/quest-item verification; a real engine squad with team identity, friendly-fire exceptions, team HUD, score/spawn rules, and client synchronization; server-only HUD markers; several true factions; dynamic server-side NPCs/mutants/PvE; arbitrary physical quest objects; dependence on undistributed asset mods; binary client mods for ordinary users; and mandatory use of unknown addresses, object layouts, or memory writes.

Asset mods are a later separate path for skins, outfits, maps, map edits, visual objects, and some UI resources. Binary client mods could later enable HUD, damage rules, or deeper network mechanics, but need trusted-event distribution, checksums, manual installation support, and handling of antivirus/trust concerns.

Unknown low-level calls are experimental until separately developed on a test server with feature flags, trusted testers, minimal rollback, and no core-bridge dependency. Even game-message sending is risky until it has sustained runtime evidence.

## 11. MVP and success criterion

MVP provides session identification; connect/disconnect and periodic presence; a private message to one linked player; friend online status; coarse companion location; `here` and `danger` signals; Telegram/Discord calling; and a location-sharing setting.

It succeeds when two permanent players on an ordinary Hardmatch server, with no client mod, can learn that the other is online, privately message, request coarse location, exchange a signal, invite through Telegram/Discord, and disable their own location sharing. FFA and friendly fire remain unchanged; backend loss has no core-server impact; malformed social input does not crash the server; and no social command needs inventory access.

## 12. Later ideas

After stabilization: temporary squads, several companions, low-pop Director, rivalry, duels, anti-camp location hints, active zones, community events, asset-mod integration, expanded outfits, virtual quest state, and external achievements.

True teams, friendly-fire exceptions, HUD markers, deep inventory, dynamic NPCs, and new client mechanics are later stages and must not block the bridge.
