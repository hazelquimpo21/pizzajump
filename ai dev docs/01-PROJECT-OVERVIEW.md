# Pizza Jump — Project Overview

## Game Concept

**Pizza Jump** ("Rise of the Slice") is a multiplayer vertical platformer on Roblox. Players are sentient pizza slices escaping a kitchen, jumping upward through themed worlds while collecting topping power-ups, avoiding hazards, and competing against other players.

**Genre:** Multiplayer vertical platformer / obby-racer hybrid
**Target Players:** 20-30 per server
**Monetization:** In-game cosmetics shop (Robux)
**MVP Scope:** 3 worlds, topping power-ups, cosmetics shop, basic multiplayer

---

## MVP Worlds

### World 1: The Kitchen (Beginner)
- **Theme:** Industrial pizza kitchen — stainless steel, flour clouds, dough
- **Platforms:** Cutting boards, stacked plates, hanging pans, rising dough balls (bouncy)
- **Hazards:** Spinning pizza cutters (horizontal blades that sweep across), oven flames rising from below (the "lava" floor), steam bursts from pots
- **Unique Mechanic:** Dough balls — springy platforms that launch you higher than normal jumps
- **Lighting:** Warm orange/yellow, industrial fluorescent overhead
- **Rising Danger:** The oven floor slowly rises with visible heat shimmer, forcing players upward

### World 2: The Dining Table (Intermediate)
- **Theme:** Giant dining table from a pizza slice's perspective — everything is oversized
- **Platforms:** Plates, overturned cups, napkin folds, salt/pepper shakers, breadsticks
- **Hazards:** Forks stabbing down from above (timed, with shadow warnings), a giant hand that sweeps across grabbing at players, spilled drinks creating slippery zones
- **Unique Mechanic:** Napkin gliders — certain napkin platforms unfold and let you glide short distances
- **Lighting:** Warm restaurant ambiance, candle flicker effects
- **Rising Danger:** The table is being cleared — a sweeping arm pushes everything off from below

### World 3: The Street (Advanced)
- **Theme:** Urban street at night — you've escaped the restaurant
- **Platforms:** Car roofs, awnings, fire escapes, dumpster lids, street signs, window ledges
- **Hazards:** Pigeons that swoop and knock you off platforms, car headlights that temporarily blind (white flash), rain making some platforms slippery
- **Unique Mechanic:** Wind gusts — periodic gusts push all players in one direction, need to compensate
- **Lighting:** Neon signs, streetlights, moody nighttime city vibe
- **Rising Danger:** A flood of marinara sauce rising from the street below

---

## Core Game Loop

```
1. Players join server → spawn in The Pizzeria (lobby)
2. Match starts → 20-30 players spawn at bottom of a world
3. Rising danger (floor) forces everyone upward
4. Players jump platform to platform, collecting toppings
5. Last-place player eliminated every 60 seconds
6. Surviving to the top = winning
7. Coins awarded based on placement
8. Return to lobby → spend coins in shop → queue for next round
```

---

## Technical Architecture

### Tool: Rojo
We use **Rojo** to sync local files ↔ Roblox Studio. This lets us write all code locally and version control with Git.

### Project Structure
```
pizza-jump/
├── default.project.json          -- Rojo project config
├── docs/                         -- These documentation files
│   ├── 01-PROJECT-OVERVIEW.md
│   ├── 02-SETUP-GUIDE.md
│   ├── 03-GAME-SYSTEMS.md
│   ├── 04-WORLD-DESIGN.md
│   ├── 05-UI-AND-SHOP.md
│   └── 06-DATA-AND-PROGRESSION.md
├── src/
│   ├── server/                   -- ServerScriptService
│   │   ├── GameManager.server.luau
│   │   ├── RoundManager.server.luau
│   │   ├── EliminationManager.server.luau
│   │   ├── DataManager.server.luau
│   │   └── ShopManager.server.luau
│   ├── client/                   -- StarterPlayerScripts
│   │   ├── InputController.client.luau
│   │   ├── CameraController.client.luau
│   │   ├── UIController.client.luau
│   │   └── EffectsController.client.luau
│   ├── shared/                   -- ReplicatedStorage
│   │   ├── modules/
│   │   │   ├── ToppingSystem.luau
│   │   │   ├── CharacterSystem.luau
│   │   │   ├── PlatformTypes.luau
│   │   │   └── Constants.luau
│   │   ├── remotes/
│   │   │   └── RemoteDefinitions.luau
│   │   └── types/
│   │       └── GameTypes.luau
│   └── ui/                       -- StarterGui
│       ├── HUD.luau
│       ├── ShopUI.luau
│       ├── LobbyUI.luau
│       └── ResultsUI.luau
├── assets/                       -- Reference images, sounds (imported manually in Studio)
└── README.md
```

### Server-Client Architecture
- **Server** controls: round state, elimination timer, player data, shop transactions, world loading, anti-cheat validation
- **Client** controls: player input, camera, UI rendering, visual effects, sound
- **Shared** modules: constants, type definitions, utility functions used by both

### Communication (RemoteEvents / RemoteFunctions)
All server↔client communication goes through defined RemoteEvents in ReplicatedStorage. Never trust the client — all game logic validated server-side.

---

## Art Direction

### Pizza Slice Character
- Triangular pizza slice with googly eyes and small legs
- Cheese drips slightly when moving
- Different "skins" are different pizza types (see cosmetics doc)
- Simple, cartoony, Roblox-friendly style

### Visual Style
- Bright, saturated colors
- Cartoony/stylized (not realistic)
- Each world has a distinct color palette
- Particle effects for power-ups, eliminations, and hazards
- Screen shake on big jumps and near-misses

### Sound Design
- Bouncy, upbeat background music per world
- Satisfying "boing" on jumps
- Sizzle sound near hazards
- Cheerful "ding" on topping collection
- Dramatic elimination sound
- Victory fanfare for winners
