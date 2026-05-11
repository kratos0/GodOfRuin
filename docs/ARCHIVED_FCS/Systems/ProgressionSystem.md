# Progression System — God of Ruin
## Status: Design — Implementation Pending
## Last Updated: 2026-05-10

---

## OVERVIEW
Health and Spirit-cap scaling driven by EXP orbs (combat) and optional Zone E
puzzles. We piggyback on the existing FCS `EquipmentComponent` XP/level
system instead of rolling our own.

---

## HEALTH SCALING
| Property | Value |
|---|---|
| Base Health | 100 |
| Max Health | 300 |
| Source | EXP orb pickups (auto-collect on proximity) |
| Increment | Existing FCS level-up handler bumps `MaxHealth` per level |

The FCS `EquipmentComponent` already replicates `Health` and `MaxHealth`
(see audit) — we just feed XP into it. **Do not add a parallel HP system.**

### EXP Orb Values
| Source | XP per orb |
|---|---|
| Grunt enemy | 1 |
| Elite enemy | 10 |
| Boss | 50 |

### Pickup Behaviour
- Auto-collect on proximity (no button press, no chest, no inventory popup).
- Implementation: small overlap volume on a lightweight `BP_EXPOrb` actor
  that calls `EquipmentComponent.ApplyXPIncrease(int)` on touch and self-destroys.

---

## SPIRIT-CAP SCALING
| Property | Value |
|---|---|
| Base Spirit max | 100 |
| Per puzzle | +15 |
| Total puzzles | 8 (one optional puzzle in Zone E of each level) |
| Final cap | 220 |

When a Zone E puzzle is solved, fire a custom event on the GoR player Blueprint that adds 15 to `MaxSpirit` and saves to slot. Tracked via a `SpiritPuzzlesSolved` array on the save game.

---

## EXACT FCS FUNCTIONS (from EquipmentComponent audit)

`EquipmentComponent` is the FCS XP+stat hub — 106 variables, 52 functions, no children.

### Functions we call directly
| Function | Purpose | Used for |
|---|---|---|
| `ApplyXPIncrease` | Adds XP to `Current_XP`, triggers level-up if `>= Max_XP` | EXP orb pickup |
| `IncreaseLevelUpStats` | Applies the per-level stat point bump | Stat chest reward |
| `Character Level Up` | Internal — ApplyXPIncrease calls into this when a level threshold is crossed | Don't call directly |
| `Leveling Up` | Internal level-up FX/handler | Don't call directly |
| `LoadCharacterStats` | Restores stats from save on level start | Save system path |
| `LoadSaveData` | Top-level load entry | Save system path |
| `LoadSaveRestoreResources` | Restores Health/Stamina/Mana from save | Save system path |
| `SetResourcesMax` | Recalculates max Health/Stamina/Mana from stats | Internal |
| `SetValuesMax` | Sets current values to their max (e.g., on level up) | Internal |

### Variables we read/write
| Variable | Type | Notes |
|---|---|---|
| `Current_XP` | int | Current accumulated XP |
| `Max_XP` | int | XP needed for next level |
| `CharacterLevel` | int | Current level |
| `Health` / `MaxHealth` | real (replicated) | Driven by stats; we read only |
| `XPIncrease` | int | The pending XP delta — `ApplyXPIncrease` consumes this |
| `OriginalXPIncrease` | int | Stored unmodified for revert/scale |
| `LeveledUpStats` | array<S_CharacterStats> | Save-game backed stat history |
| `StatPointsAvailable` | int | Available stat points (Stat chest awards via this path) |
| `StatPointsPerLevel` | int | How many stat points per level |
| `StatsToUpdate` | array<S_CharacterStats> | The active stat block |

### Multicast Delegates we bind
| Dispatcher | Fires on |
|---|---|
| `UpdateHealth` | Health value changed (HUD bar bind) |
| `UpdateXP` | XP value changed (HUD orb count bind) |
| `UpdateStats` | Any stat block changed |
| `UpdateLevel` | Character level changed (level-up notif) |
| `UpdateCharacterBarValues` | Bulk update — bind once, all bars refresh |

These already exist in stock FCS — we just bind GoR HUD widgets to them.

---

## IMPLEMENTATION

### EXP Orb Blueprint (`BP_EXPOrb`)
- **Parent:** Actor (NOT `BP_LootActorParent` — auto-collect, no interact prompt)
- **Components:** Sphere collision (~150u radius), small mesh, Niagara trail
- **Logic on `BeginOverlap`** (player only):
  1. Cast to `BP_PlayerCharacter`
  2. Get `EquipmentComponent` reference
  3. Set `XPIncrease` to the orb's value (1 / 10 / 50 by enemy tier)
  4. Call `ApplyXPIncrease` (no params — it reads `XPIncrease`)
  5. Spawn pickup VFX
  6. `DestroyActor`

### Enemy Death → Orb Spawn
On `BP_AI_Parent` death event (in the GoR child of the AI parent, NOT the FCS parent):
- Spawn N orbs based on tier (`E_AITier`):
  - Grunt → 1 orb of value 1
  - Elite → 1 orb of value 10
  - Boss → 5 orbs of value 10 = 50 total (so the player still feels the burst)

### Stat-Chest Hook (Gold/Silver use FCS Heal + XP, Stat uses this)
```
BP_ChestBase ChestType=STAT branch:
  → EquipmentComponent.IncreaseLevelUpStats
  → broadcasts UpdateStats automatically
```

### Save Integration
`BP_SaveGame.LeveledUpStats` (audited — `array<S_CharacterStats>`) already holds the per-level stat history. No new save fields needed for health scaling. Add only:
- `SpiritPuzzlesSolved` (array of name) — which optional puzzles this run cleared
- Used to compute `MaxSpirit = 100 + (count * 15)` on load

---

## TESTING
1. Spawn a grunt at L1 spawn point → kill → confirm 1 orb drops, auto-collects, XP bar moves +1.
2. Spawn enough grunts to level up → confirm `Character Level Up` fires, `MaxHealth` increases.
3. Place a Stat chest → confirm `IncreaseLevelUpStats` adds a permanent stat point.
4. Solve a Zone E test puzzle → confirm `MaxSpirit` increments by 15 and persists across save/load.

---

## OPEN QUESTIONS
- Does `ApplyXPIncrease` accept an int parameter or only read the `XPIncrease` variable?
  Audit shows the variable + a function with the same name — the user's design doc says
  `ApplyXPIncrease(int)` but the function signature wasn't fully exposed. Pin this on
  the next graph read.
- Does `IncreaseLevelUpStats` take the stat type as a param, or use `StatsToUpdate`?
  Need to read the function graph to know whether the Stat chest needs to set a variable first.
