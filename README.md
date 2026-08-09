# Saurona Server-Side Mod

Server-side Fabric utilities for the Saurona Middle-earth server.

This mod is designed to run on the dedicated server only. It provides chat tools, staff roles, prefixes, faction homes, economy, backpacks, playstyle modes, hunts, timed rewards, NPC traders, collectibles, music zones, Discord sync hooks, and staff moderation utilities.

## Quick Start

Build the jar:

```text
./gradlew build
```

On Windows:

```text
.\gradlew.bat build
```

The compiled jar is created at:

```text
build/libs/server-side-1.0.0.jar
```

Put the jar in the server `mods` folder, then restart the server once.

Most data files are written in the world root after the server starts. The main config is:

```text
world/server-side-config.json
```

Reload config files in game:

```text
/ss reload
/serverside reload
```

Reload requires OP level 4.

## Permissions

The mod uses vanilla permission levels plus its own staff-role file.

- OP level 4: full admin, config reload, staff role management, view distance, owner-level staff tools.
- OP level 3: admin-level moderation and gamemode access.
- OP level 2: lower staff utility access, teleport, fly, staff chat, collectibles.
- Staff roles can grant specific access without giving full OP.

Staff role permission behavior:

| Role | OP | OP level | Moderate | Fly | Teleport | Gamemode |
| --- | --- | ---: | --- | --- | --- | --- |
| Owner | yes | 4 | yes | yes | yes | yes |
| Manager | yes | 4 | yes | yes | yes | yes |
| Admin | yes | 3 | yes | yes | yes | no |
| Mod | no | 0 | yes | yes | yes | no |
| Advisor | no | 0 | no | no | no | no |
| Guide | no | 0 | no | no | no | no |

The staff command can be run by OP level 4 or by UUIDs listed in:

```json
"staffCommandOwnerUuids": []
```

## Config And Data Files

Main config:

```text
world/server-side-config.json
```

Feature data files:

```text
world/server-side-bank.json
world/server-side-backpacks.json
world/server-side-factions.json
world/server-side-staff-roles.json
world/server-side-playstyles.json
world/server-side-timed-rewards.json
world/server-side-mail.json
world/server-side-stats.json
world/server-side-traders.json
world/server-side-alts.json
```

Other feature folders/files:

```text
config/server-side/music_zones.json
world/server-side-collectibles/
```

Use `/ss reload` after editing config/data where supported. Restarting is safest after adding new jar fields or making large manual edits.

## Prefix System

Prefixes are configured in `server-side-config.json`.

Prefix display order in chat:

```text
[H]/[M] playstyle, staff/faction/wanderer, faction leader [L], player name
```

The tab list shows only the main prefix: staff role, faction, or Wanderer. It does not show `[H]`, `[M]`, or `[L]`.

Staff are sorted near the top of tab when `customTabSortStaffFirst` is enabled.

Current prefix defaults use gray brackets with colored text:

```text
&8[&#5E4B8BOwner&8]
&8[&#91B2D1Wanderer&8]
&8[&#6F8FAFH&8]
&8[&#C47A45M&8]
```

Faction prefix defaults:

```text
brigand            &8[&#8E6B52Brigand&8]
gondor             &8[&#E1E6EFGondor&8]
rohan              &8[&#7FA86BRohan&8]
mordor             &8[&#9B3D35Mordor&8]
isengard           &8[&#6C7480Isengard&8]
dale               &8[&#C99A4ADale&8]
shire              &8[&#92B96FShire&8]
goblin_town        &8[&#5D8A54Goblin Town&8]
hobgoblin_tribes   &8[&#B85F4AHobgoblin&8]
longbeards         &8[&#B68A3DLongbeards&8]
lothlorien         &8[&#D6C765Lorien&8]
moria              &8[&#7D6A88Moria&8]
wild_goblins       &8[&#4F7F67Wild Goblins&8]
woodland_realm     &8[&#6F9A7AWoodland&8]
```

Useful command:

```text
/prefixcheck <player>
```

Shows that player's chat and tab prefixes. Requires moderation access.

## Staff Roles

Staff roles set prefixes, grant server-side permissions, and OP the player where appropriate.

Commands:

```text
/staff add <player> <role>
/staff remove <player>
```

Roles:

```text
owner
manager
admin
mod
advisor
guide
```

Data file:

```text
world/server-side-staff-roles.json
```

Removing a staff role also de-ops the player.

## Haven And Mortal Playstyles

Haven is the default mode for all players.

Haven:

- PvP against players is blocked.
- Inventory is kept on death.
- Player hunts are blocked.
- Prefix: `[H]`.

Mortal:

- PvP is allowed only between Mortal players.
- Inventory drops normally on death.
- Player hunts are allowed.
- Prefix: `[M]`.
- Players cannot switch themselves back to Haven.

Commands:

```text
/haven
/playstyle
/mortal
/mortal confirm
/playstyle set <player> <haven|mortal>
```

`/mortal` shows a warning. `/mortal confirm` permanently opts the player into Mortal until staff changes it.

`/playstyle set` requires moderation access.

Data file:

```text
world/server-side-playstyles.json
```

Discord sync is called when a linked player switches mode.

## Player Hunts

Hunts are Mortal-only PvP events.

Commands:

```text
/hunt start <player>
/hunt aid <player>
/hunt end
```

Behavior:

- Hunter and target must both be Mortal.
- A player can only be in one hunt at a time.
- The hunt has a preparation timer before becoming active.
- Active hunt participants are prevented from teleporting far away or changing dimension.
- The hunt ends when a participant dies, runs out of time, disconnects from participation, or uses `/hunt end`.

Config:

```json
"huntPreparationSeconds": 60,
"huntDurationMinutes": 30,
"huntCooldownMinutes": 60,
"huntTeleportBlockSeconds": 30
```

Hunt state is in memory and is not persisted across server restart.

## Factions And Faction Homes

Faction data is stored by faction, not by player. That means leadership changes do not move or delete faction homes.

Data file:

```text
world/server-side-factions.json
```

Player commands:

```text
/faction home
/faction sethome
/faction homeinfo
/faction clearhome
/faction info <faction>
/faction list
```

Faction leader commands:

```text
/faction sethome
/faction clearhome
```

Staff faction commands:

```text
/faction leader set <faction> <player>
/faction leader remove <faction>
/faction set <player> <faction>
/faction remove <player>
```

Admin faction home commands:

```text
/factionadmin sethome <faction>
/factionadmin clearhome <faction>
/factionadmin homeinfo <faction>
```

Teleport behavior:

- `/faction home` automatically uses the player's current faction.
- No faction means no faction home access.
- Leaders can set and clear their own faction home.
- Staff can set, clear, and inspect any faction home.
- Teleport warmup, cooldown, cancel-on-move, cancel-on-damage, sounds, particles, and cross-dimension behavior are configurable.
- Defaults mirror TPX-style `/home`: 3 second warmup, 30 second cooldown, cancel on movement/damage, cross-dimension allowed.

Important config:

```json
"factionHomeWarmupSeconds": 3,
"factionHomeCooldownSeconds": 30,
"factionHomeMoveCooldownHours": 24,
"factionHomeCancelOnMove": true,
"factionHomeCancelOnDamage": true,
"factionHomeAllowCrossDimension": true,
"factionHomeCooldownBypassEnabled": true,
"factionHomeCooldownBypassPermissionLevel": 2,
"factionHomeWarmupBypassEnabled": true,
"factionHomeWarmupBypassPermissionLevel": 2,
"factionHomeXpCostsEnabled": false,
"factionHomeXpCostPoints": 25
```

Sounds and particles are also configurable in the main config.

## Economy And Bank

The bank stores virtual coin balances and can convert configured coin items into virtual money.

Data file:

```text
world/server-side-bank.json
```

Commands:

```text
/bank
/balance
/balance <player>
/deposit <amount>
/deposit all
/withdraw <amount>
/transfer <player> <amount>
/bank top
/baltop
/bank autodeposit
/autodeposit
```

Notes:

- `/balance <player>` requires moderation access.
- `/deposit` removes configured coin items from the player's inventory.
- `/withdraw` gives coin items back, using the largest configured coin values first.
- `/transfer` transfers virtual bank balance.
- Auto-deposit only affects picked-up coin item entities while enabled.

Default coin values:

```text
middle-earth:copper_coin        1
middle-earth:copper_coin_pile   4
middle-earth:silver_coin        10
middle-earth:silver_coin_pile   40
middle-earth:gold_coin          100
middle-earth:gold_coin_pile     400
```

## Backpacks

Backpacks are server-side virtual inventories.

Data file:

```text
world/server-side-backpacks.json
```

Commands:

```text
/backpack
/bp
/backpack upgrade
/backpack downgrade
/backpack tier
/view backpack <player>
```

Behavior:

- Backpacks start at tier 0.
- Max visible size is capped at 6 rows.
- Upgrades use bank balance.
- Default upgrade cost is 1000 coins per tier.
- Downgrading refunds the configured cost for the current tier.
- Downgrading is blocked if items exist in rows that would be removed.
- `/view backpack <player>` requires moderation access.

Timed rewards can grant a free backpack upgrade with:

```text
serverside:upgrade_backpack
```

## Timed Rewards And AFK Tracking

Timed rewards track active playtime. AFK time does not count.

Data file:

```text
world/server-side-timed-rewards.json
```

Commands:

```text
/timedrewards
/playtime
/craft
```

Behavior:

- Players become AFK after `afkTimeoutSeconds` without movement/input.
- AFK players stop gaining active playtime.
- Rewards are claimed automatically once thresholds are reached.
- The first default reward is at 10 hours.
- Later reward slots are every 20 hours up to 500 hours.
- `/craft` opens a virtual crafting table after it is unlocked.

Default first reward:

```json
"10": {
  "name": "Virtual Crafting Table",
  "commands": ["serverside:grant_craft"]
}
```

Reward commands support placeholders:

```text
{player}
{uuid}
```

Built-in reward commands:

```text
serverside:grant_craft
serverside:upgrade_backpack
```

Other commands are run from the server command source.

## Spawn Starter Items

The mod keeps required Middle-earth starter items available near spawn in the overworld.

Default items:

```text
middle-earth:starlight_phial
middle-earth:middle_earth_map
```

Config:

```json
"spawnStarterRadius": 128,
"spawnStarterItems": [
  "middle-earth:starlight_phial",
  "middle-earth:middle_earth_map"
]
```

The check looks at the player's inventory and server-side backpack. If a required item is missing while the player is in the overworld spawn area, it is returned to the player.

## Custom NPC Traders

The trader system turns explicitly configured entities into custom vanilla merchant screens without changing their appearance.

Data file:

```text
world/server-side-traders.json
```

Commands require moderation access:

```text
/trader create <id> <entity>
/trader remove <id>
/trader title <id> <title>
/trader list
/trader info <id>
/trader trade add <id> <buyA> <buyACount> <sell> <sellCount>
/trader trade clear <id>
```

Usage:

1. Stand near or target the NPC/entity.
2. Run `/trader create <id> <entity>`.
3. Add trades with `/trader trade add`.
4. Right-click the configured entity to open the merchant screen.

Notes:

- Only explicitly configured entity UUIDs become traders.
- Unconfigured NPCs keep normal interaction behavior.
- Vanilla and modded items are supported through normal item IDs.
- The saved JSON trade format includes optional `buyB`, `maxUses`, and `restock` fields. The current command path exposes one-input trades; advanced fields can be edited in the JSON file, then reloaded/restarted.
- `maxUses` of `0` means unlimited in the current command-created trades.

## Mail

Mail stores messages and optional item gifts for offline or online players.

Data file:

```text
world/server-side-mail.json
```

Commands:

```text
/mail list
/mail send <player> <message>
/mail gift <player> <message>
/mail claim <id>
/mail delete <id>
```

`/mail gift` attaches the item in the sender's main hand and removes it from the sender.

## Fellowships

Fellowships are small player groups with optional private chat and PvP toggle.

Commands:

```text
/fellowship create <name>
/fellowship info
/fellowship invite <player>
/fellowship accept <name>
/fellowship kick <player>
/fellowship transfer <player>
/fellowship rename <name>
/fellowship chat
/fellowship chat <message>
/fc <message>
/fellowship pvp
/fellowship leave
/fellowship delete
```

The fellowship owner manages invites, kicks, transfer, rename, PvP, and deletion.

## Moderation

Moderation commands require moderation access.

Commands:

```text
/tempban <player> <duration> [reason]
/tempban <player> <duration> -s [reason]
/mute <player> <duration> [reason]
/mute <player> <duration> -s [reason]
/mute <player> pds [reason]
/unmute <player>
/unmute <player> -s
/punishments
```

Duration examples:

```text
30m
2h
1d
1d12h
1w
```

PDS mute ladder:

- First offense: warning/log entry.
- Second offense: 1 hour mute.
- Third offense: 1 day mute.
- Fourth offense: 3 day mute.
- Further offenses: permanent-style mute marker.

Silent mode (`-s`) avoids public broadcast where supported.

Discord punishment logging hooks are called when Discord integration is configured.

## Chat Filter And Staff Chat

Chat filter features:

- Blocked word/phrase filtering.
- Formatting-code filtering.
- Repeated character filtering.
- Excessive caps filtering.
- Duplicate/spam rate filtering.
- Optional English-only public chat check.
- Clickable player names in chat.

Commands:

```text
/chatfilter
/chatfilter add <word or phrase>
/chatfilter remove <word or phrase>
/chatfilter check <message>
/chatfilter list
```

Staff chat:

```text
/staffchat
/staffchat <message>
/sc
/sc <message>
```

Running `/staffchat` or `/sc` with no message toggles staff chat mode.

Discord/internal bridge commands:

```text
/staffchatbridge <sender> <message>
/scbridge <sender> <message>
```

Bridge commands require OP level 4.

## Staff Utility Commands

Commands:

```text
/fly
/heal
/heal <player>
/gm 0
/gm 1
/gm 2
/gm 3
/stp <player>
/viewdistance <chunks>
/otp <player>
/view inv <player>
/view echest <player>
/view backpack <player>
/vanish
/broadcast <message>
/silence
```

Gamemode numbers:

```text
0 survival
1 creative
2 adventure
3 spectator
```

Notes:

- `/viewdistance` accepts 2-32 chunks and requires OP level 4.
- `/otp` teleports staff to an offline player's last saved logout location.
- `/view inv` and `/view echest` can inspect online or offline player data.
- `/view backpack` opens the server-side backpack.
- `/vanish` hides a staff member from normal players.
- `/silence` toggles public chat silence.

## Economy Item Cosmetics

Commands:

```text
/rename <name>
/rename clear
/engrave
/lore <text>
/lore clear
/glint
```

Behavior:

- `/rename` charges `cosmeticRenameCost` from bank balance.
- `/engrave` charges `cosmeticEngraveCost` and writes ownership lore.
- `/lore` and `/glint` require moderation access.
- Rename/engrave only work on a single held item stack.

## Stats

Stats are stored server-side and include kills, deaths, blocks, playtime, and bank balance.

Data file:

```text
world/server-side-stats.json
```

Commands:

```text
/stats
/stats <player>
/stats top <stat>
```

Top stat names:

```text
playtime
playerkills
deaths
mobkills
blocksplaced
blocksmined
bank
```

Tracked values:

- Time played.
- Player kills.
- Deaths.
- Mob kills.
- Blocks placed.
- Blocks mined.
- Bank balance.

Playtime displays in minutes before 1 hour, then converts to hours/minutes.

## Possible Alts

The alt checker records connection data and reports possible alternate accounts.

Data file:

```text
world/server-side-alts.json
```

Command:

```text
/possiblealts <player>
```

Requires moderation access.

The check is based on stored player identity/IP-style connection data captured by the server-side mod.

## Weather Vote

When bad weather starts, the server can begin a vote to clear it.

Commands:

```text
/weathervote yes
/weathervote no
/wv y
/wv yes
/wv n
/wv no
```

Players vote through the command or clickable chat buttons.

## Generators

Generators turn inventories into passive item generators.

Commands require moderation access:

```text
/gen
/gen add <item> <amountPerHour>
/gen remove
/gen remove <id>
/gen info
/gen info <id>
/gen list
/gen list <page>
/gen export
```

Usage:

1. Look directly at an inventory block within 8 blocks.
2. Run `/gen add <item> <amountPerHour>`.
3. The inventory receives generated items over time.

`/gen export` writes a readable export file for review.

Important config:

```json
"generatorsEnabled": true,
"generatorMaxPerTick": 20,
"generatorMaxItemsPerCycle": 64
```

## Collectibles

Collectibles are placed world objects that players can find for collection progress and rewards.

Config/data:

```text
world/server-side-collectibles/
world/server-side-collectible-progress.json
```

Commands require OP level 2:

```text
/collectible create <collection> <collectible>
/collectible delete <collection> <collectible>
/collectible deletecollection <collection>
/collectible move <collection> <collectible>
/collectible list
/collectible reload
/collectible info <collection>
/collectible info <collection> <collectible>
/collectible tp <collection> <collectible>
/collectible reset <player>
/collectible reset <player> <collection>
/collectible resetall
/collectible progress <player>
/collectible completed <collection>
/collectible reward xp <collection> <amount>
/collectible reward command <collection> <command>
/collectible display item <collection> <collectible>
/collectible display scale <collection> <collectible> <scale>
/collectible display offset <collection> <collectible> <x> <y> <z>
/collectible display rotation <collection> <collectible> <yaw> <pitch>
```

Collectible rewards can grant XP and run commands when a collection is completed.

## Music Zones

Music Zones play ambient disc audio based on player position. They do not require a physical jukebox.

Config path:

```text
config/server-side/music_zones.json
```

Commands require OP level 2:

```text
/musiczone list
/musiczone info <id>
/musiczone reload
/musiczone start <id> [player]
/musiczone stop <id> [player]
/musiczone create <id> <disc> <radius>
/musiczone delete <id>
/musiczone setcenter <id>
/musiczone setdisc <id> <disc>
/musiczone setradius <id> <radius>
```

`create` and `setcenter` use the command executor's current position and dimension.

When zones overlap, only one track plays for a player. The highest priority zone wins. If priority ties, the smallest matching radius wins.

Example:

```json
{
  "zones": [
    {
      "id": "spawn",
      "enabled": true,
      "dimension": "minecraft:overworld",
      "x": 0,
      "y": 80,
      "z": 0,
      "radius": 64,
      "disc": "minecraft:music_disc_cat",
      "volume": 1.0,
      "loop": true,
      "loop_delay_ticks": 20,
      "restart_on_reentry": true,
      "inner_radius": 0,
      "priority": 0,
      "fade_distance": 0,
      "stop_on_logout": true
    }
  ]
}
```

Vanilla music discs use item IDs such as:

```text
minecraft:music_disc_cat
minecraft:music_disc_otherside
```

DiscX disc IDs are also supported when DiscX is present.

## Discord Integration

The server-side mod exposes a small HTTP API and calls the Discord bot API for sync.

Main config fields:

```json
"discordIntegrationEnabled": true,
"apiBindHost": "0.0.0.0",
"apiPort": 8766,
"discordBotApiBaseUrl": "http://...",
"discordIntegrationSecret": "...",
"discordLinkCodeExpiresSeconds": 600,
"discordLinkGraceSeconds": 300,
"discordLinkBanMinutes": 60
```

Player command:

```text
/link <code>
```

Used with the Discord bot's link flow.

Discord sync hooks include:

- Minecraft account linking.
- Faction changes.
- Haven/Mortal changes.
- Punishment logs.
- Stats lookups.

## Tree Chopper And Leaf Decay

Tree Chopper is configurable and runs server-side.

Important config:

```json
"treeChopperEnabled": true,
"treeChopperRequireLeaves": true,
"treeChopperMaxTreeSize": 4096,
"treeChopperMaxWorkPerTick": 256,
"fastLeafDecay": true,
"fastLeafDecayRadius": 5,
"fastLeafDecayMaxPerTick": 96,
"fastLeafDecayDelayTicks": 20
```

When enabled, trees can be processed in batches and leaves decay faster after chopping.

## Welcome Rewards

The welcome feature rewards players for welcoming new players.

Config:

```json
"newPlayerWelcomeMessage": "&5Welcome #PLAYER to the server!",
"welcomeRewardXp": 55
```

## Messages

Most player-facing messages are configurable in:

```json
"messages": {
  "...": "..."
}
```

Color formats:

```text
&5
&6
&#5E4B8B
#N
```

`#N` inserts a new line.

The default Saurona prefix format is:

```text
&8[&#5E4B8BSaurona&8]
```

## Alias Commands

The alias manager registers command aliases from config. Aliases forward to configured target commands and preserve extra arguments.

Use aliases for short commands or server-specific command names without adding new command code.

## Common Admin Setup Checklist

1. Start the server once with the jar installed.
2. Stop the server.
3. Edit `world/server-side-config.json`.
4. Set `staffCommandOwnerUuids`.
5. Confirm prefix colors.
6. Confirm Discord API URL/secret if using the bot.
7. Set faction homes with `/factionadmin sethome <faction>`.
8. Configure timed rewards.
9. Configure custom traders.
10. Start the server and run `/ss reload` after small config edits.

## Notes For Manual JSON Editing

- Keep UUID keys quoted.
- Use full item IDs such as `minecraft:diamond` or `middle-earth:gold_coin`.
- Commands in timed rewards should not include a leading slash.
- Back up data files before large manual edits.
- Restart after editing files that are only loaded on startup, or use `/ss reload` for the core server-side config reload path.
