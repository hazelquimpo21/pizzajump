# Pizza Jump — Implementation Plan

## Current State
- Blank repo: no `src/`, no `default.project.json`, no code at all
- 7 detailed design docs in `ai dev docs/`
- Rojo tooling (`aftman`, `aftman.toml`) present but project not initialized
- We're building everything from scratch

---

## Philosophy: What Would a Savvy Indie Dev Do?

**Ship a playable vertical slice first, then iterate.**

A Roblox/indie dev would NOT build all 9 phases before testing. They'd get
a single ugly-but-functional round working ASAP, then layer on systems one
at a time — testing after every phase. Each phase ends with something
you can hit "Play" on in Studio and actually experience.

---

## The Phases

### PHASE 0: Project Skeleton (~setup)
**Goal:** Rojo project compiles, folder structure exists, Studio can connect.

Files created:
- `default.project.json` (Rojo config mapping `src/` → Roblox services)
- `src/server/` `src/client/` `src/shared/modules/` `src/shared/remotes/` `src/shared/types/` `src/ui/` (empty dirs with init files where needed)
- `.gitignore` (Roblox binaries, OS files, lock files)

**Testable:** `rojo serve` runs without errors. Studio plugin connects and shows the tree.

---

### PHASE 1: Constants, Types & Logging Utility
**Goal:** Shared foundation every other script will `require()`.

Files:
1. **`src/shared/modules/Logger.luau`** — Custom logging module with levels (DEBUG, INFO, WARN, ERROR), prefixed output, timestamps. Lua has `print()` and `warn()` but no structured logging, so we roll our own. This is your troubleshooting lifeline.
2. **`src/shared/modules/Constants.luau`** — Every magic number from the docs: movement speeds, timer durations, platform sizes, world dimensions, coin awards, etc. Single source of truth.
3. **`src/shared/types/GameTypes.luau`** — Luau type definitions: `GameState`, `PlatformType`, `ToppingType`, `EliminationReason`, `PlayerData`, etc. Catches bugs at edit-time.
4. **`src/shared/remotes/RemoteDefinitions.luau`** — Creates all RemoteEvents/RemoteFunctions in ReplicatedStorage on server start. Central registry so nothing is misspelled.

**Testable:** Require each module in Studio command bar — no errors, types resolve, remotes appear in Explorer.

---

### PHASE 2: Game Manager + Round State Machine
**Goal:** Players join, spawn on a flat platform, and the round cycles through LOBBY → COUNTDOWN → PLAYING → ROUND_END → LOBBY.

Files:
5. **`src/server/GameManager.server.luau`** — Entry point. Handles PlayerAdded/Removing, creates leaderstats, kicks off the round loop.
6. **`src/server/RoundManager.luau`** — Module (not server script) with the state machine. Manages state transitions, timers, fires `GameStateChanged` remote.
7. **`src/client/UIController.client.luau`** — Listens for `GameStateChanged`, shows basic text on screen: "LOBBY", "3..2..1..GO!", "PLAYING", "ROUND OVER". Ugly placeholder UI is fine.

**Testable:** Join in Studio. See "LOBBY" text. After countdown, state switches to PLAYING. After timeout, ROUND_END → back to LOBBY. Check Output window for Logger messages tracing every state transition.

---

### PHASE 3: World Loading + Static Platforms
**Goal:** A basic vertical world loads with simple box platforms you can jump on.

Files:
8. **`src/server/WorldManager.luau`** — Module that builds a world from code: creates a folder of Parts in Workspace, spawns platforms from config data. For now, just World 1 (Kitchen) with ~20 static box platforms stacked vertically.
9. **`src/shared/modules/PlatformTypes.luau`** — Module defining platform creation. Starts with ONLY the `Static` type. A function that takes config → returns a Part.
10. **`src/shared/modules/WorldLayouts.luau`** — Data tables defining platform positions/sizes for each world. Starts with Kitchen only. Pure data, no logic.

**Testable:** Round starts → world appears in Workspace with grey box platforms. Player can walk and jump between them to climb. Round ends → world is destroyed. Platforms are climbable with the jump power from Constants.

---

### PHASE 4: Rising Floor + Basic Elimination
**Goal:** The core competitive pressure. Floor rises, you die if you touch it, last-place player gets eliminated on a timer.

Files:
11. **`src/server/RisingFloor.luau`** — Module that creates/moves the kill plane upward. Uses Constants for speed/acceleration. Detects player contact → eliminates them.
12. **`src/server/EliminationManager.luau`** — Module tracking active/eliminated players, 60-second timed elimination of lowest player, fires `PlayerEliminated` remote.
13. Update `RoundManager` to integrate rising floor + elimination into the PLAYING state.
14. **`src/client/HUDController.client.luau`** — Basic HUD: players remaining, your current placement (by height), elimination warning if you're last.

**Testable:** Floor rises visibly. Touch it = you're eliminated (screen message). Every 60s the lowest player gets eliminated. When 1 player remains, round ends. Logger shows elimination events with player names and reasons.

---

### PHASE 5: All Platform Types
**Goal:** Moving, Bouncy, Crumbling, Slippery, Conveyor, Disappearing platforms all work.

Files:
15. Expand **`PlatformTypes.luau`** — Add behavior functions for each type. Each platform type gets its own section with clear comments explaining the mechanic.
16. Update **`WorldLayouts.luau`** — Kitchen layout now uses all platform types, ~60 platforms across 4 difficulty sections per the doc.
17. Update **`WorldManager.luau`** — Wires up platform behaviors (Heartbeat connections for moving, Touched for bouncy, etc.)

**Testable:** Jump on a bouncy platform → get launched higher. Stand on crumbling → it shakes and breaks. Moving platforms move on paths. Conveyor pushes you. Disappearing platforms toggle. Full Kitchen world is climbable.

---

### PHASE 6: Topping Power-Ups
**Goal:** Collectible power-ups with real gameplay effects.

Files:
18. **`src/shared/modules/ToppingSystem.luau`** — Spawn, collect (server-validated), apply effects, expire. Handles all 5 topping types.
19. **`src/server/ToppingManager.luau`** — Server-side: spawns toppings in the world, validates pickup requests, applies stat changes (speed boost, double jump, etc.), manages timers.
20. Update **HUDController** — Show active topping icon + timer bar.
21. **`src/client/EffectsController.client.luau`** — Visual effects for toppings: fire trail for pepper, sticky particles for cheese, etc.

**Testable:** See floating/rotating topping pickups on platforms. Walk into one → effect applies (speed feels faster, can double-jump, etc.). HUD shows which topping is active and how much time is left. One topping at a time (new replaces old). Pineapple gives random effect.

---

### PHASE 7: Hazard System
**Goal:** World-specific dangers that create exciting moments.

Files:
22. **`src/server/HazardManager.luau`** — Server module: spawns hazards per world config, manages warn→activate→deactivate cycle, knockback physics.
23. **`src/shared/modules/HazardTypes.luau`** — Definitions for each hazard: Kitchen (pizza cutter, steam burst, flame jet). Data + behavior.
24. Update **EffectsController** — Client-side hazard warnings (shadow for fork stab, sound cues, screen effects for headlight blind).

**Testable:** Pizza cutters sweep across at set heights — get hit and you're knocked sideways. Steam bursts launch you upward. Flame jets push you away from walls. Each hazard has a visible/audible warning before activating. Knockback feels fair (immunity frames prevent chain-hits).

---

### PHASE 8: Data Persistence + Coins
**Goal:** Progress saves between sessions.

Files:
25. **`src/server/DataManager.luau`** — DataStoreService wrapper: load on join, auto-save every 60s, save on leave, BindToClose, retry logic, data migration support.
26. Update **GameManager** — Wire up DataManager on PlayerAdded/Removing.
27. Update **EliminationManager / RoundManager** — Award coins at round end based on placement table from docs.
28. Update **HUD** — Show coin count.

**Testable:** Play a round, earn coins. Leave and rejoin — coins are still there. Check Output for DataManager logs showing load/save operations. Leaderstats (Coins, Wins) visible on Roblox player list.

---

### PHASE 9: Shop & Cosmetics
**Goal:** Spend coins on skins, hats, trails, emotes.

Files:
29. **`src/shared/modules/CosmeticsCatalog.luau`** — Full catalog data from the docs (all skins, hats, trails, emotes with prices/rarities).
30. **`src/server/ShopManager.luau`** — Purchase validation, equip logic, server-side character appearance updates.
31. **`src/ui/ShopUI.luau`** — Shop screen with tabs, item grid, buy/equip buttons, preview.
32. **`src/shared/modules/CharacterSystem.luau`** — Apply cosmetics to the pizza character model.

**Testable:** Open shop in lobby. Browse items by category. Buy one — coins deducted. Equip it — visible on your character. Other players see your cosmetics. Persistence across sessions.

---

### PHASE 10: Worlds 2 & 3 + Polish
**Goal:** Full content and game feel.

Files:
33. Expand **WorldLayouts** — Dining Table and Street world layouts.
34. Expand **HazardTypes** — Dining Table hazards (fork stab, giant hand, spills) and Street hazards (pigeons, headlights, rain).
35. **World-specific mechanics:** Napkin glider (World 2), Wind gusts (World 3).
36. Update **RoundManager** — Random world selection with recency weighting.
37. **`src/ui/ResultsUI.luau`** — Animated results screen at round end.
38. **`src/ui/LobbyUI.luau`** — Lobby screen with ready-up, recent results, shop/stats buttons.
39. **`src/client/CameraController.client.luau`** — Proper third-person follow cam, zoom on events, spectator mode after elimination.
40. **`src/client/InputController.client.luau`** — Clean input handling, emote binds.

**Testable:** All 3 worlds randomly selected and fully playable. Camera feels good. Results screen shows placements and coins. Lobby has all buttons working.

---

## Logging Strategy

Lua has `print()` and `warn()` but no built-in logging framework. We'll create a `Logger` module:

```
Logger.info("RoundManager", "State changed to PLAYING")
Logger.warn("DataManager", "Save failed, retrying... attempt 2/3")
Logger.error("ToppingSystem", "Invalid topping type: " .. tostring(type))
Logger.debug("EliminationManager", "Player heights:", heightTable)
```

Output looks like:
```
[INFO] [RoundManager] State changed to PLAYING
[WARN] [DataManager] Save failed, retrying... attempt 2/3
```

- Every module gets a tag so you can filter Output in Studio
- DEBUG level can be toggled off for production
- All state transitions, data operations, and errors logged
- This is non-negotiable — Roblox debugging without logs is miserable

---

## Commenting Strategy

Since you want to learn from the code, every file will have:

1. **File header** — What this file does, what service it runs on (server/client/shared), and which doc it references
2. **Section comments** — Break the file into logical sections with `-- ============` dividers
3. **"WHY" comments** — Explain design decisions, not just what the code does
4. **Beginner callouts** — `-- NOTE FOR BEGINNERS:` blocks explaining Roblox-specific patterns (WaitForChild, RemoteEvents, etc.)
5. **Link to docs** — Comments reference which section of which doc a piece of logic implements

Example style:
```lua
-- Fire the remote to ALL connected clients so their HUDs update.
-- We use FireAllClients (not FireClient) because every player needs
-- to know someone was eliminated — it updates the "players remaining"
-- counter and shows the kill feed message.
-- Reference: 06-DATA-AND-PROGRESSION.md → RemoteEvent Definitions
remotes.PlayerEliminated:FireAllClients(player.UserId, reason, placement)
```

---

## Gotchas & Things to Watch For

### Roblox-Specific Pitfalls
1. **`WaitForChild` vs direct indexing** — Always use `WaitForChild()` on client scripts when accessing replicated objects. Direct indexing (e.g., `game.ReplicatedStorage.Shared`) will error if the object hasn't replicated yet. This is the #1 Roblox beginner bug.

2. **Server vs Client boundary** — Server scripts CANNOT access `Players.LocalPlayer` or any client-only services (UserInputService, etc.). Client scripts CANNOT access `DataStoreService` or `ServerStorage`. Mixing these up produces silent nil errors.

3. **Rojo file naming** — `Name.server.luau` becomes a Script, `Name.client.luau` becomes a LocalScript, `Name.luau` becomes a ModuleScript. Get the suffix wrong and the code runs on the wrong side (or not at all).

4. **RemoteEvent security** — NEVER trust data from `OnServerEvent`. A hacked client can send anything. Always validate: is this player alive? Do they have enough coins? Are they near that topping? Every server handler needs input validation.

5. **DataStore rate limits** — Roblox limits DataStore to ~60 requests/minute per key. Our 60-second auto-save interval respects this. Don't save on every coin change — batch updates.

6. **BindToClose** — When the server shuts down, you get ~30 seconds to save all player data. If you don't handle this, players lose progress on server restart/crash. This is a must-have, not a nice-to-have.

7. **Humanoid quirks** — Setting `WalkSpeed` or `JumpPower` must happen after the character loads. Use `CharacterAdded` event and wait for `Humanoid` to exist.

### Architecture Gotchas
8. **Module require order** — If Module A requires Module B and Module B requires Module A, you get a circular dependency deadlock. Our architecture avoids this: shared modules are leaves (they don't require other game modules), server modules can require shared, client modules can require shared.

9. **Heartbeat budget** — `RunService.Heartbeat` fires every frame (~60fps). If you do expensive work (checking all platform distances for all players), you'll lag the server. Use time-gated checks (every 0.5s) for non-critical things like placement ranking.

10. **Part count / performance** — Each platform is a Part. 60+ platforms × 20+ players' effects = lots of instances. Use `Workspace.StreamingEnabled` if performance dips. Destroy worlds when not in use.

11. **Network ownership** — Roblox assigns physics ownership of parts to nearby players. Moving platforms should have their network ownership set to the server (`part:SetNetworkOwner(nil)`) so they move smoothly for everyone, not jittering based on one client's framerate.

12. **Anti-camp timer** — The 15-second anti-camp mechanic (platform crumbles if you sit too long) needs to track per-player-per-platform, not globally. Otherwise one player standing on a platform would crumble it for everyone.

### Development Process Gotchas
13. **Test with 1 player first, then multi** — Most bugs show up in single-player testing. Use Studio's "Test" tab → "Start" for solo, then "Start Server" with 2+ players for multiplayer testing.

14. **Studio Play vs Published** — DataStoreService only works if "Enable Studio Access to API Services" is ON in Game Settings. Without it, data saves silently fail.

15. **Keep files under 300 lines** — Per the project conventions. If a module grows too large, split it. Large files are hard to debug and slow to iterate on.

16. **The "it works on my screen" trap** — Cosmetics and effects must be applied server-side to replicate to all clients. If you apply them client-side, only that player sees them.

---

## Dependency Graph (Build Order Matters)

```
Phase 0: Project Skeleton
  └── Phase 1: Constants, Types, Logger, Remotes
       ├── Phase 2: GameManager + RoundManager (state machine)
       │    └── Phase 3: WorldManager + Static Platforms
       │         ├── Phase 4: Rising Floor + Elimination
       │         │    └── Phase 5: All Platform Types
       │         │         └── Phase 6: Toppings
       │         │              └── Phase 7: Hazards
       │         └── Phase 8: Data Persistence (can start after Phase 4)
       │              └── Phase 9: Shop & Cosmetics
       └── Phase 10: Worlds 2 & 3 + Polish (after everything above)
```

Note: Phase 8 (Data) can technically start after Phase 4, since it's
independent of platform types/toppings/hazards. A savvy dev might
interleave it to avoid a huge "data sprint" later. But for clarity
we'll do it sequentially.

---

## Estimated File Count at Completion

| Location | Files | Purpose |
|---|---|---|
| `src/server/` | 6 | GameManager, RoundManager, WorldManager, EliminationManager, DataManager, ShopManager, ToppingManager, HazardManager |
| `src/client/` | 4 | InputController, CameraController, UIController, EffectsController, HUDController |
| `src/shared/modules/` | 7 | Constants, Logger, PlatformTypes, ToppingSystem, HazardTypes, CharacterSystem, CosmeticsCatalog, WorldLayouts |
| `src/shared/remotes/` | 1 | RemoteDefinitions |
| `src/shared/types/` | 1 | GameTypes |
| `src/ui/` | 4 | HUD, ShopUI, LobbyUI, ResultsUI |
| Root | 2 | default.project.json, .gitignore |

~25 files total. Each under 300 lines. Small, focused, well-commented.
