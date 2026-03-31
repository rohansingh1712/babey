# The Long Day

A narrative game simulating the interrupt-driven chaos of life with a 3-month-old baby.

## 🎯 Purpose

This game captures the essence of early parenthood: trying to accomplish a simple task (getting to the grocery store) while constantly being interrupted by urgent, unpredictable demands (fires). The game mechanics mirror the experience:

- **Random interruptions** - Fires spawn at unpredictable intervals (1-10 seconds)
- **Escalating urgency** - Fires level up if ignored, demanding immediate attention
- **Limited resources** - Water buckets (representing energy/patience) deplete quickly
- **Degrading conditions** - Vision fades as fires level up, representing exhaustion and mental fog
- **Time pressure** - Only 1 minute per "day" before you need to rest
- **Persistence required** - You wake up and try again from where you left off

The game is designed to be difficult and frustrating in ways that resemble the experience it's simulating.

## 🎮 Game Mechanics

### Core Loop
1. **Navigate** a 40×40 maze using arrow keys
2. **Extinguish fires** by clicking to activate water bucket and holding on fires
3. **Reach the grocery store** before time runs out
4. **Wake up** and continue from same position if time expires

### Fire System
- **Spawn frequency**: Random (1-10 seconds between fires)
- **Initial level**: Random (1-5)
- **Level progression**: Increases every 5 seconds if not extinguished
- **Extinguish mechanic**: Click-and-hold (base 2s + 1s per level)
  - Level 1: 2 seconds of holding
  - Level 2: 3 seconds of holding
  - Level 5: 6 seconds of holding
- **Progress persistence**: Extinguishing progress accumulates across water bucket refills
- **Visual impact**:
  - Each fire level reduces maze/car opacity by 30%
  - Fire opacity reduces in 10% increments as it's extinguished (stepped visual feedback)

### Resources
- **Water bucket**: Lasts 5 seconds per activation
- **Time per day**: 1 minute
- **Movement**: Instant cell-to-cell navigation

### Visual Feedback
- Car shakes after 2 seconds of idleness
- Fire size and shake intensity increase with level
- Fire fades in 10% steps as it's being put out (0%, 10%, 20%... 100%)
- Maze/car fade as fire levels increase
- Water bucket shrinks as it depletes
- Bucket tilts with water drop icon when holding

## 🛠 Tech Stack

**Single-file architecture** - Everything lives in `the-long-day.html`

- **HTML5** - Structure and canvas element
- **CSS3** - Styling, modals, HUD layout
- **Vanilla JavaScript** - Game logic, no frameworks or libraries
- **Canvas API** - All game rendering (maze, player, fires)
- **Google Fonts** - Crimson Text for typography

### Why Single File?
- Zero dependencies
- Instant playability (open in browser)
- Easy to share and deploy
- Complete portability

## 📁 Code Structure

### Configuration (`CONFIG` object)
```javascript
{
  MAZE_SIZE: 40,              // Grid dimensions
  FIRE_INTERVAL_MS: 5000,     // Base spawn interval
  FIRE_LEVEL_UP_INTERVAL_MS: 5000,
  WATER_DURATION_MS: 5000,    // Bucket duration (5 seconds)
  DAY_DURATION_MS: 60000,     // 1 minute per day
  BASE_FIRE_SIZE_PX: 48,
  FIRE_SIZE_INCREMENT_PX: 20,
  BASE_SHAKE_AMPLITUDE_PX: 1,
  SHAKE_AMPLITUDE_INCREMENT_PX: 1
}
```

### Game State (`gameState` object)
Holds all mutable game data:
- Maze structure
- Player position
- Current day
- Fire state
- Water bucket state
- Timers and flags

### Core Systems

1. **Maze Generation** (`generateMaze()`)
   - Recursive backtracker (DFS) algorithm
   - Post-processing: Removes 20% of walls to create shortcuts
   - Creates multiple paths and loops for easier navigation

2. **Rendering** (`render()`, `renderMaze()`, `renderPlayer()`, `renderFire()`)
   - Canvas-based drawing
   - Opacity effects based on fire level
   - Shake animations

3. **Fire Management** (`spawnFire()`, `scheduleFireLevelUp()`, `extinguishFire()`)
   - Random spawn timing and levels
   - Auto-leveling with timers
   - Click-and-hold extinguishing with persistent progress
   - Opacity reduction in 10% steps as fire is extinguished

4. **Input Handling**
   - Keyboard: Arrow keys for movement
   - Mouse: Click/drag for water bucket

5. **Day/Time System** (`updateTimer()`, `showEndDayOverlay()`, `resumeDay()`)
   - Countdown timer
   - Day progression
   - Position persistence

## 🔧 Extending the Game

### Adding New Features

**Example: Add power-ups**
```javascript
// 1. Add to CONFIG
POWERUP_SPAWN_INTERVAL_MS: 30000,

// 2. Add to gameState
powerup: null,

// 3. Create spawn function
function spawnPowerup() {
  gameState.powerup = {
    x: /* random position */,
    y: /* random position */,
    type: 'extraWater' // or other types
  };
}

// 4. Add rendering in render()
if (gameState.powerup) renderPowerup();

// 5. Add collision detection in movePlayer()
// Check if player reached powerup position
```

### Modifying Difficulty

**Easy mode:**
```javascript
FIRE_INTERVAL_MS: 15000,      // Slower spawns
WATER_DURATION_MS: 4000,      // More water
DAY_DURATION_MS: 120000,      // 2 minutes per day
```

**Hard mode:**
```javascript
FIRE_INTERVAL_MS: 2000,       // Rapid spawns
WATER_DURATION_MS: 1000,      // Less water
DAY_DURATION_MS: 30000,       // 30 seconds per day
```

### Customizing Visuals

**Colors:** Update the color palette variables
```javascript
// In CSS, change these hex values:
background: #212529; // Dark background
color: #f8f9fa;      // Light text
```

**Maze size:** Adjust `MAZE_SIZE` (10-100 recommended range)

**Fonts:** Replace Google Fonts link in `<head>` and update font-family in CSS

## 🎨 Design Decisions

### Dark Theme
Reflects the exhaustion and late-night nature of early parenting.

### Serif Typography (Crimson Text)
Literary quality that emphasizes the narrative/experiential nature over arcade-game aesthetic.

### Minimal HUD
Only essential information (subtitle, day counter) to avoid clutter and let the maze dominate.

### No Score System
This isn't about winning or optimizing—it's about experiencing the struggle.

### Modal Flow
Two-step cheat prevention adds personality and discourages immediate rule-checking, forcing players to experience the confusion first-hand.

## 🚀 Running the Game

1. Open `the-long-day.html` in any modern browser
2. No build process, no server required
3. Works offline

## 📝 Future Ideas

- [ ] Sound effects (baby crying, fire crackling)
- [ ] Multiple maze layouts
- [ ] Difficulty progression (fires get worse over days)
- [ ] Achievement system
- [ ] Local storage for save states
- [ ] Mobile touch controls
- [ ] Accessibility improvements (keyboard-only mode, screen reader support)
- [ ] Multiplayer (co-parenting mode?)

## 🤝 Contributing

This is a single-file project, so modifications are straightforward:
1. Edit `the-long-day.html`
2. Test in browser
3. No compilation needed

Keep the single-file architecture for simplicity and portability.

## 📄 License

Created as an artistic expression of the parenting experience.

---

*"Life with a 3 month old when you also have other things to do"*
