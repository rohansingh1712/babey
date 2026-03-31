# Claude Context: The Long Day

## Project Overview

**"The Long Day"** is a narrative browser game that simulates the interrupt-driven chaos of life with a 3-month-old baby. The player navigates a maze from home to the grocery store while constantly being interrupted by fires (representing baby needs) that must be extinguished.

**Core Theme**: The game deliberately creates frustration and difficulty to mirror the parenting experience—unpredictable interruptions, limited resources (energy/patience), degrading conditions (exhaustion/brain fog), and the need to persist despite setbacks.

**Files**:
- `index.html` - Single-file game (HTML + CSS + JavaScript, ~1000 lines)
- `README.md` - Developer documentation
- `CLAUDE.md` - This file (Claude context)

## Technical Architecture

### Single-File Design
Everything lives in `index.html`:
- HTML structure (canvas, HUD overlays, modals)
- CSS styling (dark theme, modals, custom cursor)
- Vanilla JavaScript (no frameworks/libraries)
- Google Fonts (Crimson Text serif)

**Why single-file?**
- Zero dependencies
- Instant playability (just open in browser)
- Easy to share/deploy
- Complete portability

### Technology Stack
- **HTML5 Canvas** - All game rendering (maze, car, fires)
- **Vanilla JavaScript** - Game logic, no frameworks
- **CSS3** - Styling, modals, animations
- **Crimson Text font** - Literary aesthetic

## Game Mechanics

### Goal
Drive car 🚗 from top-left (start) to bottom-right 🏪 (grocery store) through a 40×40 maze.

### Core Systems

#### 1. Maze Generation
- **Algorithm**: Recursive backtracker (DFS)
- **Post-processing**: Removes 20% of walls randomly to create shortcuts and loops
- **Size**: 40×40 grid (configurable via `CONFIG.MAZE_SIZE`)
- **Rendering**: Light gray walls (#6c757d) on dark background (#212529)
- **Scale**: 90% of viewport (10% margin to prevent edge overflow)
- **Difficulty**: Multiple paths and interconnected routes make navigation easier

#### 2. Player Movement
- **Control**: Arrow keys
- **Behavior**: Instant snap-to-cell (no animation)
- **Visual**: Car emoji 🚗 (2x cell size, max 100px)
- **Idle Animation**: Shakes horizontally after 2 seconds of no movement

#### 3. Fire System
**Spawning**:
- Random interval: 1-10 seconds between fires
- Random initial level: 1-5
- Random position: anywhere within maze bounds
- Only spawns after previous fire is extinguished

**Leveling**:
- Increases by 1 every 5 seconds if not extinguished
- Max level: 10
- Visual: Size increases by 20px per level (base 48px)
- Shake animation increases with level

**Extinguishing**:
- Method: Click-and-hold (must hold mouse button on fire)
- Time required: `2000ms + (level - 1) × 1000ms`
  - Level 1: 2 seconds
  - Level 2: 3 seconds
  - Level 5: 6 seconds
- Progress persists across water bucket refills (cumulative)
- Requires active water bucket with remaining water
- Requires continuous hold (releasing resets progress to last 10% milestone)

**Visual Impact**:
- Each fire level reduces maze/car opacity by 30%
- Level 1: 70% opacity
- Level 4: 0% opacity (completely invisible)
- Fire opacity reduces in 10% increments as extinguishing progresses (100%, 90%, 80%... 0%)

#### 4. Water Bucket System
**Activation**:
- Click anywhere to activate
- Shows custom cursor (bucket emoji)
- Hides default cursor

**Usage**:
- Duration: 5 seconds of **holding** (click and hold)
- Depletes while holding mouse button down
- Visual feedback:
  - Bucket shrinks as water depletes (100% → 50%)
  - When holding: Shows "💧🪣" (drop + bucket)
  - When holding: Rotates -25° (tilting/pouring)
  - When not holding: Shows "🪣" upright
  - When depleted: Shows "REFILL!" text

**Refilling**:
- Click again to instantly refill (resets usedTime to 0)
- No cooldown

**Technical Implementation**:
- Tracks `usedTime` (accumulated time spent holding)
- Increments usedTime when `dragging` (mouse button held)
- Water pours immediately on click-and-hold, not requiring movement

#### 5. Day/Time System
- **Duration**: 1 minute per day
- **Timer**: Counts down (no visible display per user request)
- **Day End**: Shows modal "Day's over. Time to sleep."
- **Resume**: "Wake up" button increments day, resets timer
- **Position**: Preserved across days (same maze, same progress)
- **Win**: Reaching grocery store shows completion modal, "Play Again" resets everything

### UI/UX Elements

#### HUD (Left Side, Vertically Centered)
- **Subtitle**: "Living with a 3 month old and trying to do things." (20px, #adb5bd)
- **Day Counter**: "Day N" (20px, #f8f9fa)
- **Max Width**: 240px (prevents overlap with maze)

#### Bottom Left Link
- **Text**: "Don't cheat" (20px, #adb5bd)
- **Behavior**: Shows warning modal first, then rules

#### Modals
All use same dark style:
- Background: #212529 (matches game background)
- Max width: 1000px
- Border: None
- Buttons: Light background (#f8f9fa) with dark text (#495057)
- Font: Crimson Text serif (matches game)

**Modal Flow**:
1. Click "Don't cheat" → Warning: "You wish a 3 month old came with a cheat sheet"
2. Two options:
   - "Alright, I won't cheat" → Closes modal
   - "Just tell me how this works 😭" → Shows rules modal

## Code Structure

### CONFIG Object
```javascript
{
  MAZE_SIZE: 40,                    // Grid size (cells × cells)
  FIRE_INTERVAL_MS: 5000,           // Base spawn interval (randomized ±50%)
  FIRE_LEVEL_UP_INTERVAL_MS: 5000,  // Time between level increases
  WATER_DURATION_MS: 5000,          // Water lasts 5 seconds of active use
  DAY_DURATION_MS: 60000,           // 1 minute per day
  BASE_FIRE_SIZE_PX: 48,            // Starting fire size
  FIRE_SIZE_INCREMENT_PX: 20,       // Size increase per level
  BASE_SHAKE_AMPLITUDE_PX: 1,       // Fire shake at level 1
  SHAKE_AMPLITUDE_INCREMENT_PX: 1   // Shake increase per level
}
```

### gameState Object
Holds all mutable game state:
```javascript
{
  maze: null,                       // 2D array of cells
  cellSize: 0,                      // Calculated from viewport
  player: { x: 0, y: 0 },          // Grid position
  day: 1,                           // Current day number
  timeRemaining: 60000,             // Milliseconds left
  fire: null,                       // Current fire object or null
  fireSpawnTimer: null,             // setTimeout ID
  fireLevelUpTimer: null,           // setTimeout ID
  waterBucket: {
    active: false,                  // Bucket activated?
    startTime: null,                // Deprecated (kept for compatibility)
    usedTime: 0                     // Milliseconds of active dragging
  },
  dragging: false,                  // Mouse button down?
  lastFrameTime: 0,                 // For deltaTime calculation
  paused: false,                    // Game paused?
  mouseX: 0, mouseY: 0,            // Current mouse position
  cursorOverFire: false,            // Deprecated (kept for compatibility)
  lastMoveTime: Date.now(),         // For car idle animation
  mouseMoving: false,               // Mouse moving in last 50ms?
  mouseMoveTimeout: null            // setTimeout ID for mouseMoving flag
}
```

### Key Functions

**Maze Generation**:
- `generateMaze(size)` - Recursive backtracker (DFS) algorithm, then removes 20% of walls

**Fire Management**:
- `scheduleNextFire()` - Random 1-10s delay, then spawn
- `spawnFire()` - Creates fire at random position/level with holdTime=0 and requiredHoldTime
- `scheduleFireLevelUp()` - Increases level every 5s, updates requiredHoldTime
- `extinguishFire()` - Removes fire, schedules next
- `isMouseOverFire()` - Checks if cursor is within fire hitbox

**Rendering**:
- `render()` - Main render loop
- `renderMaze()` - Draws walls, grocery store icon, applies opacity
- `renderPlayer()` - Draws car, applies opacity, idle shake
- `renderFire()` - Draws fire emoji with size/shake/opacity (reduces in 10% steps as extinguished)

**Input Handling**:
- Arrow keys → `movePlayer(dx, dy)`
- Mouse down → Activate bucket, set dragging=true, refill if depleted
- Mouse move → Track position
- Mouse up → Set dragging=false, deactivate if water depleted

**Water System**:
- `activateWaterBucket()` - Show custom cursor, reset usedTime
- `deactivateWaterBucket()` - Hide custom cursor
- `updateCustomCursor(x, y)` - Position cursor, update visual state
- Game loop increments `usedTime` only when dragging && mouseMoving

**Timer System**:
- `updateTimer(deltaTime)` - Decrement timeRemaining
- `showEndDayOverlay()` - Pause game, show modal
- `resumeDay()` - Increment day, reset timer, preserve position

### Game Loop
```javascript
function gameLoop(timestamp) {
  const deltaTime = timestamp - lastFrameTime;

  if (!paused) {
    updateTimer(deltaTime);

    // Track water usage when holding
    if (waterBucket.active && dragging) {
      waterBucket.usedTime += deltaTime;
    }

    // Track fire hold time when holding over fire
    if (fire && waterBucket.active && dragging) {
      if (waterBucket.usedTime < WATER_DURATION_MS) {
        if (isMouseOverFire(mouseX, mouseY)) {
          fire.holdTime += deltaTime;
          if (fire.holdTime >= fire.requiredHoldTime) {
            extinguishFire();
          }
        }
      }
    }

    checkWaterBucketExpiry();

    if (waterBucket.active) {
      updateCustomCursor(mouseX, mouseY);
    }
  }

  render();
  requestAnimationFrame(gameLoop);
}
```

## Important Implementation Details

### Water Depletion Logic
**Critical**: Water depletes while **holding** (mouse button down).

**How it works**:
1. On mousedown: Set `dragging = true`
2. Game loop: If `dragging`, increment `usedTime`
3. On mouseup: Set `dragging = false`
4. Cursor: Show drops/rotation when `dragging`

**Why**: Holding depletes water, making time management critical. Must refill to continue extinguishing fires.

### Custom Cursor System
Uses a fixed-position div that follows mouse, not CSS cursor property (for dynamic content/scaling).

**States**:
- Inactive: Hidden
- Active + Not Holding: "🪣" upright, no rotation
- Active + Holding: "💧🪣" tilted -25°, shrinks as water depletes
- Active + Depleted: "REFILL!" text

### Opacity System
Fire levels directly affect visibility:
```javascript
const opacity = fire ? 1.0 - (fire.level * 0.3) : 1.0;
ctx.globalAlpha = Math.max(0, opacity);
```
Applied to both maze and car during rendering.

### Maze Scaling
Cell size calculated as 90% of viewport to prevent overflow:
```javascript
cellSize = Math.min(
  canvas.width / MAZE_SIZE,
  canvas.height / MAZE_SIZE
) * 0.9;
```

### Car Idle Animation
Shakes horizontally after 2s of no movement:
```javascript
if (Date.now() - lastMoveTime > 2000) {
  const shakeAmount = Math.sin(Date.now() / 100) * 3;
  px += shakeAmount;
}
```

## Common Modifications

### Adjust Difficulty

**Easier**:
```javascript
FIRE_INTERVAL_MS: 10000,      // Slower spawns
WATER_DURATION_MS: 10000,     // More water
DAY_DURATION_MS: 120000,      // 2 minutes
```

**Harder**:
```javascript
FIRE_INTERVAL_MS: 2000,       // Rapid spawns
WATER_DURATION_MS: 2000,      // Less water
DAY_DURATION_MS: 30000,       // 30 seconds
```

### Change Maze Size
Update `MAZE_SIZE` (recommend 20-60 range). Everything scales automatically.

### Modify Fire Behavior

**Spawn levels**:
```javascript
// Currently: random 1-5
const randomLevel = Math.floor(Math.random() * 5) + 1;

// Change to always level 1:
const randomLevel = 1;

// Change to random 1-10:
const randomLevel = Math.floor(Math.random() * 10) + 1;
```

**Hold time required**:
```javascript
// Currently: 2000ms + (level-1) × 1000ms
requiredHoldTime: 2000 + (randomLevel - 1) * 1000

// Make easier (less time):
requiredHoldTime: 1000 + (randomLevel - 1) * 500

// Make harder (more time):
requiredHoldTime: 3000 + (randomLevel - 1) * 1500
```

**Maze difficulty**:
```javascript
// Currently: removes 20% of walls
const wallsToRemove = Math.floor(size * size * 0.2);

// Make easier (more shortcuts):
const wallsToRemove = Math.floor(size * size * 0.3);

// Make harder (fewer shortcuts):
const wallsToRemove = Math.floor(size * size * 0.1);
```

### Change Visual Style

**Colors**: Search for hex codes
- Background: `#212529`
- Text: `#f8f9fa`
- Subtitle: `#adb5bd`
- Walls: `#6c757d`

**Font**: Replace Google Fonts link and update `font-family: 'Crimson Text', serif`

**Emojis**: Search for emoji characters
- Car: 🚗 (line ~630)
- Grocery: 🏪 (line ~610)
- Fire: 🔥 (line ~660)
- Bucket: 🪣 (line ~745)
- Drop: 💧 (line ~745)

## Design Decisions & Rationale

### Dark Theme
Reflects exhaustion and late-night nature of parenting a newborn.

### Serif Typography
Crimson Text gives literary quality, emphasizing narrative/experiential nature over arcade aesthetics.

### No Timer Display
Removed per user request. Creates tension without constant countdown stress.

### Two-Step Cheat Flow
Playful personality that gently discourages immediate rule-checking, forcing initial confusion/struggle (mirrors parenting experience).

### Opacity Degradation
Direct feedback that ignoring fires has consequences. At level 4+, you're playing blind—creates urgency.

### Fire Opacity Reduction
Fire fades in 10% increments as it's extinguished, providing clear visual milestones without numerical indicators.

### Hold-Based Water Depletion
Water depletes while holding, creating time pressure. Must manage water refills to complete extinguishing higher-level fires.

### Persistent Fire Progress
Extinguishing progress persists across water refills, allowing players to tackle high-level fires requiring more time than one bucket provides.

### Single File Architecture
Simplicity and portability over modularity. Easy to share, fork, remix.

## Known Issues & Considerations

### Performance
- 40×40 maze renders fine on modern browsers
- Larger mazes (80×80+) may cause frame drops on older devices
- Canvas size is full viewport (resizes on window resize)

### Mobile/Touch
Currently keyboard + mouse only. Touch controls not implemented.

### Browser Compatibility
- Requires modern browser with Canvas API support
- Emoji rendering varies by OS/browser
- Tested primarily on Chrome/Safari

### Edge Cases
- If maze generation fails (rare), page refresh required
- Fire can spawn on top of player (by design—interruptions don't wait)
- Multiple clicks during water depletion → refill works correctly

## Session History Summary

**Initial session:**
1. Building entire game from scratch
2. Implementing maze generation, fire system, water mechanics
3. Iterating on difficulty balance (water duration, fire mechanics)
4. Adding visual feedback (bucket rotation, water drops, opacity)
5. Creating modal flows for rules/cheating
6. Styling with dark theme and serif typography
7. Writing documentation (README.md, CLAUDE.md)

**Recent updates:**
1. Changed fire extinguishing from swipe-based to click-and-hold mechanic
2. Made fire progress persist across water bucket refills (cumulative progress)
3. Added stepped opacity reduction (10% increments) to fires as they're extinguished
4. Updated water bucket to deplete while holding (not just while moving)
5. Made maze easier by removing 20% of walls after generation (creates shortcuts/loops)
6. Removed progress indicators for cleaner minimalist experience

## Next Session Quick Start

To continue development:
1. Read this file for full context
2. Open `index.html` in browser to test current state
3. Check `README.md` for developer documentation
4. All code is in `index.html` - search for function names to find relevant sections
5. Test in browser, refresh to see changes (no build process)

## File Locations

```
/Users/singhroh/Desktop/ai experiements/babey/
├── index.html          # Main game file (all code)
├── README.md          # Developer documentation
└── CLAUDE.md          # This file (Claude context)
```

---

**Last Updated**: 2026-03-31
**Version**: 1.1
**Status**: Fully playable, polished, with improved mechanics
