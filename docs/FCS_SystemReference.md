# FILE 1: docs/FCS_SystemReference.md
# =====================================
# INSTRUCTIONS: This is your main FCS reference.
# Update after EVERY session when Claude Code 
# discovers new system details.
# HOW: Copy exact function names and paths from 
# Claude Code's audit outputs into the right section.
# =====================================

```markdown
# FCS System Reference
## Gods of Ruin — Living Documentation
Last Updated: May 2, 2026
Project: GodOfRuin (FCS Bundle renamed)
Engine: Unreal Engine 5

---

## HOW TO USE THIS DOCUMENT

### Claude Code Session Start Prompt:
```
Fetch and read before starting:
https://raw.githubusercontent.com/kratos0/GodOfRuin/main/docs/FCS_SystemReference.md
https://raw.githubusercontent.com/kratos0/GodOfRuin/main/docs/KnownIssues.md

Rules:
- Follow all documented patterns exactly
- Use exact function names listed here
- Check Known Issues before building anything
- Do read-only audit of any system NOT documented here before touching it
- Report new discoveries at session end for doc update
```

---

## CONTENT ROOT
All FCS content lives under:
`/Game/FlexibleCombatSystem/`
NOT /Game/GodOfRuin/ — the project was renamed but content path is unchanged.

---

## INTERACT SYSTEM

### How It Works
1. Player presses E → `PlayerInputs(E_Inputs.Interact)`
2. `CombatStatusComponent.Interact()` fires
3. Gets overlapping actors from player `CapsuleComponent`
4. Checks per actor:
   - Implements `I_Interact_C` interface
   - Has `OutlineComponent`
   - `OutlineComponent.ActorTagVisible == true`
   - `OutlineComponent.ActorIsActive == true`
5. Calls `Interact (Message)` on matching actor
6. Passes `Character` reference on the event pin

### Key Blueprints
- `CombatStatusComponent` — drives the interact call
- `I_Interact_C` — interface all interactable actors implement
- `OutlineComponent` — required on any interactable actor
- `BP_LootActorParent` — base class for all interactables

### To Make Something Interactable
- Parent: `BP_LootActorParent` (inherits everything needed)
- Override `Event Interact` only
- Do NOT call Parent event (opens loot menu instead)
- Character reference comes in on event pin directly — no GetPlayerCharacter needed

### Exact Function Names
| Function | Location | Notes |
|----------|----------|-------|
| Heal | BuffComponent | Heals to FULL — no amount param |
| ApplyXPIncrease(int) | EquipmentComponent | Adds XP |
| IncreaseLevelUpStats | EquipmentComponent | Same as level-up stat bump |
| Event Interact | I_Interact_C interface | Override this in child BPs |

### Gotchas
- `Heal` goes to full HP — no partial heal available. Need custom implementation for F-key heal
- `BP_LootActorParent` child creation fails if asset already exists at path — delete first
- `OutlineComponent` needs mesh reference via `RetrieveOutlineInfo` on BeginPlay
- `ActorTagVisible` is set when player enters the inherited `SphereComponent` radius (100 units)

---

## DAMAGE SYSTEM

### Key Components
- `TakingDamage` component — lives on `BP_AI_Parent`
- All FCS enemies inherit from `BP_AI_Parent`
- All Paragon enemy BPs = child of `BP_AI_Parent` → inherit TakingDamage automatically

### Player Damage Path
*(Full audit pending Session 3)*
- Known: `Event Any Damage` on player → routes to FCS damage handler
- Boss Toolkit bridge: Boss attack → `Event Any Damage` on FCS player

### Enemy Damage Path
- FCS player attack → `TakingDamage` component on enemy
- `TakingDamage` handles health reduction, hit reactions, death

### Key Paths
- `TakingDamage`: `/Game/FlexibleCombatSystem/Blueprints/Components/TakingDamage`
- `BP_AI_Parent`: `/Game/FlexibleCombatSystem/Blueprints/AI/BP_AI_Parent`

---

## WAVE SYSTEM

### How It Works
1. `BoxComponent` overlap → `BeginArena` custom event fires
2. 7 wave option structs: `Option 1` through `Option 1_6`
3. Random pick each round via `Random Integer in Range` → `Select`
4. `DeadEnemies` int increments on each enemy death
5. When `DeadEnemies == total spawned` → triggers next wave
6. `BP_Gate` reference stored as direct object variable
7. `OpenGates` / `CloseGates` called directly on `BP_Gate`

### Added in GodOfRuin
- `TotalRounds` (int, editable per instance) — how many rounds before complete
- `OnAllWavesComplete` event dispatcher — fires when `CurrentRound >= TotalRounds`
- Hooked: fires `OnAllWavesComplete` AND calls `OpenGates` on completion

### Key Paths
- `BP_ArenaSpawner`: `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/DemoLevels/BP_ArenaSpawner`
- `BP_ArenaGate`: `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/DemoLevels/BP_ArenaGate`

### Gotchas
- `BP_Gate` variable must be assigned in level Details panel — null reference = crash on BeginPlay
- No built-in `OnAllWavesComplete` in stock FCS — we added it
- Wave options are loose struct variables, not an array — not easily scalable

---

## CLIMBING SYSTEM

### How It Works
- `ClimbingComponent` on player drives all climbing logic
- `I_Climbable` interface on any surface actor makes it climbable
- `E_ClimbableType` enum defines surface type: `Wall` / `Ledge` / `Pole`
- No game tags — purely interface-based detection

### To Make a Surface Climbable
1. Add `I_Climbable` interface to the actor Blueprint
2. Implement the interface
3. Set `E_ClimbableType` to `Wall`, `Ledge`, or `Pole`
4. That's it — player ClimbingComponent handles the rest

### Key Paths
- `ClimbingComponent`: `/Game/FlexibleCombatSystem/Climbing/ClimbingComponent`
- `I_Climbable`: `/Game/FlexibleCombatSystem/Climbing/I_Climbable`
- `E_ClimbableType`: `/Game/FlexibleCombatSystem/Climbing/E_ClimbableType`

---

## SAVE SYSTEM

### Key Blueprints
- `BP_SaveGame`: `/Game/FlexibleCombatSystem/Blueprints/BP_SaveGame`
- `BP_SaveStorage`: `/Game/FlexibleCombatSystem/Blueprints/BP_SaveStorage`

### Stock FCS Saves
*(Audit pending — fill in after Session 3)*
- [ ] List all default saved variables here after audit

### Added in GodOfRuin
- `CompletedLevels` (int array) — which levels are done
- `UnlockedAbilities` (string array) — ability unlock tracking
- `HasCompanion` (bool, default false)
- `SelectedCharacterID` (name) — which premade was chosen

### ApplyLevelCompletion Function
Added to FCS player Blueprint. Called at end of each level.
| Level | Action |
|-------|--------|
| L1 | Add "Axe" → BP_WeaponHotkeys unlocks slot 2 |
| L2 | Health upgrade + add "MagicStaff" → slot 4 |
| L3 | Add "Bow" → slot 3 |
| L4 | HasCompanion=true, activate FCS companion |
| L5 | Add "DivineWrath" |

*(Exact FCS save function names — fill in after Session 3 audit)*

---

## HUD SYSTEM

### Existing FCS Widgets (do not rebuild)
| Widget | Purpose |
|--------|---------|
| `WB_HUD` | Main in-game HUD container |
| `WB_CharacterPortrait` | Player portrait |
| `WB_ActionBar` + `WB_ActionBarSlot` | Hotbar |
| `WB_AI-Health` | Enemy health bar (world space) |
| `WB_PopupText` | Damage numbers |
| `WB_DeathScreen` | On player death |
| `WB_PlayerMenu` | Full player menu |
| `WB_XPGain` | XP gain notification |
| `WB_LevelUp` + `WB_LevelUpScreen` | Level up UI |
| `WB_MiniMap` + `WB_Compass` | Map UI |

### Added in GodOfRuin
- Wave indicator: WAVE X/Y center-top, hidden by default
- Boss health bar: bottom-center, hidden by default
- Ability unlock notification: 2s popup
- 3 health charge icons for F-key heal system

### Widget Crash Gotchas
- `WB_PickedUpItemPopup` — requires `WB_PopupHolder` + `ItemInfo` struct params. Crashes if null.
- `WB_XPGain` — same parent widget issue. Crashes if null.
- Use `PrintString` for notifications until widget system is properly wired

---

## INPUT SYSTEM

### Inventory Key
- `IA_Open_Player_Menu`: `/Game/FlexibleCombatSystem/Input/Keybinds/Menus/IA_Open_Player_Menu`
- Status: **SUPPRESSED** in GodOfRuin — binding removed, all inventory widgets hidden

### Interact Key
- Routed through `PlayerInputs(E_Inputs.Interact)`
- Handled by `CombatStatusComponent.Interact()`

---

## KEY BLUEPRINT PATHS (QUICK REFERENCE)

### Player
```
BP_PlayerCharacter:  /Game/FlexibleCombatSystem/Blueprints/BP_PlayerCharacter
BP_PlayerController: /Game/FlexibleCombatSystem/Blueprints/BP_PlayerController
BP_PlayerHUD:        /Game/FlexibleCombatSystem/Blueprints/BP_PlayerHUD
```

### Enemy AI
```
BP_AI_Parent:    /Game/FlexibleCombatSystem/Blueprints/AI/BP_AI_Parent
BP_AI_NPC:       /Game/FlexibleCombatSystem/Blueprints/AI/AI-Types/BP_AI_NPC
BP_AIController: /Game/FlexibleCombatSystem/Blueprints/AI/BP_AIController
```

### Components
```
CombatStatusComponent: /Game/FlexibleCombatSystem/Blueprints/Components/CombatStatusComponent
TakingDamage:          /Game/FlexibleCombatSystem/Blueprints/Components/TakingDamage
WeaponCollision:       /Game/FlexibleCombatSystem/Blueprints/Components/WeaponCollision
LootComponent:         /Game/FlexibleCombatSystem/Blueprints/Components/LootComponent
InventoryComponent:    /Game/FlexibleCombatSystem/Blueprints/Components/InventoryComponent
```

### Save
```
BP_SaveGame:    /Game/FlexibleCombatSystem/Blueprints/BP_SaveGame
BP_SaveStorage: /Game/FlexibleCombatSystem/Blueprints/BP_SaveStorage
```

### Items + Loot
```
BP_LootActorParent: /Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_LootActorParent
BP_PickupItem:      /Game/FlexibleCombatSystem/Blueprints/Items/BP_PickupItem
```

### GodOfRuin Custom Blueprints
```
BP_ChestBase:       /Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_ChestBase
BP_WeaponHotkeys:   /Game/FlexibleCombatSystem/Blueprints/GodOfRuin/BP_WeaponHotkeys
BP_HealthCharges:   /Game/FlexibleCombatSystem/Blueprints/GodOfRuin/BP_HealthCharges
BP_LevelEndTrigger: /Game/FlexibleCombatSystem/Blueprints/GodOfRuin/BP_LevelEndTrigger
```

### Levels
```
L_ArenaLevelBattleArena: /Game/FlexibleCombatSystem/Maps/L_ArenaLevelBattleArena
L_BaseLevelBattleArena:  /Game/FlexibleCombatSystem/Maps/L_BaseLevelBattleArena
```

---

## SKELETON REFERENCE

### FCS Manny Skeleton (use for ALL Paragon imports)
```
/Game/FlexibleCombatSystem/GASP/UEFN-Manny/Meshes/SK_UEFN_Mannequin.SK_UEFN_Mannequin
```

### Boss Toolkit Skeletons (do not retarget to these)
```
SK_Terra_Skeleton:    /Game/BossAIToolkit/Demo/Terra/Meshes/
SK_Countess_Skeleton: /Game/BossAIToolkit/Demo/Countess/
SK_Sevarog_Skeleton:  /Game/BossAIToolkit/Demo/Sevarog/Meshes/
SK_Rampage_Skeleton:  /Game/BossAIToolkit/Demo/Rampage/Meshes/
```

---

## SESSION LOG

### Session 1 — May 2, 2026 — Status: In Progress
**Completed:**
- NeoStack connected and working
- Deep FCS audit (Prompt 0a) completed
- FCS Manny skeleton path confirmed
- BP_ArenaSpawner extended with TotalRounds + OnAllWavesComplete
- CommonInput plugin issue identified and resolved
- Project plays in PIE
- FCS interact system fully audited
- GitHub documentation system established

**In Progress:**
- BP_ChestBase — needs BP_LootActorParent inheritance + interact logic

**Blocked:**
- Nothing currently blocked

**Next Session:**
- Complete BP_ChestBase
- Build BP_LevelEndTrigger
- Extend FCS HUD + save system
- Damage bridge FCS ↔ Boss Toolkit
- Shared Blueprints + L_Template
```

