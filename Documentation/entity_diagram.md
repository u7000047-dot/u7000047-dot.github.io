# Entity Diagram

Class relationships as they exist in `side_scroller_game.html` today. `gameState` is the single source of truth for run-time data that isn't owned by one specific object (score, level, lives, upgrade stats, and the shared projectile/effect arrays).

```mermaid
classDiagram
    class gameState {
        +score, level, lives, maxLives
        +paused, statsOpen, cutsceneActive
        +upgradePoints, stats
        +fireballs[], enemyFireballs[]
        +explosions[], miniSlimes[]
    }

    class Game {
        +canvas, ctx, cameraX
        +player, house, enemies[], spawnQueue[]
        +spawnEnemies()
        +update()
        +draw()
    }

    class Player {
        +x, y, width, height
        +velocityX, velocityY, isJumping
        +speed, maxSpeed, direction
        +handleInput(keys)
        +applyGravity()
        +getCollisionBox()
    }

    class House {
        +health, maxHealth, wallThickness
        +takeDamage(amt)
        +repair(amt)
        +isDestroyed()
        +getLeftWallBox(), getRightWallBox()
    }

    class Enemy {
        +x, y, speed, centerX
        +health, maxHealth = 2
        +shootAtPlayer() -> EnemyFireball
    }

    class ChaserEnemy {
        +x, y, speed
        +health, maxHealth = 2
        +update() moves toward player
    }

    class BossSlime {
        +health, maxHealth = 500
        +rageMode, rageTriggered
        +spit() -> MiniSlime[]
        +spawnSkyRain() -> MiniSlime[]
    }

    class Fireball {
        +x, y, vx
        +radius = 10
    }

    class EnemyFireball {
        +x, y, vx, vy
        +radius = 10
    }

    class MiniSlime {
        +x, y, vx, vy
        +radius = 11
    }

    class Explosion {
        +particles[], smoke[]
        +duration
        +done
    }

    Game "1" *-- "1" Player
    Game "1" *-- "1" House
    Game "1" *-- "0..*" Enemy
    Game "1" *-- "0..*" ChaserEnemy
    Game "1" *-- "0..1" BossSlime
    Enemy ..> EnemyFireball : shootAtPlayer()
    BossSlime ..> MiniSlime : spit() / spawnSkyRain()
    Player ..> Fireball : SPACE key
    gameState "1" *-- "0..*" Fireball
    gameState "1" *-- "0..*" EnemyFireball
    gameState "1" *-- "0..*" MiniSlime
    gameState "1" *-- "0..*" Explosion
    Game --> gameState : reads/writes
```

## Notes on inheritance

There is no shared base class — `Enemy`, `ChaserEnemy`, and `BossSlime` are independent classes that happen to share a duck-typed shape (`x`, `y`, `width`/`height` or radius-equivalent, `getCollisionBox()`, `update()`, `draw(ctx)`). The collision code in `Game.update()` checks `instanceof ChaserEnemy` / `instanceof BossSlime` directly rather than dispatching through a common interface — worth knowing if a fourth enemy type gets added later, since that `instanceof` chain (see [activity_diagram.md](activity_diagram.md)) would need a new branch too.

`Fireball`, `EnemyFireball`, and `MiniSlime` are similarly independent classes with near-identical shapes (position, velocity, radius, `active` flag, `getCollisionBox()`) rather than sharing a `Projectile` base.
