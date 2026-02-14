# Pizza Jump — Data & Progression

Specifications for player data storage, progression systems, and server-side data management.

---

## 1. Player Data Schema

All player data is stored using Roblox **DataStoreService**. One DataStore per data category.

### Primary Player Data

**DataStore Name:** `PlayerData_v1`
**Key Pattern:** `Player_{UserId}`

```
PlayerData:
  -- Currency
  coins: number (default: 0)

  -- Stats
  totalRoundsPlayed: number (default: 0)
  totalWins: number (default: 0)  -- 1st place finishes
  totalTopThree: number (default: 0)  -- top 3 finishes
  totalEliminations: number (default: 0)  -- times eliminated
  totalToppingsCollected: number (default: 0)
  totalTimeAlive: number (seconds, default: 0)
  highestPlacement: number (default: 0)  -- best ever rank in a round
  longestSurvival: number (seconds, default: 0)  -- longest time alive in one round

  -- Inventory (owned cosmetics)
  ownedSkins: {string} (default: {"ClassicSlice"})
  ownedHats: {string} (default: {})
  ownedTrails: {string} (default: {})
  ownedEmotes: {string} (default: {"Wave"})

  -- Equipped cosmetics
  equippedSkin: string (default: "ClassicSlice")
  equippedHat: string (default: "")  -- empty = no hat
  equippedTrail: string (default: "")  -- empty = no trail
  equippedEmotes: {string} (default: {"Wave"})  -- up to 4 equipped emotes

  -- Settings
  musicVolume: number (default: 0.8, range 0-1)
  sfxVolume: number (default: 1.0, range 0-1)
  cameraSensitivity: number (default: 0.6, range 0-1)
  showPlayerNames: boolean (default: true)
  screenShake: boolean (default: true)
  lowQualityMode: boolean (default: false)

  -- Meta
  firstJoinDate: string (ISO date)
  lastJoinDate: string (ISO date)
  dataVersion: number (default: 1)  -- for future migrations
```

---

## 2. Data Manager (Server Module)

### Core Responsibilities
- Load player data on join
- Save player data periodically and on leave
- Handle data migration between versions
- Validate all data before saving
- Retry failed saves

### Data Flow

```
Player Joins:
  1. DataManager:LoadPlayerData(player)
  2. Attempt to read from DataStore (with retries)
  3. If no data exists → create default data
  4. If data version is old → run migrations
  5. Cache data in server memory
  6. Fire PlayerDataLoaded event

During Game:
  - All data changes go through DataManager methods
  - Changes update the in-memory cache immediately
  - Auto-save every 60 seconds for active players

Player Leaves:
  1. DataManager:SavePlayerData(player)
  2. Final save attempt with retries
  3. Clear in-memory cache

Server Shutdown:
  1. game:BindToClose() → save ALL player data
  2. Give up to 30 seconds for saves to complete
```

### Key Methods

```
DataManager:
  LoadPlayerData(player: Player) → PlayerData
  SavePlayerData(player: Player) → boolean
  GetPlayerData(player: Player) → PlayerData (from cache)

  -- Currency
  AddCoins(player: Player, amount: number, reason: string) → boolean
  RemoveCoins(player: Player, amount: number, reason: string) → boolean
  GetCoins(player: Player) → number

  -- Inventory
  PurchaseItem(player: Player, itemId: string, category: string) → boolean
  EquipItem(player: Player, itemId: string, category: string) → boolean
  OwnsItem(player: Player, itemId: string, category: string) → boolean

  -- Stats
  RecordRoundResult(player: Player, roundData: RoundResultData)
  GetStats(player: Player) → PlayerStats
```

### Data Validation Rules
- Coins can never go below 0
- Equipped items must be in owned inventory
- Stats can only increase (never decrease)
- Data version must be a positive integer
- All string values must be from predefined enums (no arbitrary strings)

### Error Handling
- DataStore read fails → retry 3 times with exponential backoff (1s, 2s, 4s)
- If all retries fail → use default data, flag player as "unsaved"
- DataStore write fails → retry 3 times, queue for next auto-save
- On BindToClose → attempt all pending saves, prioritize players with unsaved changes

---

## 3. Shop Transaction System

### Purchase Flow (Server-Side Only)

```
Client clicks "BUY":
  1. Client fires RemoteEvent: RequestPurchase(itemId, category)
  2. Server validates:
     a. Item exists in catalog
     b. Player doesn't already own it
     c. Player has enough coins
     d. Item is purchasable (not exclusive/limited)
  3. If valid:
     a. Remove coins from player data
     b. Add item to player inventory
     c. Save player data
     d. Fire RemoteEvent: PurchaseSuccess(itemId, category, newCoinBalance)
  4. If invalid:
     a. Fire RemoteEvent: PurchaseError(reason)
```

### Equip Flow

```
Client clicks "EQUIP":
  1. Client fires RemoteEvent: RequestEquip(itemId, category)
  2. Server validates:
     a. Player owns the item
     b. Item is valid for the category
  3. If valid:
     a. Update equipped item in player data
     b. Update character appearance
     c. Fire RemoteEvent: EquipSuccess(itemId, category)
```

### Catalog Data Structure

```
CatalogItem:
  id: string (unique identifier, e.g., "DeepDish", "ChefHat")
  name: string (display name)
  description: string
  category: "Skin" | "Hat" | "Trail" | "Emote"
  price: number (coins)
  rarity: "Common" | "Uncommon" | "Rare" | "Epic" | "Legendary"
  isDefault: boolean (given to all players for free)
  assetId: string? (Roblox asset ID for the model/texture/animation)
```

The full catalog is defined as a shared constant module so both server and client can reference it.

---

## 4. Coin Economy Balance

### Earning Rates
- Average round length: ~3 minutes
- Average placement for typical player: 10th-15th out of 20 (15-25 coins)
- Including bonuses: ~25-35 coins per round
- Per hour (assuming ~15 rounds/hour): 375-525 coins/hour

### Pricing Tiers vs. Earning
| Rarity | Price Range | Time to Earn |
|---|---|---|
| Common | 100-150 | 15-30 min |
| Uncommon | 200-400 | 30-80 min |
| Rare | 500-800 | 1-2 hours |
| Epic | 1000-2500 | 2-5 hours |
| Legendary | 3500-5000 | 7-12 hours |

This ensures:
- New players can buy their first item within 1-2 play sessions
- Legendary items feel aspirational but achievable
- Players always feel like they're making progress toward something

---

## 5. Leaderboards

### In-Game Leaderboard (Lobby Wall)

Display top 100 players for each category:
- **Most Wins** (total 1st place finishes)
- **Longest Survivor** (longest single round survival time)
- **Most Rounds Played** (dedication!)

**DataStore:** `Leaderboard_v1`
- Updated at the end of each round for participating players
- Use OrderedDataStore for sorted leaderboard queries
- Cache in memory, refresh every 60 seconds
- Show player name + stat value

### Roblox In-Experience Leaderboard
- Standard Roblox leaderboard (top-right of screen)
- Shows: Coins, Wins
- Updated in real-time using leaderstats

```
Implementation:
  Player joins → create "leaderstats" Folder in player
  Add IntValue "Coins" = player's coin count
  Add IntValue "Wins" = player's win count
  Update on data change
```

---

## 6. Session Data (Not Persisted)

Some data only lives for the current game session and doesn't need DataStore.

```
SessionData:
  currentRound: number (which round this session)
  isReady: boolean (queued for next round)
  isAlive: boolean (in current round)
  currentPlacement: number (live ranking)
  activeTopping: ToppingType? (currently active power-up)
  toppingTimeRemaining: number
  currentWorld: string (which world they're in)
  spawnPoint: Vector3
  lastPlatformTime: number (for anti-camp detection)
  lastPlatformId: string (for anti-camp detection)
  immunityTimer: number (knockback immunity cooldown)
```

This data is stored in a server-side dictionary keyed by player, created on round start, cleared on round end.

---

## 7. Anti-Cheat Basics

Since this is an MVP, implement basic server-side validation:

### Server Authority
- Server controls: round state, elimination, coin awards, shop purchases
- Client controls: input only (movement, jump, camera)
- Server validates: topping pickups (proximity check), position reasonableness

### Basic Checks
- **Speed check:** If player moves faster than max WalkSpeed × 1.5, flag/correct
- **Teleport check:** If player position jumps more than 50 studs in one frame, flag/correct
- **Purchase validation:** All coin transactions server-side, never trust client coin count
- **Topping pickup validation:** Server checks player is within 10 studs of topping before granting

### What NOT to worry about for MVP
- Perfect movement anti-cheat (hard problem, do later)
- Client-side prediction/reconciliation (add if needed)
- Reporting system (add later)

---

## 8. RemoteEvent Definitions

All client ↔ server communication:

### Server → Client (Events)

```
GameStateChanged(state: string, data: {})
  -- Sent when round state changes (LOBBY, COUNTDOWN, PLAYING, ROUND_END)

PlayerEliminated(playerId: number, reason: string, placement: number)
  -- Sent to all clients when someone is eliminated

ToppingCollected(playerId: number, toppingType: string)
  -- Sent to all clients when someone picks up a topping

ToppingExpired(playerId: number)
  -- Sent to all clients when a topping effect ends

CoinUpdate(newBalance: number)
  -- Sent to the specific player when their coins change

PurchaseSuccess(itemId: string, category: string, newBalance: number)
PurchaseError(reason: string)
EquipSuccess(itemId: string, category: string)

RoundResults(placements: {{playerId: number, name: string, placement: number, coins: number}})
  -- Full results at end of round

EliminationWarning(secondsRemaining: number)
  -- Sent to the player currently in last place

HazardWarning(hazardId: string, position: Vector3, delay: number)
  -- Tells clients to show visual warning for incoming hazard
```

### Client → Server (Events)

```
RequestReady()
  -- Player wants to queue for next round

RequestPurchase(itemId: string, category: string)
  -- Player wants to buy an item

RequestEquip(itemId: string, category: string)
  -- Player wants to equip an owned item

RequestToppingPickup(toppingId: string)
  -- Player thinks they touched a topping (server validates)

UpdateSettings(settings: {})
  -- Player changed their settings
```

---

## 9. Data Migration Strategy

When the data schema changes in future updates:

```
-- In DataManager, after loading raw data:
function migrateData(data)
  if data.dataVersion == nil or data.dataVersion < 1 then
    -- v0 → v1: Initial schema
    data.dataVersion = 1
  end

  if data.dataVersion == 1 then
    -- v1 → v2: Example future migration
    -- data.newField = defaultValue
    -- data.dataVersion = 2
  end

  return data
end
```

Always increment dataVersion. Never delete fields, only add or transform.
