# Pizza Jump — Claude Code Instructions

## How to Use These Docs with Claude Code

This file explains how to work with Claude Code to build Pizza Jump. Read this first!

---

## Getting Started

### 1. Set Up Your Project
Follow `02-SETUP-GUIDE.md` to install Roblox Studio, Rojo, and initialize the project.

### 2. Open Claude Code
```bash
cd pizza-jump
claude
```

### 3. Point Claude Code to the Docs
When starting a coding session, tell Claude Code:
```
Read the docs in the docs/ folder. These are the specifications for Pizza Jump,
a Roblox multiplayer platformer. I want to build this step by step.
```

Claude Code will read the markdown files and understand the full game design.

---

## Recommended Build Order

Build the game in this order. Each phase should be working and testable before moving to the next.

### Phase 1: Foundation (Get Something Running)
1. **Constants module** (`src/shared/modules/Constants.luau`)
   - All game constants: speeds, timers, sizes
   - Reference: `03-GAME-SYSTEMS.md` → movement stats, timing values
2. **Basic character spawning** (`src/server/GameManager.server.luau`)
   - Spawn players with default Roblox character (pizza character comes later)
   - Basic lobby area
3. **Simple platform** (`src/shared/modules/PlatformTypes.luau`)
   - Static platform creation from code
   - Just boxes for now — visuals come later

**Test:** Players join, spawn, can walk and jump on platforms.

### Phase 2: Core Gameplay Loop
4. **Round Manager** (`src/server/RoundManager.server.luau`)
   - State machine: LOBBY → COUNTDOWN → PLAYING → ROUND_END
   - Reference: `03-GAME-SYSTEMS.md` → Round System
5. **Rising floor** — simple kill plane that moves up
6. **Elimination system** (`src/server/EliminationManager.server.luau`)
   - Track player heights
   - Timed elimination every 60 seconds
   - Reference: `03-GAME-SYSTEMS.md` → Elimination System
7. **Basic world layout** — manually place ~20 static platforms vertically

**Test:** Round starts, floor rises, players climb, last place gets eliminated, round ends.

### Phase 3: Platform Variety
8. **All platform types** — Moving, Bouncy, Crumbling, Slippery, Conveyor, Disappearing
   - Reference: `03-GAME-SYSTEMS.md` → Platform System
9. **World 1 (Kitchen) layout** with mixed platform types
   - Reference: `04-WORLD-DESIGN.md` → World 1

**Test:** Kitchen world is playable with all platform types working correctly.

### Phase 4: Power-Ups
10. **Topping system** (`src/shared/modules/ToppingSystem.luau`)
    - Spawn collectible toppings
    - Apply effects (speed boost, double jump, sticky, gravity flip, pineapple random)
    - Reference: `03-GAME-SYSTEMS.md` → Topping Power-Up System
11. **HUD for active topping** — icon + timer bar

**Test:** Can collect toppings, effects work, timer shows, one topping at a time.

### Phase 5: Hazards
12. **Hazard system** — Kitchen hazards (pizza cutters, steam, flame jets)
    - Reference: `03-GAME-SYSTEMS.md` → Hazard System
    - Reference: `04-WORLD-DESIGN.md` → Kitchen hazards

**Test:** Hazards activate with warnings, knockback works, doesn't feel unfair.

### Phase 6: More Worlds
13. **World 2 (Dining Table)** with unique platforms + napkin glider
14. **World 3 (Street)** with unique platforms + wind gusts
15. **Random world selection** per round
    - Reference: `04-WORLD-DESIGN.md`

**Test:** All 3 worlds playable, each feels unique, world randomly selected.

### Phase 7: Data & Progression
16. **Data Manager** (`src/server/DataManager.server.luau`)
    - Save/load player data with DataStoreService
    - Reference: `06-DATA-AND-PROGRESSION.md` → Data Manager
17. **Coin system** — award coins at round end based on placement
18. **Leaderstats** — coins and wins on Roblox leaderboard

**Test:** Coins persist across sessions, leaderboard shows correctly.

### Phase 8: Shop & Cosmetics
19. **Shop UI** (`src/ui/ShopUI.luau`)
    - Reference: `05-UI-AND-SHOP.md` → Cosmetics Shop
20. **Shop Manager** (`src/server/ShopManager.server.luau`)
    - Purchase validation, equip logic
21. **Cosmetic application** — apply skins, hats, trails to characters
22. **Catalog data** as shared module

**Test:** Can buy items, equip them, they show on character, persist between sessions.

### Phase 9: Polish
23. **Full HUD** — all elements from `05-UI-AND-SHOP.md`
24. **Results screen** with animations
25. **Camera system** improvements (zoom on events, spectator mode)
26. **Sound effects and music**
27. **Lobby (The Pizzeria)** with shop kiosk and ready-up area
28. **Pizza slice character model** (replace default character)

---

## Tips for Working with Claude Code

### Be Specific
Instead of: "Make the game"
Say: "Create the RoundManager server script based on the state machine in 03-GAME-SYSTEMS.md. Start with LOBBY and PLAYING states only."

### One System at a Time
Don't try to build everything at once. Complete one phase, test it, then move on.

### Reference the Docs
Say things like:
- "Following the platform types in 03-GAME-SYSTEMS.md, create the PlatformTypes module"
- "The rising floor should use the config values from 03-GAME-SYSTEMS.md"
- "Use the RemoteEvent names defined in 06-DATA-AND-PROGRESSION.md"

### Test Frequently
After Claude Code writes a script:
1. Save the file
2. Rojo syncs it to Studio
3. Press Play in Studio
4. Check the Output window for errors
5. Tell Claude Code what happened: "I tested it and got this error: [paste error]"

### File Size
Keep individual Luau files under 300 lines. If a module is getting long, ask Claude Code to split it into sub-modules.

### Common Prompts for Each Phase

**Phase 1:**
```
Create src/shared/modules/Constants.luau with all the game constants from the docs.
Include player movement, timing, world dimensions, and platform configs.
```

**Phase 2:**
```
Create the RoundManager server script. It should implement the state machine
from 03-GAME-SYSTEMS.md with states: LOBBY, COUNTDOWN, PLAYING, ROUND_END.
Use RemoteEvents to notify clients of state changes.
```

**Phase 4:**
```
Create the ToppingSystem module. Reference 03-GAME-SYSTEMS.md for the topping
types, durations, and effects. The system should handle spawning, collection
(server-validated), applying effects, and expiration.
```

---

## Troubleshooting

### "Script doesn't appear in Studio"
- Check Rojo is running (`rojo serve`)
- Check Rojo plugin is connected in Studio
- Check file extension (`.server.luau`, `.client.luau`, or `.luau`)
- Check the file is in the correct src/ subfolder

### "Attempt to index nil"
- Usually means you're trying to use a service or object that hasn't loaded yet
- Use `WaitForChild()` instead of direct indexing
- Make sure server scripts don't reference client-only services and vice versa

### "DataStore request was added to queue"
- DataStore has rate limits — don't save too frequently
- Use the auto-save pattern (every 60 seconds) from `06-DATA-AND-PROGRESSION.md`

### "Players can't see each other's cosmetics"
- Cosmetics must be applied server-side, not client-side
- Server changes to character model replicate to all clients automatically

---

## Luau Quick Reference for Beginners

```lua
-- Variables
local speed = 20
local name = "Pizza Jump"
local isRunning = true

-- Tables (arrays and dictionaries)
local fruits = {"apple", "banana", "cherry"}
local player = {name = "Bob", score = 100}

-- Functions
local function greet(name)
    print("Hello, " .. name)
end

-- Services (how you access Roblox APIs)
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local DataStoreService = game:GetService("DataStoreService")

-- Events
Players.PlayerAdded:Connect(function(player)
    print(player.Name .. " joined!")
end)

-- RemoteEvents (client ↔ server communication)
-- Server:
local remote = ReplicatedStorage.Remotes.MyEvent
remote:FireClient(player, "data")  -- send to one client
remote:FireAllClients("data")      -- send to all clients
remote.OnServerEvent:Connect(function(player, data)
    -- handle client request
end)

-- Client:
local remote = ReplicatedStorage:WaitForChild("Remotes"):WaitForChild("MyEvent")
remote:FireServer("data")  -- send to server
remote.OnClientEvent:Connect(function(data)
    -- handle server message
end)

-- Modules
-- In MyModule.luau:
local MyModule = {}
function MyModule.doThing()
    return "done"
end
return MyModule

-- Using it:
local MyModule = require(path.to.MyModule)
MyModule.doThing()
```

---

## Project Conventions

- **File names:** PascalCase for modules (`ToppingSystem.luau`), camelCase for instances
- **Variables:** camelCase (`playerSpeed`, `isAlive`)
- **Constants:** UPPER_SNAKE_CASE (`MAX_PLAYERS`, `ROUND_DURATION`)
- **Types:** PascalCase (`PlayerData`, `ToppingType`)
- **Max file size:** 300 lines per file
- **Comments:** explain WHY, not WHAT (the code should be readable on its own)
- **Error handling:** always handle DataStore failures, always validate remote event data
