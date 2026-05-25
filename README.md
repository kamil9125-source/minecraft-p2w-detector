# P2W Detector — Fabric 1.21.11

Client-side Fabric mod MVP for detecting possible pay-to-win signals on Minecraft servers, with optional DupeDB status lookup.

## What it does

- Starts a local scan when you join a server.
- Reads normal chat messages and server/game messages.
- Scores suspicious P2W phrases such as `VIP`, `SVIP`, `crate`, `klucz`, `/fly`, `/kit`, `/repair`, `coins`, `money`, `sklep`, `buy`.
- Shows a P2W risk level: `NONE`, `LOW`, `MEDIUM`, `HIGH`.
- Can query DupeDB for known exploit records and show their database status: `WORKING`, `VERIFIED`, `PATCHED`, `UNVERIFIED`.
- Supports DupeDB OAuth 2.1 + PKCE login from inside Minecraft via a local loopback callback.
- Automatically refreshes saved DupeDB tokens when the access token expires.
- Saves a local JSON report including P2W findings and the latest DupeDB result.

## Commands

```text
/p2wscan
/p2wscan report
/p2wscan clear
/p2wscan help
/p2wscan dupedb
/p2wscan dupedb <fraza>
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

`/p2wscan dupedb <fraza>` searches by phrase, for example:

```text
/p2wscan dupedb shulker
/p2wscan dupedb backpack
/p2wscan dupedb crate
```

The mod does **not** execute exploits. It only reads status from DupeDB.

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
