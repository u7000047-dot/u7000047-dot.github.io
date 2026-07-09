# Fire Invader — Documentation

Design docs for `side_scroller_game.html`, a single-file Charmander side-scrolling defense game (protect the house, survive endless levels, evolve into Charizard at level 11).

| Doc | Covers |
|---|---|
| [game_loop.md](game_loop.md) | The 60 FPS main loop: init, per-frame update/draw, level clear, milestones, pause/stats menus |
| [activity_diagram.md](activity_diagram.md) | Frame-by-frame detail: player input/physics, and every collision category (fireballs, house, bosses, chasers, mini-slimes) |
| [entity_diagram.md](entity_diagram.md) | Class/entity relationships — Player, Enemy, ChaserEnemy, BossSlime, MiniSlime, Fireball, House, Explosion, Game |
| [level_progression.md](level_progression.md) | How difficulty scales per level, milestone rewards, boss cadence, and the level-11 evolution trigger |
| [ui_state_diagram.md](ui_state_diagram.md) | UI/app states — playing, paused, stats menu, cutscene — and how they interlock via `gameState` |
| [charizard_cutscene.md](charizard_cutscene.md) | The level-11 Charizard cutscene: Three.js integration, phase timing, hosting requirements |

## File layout (`fire_invader2/`)

| Path | Role |
|---|---|
| `side_scroller_game.html` | Everything — markup, CSS, game logic, cutscene logic (single file) |
| `Charizard006SV.glb` | 3D model + embedded textures + animation clips (incl. `rangeattack01`), ~23 MB |
| `charmander_left.png` / `charmander_right.png` | Player sprite, one per facing direction |
| `Documentation/` | This folder |

**Hosting note:** the game must be served over `http://`/`https://` (e.g. GitHub Pages, or a local dev server like VS Code Live Server / `python -m http.server`) — opening the HTML file directly via `file://` breaks the Three.js module imports and the GLB fetch due to CORS. See [charizard_cutscene.md](charizard_cutscene.md) for details.
