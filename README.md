# P2W Detector — Fabric 1.21.11

Client-side Fabric mod MVP for detecting possible pay-to-win signals on Minecraft servers, with optional DupeDB status lookup.

## What it does

- Starts a local scan when you join a server.
- Reads normal chat messages and server/game messages.
- Scores suspicious P2W phrases such as `VIP`, `SVIP`, `rank`, `rankup`, `perks`, `crate key`, `monthly crate`, `gkit`, `god kit`, `/fly`, `/kit`, `/repair`, `coins`, `money`, `store`, `buycraft`, `tebex`, `buy`.
- Shows a P2W risk level: `NONE`, `LOW`, `MEDIUM`, `HIGH`.
- Can passively fingerprint visible server plugins/software from command namespaces, command names, chat/system text, current GUI title, server brand, and visible `/plugins`-style responses if you manually run the command.
- Can query DupeDB for known exploit records and show their database status: `WORKING`, `VERIFIED`, `PATCHED`, `UNVERIFIED`.
- Supports DupeDB OAuth 2.1 + PKCE login from inside Minecraft via a local loopback callback.
- Automatically refreshes saved DupeDB tokens when the access token expires.
- Saves a local JSON report including P2W findings, plugin fingerprints and the latest DupeDB result.

## Keyword coverage

The scanner includes English and Polish P2W signals. Examples:

```text
store, shop, buy, purchase, donate, voucher, buycraft, tebex
rank, ranks, rankup, vip, svip, mvp, elite, premium, donor
perk, perks, perks shop, permissions
key, keys, crate, crate key, monthly crate, loot crate, mystery crate, lootbox
kit, kits, gkit, god kit, monthly kit, donor kit
/fly, /kit, /repair, /fix, /heal, /feed, /god
coins, money, cash, tokens, gems, emeralds
boost, booster, xp, spawner, sellwand, home, sethome
netherite, diamond, elytra, totem, beacon, enchanted
```

Polish signals such as `sklep`, `ranga`, `klucz`, `skrzynka`, `kity`, `monety`, and `kasa` are also kept.

## Plugin fingerprinting

The plugin scanner is passive. It does **not** run `/plugins`, `/pl`, `/bukkit:plugins`, or probing commands by itself. It only reads data already visible to the client:

- command namespaces such as `luckperms:lp`, `essentials:home`, `worldguard:region`,
- command names such as `/shop`, `/crate`, `/ah`, `/backpack`, `/rankup`,
- chat/system messages that mention known plugin names or plugin-specific wording,
- visible `/plugins` / `/pl` style responses if **you manually run the command** and the server prints a plugin list,
- the current GUI title when you run `/p2wscan plugins`,
- server brand/software if the connection exposes it, such as Paper, Purpur, Spigot, Bukkit.

Example visible response that will now be parsed:

```text
Plugins (4): LuckPerms, EssentialsX, WorldGuard, ExcellentCrates
```

The parsed plugin names are stored as high-confidence plugin signals with source `chat_plugins_response` or `system_plugins_response`.

Use:

```text
/p2wscan plugins
```

Then, after DupeDB login, you can search DupeDB using detected plugin names:

```text
/p2wscan dupedb plugins
```

This sends up to 5 plugin-name searches to DupeDB to avoid spamming the API.

## Commands

```text
/p2wscan
/p2wscan report
/p2wscan clear
/p2wscan help
/p2wscan plugins
/p2wscan dupedb
/p2wscan dupedb plugins
/p2wscan dupedb <query>
/p2wscan dupedb login
/p2wscan dupedb logout
/p2wscan dupedb status
/p2wscan dupedb config
/p2wscan dupedb reload
```

### DupeDB commands

`/p2wscan dupedb` checks the current server without a phrase. With a token, it calls:

```text
GET https://dupedb.net/api/exploits/search?edition=java&platform=multiplayer&serverIp=<server>&status=working,verified,patched
```

`/p2wscan dupedb <query>` searches by query, for example:

```text
/p2wscan dupedb shulker
/p2wscan dupedb backpack
/p2wscan dupedb crate
```

The mod does **not** execute exploits. It only reads status from DupeDB.

`/p2wscan dupedb plugins` searches by detected plugin names. It requires OAuth login because it uses the authenticated search API.

## DupeDB OAuth login

Register your DupeDB OAuth app as:

```text
App ID: p2w-detector
Display Name: P2W Detector
Permission Level: User / read-only
App Type: Desktop / CLI
Loopback URI: http://127.0.0.1/callback
```

Then in Minecraft run:

```text
/p2wscan dupedb login
```

The mod will:

1. start a short-lived local listener on `127.0.0.1:<random-port>/callback`,
2. open the DupeDB authorization page,
3. wait for you to click `Allow`,
4. exchange the one-time code for tokens,
5. save tokens locally in:

```text
.minecraft/config/p2w-detector/dupedb-auth.json
```

Tokens are never printed in chat. Do not share `dupedb-auth.json`.

Useful auth commands:

```text
/p2wscan dupedb status
/p2wscan dupedb logout
```

`logout` calls DupeDB token revocation and removes the local auth file. If revocation cannot reach the network, the local file is still removed.

## DupeDB configuration

On first launch, the mod creates:

```text
.minecraft/config/p2w-detector/dupedb.properties
```

Default file:

```properties
baseUrl=https://dupedb.net/api
clientId=p2w-detector
accessToken=
status=working,verified,patched
limit=10
autoScan=true
usePublicWithoutToken=true
```

`clientId` must match the App ID you registered on DupeDB. `accessToken` is only a temporary/manual fallback and normally stays empty.

Full `/exploits/search` uses OAuth. If there is no OAuth login and `usePublicWithoutToken=true`, the mod falls back to the weaker no-auth endpoint:

```text
GET https://dupedb.net/api/public/exploits
```

That fallback only returns the newest public exploit cards, so it is not a full server check.

After editing the config, use:

```text
/p2wscan dupedb reload
```

## Build

Requirements:

- JDK 21
- Gradle installed, or import the folder as a Gradle project in IntelliJ IDEA

Build from the project root:

```bash
gradle build
```

The compiled mod jar will be in:

```text
build/libs/
```

Put the jar into your Minecraft `mods` folder together with Fabric API.

## Important

This mod does not claim that a server definitely violates any rule. It only collects local evidence and estimates the probability of P2W based on text patterns. DupeDB results are shown as database status only; the mod never tests or executes exploits on a server.
