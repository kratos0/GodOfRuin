# FCS System Reference
## Gods of Ruin — Living Documentation
Last updated: May 2, 2026

---

## INTERACT SYSTEM

### How It Works
- Player presses E → PlayerInputs(E_Inputs.Interact)
- CombatStatusComponent.Interact() fires
- Gets overlapping actors from player CapsuleComponent
- Checks: implements I_Interact_C + has OutlineComponent
  + ActorTagVisible=true + ActorIsActive=true
- Calls Interact(Message) on matching actor
- Character reference passed on the event pin directly

### Key Blueprints
- CombatStatusComponent — drives the interact call
- I_Interact_C — interface all interactable actors implement
- OutlineComponent — required on any interactable actor
- BP_LootActorParent — base class for all interactables

### To Make Something Interactable
- Parent: BP_LootActorParent (inherits everything)
- Override Event Interact only
- Do NOT call Parent event (opens loot menu instead)
- Character reference comes in on event pin directly

### Exact Function Names
- Heal: BuffComponent → Heal (heals to full, no amount param)
- XP: EquipmentComponent → ApplyXPIncrease(int)
- Stats: EquipmentComponent → IncreaseLevelUpStats
- Interact trigger: I_Interact_C → Event Interact

### Gotchas
- Heal goes to full HP — no partial heal, need custom impl
- BP_LootActorParent creation fails if asset already exists at path — delete existing asset first
- CommonInput plugin must be DISABLED in this project
- OutlineComponent needs mesh ref via RetrieveOutlineInfo

---

## DAMAGE SYSTEM

### Key Components
- TakingDamage — component on BP_AI_Parent
- All Paragon enemies = child of BP_AI_Parent
- Inherits TakingDamage automatically

### To Wire Boss Toolkit → FCS Player Damage
- Boss attack → Event Any Damage on FCS player → TakingDamage component damage function

*(Full audit pending — Session 3)*

---

## WAVE SYSTEM (BP_ArenaSpawner)

### How It Works
- BoxComponent overlap → BeginArena custom event
- 7 wave option structs (Option 1 through Option 1_6)
- Random pick each round via Random Integer in Range
- DeadEnemies int tracks kills
- When DeadEnemies == spawned count → next wave
- BP_Gate reference stored directly — OpenGates/CloseGates called on it

### Added in GodOfRuin
- TotalRounds variable (editable per instance)
- OnAllWavesComplete event dispatcher
- Fires when CurrentRound >= TotalRounds

### Gotchas
- BP_Gate must be assigned in level Details panel or OpenGates crashes on null reference
- No built-in OnAllWavesComplete — we added it

---

## CLIMBING SYSTEM

### How It Works
- ClimbingComponent on player drives all climbing
- I_Climbable interface on any climbable surface actor
- E_ClimbableType enum: Wall / Ledge / Pole
- No game tags — pure interface based

### To Make a Surface Climbable
1. Add I_Climbable interface to the actor
2. Implement the interface
3. Set E_ClimbableType to Wall, Ledge, or Pole

---

## SAVE SYSTEM

### Key Blueprints
- BP_SaveGame — save data container
- BP_SaveStorage — storage manager

### Added in GodOfRuin
- CompletedLevels (int array)
- UnlockedAbilities (string array)
- HasCompanion (bool, default false)
- SelectedCharacterID (name)
- ApplyLevelCompletion(LevelNumber int) function

### ApplyLevelCompletion Logic
- L1: add "Axe" → BP_WeaponHotkeys unlocks slot 2
- L2: health upgrade + add "MagicStaff" → slot 4
- L3: add "Bow" → slot 3
- L4: HasCompanion=true, activate FCS companion
- L5: add "DivineWrath"
- Each: call FCS existing save function

*(Full function names pending Session 3 audit)*

---

## HUD SYSTEM

### Existing FCS Widgets (do not rebuild)
- WB_HUD — main in-game HUD container
- WB_CharacterPortrait — player portrait
- WB_ActionBar + WB_ActionBarSlot — hotbar
- WB_AI-Health — enemy health bar (world space)
- WB_PopupText — damage numbers
- WB_DeathScreen — on player death
- WB_PlayerMenu — full player menu (opened by IA_Open_Player_Menu)

### Added in GodOfRuin
- Wave indicator: WAVE X/Y center-top, hidden by default
- Boss health bar: bottom-center, hidden by default
- Ability unlock notification: 2s popup on new unlock
- 3 health charge icons for F-key heal system

### Widget Crash Gotchas
- WB_PickedUpItemPopup requires WB_PopupHolder + ItemInfo params — crashes if null
- WB_XPGain has same parent widget issue — also crashes if null
- Use PrintString for notifications until widget system is properly wired

---

## SKELETON REFERENCE

### FCS Manny Skeleton (use for ALL Paragon imports)

### Boss Toolkit Skeletons (do not retarget to these)
- SK_Terra_Skeleton → /Game/BossAIToolkit/Demo/Terra/Meshes/
- SK_Countess_Skeleton → /Game/BossAIToolkit/Demo/Countess/
- SK_Sevarog_Skeleton → /Game/BossAIToolkit/Demo/Sevarog/Meshes/
- SK_Rampage_Skeleton → /Game/BossAIToolkit/Demo/Rampage/Meshes/

### FCS Built-in Enemy Skeletons (not needed for Paragon work)
- Spider Boss, Wolf, Undead Knight, Ogre, Goblin — all present

---

## KEY BLUEPRINT PATHS

### Player
- BP_PlayerCharacter: /Game/FlexibleCombatSystem/Blueprints/BP_PlayerCharacter
- BP_PlayerController: /Game/FlexibleCombatSystem/Blueprints/BP_PlayerController

### Enemy AI
- BP_AI_Parent: /Game/FlexibleCombatSystem/Blueprints/AI/BP_AI_Parent
- BP_AI_NPC: /Game/FlexibleCombatSystem/Blueprints/AI/AI-Types/BP_AI_NPC
- BP_AIController: /Game/FlexibleCombatSystem/Blueprints/AI/BP_AIController

### Components
- CombatStatusComponent: /Game/FlexibleCombatSystem/Blueprints/Components/CombatStatusComponent
- TakingDamage: /Game/FlexibleCombatSystem/Blueprints/Components/TakingDamage
- WeaponCollision: /Game/FlexibleCombatSystem/Blueprints/Components/WeaponCollision
- LootComponent: /Game/FlexibleCombatSystem/Blueprints/Components/LootComponent
- InventoryComponent: /Game/FlexibleCombatSystem/Blueprints/Components/InventoryComponent

### Save
- BP_SaveGame: /Game/FlexibleCombatSystem/Blueprints/BP_SaveGame
- BP_SaveStorage: /Game/FlexibleCombatSystem/Blueprints/BP_SaveStorage

### HUD
- BP_PlayerHUD: /Game/FlexibleCombatSystem/Blueprints/BP_PlayerHUD

### Items + Loot
- BP_LootActorParent: /Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_LootActorParent
- BP_PickupItem: /Game/FlexibleCombatSystem/Blueprints/Items/BP_PickupItem

### GodOfRuin Custom
- BP_ChestBase: /Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_ChestBase
- BP_ArenaSpawner (extended): /Game/FlexibleCombatSystem/Blueprints/Miscellaneous/DemoLevels/BP_ArenaSpawner
- BP_ArenaGate: /Game/FlexibleCombatSystem/Blueprints/Miscellaneous/DemoLevels/BP_ArenaGate

### Levels
- L_ArenaLevelBattleArena: /Game/FlexibleCombatSystem/Maps/L_ArenaLevelBattleArena
- L_BaseLevelBattleArena: /Game/FlexibleCombatSystem/Maps/L_BaseLevelBattleArena

---

## INPUT SYSTEM

### Inventory Key
- IA_Open_Player_Menu: /Game/FlexibleCombatSystem/Input/Keybinds/Menus/IA_Open_Player_Menu
- Status: SUPPRESSED in GodOfRuin — inventory hidden

### Interact Key
- Routed through PlayerInputs(E_Inputs.Interact)
- Handled by CombatStatusComponent.Interact()

---

## KNOWN ISSUES + WORKAROUNDS

| Issue | Workaround |
|-------|-----------|
| CommonInput plugin missing | Click Yes to disable on project open |
| Bulk animation import crashes UE5 | Max 2-3 Paragon characters per session |
| WB_PickedUpItemPopup crashes | Requires WB_PopupHolder + ItemInfo — use PrintString instead |
| WB_XPGain crashes | Same parent widget issue — avoid for now |
| BP_LootActorParent child creation fails if asset exists | Delete existing asset at path first |
| BP_Gate null reference crash | Must assign BP_Gate in spawner Details panel in every level |
| UE5 crashes on Play | Check CommonInput disabled, check BP_Gate assigned |

---

## SESSION LOG

### Session 1 — May 2, 2026
**Status: In Progress**

Completed:
- NeoStack connected and working in UE5 Agent Chat
- Deep FCS audit (Prompt 0a) completed
- FCS Manny skeleton path confirmed
- BP_ArenaSpawner extended with TotalRounds + OnAllWavesComplete dispatcher
- CommonInput plugin issue identified and resolved — project now plays in PIE
- FCS interact system fully audited (see INTERACT SYSTEM above)
- BP_ChestBase creation in progress

In Progress:
- BP_ChestBase — needs correct BP_LootActorParent inheritance + interact logic
- BP_LevelEndTrigger — not started yet

Next Session:
- Complete BP_ChestBase
- Build BP_LevelEndTrigger
- Extend FCS HUD + save system
- Damage bridge FCS ↔ Boss Toolkit
- Build shared Blueprints + L_Template

---

## HOW TO USE THIS DOCUMENT

### At the start of every Claude Code session:
Paste this into NeoStack Agent Chat:

> "Load the FCS System Reference from docs/FCS_SystemReference.md before we begin."
