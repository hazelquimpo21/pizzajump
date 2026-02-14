# Pizza Jump — UI & Cosmetics Shop

Specifications for all user interface elements and the cosmetics progression system.

---

## 1. HUD (In-Game)

### Layout (during PLAYING state)

```
┌─────────────────────────────────────────────┐
│ [Players Left: 15/20]          [Time: 2:34] │
│                                              │
│                                              │
│                                              │
│                                              │
│ [Active Topping]                             │
│ [🍄 Mushroom ████░░ 6s]                     │
│                                              │
│              ⚠️ RISING FLOOR NEARBY ⚠️       │
│ [Placement: 8th]               [Coins: 150] │
└─────────────────────────────────────────────┘
```

### HUD Elements

| Element | Position | Details |
|---|---|---|
| Players Remaining | Top-left | "15/20" — updates on each elimination |
| Round Timer | Top-right | Counts up from 0:00, shows elapsed time |
| Active Topping | Bottom-left | Icon + name + time remaining bar. Hidden if no topping active. Pulses when < 2 seconds remaining |
| Placement Indicator | Bottom-left | "8th" — your current ranking by height. Updates every 0.5 seconds |
| Coin Counter | Bottom-right | Running total of coins earned this round so far |
| Floor Warning | Bottom-center | Appears when rising floor is within 30 studs of player. Pulses red. |
| Elimination Warning | Full screen border | Red vignette border when you're in last place and elimination timer < 10 seconds |

### Elimination Countdown
When elimination timer < 10 seconds, the last-place player sees:
- Red pulsing border around screen
- "LAST PLACE! 8... 7... 6..." countdown text
- Heartbeat sound effect increasing in speed
- If they pass someone: border disappears, relief sound plays

### Kill Feed
Top-right, below timer:
- "🍕 PlayerName was EATEN!" — shows for 3 seconds per elimination
- Stack up to 3, oldest fades first

---

## 2. Lobby UI

### The Pizzeria (Lobby Space)
Physical space in the game where players hang out between rounds.

**Features:**
- Pizza shop interior with tables, counter, arcade machines (decorative)
- Shop kiosk (interactable — opens Shop UI)
- "Ready Up" button/pad on the floor (step on it to ready up)
- Leaderboard display on the wall (top players by total wins)
- Practice area: small platforming course to warm up

### Lobby Screen UI

```
┌─────────────────────────────────────────────┐
│              🍕 PIZZA JUMP 🍕               │
│                                              │
│    Next round starting in: 15 seconds        │
│    Players ready: 8/20                       │
│                                              │
│  [🛒 SHOP]    [👤 CUSTOMIZE]    [📊 STATS]  │
│                                              │
│  Recent Results:                             │
│    🥇 CoolPlayer123                          │
│    🥈 PizzaFan99                             │
│    🥉 JumpMaster                             │
└─────────────────────────────────────────────┘
```

---

## 3. Cosmetics Shop

### Shop UI Layout

```
┌─────────────────────────────────────────────┐
│  🛒 PIZZA SHOP                    Coins: 500│
│                                              │
│  [Skins] [Hats] [Trails] [Emotes]          │
│  ─────────────────────────────────────────── │
│                                              │
│  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐       │
│  │      │ │      │ │      │ │      │       │
│  │ IMG  │ │ IMG  │ │ IMG  │ │ IMG  │       │
│  │      │ │      │ │      │ │      │       │
│  ├──────┤ ├──────┤ ├──────┤ ├──────┤       │
│  │Name  │ │Name  │ │Name  │ │Name  │       │
│  │100 🪙│ │250 🪙│ │500 🪙│ │ OWNED│       │
│  └──────┘ └──────┘ └──────┘ └──────┘       │
│                                              │
│  ◄ Page 1 of 3 ►                            │
│                                              │
│  Selected: Deep Dish                         │
│  "A thick-crusted Chicago classic"           │
│  [BUY - 250 🪙]  [PREVIEW]                  │
└─────────────────────────────────────────────┘
```

### Shop Tabs
1. **Skins** — different pizza slice appearances
2. **Hats** — accessories worn on top of the slice
3. **Trails** — visual trail behind player while moving/jumping
4. **Emotes** — animations the pizza can perform in lobby

### Item States
- **Locked:** Greyed out, shows price
- **Affordable:** Full color, shows price, "BUY" button enabled
- **Owned:** Checkmark overlay, "EQUIP" button instead of "BUY"
- **Equipped:** Green border, "EQUIPPED" label

### Preview System
- Clicking PREVIEW rotates the camera around the player's pizza with the item applied
- Preview is client-side only (doesn't actually equip)
- "Buy" button visible during preview

---

## 4. Cosmetics Catalog

### Skins (Pizza Types)

| Name | Description | Price | Rarity |
|---|---|---|---|
| Classic Slice | Default pepperoni slice | Free | Common |
| Margherita | Fresh basil and mozzarella | 100 | Common |
| Hawaiian | Pineapple and ham (controversial!) | 150 | Common |
| BBQ Chicken | Smoky BBQ with chicken chunks | 200 | Uncommon |
| Veggie Supreme | Loaded with vegetables | 200 | Uncommon |
| Deep Dish | Thick Chicago-style, extra chunky | 350 | Uncommon |
| NY Fold | Folded New York style slice | 350 | Uncommon |
| Calzone | Fully enclosed, mysterious | 500 | Rare |
| Pizza Roll | Tiny and round, rolls when moving | 500 | Rare |
| French Bread Pizza | Long rectangular shape | 500 | Rare |
| Frozen Pizza | Blue-tinted, frost particles | 800 | Rare |
| Gold Leaf Truffle | Gold shimmer effect, fancy | 1500 | Epic |
| Galaxy Pizza | Swirling space texture, star particles | 2500 | Epic |
| Lava Pizza | Glowing orange cracks, ember particles | 2500 | Epic |
| Rainbow Slice | Color-shifting chromatic effect | 5000 | Legendary |
| Invisible Pizza | Nearly transparent with faint outline | 5000 | Legendary |

### Hats (Accessories)

| Name | Description | Price | Rarity |
|---|---|---|---|
| Chef Hat | Classic white toque | 100 | Common |
| Party Hat | Colorful birthday cone | 100 | Common |
| Baseball Cap | Backwards cap, cool vibes | 150 | Common |
| Crown | Gold crown, feel like royalty | 300 | Uncommon |
| Top Hat | Classy and tall | 300 | Uncommon |
| Viking Helmet | Horned helmet, fierce | 400 | Uncommon |
| Halo | Floating golden ring above | 600 | Rare |
| Devil Horns | Red horns, mischievous | 600 | Rare |
| Propeller Beanie | Spinning propeller on top | 800 | Rare |
| Astronaut Helmet | Space bubble helmet | 1200 | Epic |
| Flaming Crown | Crown with animated fire | 2000 | Epic |
| Cheese Crown | Made of melted cheese, drips | 3500 | Legendary |

### Trails

| Name | Description | Price | Rarity |
|---|---|---|---|
| Marinara Drip | Red sauce drops behind you | 100 | Common |
| Cheese Stretch | Stretchy cheese trail | 150 | Common |
| Grease Stain | Greasy footprints | 150 | Common |
| Sparkles | Generic sparkle trail | 200 | Uncommon |
| Smoke | Gray smoke puffs | 200 | Uncommon |
| Fire Trail | Flaming footsteps | 500 | Rare |
| Ice Trail | Frosty blue crystals | 500 | Rare |
| Rainbow Trail | Rainbow streak behind you | 800 | Rare |
| Confetti | Colorful confetti burst | 800 | Rare |
| Lightning | Electric bolts trailing behind | 1500 | Epic |
| Galaxy Trail | Swirling stars and nebula | 2500 | Epic |
| Gold Coins | Coins spilling behind you | 4000 | Legendary |

### Emotes

| Name | Animation | Price | Rarity |
|---|---|---|---|
| Wave | Wiggles back and forth | Free | Common |
| Dance | Bounces up and down rhythmically | 100 | Common |
| Spin | Spins around rapidly | 150 | Common |
| Flex | Puffs up to look bigger | 200 | Uncommon |
| Cry | Sad face, cheese drips like tears | 200 | Uncommon |
| Dab | Does a dab (kids love this) | 300 | Uncommon |
| Disco | Disco lights + dance moves | 500 | Rare |
| Backflip | Does a backflip in place | 500 | Rare |
| Pizza Toss | Tosses tiny pizza into air and catches it | 1000 | Epic |
| Victory Pizza | Grows huge momentarily with golden glow | 3000 | Legendary |

---

## 5. Results Screen

Shown during ROUND_END state (10 seconds):

```
┌─────────────────────────────────────────────┐
│              🏆 ROUND COMPLETE! 🏆           │
│                                              │
│    🥇 1st: PlayerName         +100 🪙       │
│    🥈 2nd: PlayerName         +75 🪙        │
│    🥉 3rd: PlayerName         +50 🪙        │
│    4th: PlayerName            +35 🪙        │
│    5th: PlayerName            +25 🪙        │
│    ...                                       │
│                                              │
│    Your Placement: 8th        +15 🪙        │
│    Time Alive: 2:34                          │
│    Toppings Collected: 3                     │
│                                              │
│    Total Coins: 515 🪙                      │
│                                              │
│         [NEXT ROUND IN 8...]                 │
└─────────────────────────────────────────────┘
```

### Coin Awards by Placement

| Placement | Coins |
|---|---|
| 1st | 100 |
| 2nd | 75 |
| 3rd | 50 |
| 4th-5th | 35 |
| 6th-10th | 25 |
| 11th-15th | 15 |
| 16th-20th | 10 |
| 21st+ | 5 |

### Bonus Coins
- Topping collected: +2 per topping
- Survived 2+ minutes: +10
- Close call (avoided elimination at last second): +5
- First to reach a height milestone (Y=100, Y=200, Y=300): +15

---

## 6. Settings UI

Accessible from lobby or pause menu:

```
Settings:
  Music Volume: [████████░░] 80%
  SFX Volume:   [██████████] 100%
  Camera Sensitivity: [██████░░░░] 60%
  Show Player Names: [ON / off]
  Screen Shake: [ON / off]
  Low Quality Mode: [on / OFF]    -- reduces particles for low-end devices
```

---

## 7. UI Implementation Notes

### Technology
- All UI built with Roblox ScreenGui + Frames + TextLabels + ImageLabels
- Use TweenService for all animations (fade in/out, slides, pulses)
- UICorner for rounded corners on frames
- UIGradient for gradient backgrounds
- UIStroke for borders

### Color Palette

| Use | Color | Hex |
|---|---|---|
| Primary (Pizza Orange) | Warm orange | #FF8C42 |
| Secondary (Sauce Red) | Tomato red | #D64933 |
| Accent (Cheese Yellow) | Golden yellow | #FFCC00 |
| Background Dark | Dark brown | #2D1810 |
| Background Medium | Warm brown | #4A2C1A |
| Text Primary | White | #FFFFFF |
| Text Secondary | Cream | #FFF5E6 |
| Success | Green | #4CAF50 |
| Warning | Orange | #FF9800 |
| Danger | Red | #F44336 |
| Disabled | Gray | #888888 |
| Rarity Common | White | #FFFFFF |
| Rarity Uncommon | Green | #4CAF50 |
| Rarity Rare | Blue | #2196F3 |
| Rarity Epic | Purple | #9C27B0 |
| Rarity Legendary | Gold | #FFD700 |

### Font
- Primary: GothamBold (built into Roblox)
- Secondary: GothamMedium
- Numbers/Timer: Code (monospace, built into Roblox)

### Responsive Design
- All UI uses Scale (not Offset) for positioning where possible
- Test at 16:9 and 4:3 aspect ratios
- Mobile: buttons must be minimum 48px touch targets
- HUD elements stay in screen-safe areas (10% margin from edges)

### Animation Standards
- UI elements fade in over 0.3 seconds
- Buttons scale to 1.05× on hover, 0.95× on press
- Numbers (coins, timer) animate counting up/down
- Topping timer bar smoothly decreases
- Results screen entries animate in one-by-one (0.2s delay each)
