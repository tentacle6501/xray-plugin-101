# REST Bridge Ideas: Game Server, Community, and Player Cabinet

## The product idea

An external bridge could connect a Clear Sky server to a website, player cabinet, Telegram,
Discord, and operator tools. The goal is not "put an HTTP server into a DLL"; it is to create
safe, small, two-way experiences such as event announcements, opt-in player notifications,
or an approved next-spawn preference.

This page is a proposal. The examples in this repository demonstrate player enumeration,
timers, position checks, targeted outgoing messages, and the advanced respawn pattern. They do
**not** demonstrate a native REST server, TLS, inbound game-chat reading, account linking, or
arbitrary remote administration.

## Recommended boundary

Keep the injected 32-bit plugin small and keep network-facing logic out of the game process.

```text
game + narrow plugin -> bounded local event outbox -> bridge service -> REST/WebSocket -> site/bots
game + narrow plugin <- signed local action inbox  <- bridge service <- player cabinet/staff
```

The plugin should only read game-specific state and handle a short allow-list of actions. The
bridge service owns authentication, account links, persistence, rate limits, audit logs, bot
APIs, retries, and public-network exposure. Bind the bridge to loopback or a private network;
do not expose the game process directly to the internet.

## Ideas worth discussing

| Flow | Player value | Safe first interpretation |
|---|---|---|
| Game -> site | Show availability, event status, and aggregate online count. | Cached/read-only status; no public live roster or location data. |
| Game -> Telegram / Discord | The community sees an event reminder or match milestone. | Origin-labelled, rate-limited bot post with a deduplication ID. |
| Community -> game | A scheduled event announcement appears in the game. | A short, allow-listed message that expires and is audited. |
| Player cabinet | A player ranks eligible skins or loot preferences. | The server remains authoritative and applies only allow-listed preferences at a verified respawn boundary. |
| Player-to-player ping | A player asks a bot to notify another linked player. | Deliver on Telegram/Discord; show an in-game notice only with the recipient's opt-in and online state. |
| Event interaction | Supply drop, hunt, quiz, or status-driven mini-event. | Each effect gets its own separately tested plugin contract. |
| Operator console | Schedule messages, view read-only status, approve an event. | Typed, human-reviewed commands; never an arbitrary console-text box. |
| LLM character or helper | Lore hints, playful recaps, quiz prompts, draft replies. | Typed output and human review for public/high-impact messages. |
| Moderation assistant | Triage reports or group-chat messages. | Shadow-mode classification and evidence for a human moderator, never automatic punishment. |

## Hard limits and safety rules

- A nickname is not a platform account. Use an expiring one-time linking code and explicit,
  revocable consent for every cross-platform feature.
- Telegram bots normally cannot DM a person who has not started the bot. Discord delivery
  also depends on the recipient's settings. Report `delivered`, `not linked`, `opted out`, or
  `unavailable`; never claim delivery that did not happen.
- Do not treat a chat relay as an inbound game-chat API. The included API table does not prove
  that incoming game chat is observable.
- All requests need authentication, authorization, payload and rate limits, expiry,
  idempotency IDs, audit logging, and an explicit server/process epoch.
- The plugin rejects unknown action types and must never interpret arbitrary shell, Lua,
  Radmin, or memory-write text from the bridge.
- Cosmetic/loadout choices require a published allow-list and server-side fairness rules.
- An LLM may suggest or draft; it must not ban, kick, reveal player data, impersonate players,
  or initiate uncontrolled messages in the game.

## A responsible discovery sequence

1. Define one harmless outbound event, such as a server heartbeat or aggregate player count.
2. Select a bounded **local** transport and write fixtures for duplicate events, server
   restart, unavailable bridge, malformed data, and backpressure.
3. Verify one exact plugin build emits that event in a controlled Windows runtime, including
   DLL/API identity and rollback evidence.
4. Add a read-only bridge endpoint and site/bot display.
5. Add one low-impact inbound action, such as a short scheduled announcement, with full
   authorization, expiry, audit, and disable tests.
6. Consider preferences, private notices, event mechanics, and staff actions only as separate
   contracts after the earlier steps succeed.

## Questions for a voice discussion

1. What first moment would feel most alive: event status on the site, a game-originated group
   announcement, or an eligible next-spawn preference?
2. What minimum data may leave the game process?
3. Which choices are cosmetic/preference-only, and which would hurt fairness if automated?
4. Should every external action require a human approval, or can short announcements become a
   safe exception after testing?
5. What personality should an LLM bot have, and what speech, targets, or escalation actions
   are forbidden?
