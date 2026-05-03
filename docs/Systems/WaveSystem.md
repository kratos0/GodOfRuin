
# =====================================
# FILE 4: docs/Systems/WaveSystem.md
# =====================================
# INSTRUCTIONS: Fill in the blank fields
# after Session 3 when wave system is
# fully configured in Level 1.
# =====================================

```markdown
# Wave System
## Status: Extended — Testing Pending
## Last Updated: May 2, 2026
## Session: 1

---

## WHAT IT DOES
Manages enemy waves in arena combat zones.
Spawns enemies in rounds, tracks deaths, fires completion event,
opens gate when all waves done.

---

## HOW IT WORKS
1. Player walks into `BoxComponent` on `BP_ArenaSpawner`
2. `BeginArena` custom event fires (Do Once — no re-entry)
3. Stores player reference as `CharacterRef`
4. Calls `CloseGates` on `BP_Gate` directly
5. For Loop spawns enemies based on selected Option struct
6. Each enemy death → `DeadEnemies` increments
7. When `DeadEnemies == spawned count`:
   - Increments `CurrentRound`
   - If `CurrentRound >= TotalRounds` → fires `OnAllWavesComplete` + `OpenGates`
   - If not done → random picks next Option struct → spawns next wave
8. `ResetArena` can be called to clear all enemies and reset counters

---

## KEY BLUEPRINTS
```
BP_ArenaSpawner: /Game/FlexibleCombatSystem/Blueprints/Miscellaneous/DemoLevels/BP_ArenaSpawner
BP_ArenaGate:    /Game/FlexibleCombatSystem/Blueprints/Miscellaneous/DemoLevels/BP_ArenaGate
```

---

## VARIABLES

### Stock FCS Variables
| Variable | Type | Purpose |
|----------|------|---------|
| CurrentRound | int | Current round number |
| CurrentStage | int | Current stage |
| Rounds Per Stage | int | Rounds before stage increments |
| DeadEnemies | int | Running kill count |
| SpawnedMobs | array | All spawned enemy actors |
| BP_Gate | object | Direct ref to gate actor — MUST BE ASSIGNED IN LEVEL |
| CharacterRef | object | Player reference |
| Option 1 through Option 1_6 | struct | 7 wave config options |

### Added in GodOfRuin
| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| TotalRounds | int | 3 | Rounds before OnAllWavesComplete fires |
| OnAllWavesComplete | EventDispatcher | — | Fires when all waves done |

---

## GATE SYSTEM (BP_ArenaGate)

### How Gate Opens/Closes
- `OpenGates` event → plays `OpenLeGate` timeline forward + disables `Blocker` collision
- `CloseGates` event → plays `OpenLeGate` timeline reverse + enables `Blocker` collision
- Gate mesh moves via `Set Relative Location` + `Lerp (Vector)` during timeline
- Auto-saves on gate open via `SaveCheckpoint` + `UpdateSaveSlot`

### Gate Variable on Spawner
- `BP_Gate` must be manually assigned in level Details panel
- If null → crash on BeginPlay
- Always check this when placing BP_ArenaSpawner in a new level

---

## HOW TO CONFIGURE PER LEVEL

### In Level Details Panel (select BP_ArenaSpawner actor):
1. Set `TotalRounds` = number of waves for this level
2. Assign `BP_Gate` = the BP_ArenaGate actor in the level
3. Configure `Option 1` through relevant Option structs with enemy classes

### Binding OnAllWavesComplete
In level Blueprint or BP_LevelEndTrigger:
```
BP_ArenaSpawner reference → Bind Event to OnAllWavesComplete
  → Your completion logic (open next area, spawn chest, etc.)
```

---

## LEVEL CONFIGURATIONS

| Level | TotalRounds | Wave 1 | Wave 2 | Wave 3 | Wave 4 |
|-------|-------------|--------|--------|--------|--------|
| L1 Zone 2 | 1 | 4x FCS soldiers | — | — | — |
| L1 Zone 5 | 2 | 6x soldiers + Grux | +2x Grux at 50% | — | — |
| L1 Zone 7 | 2 | 8x soldiers | +2x Grux | — | — |
| L2 | *(fill in)* | | | | |
| L4 Arena | 4 | 6x soldiers | 4x Steel | Narbash+Steel | Feng Mao boss |

*(Fill in as levels are built)*

---

## TEST PROCEDURE
1. Place BP_ArenaSpawner in level
2. Place BP_ArenaGate in level
3. Assign BP_Gate in spawner Details panel
4. Set TotalRounds = 2
5. Configure Option 1 with enemy class + count
6. Hit Play
7. Walk into spawner BoxComponent
8. Verify: gate closes, enemies spawn
9. Kill all enemies
10. Verify: next wave spawns
11. Kill second wave
12. Verify: OnAllWavesComplete fires, gate opens

---

## GOTCHAS
- BP_Gate MUST be assigned in Details panel or crash
- Wave options are loose structs — not an array. Max 7 option variants per spawner.
- Stock FCS has no OnAllWavesComplete — we added it. Do not override BP_ArenaSpawner without preserving this.
- Do One node prevents BeginArena from firing twice — if testing, use ResetArena first
```

