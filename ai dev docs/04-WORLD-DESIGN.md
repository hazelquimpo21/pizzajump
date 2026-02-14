# Pizza Jump — World Design

Detailed specifications for building each world. Each world is a vertical tower of platforms that players climb. Worlds are built as Models in Roblox Studio (geometry) with code handling the dynamic elements (hazards, moving platforms, power-ups).

---

## World Architecture

### Dimensions
Each world is a vertical column:
- **Width:** 80 studs × 80 studs (play area)
- **Height:** 400 studs (bottom to top)
- **Walls:** Invisible walls on all 4 sides to prevent falling off laterally (but with gaps for visual hazards to enter)
- **Floor:** Rising danger starts at Y = 0 and moves upward
- **Finish:** Reaching Y = 380+ counts as finishing

### Vertical Sections
Each world is divided into 4 difficulty sections:

| Section | Height Range | Difficulty | Platform Density | Hazard Frequency |
|---|---|---|---|---|
| Bottom | Y 0–100 | Easy | High (close together) | Low (1-2 hazards) |
| Lower-Mid | Y 100–200 | Medium | Medium | Medium (3-4 hazards) |
| Upper-Mid | Y 200–300 | Hard | Lower (bigger gaps) | High (5-6 hazards) |
| Top | Y 300–400 | Expert | Sparse (requires skill) | Very High (constant) |

### Spawn Points
- 30 spawn points at Y = 10, spread across the bottom of the world
- Each spawn is a small platform (8×8 studs) connected to the first set of climbing platforms
- Spawn platforms are immune to the rising floor for 10 seconds (grace period)

---

## World 1: The Kitchen

### Visual Theme
- **Walls:** Stainless steel with rivets, industrial kitchen look
- **Lighting:** Warm amber from above (overhead fluorescents), orange glow from below (oven)
- **Ambient particles:** Flour dust floating, steam wisps
- **Background:** Shelves with jars, hanging utensils (non-interactive, visual only)

### Platform Palette

| Platform Visual | Type | Notes |
|---|---|---|
| Wooden cutting board | Static | Main safe platform, various sizes |
| Stack of plates | Static | Small, circular — precise jumping |
| Hanging pan (by handle) | Moving | Swings side to side like a pendulum |
| Rising dough ball | Bouncy | Large, squishy — launches player 2× jump height |
| Wet counter section | Slippery | Long platform but hard to stop on |
| Bread rack shelf | Crumbling | Thin shelves that break under weight |
| Conveyor belt | Conveyor | Industrial food conveyor, moves player left or right |
| Mixer bowl | Static/Bouncy | Large bowl shape — can bounce off the rounded interior |

### Platform Layout Guidelines

**Bottom Section (Y 0–100) — "The Counter"**
- Large cutting boards close together (easy jumps)
- Introduce ONE bouncy dough ball
- No hazards except ambient steam (visual only, no damage)
- Goal: Let players learn the controls comfortably
- ~20 platforms, gaps of 5-8 studs

**Lower-Mid (Y 100–200) — "The Prep Station"**
- Mix of cutting boards and plate stacks
- Introduce hanging pans (moving platforms)
- First pizza cutter hazard (slow, predictable)
- One slippery counter section
- ~18 platforms, gaps of 8-12 studs

**Upper-Mid (Y 200–300) — "The Oven Zone"**
- More crumbling bread racks
- Conveyor belts pushing toward hazards
- Multiple pizza cutters at different heights
- Steam burst hazards active
- Flame jets from walls
- ~15 platforms, gaps of 10-15 studs

**Top (Y 300–400) — "The Exhaust Hood"**
- Sparse platforms (mostly small plates and pans)
- Fast-moving platforms
- All hazard types active simultaneously
- Disappearing platforms introduced
- ~12 platforms, gaps of 12-15 studs with bouncy assists

### Topping Placement
- 🧀 Extra Cheese: Y ~80 (easy to get, introduces the system)
- 🌶️ Hot Pepper: Y ~150 (speed helps with moving platforms)
- 🍄 Mushroom: Y ~220 (double jump helps with bigger gaps)
- 🫒 Olive: Y ~300 (gravity flip for ceiling shortcut)
- 🍍 Pineapple: Y ~250 and ~350 (risk/reward in hard sections)

---

## World 2: The Dining Table

### Visual Theme
- **Environment:** Giant dining table from a tiny pizza's perspective — everything is oversized
- **Walls:** Table edge on sides, background shows a restaurant dining room (blurred/distant)
- **Lighting:** Warm candlelight flicker, overhead chandelier glow
- **Ambient particles:** Breadcrumb particles, gentle candle flame flicker
- **Scale:** Everything is 10× normal size relative to the pizza character

### Platform Palette

| Platform Visual | Type | Notes |
|---|---|---|
| Dinner plate | Static | Large, circular — main safe platforms |
| Overturned cup | Static | Small, requires precision |
| Folded napkin | Static/Glider | Some napkins unfold to glide (special mechanic) |
| Salt shaker | Static | Tall, thin — land on the cap |
| Pepper grinder | Moving | Slowly rotates, stand on the top knob |
| Breadstick | Crumbling | Long and thin, snaps in the middle |
| Wine glass | Bouncy | Land on the bowl, bounces you up (glass wobbles) |
| Menu (standing) | Static | Leaning menu card, angled platform |
| Candle | Moving | Slowly rises as wax melts (platform goes up) |
| Spoon | See-saw | Balanced on a knife — one end goes up, other goes down |

### Napkin Glider Mechanic
- Special to this world only
- Certain napkin platforms, when jumped off, unfold into a glider
- Player can glide horizontally for 3-4 seconds (slow descent)
- Useful for reaching distant platforms or dodging hazards
- Limited to specific marked napkins (sparkle effect to indicate)
- Press and hold Jump while in air near a glider napkin to activate

### Platform Layout Guidelines

**Bottom (Y 0–100) — "The Place Setting"**
- Large dinner plates, easy spacing
- Introduce cups and napkins
- One wine glass bounce pad
- No active hazards
- ~20 platforms

**Lower-Mid (Y 100–200) — "The Appetizers"**
- Breadstick crumbling platforms introduced
- Spilled drink hazard (slippery pools)
- Moving pepper grinder platforms
- First napkin glider opportunity
- ~17 platforms

**Upper-Mid (Y 200–300) — "The Main Course"**
- Fork stab hazards begin (watch for shadows!)
- Spoon see-saw platforms (timing challenges)
- Giant hand sweeps through every 30 seconds
- Candle platforms slowly rising (moving upward)
- ~14 platforms

**Top (Y 300–400) — "The Check"**
- Sparse platforms — mostly small cups and salt shakers
- All hazards active, fork stabs more frequent
- Disappearing platforms (dishes being cleared)
- Napkin gliders critical for crossing large gaps
- ~11 platforms

### Topping Placement
- 🧀 Extra Cheese: Y ~90 (sticky feet on slippery spills)
- 🍄 Mushroom: Y ~160 (double jump for reaching tall salt shakers)
- 🌶️ Hot Pepper: Y ~230 (speed to outrun giant hand sweep)
- 🫒 Olive: Y ~310 (gravity flip to walk under large plates)
- 🍍 Pineapple: Y ~200 and ~360

---

## World 3: The Street

### Visual Theme
- **Environment:** Nighttime city street from ground level — everything is huge
- **Walls:** Building facades on both sides with lit windows, neon signs
- **Lighting:** Neon glow (pink, blue, green), streetlights, car headlight beams
- **Ambient particles:** Rain drops, puddle splashes, exhaust fumes
- **Sky:** Dark with city light pollution glow
- **Background:** Distant skyscrapers, moon, passing clouds

### Platform Palette

| Platform Visual | Type | Notes |
|---|---|---|
| Car roof | Static | Large, flat — main platform in lower sections |
| Awning | Bouncy | Fabric canopy that bounces players |
| Fire escape | Static | Metal grating platforms with ladders between |
| Dumpster lid | Crumbling | Rusted, breaks after weight |
| Street sign | Static | Small, must land on the flat sign part |
| Window ledge | Static | Narrow, long — precision required |
| Traffic light | Moving | Swings in the wind |
| Mailbox | Static | Small, boxy — stepping stone |
| Neon sign | Disappearing | Flickers on and off (sign turns off = platform gone) |
| Satellite dish | Bouncy | On higher buildings, big bounce |
| Flagpole | Static | Tiny platform at the top |

### Wind Gust Mechanic
- Special to this world only
- Every 20 seconds, a wind gust blows all players in a random horizontal direction
- Force: moderate (doesn't throw you off a platform, but shifts you near edges)
- Visual: rain goes sideways, awnings flap, trash blows across screen
- Audio: howling wind sound, builds 2 seconds before gust
- Duration: 3 seconds of sustained push
- Strategy: position toward center of platforms before gusts, or use gust to reach far platforms

### Platform Layout Guidelines

**Bottom (Y 0–100) — "The Parking Lot"**
- Car roofs close together (like hopping across a parking lot)
- Dumpsters and mailboxes as stepping stones
- Puddles on ground (visual only, no effect)
- Awning bounces to reach first fire escape
- ~20 platforms

**Lower-Mid (Y 100–200) — "The Storefronts"**
- Fire escapes and window ledges dominate
- First pigeons start swooping
- Neon sign platforms (disappearing)
- Wind gusts begin (mild at first)
- ~16 platforms

**Upper-Mid (Y 200–300) — "The Rooftops"**
- Gaps between buildings to cross
- Satellite dish bounces to cross gaps
- Headlight beams sweep upward (blinding)
- Traffic lights swinging in wind (moving platforms)
- Rain makes rooftop edges slippery
- ~13 platforms

**Top (Y 300–400) — "The Sky's the Limit"**
- Flagpoles and antenna tops (tiny platforms)
- Strong wind gusts (more frequent)
- All hazards active
- Satellite dishes + wind gusts = skillful long-distance bounces
- ~10 platforms

### Topping Placement
- 🌶️ Hot Pepper: Y ~70 (speed across car roofs)
- 🧀 Extra Cheese: Y ~180 (grip on rain-slicked ledges)
- 🍄 Mushroom: Y ~250 (double jump between rooftops)
- 🫒 Olive: Y ~320 (gravity flip under roof overhangs)
- 🍍 Pineapple: Y ~150 and ~380

---

## World Building Notes for Roblox Studio

### What's Built in Code vs. Studio

**Built in Studio (manually):**
- World geometry: walls, decorative elements, background scenery
- Platform base models (as templates in ServerStorage)
- Lighting setup per world
- Skybox configuration
- Sound objects placed in world

**Built/Controlled in Code:**
- Platform spawning from templates + configuration data
- Hazard activation and timing
- Topping spawning and respawning
- Rising floor movement
- Moving/bouncy/crumbling platform behaviors
- Wind gusts, environmental effects

### World Model Structure in Studio

Each world is stored as a Model in ServerStorage:

```
ServerStorage/
├── Worlds/
│   ├── Kitchen/
│   │   ├── Geometry (Folder) -- walls, decorations, non-interactive
│   │   ├── PlatformTemplates (Folder) -- base parts for each platform type
│   │   ├── HazardTemplates (Folder) -- base parts for each hazard
│   │   ├── SpawnPoints (Folder) -- 30 spawn location Parts
│   │   └── WorldConfig (Configuration) -- attributes for world settings
│   ├── DiningTable/
│   │   └── (same structure)
│   └── Street/
│       └── (same structure)
```

### When a round starts, the server:
1. Clones the selected World model from ServerStorage
2. Parents it to Workspace
3. Reads WorldConfig for settings
4. Spawns platforms from PlatformTemplates according to layout data
5. Initializes hazards
6. Spawns toppings
7. Starts the rising floor
8. Teleports players to spawn points

### When a round ends, the server:
1. Destroys the World model from Workspace
2. Returns all players to the Lobby
