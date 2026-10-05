# OmniTools

**Kits, virtual chest menus, custom commands, economy, player market, duels, TPA and random teleport for Fabric servers.**

OmniTools is an all-in-one server toolkit for Minecraft 1.21.11 (Fabric). It is built around one idea: server owners
should be able to create their own menus and commands in-game, without writing a single line of code, and players
should only ever see simple, safe commands.

Everything an operator does lives under one command, `/omni`. Everything a player does goes through short
**aliases** such as `/market`, `/sell`, `/tpa` and `/rtp`. Players never get access to `/omni` itself.

- Version: 1.1.0 (build 2)
- Minecraft: 1.21.11
- Loader: Fabric Loader 0.16.0 or newer
- Java: 21 or newer
- Requires: Fabric API
- License: MIT
- Author: @sonruhin

---

## Table of contents

1. [Feature overview](#feature-overview)
2. [Requirements and installation](#requirements-and-installation)
3. [How permissions work](#how-permissions-work)
4. [Quick start](#quick-start)
5. [Kits](#kits)
6. [Chest menus](#chest-menus)
7. [Server shop slots (prices)](#server-shop-slots-prices)
8. [Aliases (custom commands)](#aliases-custom-commands)
9. [Steps, timing and placeholders](#steps-timing-and-placeholders)
10. [Economy](#economy)
11. [Player market](#player-market)
12. [Duels](#duels)
13. [TPA](#tpa)
14. [RTP](#rtp)
15. [Default player commands](#default-player-commands)
16. [Full operator command reference](#full-operator-command-reference)
17. [Configuration files](#configuration-files)
18. [settings.json reference](#settingsjson-reference)
19. [Safety design](#safety-design)
20. [Troubleshooting and FAQ](#troubleshooting-and-faq)
21. [Building from source](#building-from-source)
22. [Not included](#not-included)
23. [License](#license)

---

## Feature overview

| Module | What it does |
| --- | --- |
| Kits | Save any inventory (hotbar, main inventory, armor, offhand) as a named kit and hand it to players. |
| Chest menus | Build clickable chest GUIs in-game. Each slot can show an item, run commands, or sell something. |
| Server shop slots | Give any menu slot a price. Players pay in-game money and receive a clean copy of the item. |
| Aliases | Create your own player-facing commands out of timed steps. Steps can run commands as the player, as the console, or run a menu slot. |
| Economy | A simple balance system with pay, balance and leaderboard commands. |
| Market | A full player-to-player marketplace with a GUI: browse, search, sort, categories, confirm screen, my listings, claim box, taxes, fees, expiry and multiple markets. |
| Duels | Request-based 1v1 duels in operator-defined arenas, with your own gear and normal death rules. |
| TPA | Teleport requests (go to a player, or bring a player to you) that expire after 60 seconds. |
| RTP | Random teleport to a safe spot with a configurable radius and a cooldown. |

Other highlights:

- Operators work in-game. No datapacks and no config editing are required to create kits, menus, aliases or arenas.
- Players only need a vanilla client. All menus use the normal chest screen.
- Chests, aliases, market data and balances are saved as readable JSON files.
- Item data is stored as full item JSON, so enchantments, custom names, lore and other components are kept.
- Players cannot take items out of menus. Menus hand out copies or run commands, they never move real items.

---

## Requirements and installation

1. Install Fabric Loader 0.16.0 or newer for Minecraft 1.21.11.
2. Install Fabric API.
3. Make sure the server uses Java 21 or newer.
4. Put `omnitools-1.1.0.jar` into the `mods` folder.
5. Start the server. The folder `config/omnitools/` is created automatically.

If you previously used the older `kitmod`, remove it. OmniTools declares a conflict with it so the two cannot be loaded together.
Existing kits are not deleted automatically, but they are stored in the OmniTools format, so import them by saving
them again with `/omni kit save`.

On the first start OmniTools creates ten default aliases (see [Default player commands](#default-player-commands)).
They are only created if `aliases.json` does not exist yet, so deleting an alias is permanent.

---

## How permissions work

OmniTools does not use a permissions mod. It uses three simple levels:

| Level | Who | What they can use |
| --- | --- | --- |
| Operator | Real operators and the server console | Everything under `/omni`, including every admin branch. |
| Player | Everyone else | Only the aliases that exist (by default `/tpa`, `/rtp`, `/duel`, `/market`, `/sell`, `/mylistings`, `/claim`, `/balance`, `/pay`, `/baltop`) and any aliases you create. |
| Trusted window | A player, but only while an alias step or a menu slot step is running | `/omni` works for that player so the step can do its job, but admin branches stay blocked. |

Key points:

- `/omni` is hidden from tab-complete and cannot be run by players directly.
- Each alias can be set to **everyone** or **op** with `/omni alias permission`.
- Aliases are real commands, so they show up in tab-complete for players who are allowed to use them.
- Player-mode steps cannot use target selectors (`@s`, `@a`, `@p`, `@r`, `@e`) when the player is not an operator. Use `{player}` instead.

---

## Quick start

A five-minute tour that touches most of the mod.

**1. Save a kit from your own inventory**

```
/omni kit save starter
```

**2. Build a menu from it**

```
/omni chest fromkit large starter starter
/omni chest open large starter
```

The first `starter` after `large` is the chest name, the last one is the kit it was built from.

**3. Make the menu open with a short player command**

```
/omni alias add kits menu 1 0 omni chest open large starter
```

Now any player can type `/kits` and see the menu. Clicking an item gives them a copy.

**4. Put a price on a slot**

```
/omni chest price large starter set 1 250
```

The first slot now sells for 250 and is marked with price lore.

**5. Give someone money and try the market**

```
/omni eco give Steve 1000
```

Steve can now run `/market`, hold an item and run `/sell 500`.

---

## Kits

A kit is a saved snapshot of an inventory: hotbar, main inventory, armor slots and the offhand. Kits are stored in `kits.json`.

| Command | Description |
| --- | --- |
| `/omni kit save <name>` | Save your own inventory as a kit. |
| `/omni kit saveplayer <name> <player>` | Save another online player's inventory as a kit. |
| `/omni kit remove <name>` | Delete a kit. |
| `/omni kit rename <name> <new>` | Rename a kit. |
| `/omni kit copy <name> <new>` | Duplicate a kit. |
| `/omni kit list` | List all kits. |
| `/omni kit info <name>` | Show what is inside a kit. |
| `/omni kit give <kit> <players> [gearPercent]` | Clear the target players' inventories, then give the kit. |
| `/omni kit add <kit> <players>` | Add the kit's items to free slots without clearing anything. |
| `/omni kit giveall <kit> [playerPercent] [gearPercent]` | Give a kit to everyone online, or to a random percentage of them. |
| `/omni kit randomall <kit percent kit percent ...>` | Give every online player a random kit, using the percentages as weights. |
| `/omni kit clear <players>` | Clear the target players' inventories. |

Notes:

- `<players>` accepts normal target selectors when you run the command as an operator.
- `gearPercent` lets you roll the armor and tool durability of the kit to a percentage, which is useful for
  events where kits should arrive pre-damaged.
- Tab-complete suggests existing kit names everywhere a kit name is expected.
- Items that are full of data (custom names, enchantments, lore, shulker contents) are stored as complete item JSON.

---

## Chest menus

A chest menu is a virtual chest that only exists as data. Operators design it, players click it.
There are two sizes: **large** (6 rows, 54 slots) and **small** (3 rows, 27 slots). Names are unique per size,
so `large:shop` and `small:shop` are different menus.

### Editing a menu

```
/omni chest editchest <large|small> <name>
```

This opens the menu in edit mode. If the menu does not exist yet, it is created.

| Action in edit mode | Result |
| --- | --- |
| Place an item in a slot | The slot now shows that item. |
| Left-click an item | Pick up the whole stack. |
| Right-click an item | Pick up one item. |
| Shift-click an item | Receive a copy in your own inventory. |
| Shift-click from your inventory | Move the item into the first free slot. |
| Press `Q` over an item | Delete it from the menu. |
| Number key or `F` | Swap with or copy to your hotbar / offhand. |
| Close the screen | **Save.** Nothing else is needed. |

When you close the screen, any price that points to an empty slot is dropped automatically.

### Using a menu as a player

Menus run in read-only mode for players:

- Clicking an item gives a **copy**. The item stays in the menu and nothing can be put in.
- Drag, double-click collect and shift-click from the inventory are blocked so items can never leak into or out of the menu.
- If a slot has commands, a left or shift click **runs the commands** instead of giving a copy.
- If a slot has a price, a left or shift click **buys** the item (see below).

### Menu commands

| Command | Description |
| --- | --- |
| `/omni chest open <size> <name> [players]` | Open a menu for yourself or for other players. If no chest has that name, a kit with that name is shown as a menu. |
| `/omni chest run <size> <name> <slot> [players]` | Run a slot's steps directly, without opening the menu. |
| `/omni chest editchest <size> <name>` | Edit a menu in-game. |
| `/omni chest fromkit <size> <name> <kit>` | Build or overwrite a menu from a kit. Items that do not fit are reported. |
| `/omni chest list` | List all menus. |
| `/omni chest info <size> <name>` | Show item count, number of steps and priced slots. |
| `/omni chest delete <size> <name>` | Delete a menu. |
| `/omni chest copy <size> <name> <new>` | Copy a menu. |
| `/omni chest rename <size> <name> <new>` | Rename a menu. Aliases and other menus that point at it are updated automatically. |

### Commands on a slot

Any slot that contains an item can run a sequence of steps when a player clicks it.

| Command | Description |
| --- | --- |
| `/omni chest cmd <size> <name> add <slot> <step> <number> <time> <command>` | Add a step that runs as the player. |
| `/omni chest cmd <size> <name> addconsole <slot> <step> <number> <time> <command>` | Add a step that runs as the console. |
| `/omni chest cmd <size> <name> remove <slot> <step>` | Remove a step. |
| `/omni chest cmd <size> <name> settime <slot> <step> <time>` | Change the delay of a step. |
| `/omni chest cmd <size> <name> clear <slot>` | Remove all steps from a slot. |
| `/omni chest cmd <size> <name> list <slot>` | Show the steps of one slot. |
| `/omni chest cmd <size> <name> listall` | Show every slot that has steps. |
| `/omni chest cmd <size> <name> close <slot> <true\|false>` | Choose whether clicking the slot also closes the menu. |

Slots are numbered from 1. See [Steps, timing and placeholders](#steps-timing-and-placeholders) for the meaning of
`<step>`, `<number>` and `<time>`.

---

## Server shop slots (prices)

Any menu slot can be turned into a fixed-price shop slot. This is the "server shop": money is taken from the player
and removed from the economy. It is not paid to anyone.

| Command | Description |
| --- | --- |
| `/omni chest price <size> <name> set <slot> <price>` | Set a price. |
| `/omni chest price <size> <name> clear <slot>` | Remove a price. |
| `/omni chest price <size> <name> list` | Show all prices in a menu. |

How it behaves:

- Priced slots automatically get lore that shows the price and "Click to buy". You do not edit the item yourself.
- A left click or shift click buys. Right click and other click types do nothing, so nothing is ever copied for free.
- The buyer receives a **clean copy** of the original item (without the price lore).
- If the buyer's inventory is full, the rest goes into their claim box and they are told so.
- If the slot also has commands, **commands win** and the price is ignored. OmniTools warns you when you set up that combination.
- If the player does not have enough money, they get a message showing the price and their balance.

---

## Aliases (custom commands)

An alias is a real root command created by an operator. Each alias is a list of ordered steps. Players run the alias
with a single short command and the steps run in order, each after its own delay.

Typical uses: a `/spawn` that teleports after 3 seconds, a `/kits` that opens a menu, a `/vote` that tells players where to vote,
a `/shop` that runs a console command with the player's name filled in.

### Alias commands

| Command | Description |
| --- | --- |
| `/omni alias add <alias> <step> <number> <time> <command>` | Add a step that runs as the player. Creates the alias if it does not exist. |
| `/omni alias addconsole <alias> <step> <number> <time> <command>` | Add a step that runs as the console. |
| `/omni alias addslot <alias> <step> <number> <time> <size> <chest> <slot>` | Add a step that runs the steps of a menu slot. |
| `/omni alias delete command <alias> <step>` | Remove one step. |
| `/omni alias delete alias <alias>` | Delete the whole alias. |
| `/omni alias list` | List all aliases. |
| `/omni alias info <alias>` | Show every step of an alias. |
| `/omni alias run <alias> [players]` | Run an alias for yourself or for other players. |
| `/omni alias settime <alias> <step> <time>` | Change a step's delay. |
| `/omni alias cooldown <alias> <time>` | Set a per-player cooldown. Operators ignore it. |
| `/omni alias permission <alias> <everyone\|op>` | Choose who may use the alias. |

### Rules

- Alias names use lowercase letters, numbers, `_` and `-`, up to 32 characters.
- The name `omni` is reserved, and an alias cannot use the name of an existing command.
- An alias cannot call itself.
- New aliases are registered live and tab-complete is refreshed for everyone online. No restart is required.
- A deleted alias simply stops working. Its command node stays until the next restart, but nobody can use it.
- Alias steps that run a menu slot read the slot's steps **when the alias runs**, so you can edit the slot later without touching the alias.

---

## Steps, timing and placeholders

Both aliases and menu slots are made of steps. Every step has:

| Part | Meaning |
| --- | --- |
| `step` | A name for the step: letters, numbers and `_ . + -`, up to 32 characters. Unique inside its alias or slot. |
| `number` | Its position in the sequence. Adding a step at a position that is already taken pushes the later steps down. A number higher than the current count adds the step at the end. |
| `time` | The delay before the step runs, counted from the previous step. |
| `command` | The command to run, without the leading `/`. |

### Time format

| Example | Meaning |
| --- | --- |
| `0` | No delay |
| `500ms` | 500 milliseconds |
| `5sec` | 5 seconds |
| `2min` | 2 minutes |

Short forms such as `s`, `secs`, `m` and `mins` also work. A bare number other than `0` is rejected, so there is
never any doubt about the unit. The maximum is 60 minutes.

### Placeholders

Placeholders are replaced when the step runs:

| Placeholder | Replaced with |
| --- | --- |
| `{player}` | The player's name |
| `{uuid}` | The player's UUID |
| `{x}` `{y}` `{z}` | The player's block position |
| `{world}` | The player's dimension id |
| `{args}` | Whatever the player typed after the alias |

### About `{args}`

`{args}` lets players pass a value, for example `/tpa Steve` becomes `omni tpa Steve`. To keep this safe:

- Arguments are limited to 5 words and 32 characters per word.
- Only letters, numbers and `_ - . :` are allowed. Selectors, quotes, braces, `~`, `^` and similar characters are rejected.
- If an operator uses `{args}` in a console step, or in a step that runs `/omni`, OmniTools shows a warning because players
  choose the words that go there.

### Examples

```
# A kit menu command
/omni alias add kits menu 1 0 omni chest open large starter

# Teleport to spawn after a 3 second countdown message
/omni alias addconsole spawn msg 1 0 tellraw {player} {"text":"Teleporting in 3 seconds...","color":"yellow"}
/omni alias addconsole spawn tp 2 3sec tp {player} 0 100 0

# An alias that opens a menu and later runs a slot of another menu
/omni alias addslot daily open 1 0 large rewards 5

# A command with an argument
/omni alias add duel duel 1 0 omni duel {args}
```

---

## Economy

A small and predictable balance system. Balances are whole numbers and are stored in `economy.json`.

- New players start with a configurable starting balance.
- Balances have a configurable maximum.
- The currency symbol is configurable.
- A player's account is created when they join, so operators can pay offline players who have joined before.

### Player commands (through aliases)

| Alias | Command |
| --- | --- |
| `/balance [player]` | Show your balance or someone else's. |
| `/pay <player> <amount>` | Send money to another player. You cannot pay yourself, and the receiver cannot go over the maximum balance. |
| `/baltop` | Show the richest players. |

Payments to offline players leave a notice that is shown when they join.

### Operator commands

| Command | Description |
| --- | --- |
| `/omni eco give <player> <amount>` | Add money. |
| `/omni eco take <player> <amount>` | Remove money. Fails if the player does not have enough. |
| `/omni eco set <player> <amount>` | Set an exact balance. |
| `/omni eco reset <player>` | Reset to the starting balance. |
| `/omni eco balance [player]` | Show a balance. |
| `/omni eco pay <player> <amount>` | Same as `/pay`. |
| `/omni eco top [page]` | Leaderboard, ten players per page. |

---

## Player market

The market is a complete player-to-player marketplace with a chest GUI. It works with items of any kind, including
enchanted and renamed ones, and keeps every detail of the item.

### Selling

There are two ways to sell:

**With a command (fast):**

```
/sell <price> [amount]
```

Sells what you are holding. If you leave out the amount, the whole stack is sold.

**With the GUI:**

```
/market      then click "Sell an item"
```

1. Click an item in your own inventory (the bottom half of the screen).
2. Set the price with the +/- buttons (1, 10, 100, 1000), or halve or double it.
3. Choose how many items to sell.
4. Check the summary: item, price, listing fee, sales tax and what you will receive.
5. Click **List for sale**.

If the item has been sold before, the starting price is suggested from the average sold price.

### Buying

`/market` opens the browser. Click a listing to buy it. If purchase confirmation is on, a confirm screen shows the item,
the price and your balance first. The buyer receives the exact original item. If their inventory is full, the item goes to
their claim box.

### The browser

- 45 listings per page with page buttons.
- **Category filter:** All items, Blocks, Tools, Weapons, Armor, Food, Potions, Enchanted books, Resources, Miscellaneous.
- **Sorting:** Newest first, Oldest first, Price low to high, Price high to low, Name A-Z. Left-click cycles forward, right-click cycles backward.
- Each tile shows the price, the seller, time left, the average sold price per item and the listing id.
- Right-click the info button to clear all filters.
- Quick buttons for selling, your listings, your claim box and refreshing.

### Searching and filtering by command

| Command | Description |
| --- | --- |
| `/omni market open [market]` | Open the main market or another market. |
| `/omni market search <text>` | Search listings by item name or data (for example an enchantment). |
| `/omni market seller <name>` | Show only one seller's listings. |
| `/omni market history` | Your last ten sales. |

### Managing your listings

`/mylistings` opens your listings:

| Click | Action |
| --- | --- |
| Left-click | Cancel the listing. The item comes back to you. |
| Right-click | Change the price. |
| Shift-click | Renew the listing for the full duration again. |

Command versions exist too: `/omni market price <id> <price>` and `/omni market cancel <id>`.

### Claim box

`/claim` opens the claim box. Anything that could not be delivered goes here instead of being lost or dropped:

- Items from expired listings
- Items bought while the inventory was full
- Items from listings an operator removed
- Overflow from shop slot purchases

Click an item to claim it, or use **Take all**. If your inventory is full the item simply stays in the box.

### Money flow

- The listing price is paid by the buyer.
- The seller receives the price minus the **sales tax** (a percentage). Taxes are removed from the economy.
- An optional flat **listing fee** is charged when the item is listed.
- Sellers are credited immediately, even when offline, and get a notice when they join.
- Listings expire after a configurable number of hours. Expired items move to the claim box automatically.

### Markets

There is one main market by default. Operators can create extra markets (for example a "black market" or an event market)
and open them with `/omni market open <name>` or through an alias. Listings belong to the market they were created in.

### Operator tools

| Command | Description |
| --- | --- |
| `/omni market list [player]` | List active listings, optionally of one player. |
| `/omni market remove <id>` | Remove any listing. The item goes to the owner's claim box. |
| `/omni market log [n]` | Show the last sales and actions from the market log. |
| `/omni market create <name>` | Create a market. |
| `/omni market delete <name>` | Delete an empty market. The main market cannot be deleted. |
| `/omni market blacklist add\|remove\|list <item>` | Forbid items from being sold. Wildcards work, for example `minecraft:*_command_block` or `modid:*`. |
| `/omni market set <key> <value>` | Change a setting live. Keys: `tax`, `fee`, `minprice`, `maxprice`, `maxlistings`, `duration`, `confirm`, `currency`. |
| `/omni market settings` | Show current settings. |
| `/omni market limit <player> <n\|reset>` | Give one player a custom listing limit. |
| `/omni market sweep` | Move all expired listings to claim boxes immediately. |

### Safety in the market

- All market commands always act as the player who ran them. Nobody can act on behalf of another player.
- The item is taken from the player at the moment of listing, using the live inventory, and the listing is checked
  again at the moment of purchase. An item that moved or changed cannot be listed by accident.
- A purchase and the payment are completed together. If the money or the item cannot be delivered, nothing is lost:
  undelivered items go to the claim box.
- Every sale and action is written to `market-log.jsonl`.

---

## Duels

Duels are request-based 1v1 fights in arenas that operators define.

### For players

```
/duel request <player> <arena>
/duel accept
/duel deny
/duel forfeit
```

- A request expires after 60 seconds.
- Both players fight with **their own gear**, and normal death rules apply. OmniTools does not touch inventories.
- Both players are teleported to the arena's two spawn points when the duel starts.
- `/duel forfeit` ends the duel immediately and the other player wins.
- A duel also ends if a player dies or leaves.

### For operators

```
/omni duel arena set <name> <1|2>
/omni duel arena remove <name>
/omni duel arena list
```

Stand where you want spawn point 1, run `/omni duel arena set <name> 1`, then move to spawn point 2 and run it again with `2`.
An arena needs both points before it can be used.

---

## TPA

Players can ask to teleport to each other.

```
/tpa <player>          go to the player
/tpa here <player>     bring the player to you
/tpa accept
/tpa deny
/tpa cancel
```

- A request expires after 60 seconds.
- Only the player who received the request can accept or deny it.
- The sender can cancel a pending request at any time.

---

## RTP

Random teleport to a safe location.

```
/rtp
/rtp <world>
/rtp <world> <minRadius> <maxRadius>
```

- Default radius is 500 to 5000 blocks around the world center.
- OmniTools tries up to 40 random spots and picks one that is safe to stand on.
- A 30 second cooldown applies per player.
- The world argument is tab-completed with the worlds of the server.

---

## Default player commands

These aliases are created the first time the mod starts. You can change, remove or rename them.

| Alias | Runs |
| --- | --- |
| `/tpa` | `omni tpa {args}` |
| `/rtp` | `omni rtp {args}` |
| `/duel` | `omni duel {args}` |
| `/market` | `omni market {args}` |
| `/sell` | `omni market sell {args}` |
| `/mylistings` | `omni market mine` |
| `/claim` | `omni market claim` |
| `/balance` | `omni eco balance {args}` |
| `/pay` | `omni eco pay {args}` |
| `/baltop` | `omni eco top` |

The names shown in help messages come from the `commands` block of `settings.json`, so if you rename an alias the help text can follow.

---

## Full operator command reference

All of these require operator permission (or the console).

### General

```
/omni
/omni help
/omni about
/omni reload
```

`/omni reload` re-reads the config files, registers any new aliases and refreshes tab-complete.

### Kits

```
/omni kit save <name>
/omni kit saveplayer <name> <player>
/omni kit remove <name>
/omni kit rename <name> <new>
/omni kit copy <name> <new>
/omni kit list
/omni kit info <name>
/omni kit give <kit> <targets> [gear]
/omni kit add <kit> <targets>
/omni kit giveall <kit> [players] [gear]
/omni kit randomall <kit percent kit percent ...>
/omni kit clear <targets>
```

### Chest menus

```
/omni chest open <large|small> <name> [targets]
/omni chest run <large|small> <name> <slot> [targets]
/omni chest editchest <large|small> <name>
/omni chest fromkit <large|small> <name> <kit>
/omni chest list
/omni chest info <large|small> <name>
/omni chest delete <large|small> <name>
/omni chest copy <large|small> <name> <new>
/omni chest rename <large|small> <name> <new>
/omni chest cmd <size> <name> add <slot> <step> <number> <time> <command>
/omni chest cmd <size> <name> addconsole <slot> <step> <number> <time> <command>
/omni chest cmd <size> <name> remove <slot> <step>
/omni chest cmd <size> <name> settime <slot> <step> <time>
/omni chest cmd <size> <name> clear <slot>
/omni chest cmd <size> <name> list <slot>
/omni chest cmd <size> <name> listall
/omni chest cmd <size> <name> close <slot> <true|false>
/omni chest price <size> <name> set <slot> <price>
/omni chest price <size> <name> clear <slot>
/omni chest price <size> <name> list
```

### Aliases

```
/omni alias add <alias> <step> <number> <time> <command>
/omni alias addconsole <alias> <step> <number> <time> <command>
/omni alias addslot <alias> <step> <number> <time> <size> <chest> <slot>
/omni alias delete command <alias> <step>
/omni alias delete alias <alias>
/omni alias list
/omni alias info <alias>
/omni alias run <alias> [targets]
/omni alias settime <alias> <step> <time>
/omni alias cooldown <alias> <time>
/omni alias permission <alias> <everyone|op>
```

### Economy

```
/omni eco balance [player]
/omni eco pay <player> <amount>
/omni eco top [page]
/omni eco give <player> <amount>
/omni eco take <player> <amount>
/omni eco set <player> <amount>
/omni eco reset <player>
```

### Market

```
/omni market
/omni market open [market]
/omni market search <text>
/omni market seller <name>
/omni market sell
/omni market sell <price> [amount]
/omni market mine
/omni market claim
/omni market history
/omni market price <id> <price>
/omni market cancel <id>
/omni market list [player]
/omni market remove <id>
/omni market log [n]
/omni market create <name>
/omni market delete <name>
/omni market blacklist add <item>
/omni market blacklist remove <item>
/omni market blacklist list
/omni market set <key> <value>
/omni market settings
/omni market limit <player> <n|reset>
/omni market sweep
```

### Duels, TPA and RTP

```
/omni duel request <player> <arena>
/omni duel accept
/omni duel deny
/omni duel forfeit
/omni duel arena set <name> <1|2>
/omni duel arena remove <name>
/omni duel arena list

/omni tpa <player>
/omni tpa here <player>
/omni tpa accept
/omni tpa deny
/omni tpa cancel

/omni rtp [world] [minRadius] [maxRadius]
```

---

## Configuration files

All data lives in `config/omnitools/`. Files are plain JSON (the market log is JSON Lines) and can be edited while the
server is stopped. Run `/omni reload` after editing while the server is running.

| File | Contents |
| --- | --- |
| `kits.json` | Saved kits |
| `chests.json` | Menus: items, slot steps and prices |
| `aliases.json` | Aliases and their steps, cooldowns and permissions |
| `arenas.json` | Duel arenas |
| `settings.json` | Command names, economy and market settings, blacklist |
| `economy.json` | Player balances |
| `market.json` | Active listings, claim boxes, markets, limits and join notices |
| `market-log.jsonl` | Sales and market actions, one JSON object per line |

Safety measures for your data:

- Writes are made safely so a crash during saving does not leave a half-written file.
- A `.bak` copy of the previous version is kept next to each file.
- If a file cannot be read, OmniTools logs the problem and does not overwrite it with empty data.
- Older menu formats (single command, entries list, steps list) are converted automatically when they are loaded.

---

## settings.json reference

`settings.json` is created on first start. The structure looks like this:

```json
{
  "commands": {
    "tpa": "tpa",
    "rtp": "rtp",
    "duel": "duel",
    "market": "market",
    "sell": "sell",
    "mylistings": "mylistings",
    "claim": "claim",
    "balance": "balance",
    "pay": "pay",
    "baltop": "baltop"
  },
  "economy": {
    "currencySymbol": "$",
    "startingBalance": 100,
    "maxBalance": 1000000000000
  },
  "market": {
    "listingFee": 0,
    "salesTaxPercent": 5,
    "minPrice": 1,
    "maxPrice": 1000000000,
    "maxListingsPerPlayer": 10,
    "listingDurationHours": 168,
    "confirmPurchase": true
  },
  "blacklist": []
}
```

| Setting | Default | Meaning |
| --- | --- | --- |
| `commands.*` | the key itself | The command name shown in help and messages for that feature. |
| `economy.currencySymbol` | `$` | Symbol put in front of amounts (1 to 4 characters). |
| `economy.startingBalance` | 100 | Balance of a new account. |
| `economy.maxBalance` | 1,000,000,000,000 | No account can hold more than this. |
| `market.listingFee` | 0 | Flat fee charged when listing an item. |
| `market.salesTaxPercent` | 5 | Percent of the price removed from the seller's payout. |
| `market.minPrice` | 1 | Lowest allowed listing price. |
| `market.maxPrice` | 1,000,000,000 | Highest allowed listing price. |
| `market.maxListingsPerPlayer` | 10 | Active listings per player, unless overridden with `/omni market limit`. |
| `market.listingDurationHours` | 168 | Hours before a listing expires (168 is one week). |
| `market.confirmPurchase` | true | Show a confirm screen before a purchase. |
| `blacklist` | empty | Item ids that cannot be sold. `*` wildcards are allowed. |

Most market values can also be changed live with `/omni market set <key> <value>`, and the file is updated automatically.

---

## Safety design

OmniTools is built so that giving players commands does not give them power over the server.

- **Operators only for `/omni`.** Players cannot run it, see it, or tab-complete it.
- **Trusted window.** When an alias or menu slot runs a step, `/omni` is allowed for that player only for the duration of the step, and only for non-admin branches.
- **No selectors for players.** Player-mode steps reject `@s`, `@a`, `@p`, `@r` and `@e` when the player is not an operator.
- **Strict `{args}`.** At most 5 words, a limited character set, no quotes, braces or selectors.
- **Console steps are operator-defined.** Console steps always use exactly the command you wrote, with only the documented placeholders replaced.
- **Read-only menus.** Players cannot move real items in or out of a menu. Drags, double-click collect and shift-click transfers are blocked.
- **No free copies from shop slots.** A priced slot only reacts to a plain left click or shift click, and only after payment.
- **Safe market handling.** Items are re-checked at listing and purchase time, and undelivered items go to the claim box instead of disappearing.
- **Server-side only.** Players need nothing installed on their client.

---

## Troubleshooting and FAQ

**Players can run `/omni`.**
They cannot. If a player seems able to, check that they are not an operator and that no other mod grants command permissions.

**A new alias does not appear in tab-complete.**
OmniTools refreshes the command tree automatically for everyone online. If it does not show up, the name probably conflicts with an
existing command. `/omni alias list` marks conflicting names as "not registered". Pick a different name.

**My alias says "Invalid arguments".**
Arguments passed to an alias are limited to 5 words and a safe character set. Selectors, quotes and braces are not allowed.

**`@s` does not work in a player step.**
That is intentional. Use `{player}` instead. Operators running the alias are not restricted, but players are.

**A menu slot has a price but nothing happens when clicked.**
Check that the slot has an item, and that it has no commands. A slot with commands runs the commands and ignores the price.
Use `/omni chest cmd <size> <name> list <slot>` to look.

**A market purchase says my inventory is full.**
The item is not lost. It is placed in your claim box, which you can open with `/claim`.

**The server shows `could not hook into: ...` in the log.**
OmniTools registers its Fabric API events with a fallback so that one missing hook only logs a message instead of
stopping the server. Make sure Fabric API is installed and matches your Minecraft version.

**Kits give an item with strange data or fail to load an item.**
Items are stored as full JSON. If an item from a removed mod cannot be decoded, it is skipped and a warning is written to
`logs/latest.log`. The rest of the kit still loads.

**Where are my balances and listings stored?**
In `config/omnitools/economy.json` and `config/omnitools/market.json`. Back them up before manual edits.

**Can I turn off a feature?**
Delete or restrict its alias. For example, delete the `duel` alias and players can no longer start duels. Operators can still use `/omni`.

---

## Building from source

The source uses intermediary names, so it is compiled directly against the Minecraft intermediary jar and the libraries that
ship with your Fabric setup. There is no Gradle project in this repository.

Layout:

```
src/com/example/omni_tools/   Java sources
resources/                    fabric.mod.json, icons, LICENSE, build id file
manifest.mf                   jar manifest
update.bat                    Windows build script
```

On Windows, place the project folder next to `intermediary.jar` and the `libs` folder, then run `update.bat`.
It copies the patch files, checks that every required class is present, compiles with `javac --release 21`, copies the resources
and creates `omnitools-1.1.0.jar`. If compilation fails, the compiler output is saved to `build_log.txt`.

Source files:

| File | Purpose |
| --- | --- |
| `OmniTools.java` | Mod entry point and Fabric event hooks |
| `OmniCommand.java` | The `/omni` command tree, kit and chest commands, shared helpers |
| `AliasCmd.java` | Alias management and registration |
| `MarketCmd.java` | Market and economy commands |
| `MarketGui.java` | Market screens |
| `Market.java` | Market logic, listings, claims, history |
| `Eco.java` | Balances |
| `OmniMenu.java` | Virtual chest menu, clicks and saving |
| `Gui.java`, `Ui.java` | Generic button-based screens and item tiles |
| `Seq.java` | Timed step sequences |
| `Store.java` | Loading and saving of JSON data |
| `Settings.java` | `settings.json` |
| `Trust.java` | The trusted window |
| `Kit.java`, `Ticker.java`, `Mc.java` | Kit helpers, scheduling and Minecraft helpers |
| `DuelCmd.java`, `TpaCmd.java`, `RtpCmd.java` | Duels, teleport requests, random teleport |

Because the code uses intermediary names, a build is tied to one Minecraft version. A new Minecraft version needs
the names to be checked again.

---

## Not included

These features are not part of OmniTools 1.1.0:

- Scoreboard display of balances
- Item-based currency
- Per-item price mode for shop slots
- A refund command

---

## License

OmniTools is released under the MIT License. See the `LICENSE` file for the full text.

Copyright (c) 2026 @sonruhin
