# UI / App State

`gameState.gameRunning` gates the entire `Game.update()` step (see [game_loop.md](game_loop.md)); `draw()` always runs regardless, which is what lets menus and the cutscene overlay render on top of a frozen last frame.

Three independent-ish flags control this: `paused`, `statsOpen`, `cutsceneActive`. They're each set by their own show/close functions rather than derived from one another — see the **Known quirk** below.

```mermaid
stateDiagram-v2
    [*] --> Playing

    Playing --> Paused: ESC (showPauseMenu)\npaused=true, gameRunning=false
    Paused --> Playing: ESC (resumeGame)\npaused=false, gameRunning=true

    Playing --> StatsOpen: Stats button\nstatsOpen=true, gameRunning=false
    StatsOpen --> Playing: Close button\nstatsOpen=false, gameRunning=true

    Paused --> PausedWithStats: Stats button\nstatsOpen=true (gameRunning already false)
    PausedWithStats --> Paused: Close button\nstatsOpen=false (gameRunning stays false, still paused)

    Paused --> CutscenePlaying: 'C' key (debug)\ncloses pause menu, paused=false,\ncutsceneActive=true, gameRunning=false
    Playing --> CutscenePlaying: level becomes 11\ncutsceneActive=true, gameRunning=false
    CutscenePlaying --> Playing: onComplete\ncutsceneActive=false, gameRunning=true\n(+ spawnEnemies() if triggered by level-up)

    Paused --> Playing: Reset Game button\n(full state reset, see game_loop.md)
    Paused --> Playing: Export/Import State\n(Import also resumes)
```

## Known quirk: `PausedWithStats → Playing` via ESC

`resumeGame()` unconditionally sets `paused = false` and `gameRunning = true` and hides the pause menu — it does **not** check or close the stats menu. So the sequence ESC (pause) → open Stats → ESC (resume) leaves the Stats overlay visually on screen (`statsOpen` still `true`, `#statsMenu` still `display:block`) while the game underneath starts running again. Not currently guarded against; worth knowing if this ever gets reported as "the game moves behind the stats screen."

## Debug cutscene trigger

Pressing `C` while `paused` is true (pause menu open) calls `window.playCharizardCutscene()` directly, bypassing level logic entirely — it's a preview tool, not tied to `gameState.level`. See [charizard_cutscene.md](charizard_cutscene.md).
