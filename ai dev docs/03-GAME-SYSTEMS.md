# Pizza Jump — Game Systems

Detailed specifications for all core gameplay systems. Each section includes the data structures, logic flow, and implementation notes needed to code the system.

---

## 1. Round System

### States
The game follows a state machine with these states:

```
LOBBY → COUNTDOWN → PLAYING → ROUND_END → LOBBY
```

| State | Duration | What Happens |
|---|---|---|
| `LOBBY` | Until 4+ players ready (min 15s, max 60s wait) | Players in Pizzeria hub, can browse shop, emote |
| `COUNTDOWN` | 5 seconds | Players teleported to world start, frozen in place, "3-2-1-GO!" on screen |
| `PLAYING` | Until 1 player remains or 5 min max | Active gameplay, eliminations happening |
| `ROUND_END` | 10 seconds | Show results screen, award coins, then back to lobby |

### Round Manager Logic (Server)

```
RoundManager:
  currentState: GameState
  currentWorld: string (randomly selected from available worlds)
  activePlayers: {Player}
  eliminatedPlayers: {Player}
  roundTimer: number
  eliminationTimer: number (resets every 60 seconds)
  risingFloorHeight: number

  Events:
    RoundStateChanged(newState) → fires to all clients
    PlayerEliminated(player, reason) → fires to all clients
    RoundResults(placements) → fires to all clients
```

### World Selection
- Each round randomly selects one of the 3 worlds
- Weighted random: slightly favor worlds the server hasn't played recently
- All players play the same world per round

### Rising Floor (The Danger)
- Starts at the bottom of the world
- Rises at a configurable speed that **increases over time**
- Speed formula: `baseSpeed + (elapsedTime * accelerationRate)`
- Touching the floor = instant elimination
- Visual: glowing red/orange kill plane with particle effects above it
- The floor has a "warning zone" above it (10 studs) that plays a sizzle sound and makes screen edges glow red

```
Rising Floor Config:
  Kitchen:
    baseSpeed: 1 stud/second
    accelerationRate: 0.05 studs/second²
    visual: "OvenFlames" — orange fire particles
  DiningTable:
    baseSpeed: 1.2 studs/second
    accelerationRate: 0.06 studs/second²
    visual: "SweepingArm" — giant arm model moving upward
  Street:
    baseSpeed: 1.5 studs/second
    accelerationRate: 0.07 studs/second²
    visual: "MarinaraFlood" — red liquid with splash particles
```

---

## 2. Player Character System

### Pizza Slice Character
Players don't use the default Roblox character. Instead, they use a custom pizza slice model.

**Character Structure:**
```
PizzaCharacter (Model)
├── HumanoidRootPart (invisible, handles physics)
├── Humanoid (for built-in movement/jumping)
├── PizzaBody (MeshPart — triangular pizza slice shape)
├── LeftEye (Part — googly eye)
├── RightEye (Part — googly eye)
├── LeftLeg (Part — small stubby leg)
├── RightLeg (Part — small stubby leg)
└── CheeseEffect (ParticleEmitter — subtle cheese drip while moving)
```

**Movement Stats (Base):**
```
WalkSpeed: 20 (Roblox default is 16)
JumpPower: 60 (Roblox default is 50)
Gravity: 120 (set in Workspace, default is 196.2 — lower = floatier jumps)
```

**Why custom gravity:** Lower gravity makes platforming feel better. Jumps are more forgiving and the game feels bouncier, which matches the pizza theme.

### Character Loading
1. Player joins → default character is loaded
2. Server replaces it with PizzaCharacter model
3. Apply any equipped cosmetics (skin, hat, trail)
4. Character respawns at current checkpoint or world start

### Death / Elimination
- **Falling into rising floor** → eliminated from round
- **Falling off the map** (below floor) → eliminated
- **No fall damage** — only hazards and the rising floor can eliminate you
- On elimination: play dramatic animation, camera pulls back, show "EATEN!" text, transition to spectator

### Spectator Mode
- Eliminated players can spectate remaining players
- Free camera or follow-a-player mode
- Can still chat
- See a mini leaderboard of remaining players

---

## 3. Platform System

Platforms are the core building blocks of each world. Each platform type has specific behaviors.

### Platform Types

| Type | Behavior | Visual Cue |
|---|---|---|
| `Static` | Doesn't move, always solid | Solid color, no effects |
| `Moving` | Moves on a path (horizontal, vertical, or circular) | Slight glow, arrows showing direction |
| `Bouncy` | Launches player higher than normal jump | Bright color, squish animation on land |
| `Crumbling` | Breaks 1.5 seconds after being stepped on, respawns after 5 seconds | Cracked texture, shakes when stood on |
| `Slippery` | Reduced friction, player slides | Glossy/wet appearance, drip particles |
| `Conveyor` | Moves player in a direction while standing on it | Animated arrows on surface |
| `Disappearing` | Toggles visibility on a timer (2s on, 2s off) | Flashes/pulses before disappearing |

### Platform Data Structure

```
PlatformConfig:
  type: PlatformType (enum of types above)
  size: Vector3
  position: Vector3
  -- Type-specific properties:
  moveSpeed: number? (for Moving)
  movePath: {Vector3}? (for Moving — waypoints)
  bounceMultiplier: number? (for Bouncy, default 2.0)
  crumbleDelay: number? (for Crumbling, default 1.5)
  respawnDelay: number? (for Crumbling, default 5.0)
  friction: number? (for Slippery, default 0.1)
  conveyorDirection: Vector3? (for Conveyor)
  conveyorSpeed: number? (for Conveyor)
  visibleDuration: number? (for Disappearing)
  hiddenDuration: number? (for Disappearing)
```

### Platform Placement Rules
- Minimum gap between platforms: player must be able to reach with a normal jump
- Maximum gap: player needs a running jump or a Bouncy platform
- Horizontal max gap: ~15 studs (with WalkSpeed 20 and JumpPower 60)
- Vertical max gap: ~12 studs (normal jump), ~24 studs (from Bouncy platform)
- Each world should have 50-80 platforms in the main path
- Alternate paths (shortcuts for skilled players) should have 10-15 additional platforms

---

## 4. Topping Power-Up System

Toppings are collectible power-ups scattered across platforms. They float and rotate above platforms with a glowing particle effect.

### Topping Types

| Topping | Icon Color | Duration | Effect |
|---|---|---|---|
| Extra Cheese 🧀 | Yellow glow | 8 seconds | Sticky landing — player doesn't slide on any surface, perfect grip |
| Hot Pepper 🌶️ | Red glow | 6 seconds | Speed boost (WalkSpeed × 1.5) + fire trail visual behind player |
| Mushroom 🍄 | Brown/white glow | 10 seconds | Double jump — press jump again in midair for a second jump |
| Olive 🫒 | Dark green glow | 5 seconds | Gravity flip — player walks on ceilings/undersides of platforms |
| Pineapple 🍍 | Golden glow | Instant | Random effect — 50% chance helpful (any of the above), 50% chance harmful (inverted controls for 4 seconds, shrink to half size for 5 seconds, or screen goes blurry for 3 seconds) |

### Power-Up Rules
- Only ONE topping active at a time (new one replaces current)
- Pineapple is the exception — it applies instantly and can stack with current topping
- Toppings respawn 15 seconds after being collected
- ~3-5 toppings per world section (not too common, not too rare)
- Topping indicators show on HUD: icon + remaining time bar

### Power-Up Data Structure

```
ToppingConfig:
  type: ToppingType (enum)
  duration: number (seconds, 0 for instant)
  spawnPosition: Vector3
  respawnDelay: number (default 15)

ActiveTopping:
  type: ToppingType
  remainingTime: number
  player: Player

-- For Pineapple random effects:
PineappleEffect (enum):
  GOOD_CHEESE
  GOOD_PEPPER
  GOOD_MUSHROOM
  GOOD_OLIVE
  BAD_INVERTED_CONTROLS (4 seconds)
  BAD_SHRINK (5 seconds, character scales to 0.5)
  BAD_BLUR (3 seconds, camera blur effect)
```

### Server-Side Validation
- Server tracks which toppings exist and their cooldowns
- Client sends "I touched topping X" → server validates proximity → server applies effect
- Never trust client to apply power-up effects directly

---

## 5. Elimination System

### Timed Elimination
- Every 60 seconds during PLAYING state, the player in last place is eliminated
- "Last place" = lowest Y position (height) among living players
- If tied, the player who has been lowest for the longest time is eliminated
- Warning: at 10 seconds before elimination, the lowest player gets a red screen border and warning sound
- At elimination: dramatic zoom, "EATEN!" text with bite animation, player enters spectator

### Other Elimination Triggers
- Touching the rising floor
- Falling below the world bounds (Y < -50)
- Disconnecting (counts as eliminated, no coins awarded)

### Anti-Camping
- If a player stays on the same platform for more than 15 seconds, their platform starts to crumble regardless of type
- Visual warning at 10 seconds (platform shakes)

### Elimination Event Data
```
EliminationData:
  player: Player
  reason: "timed" | "floor" | "fell" | "disconnect"
  placement: number (e.g., 15th out of 20)
  coinsEarned: number
  timeAlive: number (seconds)
```

---

## 6. Hazard System

Each world has unique hazards. All hazards are server-authoritative with client-side visual predictions.

### Hazard Types by World

**Kitchen:**
| Hazard | Behavior | Warning | Damage |
|---|---|---|---|
| Pizza Cutter | Horizontal blade sweeps across at set heights, moves back and forth | Visible spinning blade approaching, audio cue | Knockback (pushes player off platform, doesn't directly eliminate) |
| Steam Burst | Vertical steam jets from pipes, activates periodically | Pipe rattles 1 second before burst, subtle particle leak | Launches player upward uncontrollably for 2 seconds |
| Flame Jets | Short bursts of fire from oven vents on walls | Orange glow brightens 0.5s before burst | Knockback away from wall |

**Dining Table:**
| Hazard | Behavior | Warning | Damage |
|---|---|---|---|
| Fork Stab | Giant fork stabs down from above at marked locations | Shadow appears on platform 1.5 seconds before stab | Eliminates if hit directly, knockback if near miss |
| Giant Hand | Sweeps horizontally across the play area every 30 seconds | Fingers appear at edge of screen 2 seconds before sweep | Knockback in sweep direction |
| Spilled Drink | Liquid pools on certain platforms | Visible wet texture | Makes platform Slippery type for 10 seconds |

**Street:**
| Hazard | Behavior | Warning | Damage |
|---|---|---|---|
| Pigeons | Fly in from edges, home toward nearest player | Cooing sound + shadow 1 second before arrival | Knockback, knocks off platform |
| Headlights | Car headlight beams sweep up the world periodically | Engine sound + light beam visible approaching | 2-second screen whiteout (can't see, can still move) |
| Rain | Periodic rain makes certain platforms temporarily slippery | Clouds darken above affected area | Slippery surfaces for duration |

### Hazard Implementation Pattern

```
HazardBase:
  type: string
  warningDuration: number (seconds before hazard activates)
  activeDuration: number (seconds hazard is active)
  cooldown: number (seconds between activations)
  affectedArea: Region3 or Part

  Methods:
    warn() -- show visual/audio warning to nearby clients
    activate() -- server enables hitbox/effect
    deactivate() -- server disables hitbox/effect
    onPlayerHit(player) -- apply knockback/elimination/effect
```

### Knockback Formula
- Direction: away from hazard center
- Force: `knockbackPower * direction.Unit`
- Default knockback power: 80
- Player is stunned (no input) for 0.3 seconds during knockback
- Cannot be knocked back again for 1 second (immunity frames)

---

## 7. Camera System (Client)

### Camera Behavior
- **Third-person follow camera** — camera sits behind and slightly above the player
- Camera offset: `Vector3.new(0, 8, 15)` from character (behind and above)
- Camera always looks slightly ahead of player movement direction
- Smooth lerp between positions (no jarring snaps)

### Camera Events
- **On big jump:** camera pulls back slightly (zoom out)
- **On near miss (close to hazard):** slight screen shake
- **On elimination:** camera zooms out dramatically, slow motion for 1 second
- **On power-up collect:** brief zoom-in pulse
- **On rising floor nearby:** camera tilts slightly to show floor approaching from below

### Spectator Camera
- After elimination, switches to following another player
- Left/right click or Q/E to cycle through remaining players
- Free cam option with WASD

---

## 8. Audio System

### Music
- Each world has its own background track (looping)
- Music speeds up slightly as fewer players remain
- Lobby has a chill pizza parlor vibe track
- Music crossfades between states (2-second fade)

### Sound Effects (organized by trigger)

```
Sounds:
  -- Player actions
  Jump: short bouncy "boing"
  Land: soft thud
  DoubleJump: higher-pitched boing with sparkle
  CollectTopping: cheerful "ding" + sizzle

  -- Hazards
  HazardWarning: rising tone alert
  HazardHit: impact thud + "oof"
  Knockback: whoosh

  -- Round events
  Countdown321: tick sounds
  CountdownGo: air horn / bell
  EliminationWarning: heartbeat getting faster
  Eliminated: dramatic bite chomp sound
  Victory: triumphant fanfare + crowd cheer

  -- UI
  ButtonClick: soft click
  PurchaseSuccess: cash register cha-ching
  ErrorBuzz: gentle buzz for failed actions

  -- Ambient (per world)
  Kitchen: sizzling, clanking, oven rumble
  DiningTable: murmur of conversation, clinking glasses
  Street: traffic, distant music, rain
```

### Audio Implementation
- Use SoundService for global sounds (music, UI)
- Use Part-based Sound objects for positional audio (hazards, ambient)
- Sound volumes should be configurable in settings
- All sounds loaded via SoundService:PreloadAsync() at game start
