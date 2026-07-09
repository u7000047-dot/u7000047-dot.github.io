# Game Loop

The main loop as it stands today: level banners, spawn queues, a house to defend, a boss every 5 levels, upgrade points, and a one-time cutscene at level 11. Collision detail is broken out separately in [activity_diagram.md](activity_diagram.md).

```mermaid
flowchart TD
    Start([Page Load]) --> Init["new Game()\n• Player at (500, groundY)\n• spawnEnemies() for level 1"]
    Init --> Loop["run(): setInterval @ 60 FPS\nupdate() then draw()"]

    Loop --> Running{gameRunning?}
    Running -- "No (paused / stats open / cutscene playing)" --> Draw[draw current frame only]
    Draw --> Loop

    Running -- Yes --> Banner{showLevelBanner?}
    Banner -- "Yes" --> BannerTick["levelBannerTimer++\n(clears after 1.5s)"]
    BannerTick --> Draw

    Banner -- No --> Drain["Drain spawnQueue:\neach entry's delay counts down,\nspawns Enemy or ChaserEnemy at 0"]
    Drain --> UpdateAll["Update player, house, all enemies,\nfireballs, enemyFireballs, explosions, miniSlimes"]
    UpdateAll --> Collisions["Resolve collisions\n(see activity_diagram.md)"]
    Collisions --> Clear{"enemies empty AND\nspawnQueue empty AND\nminiSlimes empty?"}

    Clear -- No --> Draw
    Clear -- Yes --> LevelUp["level++"]
    LevelUp --> M10{"level % 10 == 0?"}
    M10 -- Yes --> Life["maxLives += 1\nlives = maxLives"]
    M10 -- No --> M5
    Life --> M5{"level % 5 == 0?"}
    M5 -- Yes --> Points["upgradePoints += level/5\nupdateStatsBadge()"]
    M5 -- No --> ShowBanner
    Points --> ShowBanner["showLevelBanner = true"]

    ShowBanner --> Is11{level === 11?}
    Is11 -- Yes --> Cutscene["gameRunning = false\ncutsceneActive = true\nplayCharizardCutscene(onComplete)"]
    Cutscene --> CutsceneDone["onComplete:\ncutsceneActive = false\ngameRunning = true\nspawnEnemies()"]
    CutsceneDone --> Draw
    Is11 -- No --> Spawn["spawnEnemies()\n(fresh House, new spawnQueue,\nBossSlime immediately if level % 5 == 0)"]
    Spawn --> Draw

    %% --- Menus (interrupt the loop via gameRunning, not the flow above) ---
    ESC([ESC key]) --> PauseToggle{paused?}
    PauseToggle -- No --> OpenPause["showPauseMenu()\npaused = true, gameRunning = false"]
    PauseToggle -- Yes --> ClosePause["resumeGame()\npaused = false, gameRunning = true"]

    StatsBtn([Stats button]) --> OpenStats["showStatsMenu()\nstatsOpen = true\ngameRunning = false (if not already paused)"]
    OpenStats --> Spend["upgradeStat(stat)\nspends 1 upgradePoint\n(damage / lives / house HP / speed)"]
    Spend --> CloseStats["closeStatsMenu()\nstatsOpen = false\ngameRunning = true (if not paused)"]

    DebugC(["'C' key\n(while paused)"]) --> DebugCutscene["Close pause menu\nplayCharizardCutscene(...)\n(preview only — does not touch level/spawns)"]

    PauseMenu2["Pause menu also offers:\nReset Game · Export State (JSON) · Import State (JSON)"]
```

## Notes

- **No real "Game Over"**: the `#gameOver` overlay exists in the markup/CSS but nothing in the code ever shows it. Running out of lives just resets `lives` to `maxLives` and calls `spawnEnemies()` again for the *same* level — the house also resets to full HP whenever `spawnEnemies()` runs. The house visually crumbles to rubble at 0 HP but that doesn't end the level or the game; it just looks damaged until the next level starts. This is effectively an endless mode.
- **Save/Load**: Export/Import (in the pause menu) round-trip `score`, `level`, `lives`, `maxLives`, `upgradePoints`, `stats`, player position/direction, and house health as JSON — everything else (enemies, projectiles, spawn queue) is regenerated fresh via `spawnEnemies()`.
