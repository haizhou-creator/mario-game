# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file browser games built with vanilla HTML5 Canvas and JavaScript. No build tools, no dependencies, no package manager. Each game is one self-contained `.html` file that opens directly in a browser.

## Running the game

```powershell
Start-Process "mario.html"
```

Or double-click the file in Explorer. There is no build step, server, or install required.

## Git workflow

Every feature or fix gets its own commit and is pushed to GitHub immediately after:

```powershell
git add mario.html
git commit -m "feat: describe what changed and why"
git push
```

Commit message convention: `feat:`, `fix:`, or `chore:` prefix. Keep messages descriptive — include what the mechanic does, not just what file changed.

Remote: `https://github.com/haizhou-creator/mario-game` (only `mario.html` is tracked; `tictactoe.html` is intentionally untracked).

## mario.html architecture

All game code lives in a single `<script>` block. The structure is:

**Level data (top)** — plain arrays that define the world:
- `PLATFORM_DATA` — `[x, y, w, h]` rectangles
- `ENEMY_DATA` — `[x, y, patrolRange]`
- `COIN_DATA` — `[x, y]`
- `QBOX_DATA` — `[x, y]` question boxes; hitting from below spawns a mushroom

**Constants** — physics (`GRAVITY`, `JUMP_VEL`, `SPEED`) and sizes (`SMALL_W/H`, `BIG_W/H`, `MUSHROOM_W/H`, `QBOX_W/H`)

**Game state** — flat globals: `state` (`'start' | 'play' | 'gameover' | 'win'`), `score`, `lives`, `cameraX`, and entity arrays (`platforms`, `enemies`, `coins`, `mushrooms`, `qboxes`, `flag`)

**Entity lifecycle** — `buildLevel()` constructs all entity objects from the data arrays and resets player state. `resetGame()` also zeroes score/lives and calls `buildLevel()`.

**Player object** — `{x, y, vx, vy, w, h, big, onGround, dead, invincible, facing}`. Width/height change when Mario grows (`growPlayer()`) or shrinks (`shrinkPlayer()`). All collision code uses `player.w` / `player.h` — never the raw constants.

**Collision resolution** — `updatePlayer()` builds a combined `solids` array from `platforms` + `qboxes` each frame and resolves AABB overlap using minimum-overlap axis. A `_box` reference on qbox entries triggers `hitBox()` on head-bump (vy < 0, top overlap).

**Mushroom physics** — spawned mushrooms (`mushrooms[]`) have gravity and platform collision; they are created dynamically by `hitBox()`, not from static data.

**Render order** — background → flag → platforms → qboxes → mushrooms → coins → enemies → player → HUD

**Camera** — horizontal-only scroll; `cameraX` is subtracted from all world-space x positions before drawing.
