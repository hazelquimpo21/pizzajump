# Pizza Jump — Setup Guide

This guide walks through setting up your development environment from scratch. You need: Roblox Studio, Rojo, Git, and Claude Code.

---

## Step 1: Install Roblox Studio

1. Go to https://create.roblox.com/ and create a Roblox account (or log in)
2. Download and install **Roblox Studio**
3. Open Studio → Create New → Baseplate (just to verify it works)
4. Close it for now

---

## Step 2: Install Rojo

Rojo syncs our local code files into Roblox Studio.

### Install Rojo CLI (choose one method)

**Option A: Using Aftman (recommended Roblox toolchain manager)**
```bash
# Install Aftman first: https://github.com/LPGhatguy/aftman
aftman init
aftman add rojo-rbx/rojo
```

**Option B: Using Foreman**
```bash
# Install Foreman: https://github.com/Roblox/foreman
# Add to foreman.toml:
[tools]
rojo = { github = "rojo-rbx/rojo", version = "7.x" }

# Then run:
foreman install
```

**Option C: Direct download**
- Go to https://github.com/rojo-rbx/rojo/releases
- Download the latest release for your OS
- Add to your PATH

### Install Rojo Plugin in Roblox Studio
1. Open Roblox Studio
2. Go to **Plugins** tab → **Manage Plugins**
3. Search for "Rojo" and install it
4. You should see a Rojo panel in Studio

### Verify Rojo Works
```bash
rojo --version
# Should print something like: rojo 7.x.x
```

---

## Step 3: Initialize the Project

```bash
# Create project directory
mkdir pizza-jump
cd pizza-jump

# Initialize Git
git init

# Initialize Rojo
rojo init

# This creates default.project.json — we'll customize it
```

### Replace `default.project.json` with our structure:

```json
{
  "name": "PizzaJump",
  "tree": {
    "$className": "DataModel",

    "ServerScriptService": {
      "$className": "ServerScriptService",
      "$path": "src/server"
    },

    "StarterPlayer": {
      "$className": "StarterPlayer",
      "StarterPlayerScripts": {
        "$className": "StarterPlayerScripts",
        "$path": "src/client"
      }
    },

    "ReplicatedStorage": {
      "$className": "ReplicatedStorage",
      "Shared": {
        "$path": "src/shared"
      }
    },

    "StarterGui": {
      "$className": "StarterGui",
      "$path": "src/ui"
    },

    "Workspace": {
      "$className": "Workspace",
      "$properties": {
        "Gravity": 120
      }
    },

    "Lighting": {
      "$className": "Lighting",
      "$properties": {
        "Ambient": [0.3, 0.3, 0.3],
        "ClockTime": 14,
        "GlobalShadows": true
      }
    },

    "SoundService": {
      "$className": "SoundService"
    },

    "ReplicatedFirst": {
      "$className": "ReplicatedFirst"
    }
  }
}
```

### Create folder structure:

```bash
mkdir -p src/server
mkdir -p src/client
mkdir -p src/shared/modules
mkdir -p src/shared/remotes
mkdir -p src/shared/types
mkdir -p src/ui
mkdir -p assets
mkdir -p docs
```

---

## Step 4: Development Workflow

### Starting a dev session:

```bash
# Terminal 1: Start Rojo server
cd pizza-jump
rojo serve

# Terminal 2: Use Claude Code for writing scripts
claude
```

### In Roblox Studio:
1. Open your Pizza Jump place (or create a new Baseplate)
2. Click the **Rojo** plugin button
3. Click **Connect** — it will connect to `localhost:34872`
4. You should see your src/ files appear in the Explorer panel
5. Changes you make locally sync into Studio automatically

### Testing:
1. Write code with Claude Code → saves to src/
2. Rojo syncs to Studio automatically
3. Press **Play** in Studio to test
4. Check Output window for errors
5. Iterate

### Important Notes for New Roblox Devs:
- **Server scripts** end in `.server.luau` — they run on the Roblox server
- **Client scripts** end in `.client.luau` — they run on each player's computer
- **Module scripts** end in `.luau` (no prefix) — they're shared libraries
- The **Output** window in Studio shows print() statements and errors
- **Explorer** panel shows the game hierarchy (like a file tree)
- **Properties** panel shows selected object's properties
- You'll still build the physical world (parts, models) in Studio manually — Rojo handles the code

---

## Step 5: Git Setup

### `.gitignore`:
```
# Roblox
*.rbxlx
*.rbxl
*.rbxm
*.rbxmx

# OS
.DS_Store
Thumbs.db

# Node
node_modules/

# Rojo
rojo.lock
```

### Initial commit:
```bash
git add .
git commit -m "Initial Pizza Jump project setup"
```

### Push to GitHub:
```bash
# Create a repo on GitHub first, then:
git remote add origin https://github.com/YOUR_USERNAME/pizza-jump.git
git branch -M main
git push -u origin main
```

---

## Step 6: Useful Studio Settings for Beginners

1. **Enable Output window:** View → Output
2. **Enable Explorer:** View → Explorer
3. **Enable Properties:** View → Properties
4. **Enable Command Bar:** View → Command Bar (lets you run Luau snippets)
5. **Enable Script Analysis:** View → Script Analysis (shows warnings)
6. **Set auto-save:** File → Studio Settings → Studio → Auto-save

---

## Step 7: Roblox Game Settings

Before publishing:
1. In Studio: Game Settings → Basic Info
   - Name: "Pizza Jump"
   - Description: "Escape the kitchen! Jump your way through wild worlds as a sentient pizza slice. Collect toppings, dodge hazards, and race to the top!"
   - Genre: Adventure
   - Max Players: 30
2. Game Settings → Security
   - Enable Studio Access to API Services: ON (needed for DataStoreService during testing)
   - Allow Third Party Teleports: OFF

---

## File Naming Conventions

| File Pattern | Type | Runs On |
|---|---|---|
| `Name.server.luau` | Server Script | Server only |
| `Name.client.luau` | Client Script | Each player's machine |
| `Name.luau` | ModuleScript | Wherever it's required |
| `init.luau` | ModuleScript | Represents the folder itself |
| `init.server.luau` | Server Script | Represents the folder (server) |
| `init.client.luau` | Client Script | Represents the folder (client) |
