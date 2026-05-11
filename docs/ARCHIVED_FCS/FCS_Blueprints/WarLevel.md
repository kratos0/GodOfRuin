# WarLevel — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/WarLevel/`
> Total assets: 4 | All relevant — none skipped

### System Overview

WarLevel is FCS's large-scale battle mode — it spawns two opposing armies in formation and lets them fight autonomously, with the player watching from a top-down camera pawn. The system has four parts:

```
BP_WarLevel          ← GameMode governing the battle session
BP_WarLevelCameraPawn  ← Top-down/RTS camera for watching the battle
BP_TeamManager       ← Per-team root actor; owns a spawner reference
BP_UnitSpawner       ← Spawns one formation (N rows × M columns) of a chosen unit type
```

**Setup pattern:** Place one `BP_TeamManager` per side. Each `BP_TeamManager` holds a reference to a `BP_UnitSpawner`. Configure the spawner's unit type and formation dimensions. Set `BP_WarLevel` as the World Settings GameMode. The player's Pawn is `BP_WarLevelCameraPawn`.

---

## BP_WarLevel
- **Type:** Blueprint
- **Parent:** GameModeBase
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/WarLevel/BP_WarLevel`
- **Purpose:** Minimal GameMode for the war battle level — sets up the game rules context for a mass-battle session; all actual unit management is delegated to the `BP_TeamManager` / `BP_UnitSpawner` actors in the level.
- **Key Variables:** None own (0 variables, 10 nodes)
- **Key Components:**
  - `DefaultSceneRoot` : SceneComponent
- **GodOfRuin Use:** **USE** — set as the GameMode in World Settings for any large-scale battle level. If GodOfRuin needs battle-specific game rules (victory condition, timer, score), extend this BP and add the logic here rather than in `BP_TeamManager`.
- **Notes:** Shell GameMode — only 10 nodes across 2 graphs. All the substantive battle logic is in the spawner/manager actors. This is intentionally thin so the level design drives the battle configuration.

---

## BP_TeamManager
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/WarLevel/BP_TeamManager`
- **Purpose:** Per-team root actor — holds a reference to the team's `BP_UnitSpawner` and provides a pivot transform (via `Rotator` SceneComponent) that orients the spawned formation to face the enemy.
- **Key Variables:**
  - `TeamSpawner` (object) — reference to this team's `BP_UnitSpawner`; set in-editor by pointing at the spawner placed in the level
- **Key Components:**
  - `Rotator` : SceneComponent — the facing direction pivot; the spawner uses this transform to orient the unit formation toward the enemy team
  - `Spawner` : SceneComponent — the world position from which unit formation grid coordinates are calculated
  - `DefaultSceneRoot` : SceneComponent
- **GodOfRuin Use:** **USE** — place two instances (one per army) in any war-level. Rotate each `BP_TeamManager` to face the opposing team; the `Rotator` component orientation drives which way units face when spawned. Set `TeamSpawner` to point at the corresponding `BP_UnitSpawner`.
- **Notes:** The separation of `TeamManager` (orientation pivot) from `UnitSpawner` (formation logic) allows the spawner to be repositioned independently of the team's facing direction — useful when terrain forces the spawn point away from the front line.

---

## BP_UnitSpawner
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/WarLevel/BP_UnitSpawner`
- **Purpose:** Formation spawner — places AI units in a configurable grid (rows × columns) at runtime; supports 10 FCS unit archetypes selectable by `UnitType` byte, each configured via an individual struct variable.
- **Key Variables:**
  - `UnitType` (byte) — selects which unit archetype to spawn for this formation (0=Warrior, 1=Duelist, 2=Archer, 3=Assassin, 4=Tank, 5=Boxer, 6=Fire Mage, 7=Poison Mage, 8=Lightning Mage, 9=Frost Mage)
  - `Unit` (byte) — secondary unit selector (variant within a type)
  - `Faction` (byte) — which faction/team these units belong to; fed to `CombatStatusComponent` so units attack the opposing faction
  - `NumberOfRows` (int) — how many rows deep the formation is
  - `UnitsPerRow` (int) — how many units wide each row is
  - `SpacePerRow` (real) — Z-axis (depth) spacing between rows in world units
  - `SpacePerColumn` (real) — X-axis (width) spacing between units per row
  - `CurrentColumn` (int) — internal counter tracking current column during spawn loop
  - `UnitTransform` (struct) — computed world transform for the next unit to spawn
  - `SpawnedUnits` (object) — array reference to all spawned unit actors; used for tracking/cleanup
  - `BP_TeamManager` (object) — reference back to the owning team manager (for orientation)
  - `NewVar_0` (object) — unnamed variable; likely a leftover or in-progress addition
  - **Unit config structs (one per archetype):** `Warrior`, `Duelist`, `Archer`, `Assassin`, `Tank`, `Boxer`, `Fire Mage`, `Poison Mage`, `Lightning Mage`, `Frost Mage` — each struct defines the unit BP class, count, and any archetype-specific parameters for this spawner instance
- **Key Components:**
  - `LocationCreator` : SceneComponent — origin point for the formation grid; units are placed relative to this component's world transform
  - `DefaultSceneRoot` : SceneComponent
- **GodOfRuin Use:** **USE/EXTEND** — use for large battle sequences. Set `UnitType` to select the archetype, configure `NumberOfRows` × `UnitsPerRow` for army size, and tune `SpacePerRow` / `SpacePerColumn` for formation density. To add a Paragon character as a unit type, add a new struct variable and a new case in the `UnitType` switch in EventGraph.
- **Notes:**
  - **Known scalability issue:** Each of the 10 unit archetypes has its own dedicated struct variable rather than a single `TArray<FUnitConfig>`. Adding an 11th unit type requires adding a new variable and editing the EventGraph switch — there is no data-driven unit registration. See KnownIssues.md (wave system 7-struct pattern — same architectural limitation applies here with 10 structs).
  - `NewVar_0` (unnamed object variable) is a dev leftover — do not rely on it; it may be removed in a future FCS update.
  - Formation size scales linearly with performance cost — 10 rows × 20 units = 200 AI characters simultaneously simulating full FCS combat AI. Profile before committing to large formation counts; the BT_MixedCombat behavior tree runs on every unit.
  - `Faction` byte must match the faction system used by `CombatStatusComponent` — verify the faction enum values align with your AI setup before placing in levels.

---

## BP_WarLevelCameraPawn
- **Type:** Blueprint
- **Parent:** DefaultPawn
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/WarLevel/BP_WarLevelCameraPawn`
- **Purpose:** Top-down/RTS-style observer camera pawn for watching the battle — the player possesses this pawn in the war level instead of a character; stores starting position and rotation for reset.
- **Key Variables:**
  - `StartLocation` (struct) — the world position to spawn/reset the camera to
  - `CamStartLoca` (struct) — the camera component's starting local transform
  - `ControlRot` (struct) — the starting control rotation (pitch/yaw for the overhead angle)
- **Key Components:**
  - `Camera` : CameraComponent — the battle overview camera
- **GodOfRuin Use:** **USE** — set as the default pawn class in `BP_WarLevel` (or via World Settings). Position `StartLocation` to frame the battlefield from a good overview angle. If GodOfRuin's battle sequences use a cinematic camera instead of player control, this pawn can be possessed by a Sequencer camera cut or replaced entirely with a Cine Camera Actor.
- **Notes:** Extends `DefaultPawn` — it has built-in WASD/mouse movement for panning and rotating the view. If the battle should be fully cinematic with no player camera control, disable input on possession or swap to a non-pawn camera solution.
