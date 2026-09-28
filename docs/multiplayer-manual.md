# Clear Sky Multiplayer: Deathmatch and the Hardmatch Layer

## Status and scope

This is a guide to the documented **Clear Sky 1.5.x XRAY PROJECT** multiplayer baseline.
It is not a live-server status page: the current map, running game mode, loaded plugin
binary, item availability, and anti-cheat compatibility must be verified in the actual
server runtime before being promised to players.

The key distinction is:

```text
vanilla Deathmatch (DM) rules and buy menu
  +
the optional _spawn_v6 respawn plugin
  =
the historical "Hardmatch" setup
```

Hardmatch is not a stock engine game mode. It is a custom, name-aware respawn loadout layer
on top of DM. It does not make nickname groups into friendly teams, add persistent progress,
or add a web account system.

## Vanilla Deathmatch

In the documented vanilla DM reference, a new player starts with a knife and PM. The normal
buy menu supplies weapons, ammunition, armor, grenades, and equipment according to the active
server configuration. DM is still a free-for-all match: a community role or a faction word in
a nickname is not a team alliance.

### Weapons in the retained DM catalogue

| Family | Weapons |
|---|---|
| Pistols | PM, PB, Fort, HPSA, Walther, Colt 1911, Beretta, SIG 220, USP, Desert Eagle |
| Shotguns | BM-16, TOZ-34, Winchester 1300, SPAS-12 |
| Assault / SMG | AK-74U, MP5, AK-74, L85, Abakan, LR-300, Groza, SIG 550, VAL, G36, PKM, FN F2000 |
| Precision | Vintorez, SVU, SVD, Gauss |
| Heavy | RPG-7, RG-6 |
| Throwables | RGD-5, F1, GD-05 smoke, plus launcher and RPG ammunition |

Engine section names are exact and sometimes contain legacy spellings; for example,
`mp_wpn_wincheaster1300` is intentionally spelled that way. Do not "correct" a section name
in a plugin without confirming it in the target gamedata.

### Outfits and utilities

The documented multiplayer buy-menu reference names three MP armor sections:

| Outfit | Section |
|---|---|
| Scientific / SEVA | `mp_scientific_outfit` |
| Military Stalker / Bulat | `mp_military_stalker_outfit` |
| Exoskeleton | `mp_exo_outfit` |

It also contains medkits, anti-rad, scopes, silencers, grenade launchers, binoculars,
a torch, and the grenades above. A section being present in one reference configuration is
not a runtime test for another server build.

## The historical Hardmatch layer

The included advanced example, [`_spawn_v6_sanitized.asm`](../advanced/_spawn_v6_sanitized.asm),
shows the model: after a respawn, it looks up the exact nickname in priority-ordered role
lists, chooses one armor branch, and may spawn bonus consumables. Its role names and player
identities are sanitized in this public repository.

The behavior is per-life, not persistent:

1. A periodic callback observes a respawn.
2. The first matching role wins; an unlisted player receives the default branch.
3. The plugin picks or fixes an outfit and may add the role's consumables.
4. On the next life, the process repeats.

Nickname matching is case-sensitive and old Cyrillic names are byte-encoded. A game nickname
is not proof of Telegram, Discord, or website ownership.

### Historical profile families

| Profile family | Typical historical effect |
|---|---|
| Default | A weighted light/mid/flicker suit distribution; no bonus items. |
| Regular / community | A regular suit distribution and, in some branches, army medkit and bread. |
| Community-chat variants | A regular or pro suit branch plus historical food or scientific-medkit bonuses. Chat membership must be explicitly managed, never inferred. |
| Pro | A more constrained armor distribution; variants can add food or an army medkit. |
| Patron / premium | A special heavy-suit chance or heavy-suit branch with a historical medkit/food bonus. |
| Tester, operator, event, or restriction accounts | Explicit special cases: fixed suit, no loadout, or test-marker items. |

The exact balance is an operator decision. Changing it affects fairness and needs source,
binary, and controlled-runtime evidence before rollout.

### Armor pools in `_spawn_v6`

| Pool | Historical contents | Important caveat |
|---|---|---|
| Light | novice, bandit, stalker, Duty, Clear Sky light, Freedom light, scientific | Several are single-player-style names and must be tested in MP. |
| Mid | Clear Sky heavy, specops, Freedom heavy | Must be verified against the target build. |
| Flicker | MP military and MP exo | Historical trade-offs include NVG and sprint behavior. |
| Heavy | Duty heavy, military/Bulat, Freedom exo, exo | Some entries are explicitly unconfirmed in MP. |

The documented default branch is 55% light, 30% mid, 9% MP military, and 6% MP exo. These
figures describe the preserved example, not a universal balance recommendation.

## What Hardmatch does not currently provide

- No player-selected loot, skin, or live inventory editing.
- No persistent levels, currency, profile, or account linking.
- No confirmed REST API, HTTPS listener, or website/Telegram/Discord connection.
- No confirmed inbound game-chat commands or AI moderation feed.
- No generic remote administration channel or automatic punishment system.

The separate idea of a single plugin that rolls pistols and primary weapons by player profile
is a proposal, not behavior implemented by `_spawn_v6`. Do not confuse a design document or
an archived Guns experiment with an active server rule.

## Before calling a feature live

Verify the exact game build, server configuration, loaded plugin hash, `xrCPU_Pipe.dll` API,
anti-cheat state, item sections, behavior in a controlled match, and a rollback procedure.
Until then, the honest label is **documented baseline**, not **live verified**.

For exact plugin behavior and known limitations, read
[`advanced/_spawn_v6_spec.md`](../advanced/_spawn_v6_spec.md).
