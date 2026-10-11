# OmniTools

**An all-in-one server toolkit for Minecraft 1.21.11 (Fabric): SMP essentials, ranked PvP, minigames, custom ranks, a universal scoreboard, kits, chest menus, custom commands, economy and a player market.**

OmniTools is built around one idea: server owners should be able to create their own menus, commands, ranks, boards and arenas
in-game, without writing code, while players only ever see simple, safe commands.

Everything an operator does lives under one command, `/omni`. Everything a player does goes through short **aliases**
such as `/home`, `/tpa`, `/pvp`, `/market` or `/board`. Players never get access to `/omni` itself.

- Version: 2.1.0 (build 5)
- Minecraft: 1.21.11
- Loader: Fabric Loader 0.16.0 or newer
- Java: 21 or newer
- Requires: Fabric API
- License: MIT
- Author: @sonruhin

> Upgrading from 1.1.0? Read and [Update New Version](https://github.com/sonruhin/omni-tools/releases#release-Omni-Tools-Beta) first. Stored commands are migrated automatically.

---

## Table of contents

1. [Feature overview](#feature-overview)
2. [Requirements and installation](#requirements-and-installation)
3. [The /omni layout](#the-omni-layout)
4. [How permissions work](#how-permissions-work)
5. [Quick start](#quick-start)
6. [SMP: homes, warps, claims, chat](#smp-homes-warps-claims-chat)
7. [Ranks](#ranks)
8. [PvP](#pvp)
9. [Minigames](#minigames)
10. [Universal scoreboard](#universal-scoreboard)
11. [Admin toolbox](#admin-toolbox)
12. [Kits](#kits)
13. [Chest menus](#chest-menus)
14. [Server shop slots (prices)](#server-shop-slots-prices)
15. [Aliases (custom commands)](#aliases-custom-commands)
16. [Steps, timing and placeholders](#steps-timing-and-placeholders)
17. [Economy](#economy)
18. [Player market](#player-market)
19. [Duels, TPA and RTP](#duels-tpa-and-rtp)
20. [Default player commands](#default-player-commands)
21. [Fabric API hooks](#fabric-api-hooks)
22. [Configuration files](#configuration-files)
23. [settings.json reference](#settingsjson-reference)
24. [Safety design](#safety-design)
25. [Troubleshooting and FAQ](#troubleshooting-and-faq)
26. [Building from source](#building-from-source)
27. [Known limits](#known-limits)
28. [License](#license)

---

## Feature overview

| Module | What it does |
| --- | --- |
| SMP | Homes, warps, spawn, `/back`, land claims, teleport warmup and cooldown, chat channels, mutes, nicknames. |
| Ranks | Ranks you design yourself: prefix and suffix next to the name (above the head, tab list, chat), permissions, parent ranks, numeric limits, power up to operator level. |
| PvP | Ranked queue with Elo and leagues, arenas with snapshots and auto-reset, loadouts, kill rewards with anti-farm, assists, kill streaks, bounties, combat tag, seasons, open-world PvP. |
| Minigames | Spleef, TNT run, parkour, bedwars, skywars, KOTH, team deathmatch, free for all, last man standing. One engine runs both PvP matches and minigames. |
| Universal scoreboard | Profiles with a trigger, priority and placeholders. One board per player or one shared board, plus tab header/footer and action bar. |
| Admin toolbox | Server health (TPS, memory, chunks, entities), drop clean-up, inspect, freeze, audit log reader, in-game settings editor, backups, announcements, maintenance mode. |
| Kits | Save any inventory as a named kit and hand it to players. |
| Chest menus | Build clickable chest GUIs in-game. Each slot can show an item, run commands, or sell something. |
| Server shop slots | Give any menu slot a price. Players pay in-game money and receive a clean copy. |
| Aliases | Create player commands out of timed steps. A usage text shows players what to type (`/tpa <player>`). |
| Economy | Balances with pay, balance and leaderboard. |
| Market | Full player-to-player marketplace with a GUI, taxes, fees, expiry, claim box and multiple markets. |
| Duels, TPA, RTP | Request-based duels, teleport requests, random teleport. |

Other highlights:

- Operators work in-game. No datapacks and no config editing are required.
- Players only need a vanilla client. All menus use the normal chest screen.
- Data is saved as readable JSON, with a `.bak` copy and an audit log.
- Item data is stored as full item JSON, so enchantments, custom names and lore are kept.
- Players cannot take items out of menus.

---

## Requirements and installation

1. Install Fabric Loader 0.16.0 or newer for Minecraft 1.21.11.
2. Install Fabric API.
3. Use Java 21 or newer.
4. Put `omnitools-2.1.0.jar` into the `mods` folder.
5. Start the server. `config/omnitools/` is created automatically.
6. As an operator run `/omni admin hooks` (all hooks should be green) and `/omni admin install` (creates the player commands such as `/home` and `/pvp`).

If you previously used the older `kitmod`, remove it. OmniTools declares a conflict with it.

On the first start OmniTools creates ten default aliases (see [Default player commands](#default-player-commands)). They are only
created if `aliases.json` does not exist yet, so deleting an alias is permanent.

---

## The /omni layout

`/omni` has three helpers and six categories.

| Command | Purpose |
| --- | --- |
| `/omni about` | Version and build information. |
| `/omni help [category]` | Category list, or the commands of one category. |
| `/omni search <word>` | Find a command by word. |

| Category | Contents |
| --- | --- |
| `/omni smp` | Homes, warps, spawn, back, `tpa`, `rtp`, land claims (`land`), `money`, `market`, `chat`, `nick`, `rank`. |
| `/omni pvp` | `queue`, `stats`, `top`, `tier`, `bounty`, `duel`, `arenas`, `leave`, `spectate`, and for operators `arena`. |
| `/omni games` | `list`, `join`, `leave`, `spectate`, `team`, and for operators `game`. |
| `/omni board` | Players: `toggle`, `on`, `off`. Operators: profiles, lines, triggers. |
| `/omni tools` | Operators: `kit`, `chest`, `alias`. |
| `/omni admin` | Staff and operators: the admin toolbox, ranks, eco, market tools, reload, hooks. |

Players use short aliases for the player parts (`/home`, `/pvp`, `/mg`, `/board`, ...). Operators use the full paths.
Tab-complete shows only what the viewer is allowed to use.

---

## How permissions work

OmniTools has its own simple levels and does not need a permissions mod.

| Level | Who | What they can use |
| --- | --- | --- |
| Operator | Vanilla operators, the console, and players whose rank has power 3 or higher | Everything under `/omni`. |
| Staff | Rank power 1 and 2 | Parts of `/omni admin`: power 1 sees it, power 2 can use perf, entities, inspect, freeze, clear, audit and mute. |
| Player | Everyone else | Only the aliases that exist. |
| Trusted window | A player, only while an alias or menu step runs | `/omni` works for that step, but admin branches stay blocked. |

Rank power: 0 player, 1 helper, 2 moderator, 3 admin (counts as operator for every OmniTools command), 4 owner.
The threshold is `admin.rankAdminPower` (default 3). A rank can also give real vanilla op (`vanillaop`), see [Ranks](#ranks).

Other rules:

- `/omni` is hidden from tab-complete for players and cannot be run by them directly.
- Each alias can be set to **everyone** or **op** with `/omni tools alias permission`.
- Rank permissions (`omni.nick`, `omni.chat.color`, ...) restrict anything only when `smp.enforcePermissions` is `true`. By default everything a player may do stays allowed.

---

## Quick start

**1. Save a kit and build a menu from it**

```
/omni tools kit save starter
/omni tools chest fromkit large starter starter
/omni tools chest open large starter
```

**2. Make the menu open with a short player command**

```
/omni tools alias add kits menu 1 0 omni tools chest open large starter
```

**3. Create the player commands for SMP, PvP and boards**

```
/omni admin install
```

**4. Make a rank and give it to a player**

```
/omni admin rank create vip
/omni admin rank prefix vip &6[VIP]
/omni admin rank attr vip homes 6
/omni admin rank set Steve vip
```

Steve now has `[VIP]` next to his name above his head, in the tab list and in chat, and may set 6 homes.

**5. Build a duel arena and queue for it**

```
/omni pvp arena create pit duel
/omni pvp arena spawn pit add        (stand at spawn 1, run again at spawn 2)
/omni pvp arena spec pit             (where the loser watches)
/pvp queue join duel                 (two players)
```

**6. See the board and server health**

```
/omni board preview default
/omni admin perf
```

---

## SMP: homes, warps, claims, chat

### Homes, warps, spawn and back

| Command | Description |
| --- | --- |
| `/sethome [name]` | Save a home (default name `home`). The limit comes from the rank attribute `homes` (default `smp.maxHomes`). |
| `/home [name]` | Teleport to a home. |
| `/delhome <name>` | Delete a home. |
| `/homes` | List your homes. |
| `/warp <name>` and `/warps` | Teleport to a server warp, list warps. |
| `/spawn` | Teleport to the server spawn. |
| `/back` | Go to your last death place or teleport origin, whichever came last. |

Teleport rules (all of these apply to homes, warps, spawn, back):

- A **warmup** (default 3 s) that is cancelled if you move or take damage. Rank attribute `tpwarmup`.
- A **cooldown** (default 10 s). Rank attribute `tpcooldown`.
- **Combat tag:** while you are in combat you cannot teleport (see [PvP](#pvp)). Rank attribute `bypasscombat` ignores this.
- Operators skip warmup and cooldown.

Operators create warps with `/omni admin setwarp <name>`, `/omni admin delwarp <name>` and `/omni admin setspawn`.
Names use a-z, 0-9, `_`, `-` (up to 16).

### Land claims

Claims protect land against other players. They are full-height columns.

| Command | Description |
| --- | --- |
| `/landclaim pos1` / `pos2` | Set the two corners at your position. A **golden shovel** also works: right-click a block for corner 1, again for corner 2. |
| `/landclaim create` | Create the claim. Limits: rank attributes `claims` (count) and `claimarea` (blocks), defaults `smp.maxClaims` and `smp.maxClaimArea`. |
| `/landclaim delete [id]` | Delete the claim you stand in, or by id. |
| `/landclaim list` and `info` | Your claims, and the claim you stand in. |
| `/landclaim trust <player>` and `untrust <player>` | Let another player build in your claim. |
| `/landclaim flag pvp <true\|false>` | Allow or forbid player-vs-player damage in the claim. |
| `/omni smp land admin delete <id>` | Operators: delete any claim. |

Protected: breaking blocks, using items on blocks, and opening containers (needs the block hooks), and PvP damage when the pvp flag is off.
Doors and buttons stay usable with an empty hand. Claims cannot overlap. (`/claim` stays the market claim box.)

### Chat, nicknames and mutes

| Command | Description |
| --- | --- |
| `/chat <global\|local\|staff\|trade>` | Choose your channel. `local` reaches `smp.localChatRange` blocks (default 100). `staff` is for rank power 1 and up. |
| `/nick <name>` and `/nick off` | A chat nickname (3-16 letters, digits, `_`). It shows in chat and on boards. The real name stays above the head, but the **rank prefix does show** there. |
| `/omni admin mute <player> [minutes] [reason]` | Mute (0 or no minutes = permanent). Power 2 and up. |
| `/omni admin unmute <player>` and `mutes` | Unmute, list mutes. |

Chat format: `[channel] [rank prefix] <name> [suffix] message`. `&` color codes in messages need the rank attribute `chatcolor` or the
permission `omni.chat.color`. Chat formatting and mutes need the chat hook; turn formatting off with `smp.chatFormat false`.

### Rank info for players

`/rank` shows your rank, power, homes, claims, teleport warmup and cooldown.

---

## Ranks

Ranks are made by operators, entirely in game. Command: `/omni admin rank ...` (power 3 or operator).

| Command | Description |
| --- | --- |
| `create <name> [power]` | New rank (name: a-z, 0-9, `_`, up to 12). |
| `clone <rank> <new>` | Copy a rank. |
| `delete <rank>` | Delete a rank (its players become `default`). The `default` rank cannot be deleted. |
| `prefix <rank> <text>` and `suffix <rank> <text>` | Text next to the name. `&` codes and `&#rrggbb` work. Use `none` to clear. Up to 24 visible characters. |
| `color <rank> <color>` | Name color (a color name or a code such as `&c`). |
| `weight <rank> <0-99>` | Order in the tab list (higher first). |
| `power <rank> <0-4>` | See [permissions](#how-permissions-work). |
| `parent <rank> <parent\|none>` | Inherit permissions and attributes. Loops are refused. |
| `perm <rank> add\|remove <perm>` | Permissions. `omni.*` matches everything below, `*` matches all, `-perm` removes an inherited one. |
| `attr <rank> <key> [value]` | Numeric limits (see below). No value resets it. |
| `vanillaop <rank> <true\|false>` | Players who get this rank also get real vanilla op. It is removed again when the rank is taken away. |
| `set <player> <rank>` | Give a rank (works for offline players who joined before). |
| `player <player>`, `info <rank>`, `list`, `sync` | Look things up, rebuild the rank teams. |

**Attributes:** `homes`, `claims`, `claimarea`, `tpwarmup`, `tpcooldown`, `kitcooldown`, `chatcolor`, `nick`, `bypasscombat`.
The first value found going from the rank up through its parents is used.

**How the look works:** every rank with a prefix, suffix or color gets a scoreboard team. The prefix therefore shows next to the name
**above the head, in the tab list and in chat**, with no client mod. Each team name begins with the rank weight (for example `r90_vip`), which is what sorts the tab list.
A player who is in another team (for example from a datapack) is moved into the rank team.

Data: `config/omnitools/ranks.json`. Ranks created with build 4 are imported once.

---

## PvP

Command: `/pvp ...` (alias of `/omni pvp ...`).

### Queue, stats and rewards

| Command | Description |
| --- | --- |
| `/pvp queue join <mode> [arena]` | Join the ranked queue. Modes: `duel ffa tdm koth lms practice`. |
| `/pvp queue leave` and `status` | Leave, see your waiting time and Elo window. |
| `/pvp stats [player]` | Elo, league, wins/losses/draws, kills/deaths, streak. |
| `/pvp top [mode]` | Top 10 by Elo. |
| `/pvp tier [mode]` | Your league and points to the next. |
| `/pvp arenas` | Ready arenas and their state. |
| `/pvp leave` and `/pvp spectate <arena>` | Leave a match, or watch one. |
| `/pvp bounty set <player> <amount>` and `bounty list` | Put money on a player's head (minimum `pvp.bountyMin`, tax `pvp.bountyTaxPercent`). Whoever kills the target collects it. |
| `/pvp duel request <player> <arena>` ... | The classic duel, see [Duels, TPA and RTP](#duels-tpa-and-rtp). |

How matchmaking works: players with a similar Elo are paired. The accepted gap starts at 200 and grows by 50 every 10 seconds of waiting.
When a match is found both players get a 5 second countdown. The queue is saved, so a restart does not lose waiting players (entries expire after 30 minutes).

Rules that apply in arenas **and** in the open world (open world needs the death hooks and `pvp.openWorldPvp`):

- **Kill reward** `pvp.killReward` and **death penalty** `pvp.deathPenalty`.
- **Anti-farm:** killing the same victim again within 30 s pays nothing, within 60 s pays half.
- **Kill streaks:** bonus money `pvp.streakBonus` per streak step from the 3rd kill (capped), server announcement at 3 and every 5.
- **Assists:** players who did at least 4 damage within 10 s before the kill get `pvp.assistReward`.
- **Bounties:** paid to the killer, minus tax.
- **Combat tag:** after player-vs-player damage both players are tagged for `pvp.combatSeconds` (default 15). They cannot use teleport commands. Logging out
  while tagged kills the player (`pvp.combatLogKill`) and announces it.
- **Leagues:** names and Elo limits from `pvp.tiers` (default Bronze 0, Silver 1000, Gold 1200, Platinum 1400, Diamond 1600, Master 1800, Grandmaster 2000).
- **Elo** uses K = 32 with a floor of 100. Free-for-all modes compare each player with the average of the others. Team modes do not change Elo.

### Arenas (operators)

`/omni pvp arena <sub> ...` (the same builder serves minigames, see below).

| Command | Description |
| --- | --- |
| `create <name> <type>` | New arena. Types: `duel ffa tdm koth lms practice spleef tnt_run parkour bedwars skywars`. |
| `delete <name>`, `list`, `info <name>` | Manage and inspect. `info` says what is still missing. |
| `spawn <name> add\|clear` | Spawn points at your position (all in the same world). |
| `lobby <name>` and `spec <name>` | Lobby point and spectator point. |
| `region <name> pos1\|pos2` | The arena box. Players cannot leave it. Active arenas are protected from outsiders. |
| `zone <name> pos1\|pos2` | KOTH hill. |
| `finish <name> pos1\|pos2` | Parkour finish. |
| `bed <name> <team>` | Bedwars bed of a team (stand on the bed). |
| `set <name> <key> <value>` | `min max countdown timelimit teams reset lives winscore rewardwin rewardlose rewardkit loadout pvp build keepinv friendly sidebar elo killrewards`. |
| `snapshot <name>` and `reset <name>` | Save the region's blocks, and restore them. The arena also resets itself after every match. |
| `start <name>` and `stop <name>` | Force-start a countdown, or stop. |

Details:

- **Loadouts:** `set <name> loadout <kit>` gives the kit at the start. The player's own inventory is saved in a vault (`pvp/vault.json`) and returned after the match,
  and also after a crash when the player joins again.
- **Death handling:** deaths inside a match are cancelled (no respawn screen, no item loss). The loser becomes a spectator, or respawns after 3 s when the type uses respawns (`lives 0`).
- **Snapshots:** only block states are stored (not chest contents). The limit is `pvp.snapshotMaxBlocks` (250000). Reset changes only differing blocks, 3000 per 100 ms, so the server does not lag.
- **Spectators** cannot use blocks and cannot leave the arena surroundings.

### Seasons

`/omni admin season new [keepPercent]` makes a backup, pulls every rating toward 1000 (default keep 50 %), and clears wins, losses, streaks and kill counters.

---

## Minigames

Command: `/mg ...` (alias of `/omni games ...`). Operators build games with `/omni games game ...`, which has exactly the same sub-commands as `/omni pvp arena` (see [Arenas](#arenas-operators)).

| Command | Description |
| --- | --- |
| `/mg list` | Games and their state. |
| `/mg join <game>` | Join. Players wait in the lobby until `min` players are there, then the countdown starts. |
| `/mg leave` and `/mg spectate <game>` | Leave, watch. |
| `/mg team <color>` | Choose a team in the lobby (`red blue green yellow`). Otherwise teams are balanced automatically. |

Phases: lobby, countdown (players are frozen with slowness and jump boost), running, ending, reset.

| Type | Win condition |
| --- | --- |
| `ffa`, `lms`, `duel`, `skywars` | Last player alive. |
| `tdm` | First team to `winscore` kills, or the best team when time runs out. |
| `koth` | First to `winscore` points for staying alone in the zone. |
| `spleef`, `tnt_run` | Last player alive. Falling below the region eliminates. |
| `parkour` | First player into the finish region. |
| `bedwars` | Last team with a bed or members alive. A destroyed bed means no respawn. Own beds are protected. |
| `practice` | No end. |

Lives (`lives`), time limit (`timelimit`), friendly fire, building (`build`) and rewards (`rewardwin`, `rewardlose`, `rewardkit`) are set per game.
Minigame results are saved under the mode `mg:<type>`.

---

## Universal scoreboard

`/omni board` shows what you see, and lets operators design everything. Players switch their own board with `/board toggle`, `on` and `off`.

### Profiles

A **profile** has a title, up to 15 lines, an optional tab header and footer, an optional action bar, a **trigger**, a **priority** and a **scope**.
Each player gets the matching profile with the highest priority.

| Scope | Behavior |
| --- | --- |
| `player` | Every player has his own board (packets to that player only). Placeholders show his own values. |
| `global` | One shared sidebar for everybody (vanilla scoreboard commands). Player placeholders stay empty. Works without packets. |

Built-in profiles (edit, copy or delete them): `default` (SMP), `queue`, `pvp`, `minigame`, `combat`.

### Triggers

A trigger is a list of terms separated by commas. All terms must match.

| Term | Meaning |
| --- | --- |
| `key` | The fact exists and is not empty or `false`. |
| `key:value` | Equal (ignoring case). A trailing `*` means "starts with". |
| `!term` | Not. |
| `always` | Always. |

Facts: `world`, `rank`, `power`, `kind` (`pvp`, `mg` or `none`), `type`, `game`, `phase`, `team`, `queue`, `combat`, `channel`.

Examples: `always` - `kind:pvp` - `kind:mg,type:spleef` - `rank:vip` - `queue` - `combat,!kind:pvp` - `world:minecraft:the_nether`.

### Commands (operators)

| Command | Description |
| --- | --- |
| `/omni board list`, `status`, `reload`, `placeholders` | Overview, packet status, reload, placeholder list. |
| `create <name> [copy]` and `delete <profile>` | New profile (a new one starts with trigger `never`). |
| `title`, `tabheader`, `tabfooter`, `actionbar <profile> <text>` | Texts. `&` colors, `%placeholders%`, `\n` in tab texts, `none` clears. |
| `line add\|set\|insert\|remove\|clear <profile> ...` | Lines (1-15). |
| `scope <profile> player\|global` | Scope. |
| `trigger <profile> <expr>` and `priority <profile> <n>` | When it shows, which one wins. |
| `numbers <profile> <true\|false>` | Show or hide the red score numbers. |
| `preview <profile>` | Show the profile in chat with your own values. |

### Placeholders

Write `%name%` or `%name:argument%`. Unknown placeholders stay visible, so a typo is easy to spot. `%%` gives a `%`.

| Group | Placeholders |
| --- | --- |
| Player | `player`, `nick`, `rank`, `rank_prefix`, `rank_display`, `power`, `balance`, `balance_raw`, `health`, `world`, `world_short`, `x`, `y`, `z`, `channel` |
| Server | `online`, `max`, `tps`, `mspt`, `time`, `date`, `server`, `games_running` |
| SMP | `homes`, `homes_max`, `claims`, `claims_max`, `claimbox`, `listings` |
| PvP | `pvp_elo`, `pvp_tier`, `pvp_next`, `pvp_wins`, `pvp_losses`, `pvp_kills`, `pvp_deaths`, `pvp_kd` (all take `:mode`, default `duel`), `pvp_streak`, `pvp_bounty`, `combat`, `queue`, `queue_wait`, `queue_size` |
| Game | `game`, `game_type`, `game_phase`, `game_time`, `game_players`, `game_alive`, `game_kills`, `game_deaths`, `game_score`, `game_lives`, `team`, `team_color`, `team_score:<color>` |
| Tops | `baltop_name:<n>`, `baltop_value:<n>`, `pvptop_name:<n>[:mode]`, `pvptop_value:<n>[:mode]` |

One placeholder system serves every module, so one board can mix SMP, PvP and games, or each system can have its own profile.

Data: `config/omnitools/boards.json`. Per-player boards and the tab list depend on packet signatures that are checked at start; `/omni board status` tells you whether they are ready.

---

## Admin toolbox

`/omni admin ...`. Levels: power 2 = moderator rank or higher, power 3 = admin rank or operator. Vanilla operators can always do everything.

| Command | Level | Description |
| --- | --- | --- |
| `perf` | 2 | TPS, average and worst tick time, memory, uptime, players, chunks and entities per world, running sessions. |
| `chunks` | 2 | Loaded chunks per world. |
| `entities [world]` | 2 | The 12 most common entity types. |
| `clear items\|xp\|arrows\|all [world]` | 2 | Remove dropped items, XP orbs, arrows. |
| `autoclear on\|off\|<minutes>` | 3 | Scheduled clean-up with warnings at 30, 10 and 5 seconds. |
| `gc` | 3 | Run garbage collection and show memory before and after. |
| `inspect <player>` | 2 | Rank, balance, homes, claims, position, health, combat, game, mute. |
| `freeze <player>` and `unfreeze <player>` | 2 | Hold a player in place. |
| `broadcast <text>` | 2 | Message to everyone (`&` colors). |
| `audit tail [n]` and `audit search <word>` | 2 | Read `audit.jsonl` in game. |
| `settings list\|get\|set <key> <value>` | 3 | Change 35 settings in game. Every number is range-checked, every change is audited. |
| `backup now` and `backup list` | 3 | Zip of the OmniTools data in `config/omnitools/backups`. Automatic every `admin.backupHours` (0 = off), the newest `admin.backupKeep` are kept. |
| `announce on\|off\|list\|now\|interval <min>\|add <text>\|remove <n>` | 3 | Rotating announcements. |
| `maintenance on [message]\|off\|status` | 3 | Kick everyone below power 2 and refuse them until it is off. |
| `season new [keep%]` | 3 | New PvP season. |
| `mute`, `unmute`, `mutes` | 2 | See [Chat, nicknames and mutes](#chat-nicknames-and-mutes). |
| `rank ...` | 3 | See [Ranks](#ranks). |
| `eco give\|take\|set\|reset` | 3 | See [Economy](#economy). |
| `market list\|remove\|log\|create\|delete\|blacklist\|set\|settings\|limit\|sweep` | 3 | See [Player market](#player-market). |
| `setwarp`, `delwarp`, `setspawn`, `install` | 3 | Warps, spawn, create the player aliases. |
| `reload` and `hooks` | 3 | Reload files; show hook and scoreboard status. |

`/omni reload` waits (at most 2 minutes) for running command sequences before it reloads.

---

## Kits

A kit is a saved snapshot of an inventory: hotbar, main inventory, armor slots and the offhand. Kits are stored in `kits.json`.

| Command | Description |
| --- | --- |
| `/omni tools kit save <n>` | Save your own inventory as a kit. |
| `/omni tools kit saveplayer <n> <player>` | Save another online player's inventory. |
| `/omni tools kit remove <n>` | Delete a kit. |
| `/omni tools kit rename <n> <new>` | Rename a kit. |
| `/omni tools kit copy <n> <new>` | Duplicate a kit. |
| `/omni tools kit list` | List all kits. |
| `/omni tools kit info <n>` | Show what is inside a kit. |
| `/omni tools kit give <kit> <players> [gearPercent]` | Clear the target players' inventories, then give the kit. |
| `/omni tools kit add <kit> <players>` | Add the kit's items to free slots without clearing anything. |
| `/omni tools kit giveall <kit> [playerPercent] [gearPercent]` | Give a kit to everyone online, or to a random percentage. |
| `/omni tools kit randomall <kit percent kit percent ...>` | Give every online player a random kit, using the percentages as weights. |
| `/omni tools kit clear <players>` | Clear the target players' inventories. |

Notes:

- `gearPercent` rolls armor and tool durability to a percentage.
- A kit slot that holds armor (slots 36-39) or the offhand (40) goes to that slot when the kit is added with `kit add`; with `kit give` the whole inventory is replaced slot by slot.
- When the inventory is full, items from `kit add` go to the player's market claim box instead of being lost.
- Items are stored as complete item JSON (custom names, enchantments, lore, shulker contents).

---

## Chest menus

A chest menu is a virtual chest that only exists as data. There are two sizes: **large** (6 rows) and **small** (3 rows). Names are unique per size.

### Editing a menu

```
/omni tools chest editchest <large|small> <n>
```

| Action in edit mode | Result |
| --- | --- |
| Place an item in a slot | The slot shows that item. |
| Left-click an item | Pick up the stack. |
| Right-click an item | Pick up one item. |
| Shift-click an item | Receive a copy in your inventory. |
| Shift-click from your inventory | Move the item into the first free slot. |
| Press `Q` over an item | Delete it from the menu. |
| Close the screen | **Save.** |

When you close the screen, prices that point to an empty slot are dropped. If an item cannot be encoded it is kept as it was (or handed back to you), never silently deleted.
With `market.blockShulkerInShopChests` on, a shulker box with contents cannot be saved into a chest that has prices.

### Using a menu as a player

- Clicking an item gives a **copy**. Nothing can be put in.
- Drag, double-click collect and shift-click from the inventory are blocked.
- If a slot has commands, a left or shift click runs them. If it has a price, it buys.

### Menu commands

| Command | Description |
| --- | --- |
| `/omni tools chest open <size> <n> [players]` | Open a menu. If no chest has that name, a kit with that name is shown. |
| `/omni tools chest run <size> <n> <slot> [players]` | Run a slot's steps directly. |
| `/omni tools chest editchest <size> <n>` | Edit in game. |
| `/omni tools chest fromkit <size> <n> <kit>` | Build a menu from a kit. |
| `/omni tools chest list`, `info <size> <n>` | List, details. |
| `/omni tools chest delete <size> <n>`, `copy <size> <n> <new>`, `rename <size> <n> <new>` | Manage. Renames update aliases that point at the menu. |

### Commands on a slot

| Command | Description |
| --- | --- |
| `/omni tools chest cmd <size> <n> add <slot> <step> <number> <time> <command>` | Step that runs as the player. |
| `/omni tools chest cmd <size> <n> addconsole <slot> <step> <number> <time> <command>` | Step that runs as the console. |
| `/omni tools chest cmd <size> <n> remove <slot> <step>` | Remove a step. |
| `/omni tools chest cmd <size> <n> settime <slot> <step> <time>` | Change the delay. |
| `/omni tools chest cmd <size> <n> flag <slot> <step> selector\|dangerous <true\|false>` | Safety exceptions, see [Safety design](#safety-design). |
| `/omni tools chest cmd <size> <n> clear <slot>`, `list <slot>`, `listall` | Clear, show. |
| `/omni tools chest cmd <size> <n> close <slot> <true\|false>` | Close the menu after the click or not. |

---

## Server shop slots (prices)

Any menu slot can become a fixed-price shop slot. Money is taken from the player and removed from the economy.

| Command | Description |
| --- | --- |
| `/omni tools chest price <size> <n> set <slot> <price>` | Set a price. |
| `/omni tools chest price <size> <n> clear <slot>` | Remove a price. |
| `/omni tools chest price <size> <n> list` | Show prices. |

- Priced slots get lore with the price and "Click to buy".
- A left or shift click buys. The buyer receives a **clean copy**.
- Everything is checked **before** any money is taken. If the inventory cannot hold the item and the claim box is full (`market.maxClaimsPerPlayer`), the purchase is refused and nothing is charged.
- If the slot also has commands, commands win and the price is ignored.
- Every shop purchase is written to the audit log.

---

## Aliases (custom commands)

An alias is a real root command created by an operator. Each alias is a list of ordered steps, each after its own delay.

### Alias commands

| Command | Description |
| --- | --- |
| `/omni tools alias add <alias> <step> <number> <time> <command>` | Step that runs as the player. Creates the alias if needed. |
| `/omni tools alias addconsole <alias> <step> <number> <time> <command>` | Step that runs as the console. |
| `/omni tools alias addslot <alias> <step> <number> <time> <size> <chest> <slot>` | Step that runs a menu slot. |
| `/omni tools alias delete command <alias> <step>` / `delete alias <alias>` | Remove a step or the alias. |
| `/omni tools alias list`, `info <alias>` | Show aliases (with their usage). |
| `/omni tools alias run <alias> [players]` | Run for yourself or others. |
| `/omni tools alias settime <alias> <step> <time>` | Change a delay. |
| `/omni tools alias cooldown <alias> <time>` | Per-player cooldown. |
| `/omni tools alias permission <alias> <everyone\|op>` | Who may use it. |
| `/omni tools alias usage <alias> <text>` | What players type after the command (next section). |
| `/omni tools alias flag <alias> <step> selector\|dangerous <true\|false>` | Safety exceptions. |

### Showing players what to type: usage

Without help the client shows only a nameless argument after an alias that uses `{args}`. A **usage** text fixes that: the command tree then shows
`/tpa <player>`, `/pay <player> <amount>` and so on, in tab-complete and in lists.

```
/omni tools alias usage tpa <player>;here <player>;accept;deny;cancel
/omni tools alias usage pay <player> <amount>
/omni tools alias usage nick <name>;off
```

| Token | Meaning |
| --- | --- |
| `word` | A fixed word. `a\|b` gives several fixed words. |
| `<name>` | A required single word. A name `player` or `target` suggests online players. |
| `[name]` | An optional single word. |
| `<name...>` or `[name...]` | The rest of the line. |
| `;` | Separates forms (up to 16 forms, up to 6 tokens each). |

Rules: an alias with a usage accepts **only** the forms in it. Without a usage the alias accepts one free argument shown as `<args>` (only if a step uses `{args}`).
Use `none` to clear a usage. The ten default aliases and the commands made by `/omni admin install` already have a usage.

### Rules

- Alias names use lowercase letters, numbers, `_` and `-`, up to 32 characters.
- Reserved names cannot be used: `omni`, the six categories, vanilla admin commands (`op`, `ban`, `kick`, `stop`, ...), and common names such as `help`, `tell`, `msg`, `list`, `tps`.
- An alias cannot call itself.
- New aliases are registered live and tab-complete is refreshed for everyone.
- A deleted alias is hidden and cannot be run.

---

## Steps, timing and placeholders

Both aliases and menu slots are made of steps. Every step has a `step` name, a `number` (position), a `time` (delay after the previous step) and a `command`.

| Time | Meaning |
| --- | --- |
| `0` | No delay |
| `500ms` | 500 milliseconds |
| `5sec` | 5 seconds |
| `2min` | 2 minutes |

The maximum is 60 minutes. A bare number other than `0` is rejected so the unit is never in doubt.

Step placeholders (different from board placeholders):

| Placeholder | Replaced with |
| --- | --- |
| `{player}` | The player's name |
| `{uuid}` | The player's UUID |
| `{x}` `{y}` `{z}` | The player's block position |
| `{world}` | The player's dimension id |
| `{args}` | What the player typed after the alias |

`{args}` is limited to 5 words, 32 characters per word, and the characters letters, digits, `_ - . :`. Selectors, quotes, braces, `~` and `^` are rejected.

Examples:

```
/omni tools alias add kits menu 1 0 omni tools chest open large starter
/omni tools alias addconsole spawn msg 1 0 tellraw {player} {"text":"Teleporting in 3 seconds...","color":"yellow"}
/omni tools alias addconsole spawn tp 2 3sec tp {player} 0 100 0
/omni tools alias add duel duel 1 0 omni pvp duel {args}
/omni tools alias usage duel request <player> <arena>;accept;deny;forfeit
```

---

## Economy

Balances are whole numbers stored in `economy.json`. New players start with `economy.startingBalance`, balances are capped at `economy.maxBalance`.
Every change is written to the audit log with the balance before and after.

### Player commands (`/omni smp money ...`, normally through aliases)

| Alias | Command |
| --- | --- |
| `/balance [player]` | Show a balance. |
| `/pay <player> <amount>` | Send money. Payment is atomic: it fails with a message if you lack money or the receiver is at the maximum. Nothing is destroyed. |
| `/baltop` | Richest players. |

### Operator commands (`/omni admin eco ...`)

`give <player> <amount>`, `take <player> <amount>`, `set <player> <amount>`, `reset <player>`.

---

## Player market

A complete player-to-player marketplace with a chest GUI. Commands are under `/omni smp market ...`; players normally use `/market`, `/sell`, `/mylistings` and `/claim`.

### Selling and buying

- `/sell <price> [amount]` sells what you hold. Or open `/market` and use the "Sell an item" screen (price buttons, amount, summary with fee and tax).
- `/market` opens the browser. Click a listing to buy. A confirm screen is shown when `market.confirmPurchase` is on.
- Browser: 45 listings per page, category filter, sorting (newest, oldest, price, name), average sold price per item.
- `/mylistings`: left-click cancels, right-click changes the price, shift-click renews.
- `/claim` opens your claim box (expired listings, purchases made with a full inventory, removed listings, shop overflow).

Player sub-commands: `open [market]`, `search <text>`, `seller <n>`, `history`, `sell <price> [amount]`, `mine`, `claim`, `price <id> <price>`, `cancel <id>`.

### Money flow and safety

- The seller gets the price minus the **sales tax**. An optional flat **listing fee** is charged when listing.
- A purchase checks everything first: the seller's balance has room, the item fits the inventory or the claim box has room, the money is taken in one checked step. If any check fails, nothing changes.
- A claim box holds at most `market.maxClaimsPerPlayer` entries for purchases (system claims such as expired listings are never refused, so no item is lost).
- Listings expire after `market.listingDurationHours`. Sales and actions are written to `market-log.jsonl`.

### Operator tools (`/omni admin market ...`)

`list [player]`, `remove <id>`, `log [n]`, `create <n>`, `delete <n>`, `blacklist add|remove|list <item>` (wildcards work), `set <key> <value>`
(keys `tax fee minprice maxprice maxlistings duration confirm currency`), `settings`, `limit <player> <n|reset>`, `sweep`.

---

## Duels, TPA and RTP

### Duels (`/omni pvp duel ...`, alias `/duel`)

```
/duel request <player> <arena>
/duel accept   /duel deny   /duel forfeit
```

A request expires after 60 seconds. Both players fight with their own gear and normal death rules. Duel arenas are set up by operators:

```
/omni pvp duel arena set <n> <1|2>
/omni pvp duel arena remove <n>
/omni pvp duel arena list
```

For ranked duels with Elo, snapshots and rewards use the PvP queue and `/omni pvp arena` instead. A player cannot be in a duel and in a PvP match or minigame at the same time.

### TPA (`/omni smp tpa ...`, alias `/tpa`)

```
/tpa <player>   /tpa here <player>   /tpa accept   /tpa deny   /tpa cancel
```

Requests expire after 60 seconds. The combat tag blocks teleports.

### RTP (`/omni smp rtp ...`, alias `/rtp`)

```
/rtp   /rtp <world>   /rtp <world> <minRadius> <maxRadius>
```

Default radius 500 to 5000, up to 40 tries to find a safe spot, 30 second cooldown.

---

## Default player commands

Created the first time the mod starts (only if `aliases.json` does not exist yet).

| Alias | Runs | Shows |
| --- | --- | --- |
| `/tpa` | `omni smp tpa {args}` | `<player>`, `here <player>`, `accept`, `deny`, `cancel` |
| `/rtp` | `omni smp rtp {args}` | `[world] [min] [max]` |
| `/duel` | `omni pvp duel {args}` | `request <player> <arena>`, `accept`, `deny`, `forfeit` |
| `/market` | `omni smp market {args}` | `<args>` |
| `/sell` | `omni smp market sell {args}` | `<price> [amount]` |
| `/mylistings` | `omni smp market mine` | |
| `/claim` | `omni smp market claim` | |
| `/balance` | `omni smp money balance {args}` | `[player]` |
| `/pay` | `omni smp money pay {args}` | `<player> <amount>` |
| `/baltop` | `omni smp money top` | |

`/omni admin install` adds more (existing aliases are never changed):

`/sethome` `/home` `/delhome` `/homes` `/warp` `/warps` `/spawn` `/back` `/landclaim` `/chat` `/nick` `/rank` `/board` `/pvp` `/mg` `/tpaccept` `/tpdeny`.

Names come from the `commands` block of `settings.json` where applicable.

---

## Fabric API hooks

Some features need Fabric API events. They are installed by reflection; if one is missing the log says so and only that feature is off. `/omni admin hooks` shows the state (green = installed).

| Hook | Needed for |
| --- | --- |
| `death` | Cancelling deaths in matches (no respawn screen, no item loss). Do not run matches without it. |
| `damage` | Match rules, friendly fire, claim PvP flag, combat tag, assists. |
| `after-death` | Open-world kills, streaks and bounties. |
| `block-break` and `use-block` | Claim protection, arena build rules, the golden shovel. |
| `chat` | Chat channels, rank prefix in chat, mutes. |
| `join` and `leave` | Maintenance mode, rank team sync, combat log penalty, leaving a match on disconnect. |

`/omni admin hooks` also shows the **per-player scoreboard** status. If a packet signature does not match, per-player boards and tab lists are switched off by themselves and `scope global` boards keep working.

---

## Configuration files

All data lives in `config/omnitools/`. Files are plain JSON (logs are JSON Lines). Edit them only while the server is stopped, or run `/omni admin reload` afterwards.

| File | Contents |
| --- | --- |
| `kits.json`, `chests.json`, `aliases.json`, `arenas.json` | Kits, menus (items, steps, prices), aliases (steps, cooldowns, permissions, usage), duel arenas |
| `settings.json` and `settings.schema.json` | Settings, and the schema for editors (rewritten at every start) |
| `economy.json`, `market.json`, `market-log.jsonl` | Balances, listings and claim boxes, market log |
| `ranks.json` | Ranks, player ranks, who got vanilla op from a rank |
| `boards.json` | Scoreboard profiles and players who turned their board off |
| `admin.json` | Maintenance, announcements, last backup time |
| `games.json` | PvP arenas and minigames |
| `smp/homes.json`, `smp/warps.json`, `smp/claims.json`, `smp/social.json` | Homes, warps and spawn, land claims, nicknames and mutes |
| `pvp/stats.json`, `pvp/queue.json`, `pvp/bounties.json`, `pvp/vault.json` | Stats, queue, bounties, saved inventories during loadout matches |
| `snapshots/<arena>.snap` | Arena snapshots |
| `backups/*.zip` | Backups made by `/omni admin backup` |
| `audit.jsonl` | Money changes, listings, purchases, claims, kit gives, chest opens, alias runs, flags, blocked steps, game results, rank changes, settings changes (rotates at 16 MB) |

Safety for your data:

- Writes are atomic and a `.bak` copy of the previous version is kept.
- **Safe mode:** a file that cannot be read is moved to `<name>.corrupt-<time>` (never deleted or overwritten with empty data) and defaults are used. The log says so, and `/omni admin reload` tells the operator.
- Older menu formats are converted automatically.
- Stored commands from 1.x are moved to the new `/omni` paths once (see [UPDATE-2.1.0.md](UPDATE-2.1.0.md)).

---

## settings.json reference

Created on first start. Edit by hand, or use `/omni admin settings set <key> <value>` for most numbers. Out-of-range values are clamped and logged.

```json
{
  "commands": { "tpa": "tpa", "rtp": "rtp", "duel": "duel", "market": "market", "sell": "sell",
                "mylistings": "mylistings", "claim": "claim", "balance": "balance", "pay": "pay", "baltop": "baltop" },
  "economy": { "currencySymbol": "$", "startingBalance": 100, "maxBalance": 1000000000000 },
  "market":  { "listingFee": 0, "salesTaxPercent": 5, "minPrice": 1, "maxPrice": 1000000000, "maxListingsPerPlayer": 10,
               "listingDurationHours": 168, "confirmPurchase": true, "maxClaimsPerPlayer": 200,
               "blockShulkerInShopChests": false, "blacklist": [] },
  "safety":  { "stepsPerSecond": 10 },
  "smp":     { "maxHomes": 3, "teleportWarmup": 3, "teleportCooldown": 10, "baseHomes": 3, "maxClaims": 3, "maxClaimArea": 10000,
               "localChatRange": 100, "enforcePermissions": false, "chatFormat": true },
  "pvp":     { "killReward": 10, "deathPenalty": 5, "snapshotMaxBlocks": 250000, "combatSeconds": 15, "combatLogKill": true,
               "assistReward": 3, "streakBonus": 5, "bountyTaxPercent": 10, "bountyMin": 50, "openWorldPvp": true,
               "tiers": "Bronze:0,Silver:1000,Gold:1200,Platinum:1400,Diamond:1600,Master:1800,Grandmaster:2000" },
  "admin":   { "rankAdminPower": 3, "layoutVersion": 2, "autoClear": false, "autoClearMinutes": 15, "backupHours": 24, "backupKeep": 10 }
}
```

| Setting | Meaning |
| --- | --- |
| `commands.*` | Command names used in help and messages (a-z, 0-9, `_`, `-`). Duplicates fall back to the default. |
| `economy.*` | Currency symbol (up to 8 characters), starting balance, balance cap. |
| `market.*` | Fee, tax, price limits, listings per player, duration (hours), purchase confirm, claim box limit for purchases, shulker rule for shop chests, item blacklist (`*` wildcards). |
| `safety.stepsPerSecond` | Steps one non-operator player may trigger per second (1-100). Extra steps are delayed, not dropped. |
| `smp.maxHomes` | Default home limit when the rank has no `homes` attribute. (`baseHomes` is kept for old files and is not used.) |
| `smp.teleportWarmup`, `teleportCooldown` | Seconds, defaults for ranks without `tpwarmup` / `tpcooldown`. |
| `smp.maxClaims`, `maxClaimArea` | Defaults for ranks without `claims` / `claimarea`. |
| `smp.localChatRange` | Blocks reached by local chat. |
| `smp.enforcePermissions` | `true`: rank permissions restrict (home count aside). `false`: everything stays allowed. |
| `smp.chatFormat` | Format global chat with rank prefix and nickname. |
| `pvp.*` | Money for kills, assists and streaks, death penalty, combat tag seconds (0 = off), combat log kill, bounty tax and minimum, open-world PvP, league table, snapshot size limit. |
| `admin.rankAdminPower` | Rank power that counts as operator (1-4). |
| `admin.layoutVersion` | 2 = the 2.1.0 command layout is in use. Set by the mod. |
| `admin.autoClear*`, `backup*` | Scheduled clean-up and backups. |

---

## Safety design

OmniTools is built so that giving players commands does not give them power over the server.

- **Operators only for `/omni`.** Players cannot run it, see it, or tab-complete it. Staff ranks see only the parts their power allows.
- **Trusted window.** While an alias or menu step runs, `/omni` is allowed for that player only for non-admin branches.
- **Console steps are checked for non-operators.** A console step is refused when it runs a blocked command (`op deop ban ban-ip pardon pardon-ip kick stop reload whitelist save-all save-off save-on`, also hidden inside `execute ... run` or `return run`), when `{args}` is used as the command itself, or when it combines `{args}` with a selector.
  Allow a deliberate exception per step with `flag selector true` or `flag dangerous true` (see the alias and chest commands). At every start the log lists console steps that use `{args}`.
- **Step rate limit.** `safety.stepsPerSecond` per non-operator player.
- **No selectors for players** in player-mode steps.
- **Strict `{args}`.** At most 5 words, safe characters only.
- **Read-only menus.** Drag, double-click collect and shift-click transfers are blocked.
- **No free copies from shop slots.** Checks come before payment.
- **Atomic money.** Pay, market purchases and shop purchases check everything first and never destroy money or lose items. Undelivered items go to the claim box.
- **Audit log** (`audit.jsonl`) with before/after values. It is a log, not an automatic undo tool.
- **Backups and safe mode** for data (see [Configuration files](#configuration-files)).
- **One background scheduler** with a queue limit (5000 delayed steps). Background threads never touch game state.
- **Server-side only.** Players need nothing installed.

---

## Troubleshooting and FAQ

**Players can run `/omni`.**
They cannot. Check that they are not operators and that their rank power is below `admin.rankAdminPower`.

**The rank prefix does not show.**
Run `/omni admin rank sync`. The prefix needs a non-empty prefix, suffix or color on the rank; the player must not be moved into another team by a datapack.

**`/tpa ` shows `<args>` instead of `<player>`.**
Give the alias a usage: `/omni tools alias usage tpa <player>;here <player>;accept;deny;cancel`.

**A console alias step says "blocked for safety".**
The step runs a blocked command or uses `{args}` unsafely. Flag it only if every possible input is safe: `/omni tools alias flag <alias> <step> dangerous true`.

**Matches do not protect players / players die in arenas.**
The `death` hook is missing. See `/omni admin hooks`.

**Claims do not protect anything.**
The `block-break` and `use-block` hooks are missing, or the player is an operator.

**Chat looks unchanged.**
The `chat` hook is missing, or `smp.chatFormat` is `false` and the channel is global.

**My own board does not show.**
`/omni board status` says whether per-player packets are ready. Use `/board toggle`, or check that a profile matches (`/omni board list`). A `scope global` profile works without packets.

**A new alias does not appear in tab-complete.**
The name probably conflicts with an existing command. `/omni tools alias list` marks such aliases as "not registered".

**A menu slot has a price but nothing happens.**
A slot with commands runs the commands and ignores the price.

**A market purchase says my inventory and claim box are full.**
Free some space or `/claim` items. Nothing was charged.

**Players lost their inventory after a crash during a loadout match.**
It is saved in `pvp/vault.json` and returned when the player joins again.

**Where are my balances and listings?**
`config/omnitools/economy.json` and `market.json`. Backups are in `config/omnitools/backups`.

**Can I turn off a feature?**
Delete or restrict its alias. Operators can still use `/omni`.

---

## Building from source

The source uses intermediary names and is compiled directly against the Minecraft intermediary jar and the libraries of your Fabric setup. There is no Gradle project.

```
src/com/example/omni_tools/   Java sources
resources/                    fabric.mod.json, icons, LICENSE
patch/                        files that update.bat copies into src
manifest.mf                   jar manifest
update.bat                    Windows build script
tools/                        dump_mc.bat / dump_mc.ps1 / McDump.java (extract class and method signatures)
tests/                        unit tests of the pure logic (plain Java, no Minecraft needed)
```

On Windows, place the project folder next to `intermediary.jar` and the `libs` folder and run `update.bat`. It copies the patch files, checks the required classes, compiles with `javac --release 21`, and creates `omnitools-2.1.0.jar`. Compiler output is saved to `build_log.txt`.

Unit tests (no Minecraft jars needed): from the `tests` folder run
`javac -d out *.java` then `java -ea -cp out com.example.omni_tools.GuardTest` (also `LogicTest`, `Logic2Test`).

Because the code uses intermediary names, a build is tied to one Minecraft version. Use `tools/dump_mc.bat` to extract the signatures of a new version.

Source files by area:

| Area | Files |
| --- | --- |
| Entry and shared | `OmniTools`, `Mc`, `Store`, `Settings`, `SettingsReg`, `Ticker`, `Scheduler`, `Trust`, `Hooks`, `Events`, `Audit`, `MiniSchema`, `Perf` |
| Command layout and help | `OmniCommand`, `HelpData`, `HelpIndex`, `Migrate` |
| Aliases and safety | `AliasCmd`, `AliasUsage`, `Seq`, `Guard` |
| Kits, menus, GUI | `Kit`, `OmniMenu`, `Gui`, `Ui` |
| Economy and market | `Eco`, `Market`, `MarketCmd`, `MarketGui` |
| SMP | `SmpCmd`, `SmpData`, `Homes`, `Warps`, `Back`, `Tp`, `Land`, `ClaimIndex`, `Social`, `RankLogic` |
| Ranks | `Ranks`, `RankModel`, `LegacyText` |
| PvP and games | `PvpCmd`, `PvpCore`, `PvpStats`, `Elo`, `AntiFarm`, `DamageLog`, `CombatTag`, `PvpTier`, `Matchmaker`, `GameDef`, `GameAdmin`, `MinigameCmd`, `Session`, `Snapshot`, `SnapCodec`, `Region` |
| Scoreboard | `Board`, `BoardNet`, `BoardLogic`, `Placeholders` |
| Admin | `AdminCmd` |
| Teleport and duels | `TpaCmd`, `RtpCmd`, `DuelCmd` |

---

## Known limits

- **Not verified on a server yet.** The code is type-checked and its pure logic is unit-tested, but the full build and in-game behavior must be confirmed on your setup. Use the checklist in `docs/REPORT-2.1.0.md`.
- **Per-player scoreboard and tab list** depend on packet constructor signatures that are discovered at start (`/omni board status`). `scope global` boards do not depend on them.
- **Nicknames** change chat and boards only. The real name stays above the head (the rank prefix shows there).
- **Claim flags** exist only for PvP. Explosions, fire, fluids and mob griefing need engine hooks that are not available.
- **Arena rules:** hunger and natural regeneration cannot be changed per arena.
- **Snapshots** store block states, not chest contents.
- **Minigames:** `tnt_run` does not remove the floor yet, `skywars` does not fill chests, `bedwars` has no generators or shops, `parkour` has no checkpoints. They run with the generic rules.
- **Not included:** shop signs, item-based currency, per-item price mode for shop slots, a refund command, an automatic undo from the audit log.

---

## License

OmniTools is released under the MIT License. See the `LICENSE` file for the full text.

Copyright (c) 2026 @sonruhin
