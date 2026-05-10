# Checkpoint System
## Status: Audited
## Last Updated: 2026-05-10

---

## WHAT IT DOES
FCS provides `BP_CheckPoint` — a placement actor that records the player's
location and triggers a save when the player enters its overlap volume. On
load, the player respawns at the last-used checkpoint.

For God of Ruin we use this for **invisible auto-save points** between zones.
No UI, no player input — overlap fires the save silently.

---

## EXACT STRUCTURE (from audit)

### Blueprint Path
```
/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/BP_CheckPoint
```

### Parent Class
`Actor`

### Variables (2)
| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| `Load?` | bool | (per instance) | Marks this as a "load on next spawn" checkpoint |
| `PlayerLocation` | Vector | — | Stored player location written on overlap |

### Components (4)
| Component | Class | Purpose |
|-----------|-------|---------|
| `DefaultSceneRoot` | SceneComponent | Root |
| `Box` | BoxComponent | Overlap volume — drives the save trigger |
| `FloppyDisk` | StaticMeshComponent | Default visible mesh (disable for invisible checkpoints) |
| `RotatingMovement` | RotatingMovementComponent | Spins the floppy disk visual |

### Graphs (2)
- `EventGraph` — overlap → write `PlayerLocation` → mark `Load?` → save
- `UserConstructionScript` — construction-time setup

### Stats
- 33 nodes, 1 function, 0 macros, 0 interfaces

---

## SAVE INTEGRATION
The checkpoint pairs with `BP_SaveGame` which has these relevant fields
(audited — see `BP_SaveGame` section below):

| BP_SaveGame Variable | Type | Role |
|---|---|---|
| `CheckpointsUsed` | array<S_ActorSaveInfo> | Per-checkpoint metadata (which was last used) |
| `CharactersLocation_SaveGame` | Transform | Player respawn transform |
| `SpawnOnLevelStart` | bool | If true, use `CharactersLocation_SaveGame` on next load |
| `UseCharacterLastCameraView` | bool | Restore camera angle on respawn |
| `PlayerCameraRotation` | Rotator | Stored camera rotation |
| `CurrentHealth` / `CurrentStamina` / `CurrentMana` | real | Restored on respawn |

The checkpoint's overlap event writes its `PlayerLocation` → `BP_SaveGame.CharactersLocation_SaveGame`,
appends to `CheckpointsUsed`, and flips `SpawnOnLevelStart` to true.

---

## HOW TO PLACE IN A LEVEL

### Standard (visible) checkpoint
1. Drag `BP_CheckPoint` into the level.
2. Position the `Box` overlap volume across the player's path between zones.
3. Leave default floppy disk mesh visible if you want the player to see it.
4. Test: walk through, exit PIE, re-enter — player respawns at last checkpoint.

### God of Ruin invisible checkpoint
1. Drag `BP_CheckPoint` in.
2. **Disable visibility** on the `FloppyDisk` mesh (or set `Hidden in Game = true`).
3. **Disable** the `RotatingMovement` component (no need to tick if invisible).
4. Place at zone boundaries:
   - Zone A → Zone B (after intro puzzle clears)
   - Zone D → Zone E (after mob gauntlet clears)
   - Pre-F (right before climax arena door)

### Per-level placement (from GodOfRuin_GameDesign.md)
| Level | Checkpoint locations |
|-------|---------------------|
| L1–L8 | Zone A→B, Zone D→E, pre-F (3 per level, 24 total) |

---

## RESPAWN BEHAVIOUR
- On player death: `BP_PlayerCharacter` death handler → load `BP_SaveGame`.
- Player spawns at `CharactersLocation_SaveGame` if `SpawnOnLevelStart` is true.
- Health, stamina, mana restored from `CurrentHealth/Stamina/Mana`.
- Camera rotation restored if `UseCharacterLastCameraView` is true.
- No respawn UI shown (matches the "seamless, invisible" design rule).

---

## INTEGRATION CHECKLIST
- [ ] Verify overlap event in `BP_CheckPoint.EventGraph` writes to `BP_SaveGame`
  (open in editor — confirm save call is wired before relying on this for L1).
- [ ] Confirm `SpawnOnLevelStart` is the flag the player respawn path checks
  (audit `BP_PlayerCharacter` BeginPlay if respawn doesn't fire).
- [ ] Hide `FloppyDisk` for all GodOfRuin level instances.
- [ ] Tag each checkpoint with its zone in the actor label (e.g., `CP_L1_A_to_B`).

---

## GOTCHAS
- Don't enable `Load?` on every checkpoint — only the most recent should be true.
  The standard FCS pattern toggles old ones false when a new one fires.
- Box overlap fires on *any* pawn — wrap the save call in a player-class check
  if you have AI passing through the same volume.
- Visible floppy disk + rotating movement is the FCS demo aesthetic — always
  disable both for GodOfRuin (per the "no UI, invisible" rule).

---

## OPEN QUESTIONS
- Does `BP_CheckPoint.EventGraph` directly call SaveGame, or does it route
  through a save manager? Need to read the EventGraph to confirm. *(33 nodes
  total — small enough to trace in one read pass.)*
- What clears `CheckpointsUsed` between runs? Probably level transition
  resets it — verify before L8 → L1 New Game flow.
