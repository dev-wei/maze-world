# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Maze World is a Roblox game where players solve dynamically generated mazes. The codebase is version-controlled and synced to Roblox Studio via Rojo. It uses a Roact/Rodux architecture (React/Redux pattern for Roblox) with server-client state replication.

## Build & Run

```bash
# Install wally packages (required after fresh clone)
wally install

# Install dev dependencies (prettier for Lua formatting)
npm i

# Build the game file and open in Roblox Studio
./scripts/build-and-open.sh
# (runs: rojo build -o Game.rbxlx && open Game.rbxlx)

# Extract models from rbxlx file (after editing in Studio)
remodel run ./get-models.lua

# Sync assets to Roblox CDN
tarmac sync --target roblox --auth ROBLOSECURITY
```

**Toolchain (managed via rokit.toml):** Rojo 7.6.1, Lune 0.10.4

After building, connect Rojo from VS Code and use the Rojo plugin in Roblox Studio to sync live changes.

## Architecture

### State Management (Rodux)

The game uses Rodux (Redux for Roblox) with a shared reducer architecture:

- **`src/common/commonReducers`** — Shared state (rooms, shop, leaderboards, inventories) replicated between server and client
- **`src/server/serverReducers`** — Server-only state
- **`src/client/clientReducers`** — Client-only state

State replication: Server dispatches actions and the `networkMiddleware` automatically replicates actions that affect `commonReducers` to all clients, or to specific players via `replicateTo`/`replicateBroadcast` flags.

### Client-Server Communication

- **`src/server/ServerApi.lua`** — Defines RemoteEvents the server exposes
- **`src/client/ClientApi.lua`** — Client connects to server RemoteEvents
- Actions in `src/common/actions/toClient/` are server-to-client dispatches

### Source Layout (maps to Rojo project in default.project.json)

| Directory | Roblox Location | Purpose |
|-----------|----------------|---------|
| `src/server/` | ServerScriptService | Server scripts, game loop, API |
| `src/client/` | StarterPlayerScripts | Client UI, Roact components |
| `src/common/` | ReplicatedStorage.Modules.src | Shared code (reducers, actions, thunks, utilities) |
| `src/serverStorage/` | ServerStorage | GameAnalytics |
| `models/` | ReplicatedStorage.Models | Game models (Prefabs, Pets, Walls, etc.) |
| `modules/` | ReplicatedStorage.Modules | Third-party libraries (git submodules) |

### Key Systems

- **MazeGenerator** (`src/common/MazeGenerator.lua`) — Procedural maze generation using recursive backtracking
- **RoomManager** (`src/common/RoomManager.lua`) — Manages maze rooms with ZonePlus for player detection
- **GameDatastore** (`src/common/GameDatastore.lua`) — Persistent player data (coins, pets, inventory) via DataStore2
- **Thunks** (`src/common/thunks/`) — Async game logic (room game loops, player events)
- **TagItem** (`src/common/TagItem.lua`) — CollectionService-based interaction system for kill bricks, coins, collectables, triggers

### UI Layer

Roact components in `src/client/Components/` with RoactMaterial for styling. The root component is `Game.lua`, mounted via `RoactRodux.StoreProvider`.

### Plugin

`plugin/` contains a standalone Roblox Studio plugin for generating mazes with configurable parameters (size, materials, options).

## Code Style

- Lua with Roblox globals (`selene.toml`: `std = "roblox"`)
- Prettier with plugin-lua for formatting (100 char width, 4-space indent, single quotes)
- Hot-reloading supported: `src/HotReloadServer.lua` and `src/HotReloadClient.lua` manage context/destructors

## Dependencies

Managed by [Wally](https://wally.run) (Roblox package manager). Run `wally install` to populate `Packages/`.

**Wally packages** (in `wally.toml`):
Roact, Rodux, RoactRodux, Promise, t, Knit, DataStore2, ZonePlus

**Vendored** (in `vendor/`, not on Wally registry):
Moses (utility library), RoactMaterial, RoactAnimate, Maid (cleanup utility)
