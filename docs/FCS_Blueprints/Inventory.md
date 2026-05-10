# FCS Blueprint Inventory

> **Generated:** 2026-05-09 via Lua `open_asset():info()` over `/Game/FlexibleCombatSystem/Blueprints/`
> **Scope:** Every asset under the FCS Blueprints folder (not just Blueprint class).
> **Purpose:** Quick lookup of every FCS asset with parent class + structural counts. For curated descriptions of key assets, see the per-folder docs (`AI.md`, `Components.md`, etc.) in this directory.

## Totals

- **Total assets:** 296
- **Blueprint-class assets:** 287
- **Non-Blueprint assets:** 9 (BehaviorTree, BlackboardData, EnvQuery, UserDefinedEnum, UserDefinedStruct, WaterWavesAsset)
- **Aggregate node count:** 41,675
- **Aggregate variable count:** 1,922
- **Folders:** 43

## Top 10 by Node Count

| Asset | Parent | Nodes | Vars | Funcs |
|---|---|---:|---:|---:|
| `BP_PlayerCharacter` | Character | 2,643 | 87 | 53 |
| `CombatStatusComponent` | ActorComponent | 2,597 | 77 | 54 |
| `EquipmentComponent` | ActorComponent | 2,474 | 106 | 52 |
| `InventoryComponent` | ActorComponent | 2,457 | 74 | 55 |
| `TakingDamage` | ActorComponent | 1,989 | 80 | 46 |
| `BP_AI_Parent` | Character | 1,932 | 62 | 49 |
| `RangedComponent` | ActorComponent | 1,590 | 69 | 48 |
| `MeleeComponent` | ActorComponent | 1,202 | 51 | 33 |
| `FunctionLibrary` | BlueprintFunctionLibrary | 932 | 0 | 54 |
| `SwimmingComponent` | ActorComponent | 871 | 34 | 20 |

## Top 10 by Variable Count

| Asset | Parent | Vars | Funcs | Nodes |
|---|---|---:|---:|---:|
| `EquipmentComponent` | ActorComponent | 106 | 52 | 2,474 |
| `BP_PlayerCharacter` | Character | 87 | 53 | 2,643 |
| `TakingDamage` | ActorComponent | 80 | 46 | 1,989 |
| `CombatStatusComponent` | ActorComponent | 77 | 54 | 2,597 |
| `InventoryComponent` | ActorComponent | 74 | 55 | 2,457 |
| `RangedComponent` | ActorComponent | 69 | 48 | 1,590 |
| `BP_AI_Parent` | Character | 62 | 49 | 1,932 |
| `BP_AIController` | AIController | 59 | 22 | 787 |
| `MeleeComponent` | ActorComponent | 51 | 33 | 1,202 |
| `MagicComponent` | ActorComponent | 45 | 29 | 857 |

## Folder Index

- [(root)](#root) — 11 assets
- [AI](#ai) — 5 assets
- [AI/AI-Spawners](#aiai-spawners) — 6 assets
- [AI/AI-Types](#aiai-types) — 3 assets
- [AI/BehaviourTrees](#aibehaviourtrees) — 4 assets
- [AI/Dummy](#aidummy) — 5 assets
- [AI/EQS](#aieqs) — 2 assets
- [AI/Tasks/GeneralCombat](#aitasksgeneralcombat) — 16 assets
- [AI/Tasks/MagicSpecificCombat](#aitasksmagicspecificcombat) — 6 assets
- [AI/Tasks/MeleeSpecificCombat](#aitasksmeleespecificcombat) — 8 assets
- [AI/Tasks/RangedSpecificCombat](#aitasksrangedspecificcombat) — 6 assets
- [AI/Tasks/Service](#aitasksservice) — 5 assets
- [Abilities](#abilities) — 10 assets
- [Abilities/Children](#abilitieschildren) — 11 assets
- [Abilities/SpawnedItems](#abilitiesspawneditems) — 3 assets
- [AnimNotifies](#animnotifies) — 4 assets
- [AnimNotifies/Magic](#animnotifiesmagic) — 9 assets
- [AnimNotifies/Melee](#animnotifiesmelee) — 10 assets
- [AnimNotifies/Ranged](#animnotifiesranged) — 7 assets
- [AttackCombos](#attackcombos) — 1 asset
- [Components](#components) — 33 assets
- [Crafting](#crafting) — 7 assets
- [GodOfRuin](#godofruin) — 1 asset
- [Interfaces](#interfaces) — 25 assets
- [Items](#items) — 1 asset
- [Items/Arrows](#itemsarrows) — 11 assets
- [Items/Containers](#itemscontainers) — 3 assets
- [Items/LootActors](#itemslootactors) — 4 assets
- [Items/PickUpItems](#itemspickupitems) — 7 assets
- [Miscellaneous](#miscellaneous) — 8 assets
- [Miscellaneous/ActivateActors](#miscellaneousactivateactors) — 2 assets
- [Miscellaneous/ActivateActors/ActivatingActors(Switch)](#miscellaneousactivateactorsactivatingactorsswitch) — 4 assets
- [Miscellaneous/ActivateActors/ActorsToActivate/Message](#miscellaneousactivateactorsactorstoactivatemessage) — 4 assets
- [Miscellaneous/ActivateActors/ActorsToActivate/Overlap](#miscellaneousactivateactorsactorstoactivateoverlap) — 1 asset
- [Miscellaneous/ActivateActors/ActorsToActivate/Touch](#miscellaneousactivateactorsactorstoactivatetouch) — 3 assets
- [Miscellaneous/DemoLevels](#miscellaneousdemolevels) — 21 assets
- [Miscellaneous/DragDropOperation](#miscellaneousdragdropoperation) — 1 asset
- [Miscellaneous/MainMenu](#miscellaneousmainmenu) — 11 assets
- [ResourceCollection](#resourcecollection) — 2 assets
- [ResourceCollection/Mining](#resourcecollectionmining) — 3 assets
- [ResourceCollection/Woodchopping](#resourcecollectionwoodchopping) — 3 assets
- [Swimming](#swimming) — 5 assets
- [WarLevel](#warlevel) — 4 assets

---

## Column Legend

- **Parent** — Direct parent class (`_C` suffix indicates another Blueprint).
- **V / F / C / M** — Variables / Functions / Components / Macros.
- **EG** — Event graphs (almost always 1 for actor BPs, 0 for interfaces and function libraries).
- **Nodes** — Total nodes across all graphs (rough proxy for complexity).
- **I** — Implemented interfaces.

## (root)

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_GameModeFCS` | Blueprint | GameModeBase | 18 | 21 | 1 | 1 | 1 | 615 | 1 |
| `BP_GameModeMainMenu` | Blueprint | GameModeBase | 0 | 1 | 1 | 0 | 1 | 2 | 0 |
| `BP_PlayerCharacter` | Blueprint | Character | 87 | 53 | 41 | 2 | 1 | 2,643 | 7 |
| `BP_PlayerController` | Blueprint | PlayerController | 28 | 17 | 0 | 1 | 1 | 649 | 1 |
| `BP_PlayerControllerMainMenu` | Blueprint | PlayerController | 1 | 1 | 0 | 0 | 1 | 16 | 1 |
| `BP_PlayerHUD` | Blueprint | HUD | 5 | 1 | 1 | 0 | 1 | 60 | 0 |
| `BP_SaveGame` | Blueprint | SaveGame | 31 | 0 | 0 | 0 | 1 | 0 | 0 |
| `BP_SaveStorage` | Blueprint | SaveGame | 8 | 0 | 0 | 0 | 1 | 0 | 0 |
| `FunctionLibrary` | Blueprint | BlueprintFunctionLibrary | 0 | 54 | 0 | 0 | 0 | 932 | 0 |
| `GenericUtilsLibrary` | Blueprint | BlueprintFunctionLibrary | 0 | 3 | 0 | 0 | 0 | 44 | 0 |
| `StructUtilsLibrary` | Blueprint | BlueprintFunctionLibrary | 0 | 2 | 0 | 0 | 0 | 7 | 0 |

## AI

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BB_AI` | BlackboardData | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `BP_AI-Jump-Link` | Blueprint | NavLinkProxy | 4 | 2 | 0 | 0 | 1 | 59 | 0 |
| `BP_AIController` | Blueprint | AIController | 59 | 22 | 1 | 2 | 1 | 787 | 1 |
| `BP_AI_Parent` | Blueprint | Character | 62 | 49 | 8 | 2 | 1 | 1,932 | 6 |
| `BP_PatrolPath` | Blueprint | Actor | 8 | 1 | 11 | 0 | 1 | 90 | 0 |

## AI/AI-Spawners

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_AISpawnMarker` | Blueprint | Actor | 1 | 1 | 1 | 0 | 1 | 4 | 1 |
| `BP_Spawner-Random` | Blueprint | Actor | 13 | 4 | 3 | 0 | 1 | 148 | 1 |
| `BP_Spawner-Set` | Blueprint | Actor | 9 | 4 | 2 | 0 | 1 | 111 | 1 |
| `E_AISpawnMethod` | UserDefinedEnum | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `I_SpawnMarker` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 2 | 0 |
| `S_AISpawnData` | UserDefinedStruct | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

## AI/AI-Types

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_AI_Animal` | Blueprint | BP_AI_Parent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_AI_Human` | Blueprint | BP_AI_Parent_C | 0 | 1 | 0 | 0 | 1 | 2 | 0 |
| `BP_AI_NPC` | Blueprint | BP_AI_Parent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |

## AI/BehaviourTrees

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BT_AnimalCombat` | BehaviorTree | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `BT_AnimalCompanionCombat` | BehaviorTree | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `BT_CompanionMixedCombat` | BehaviorTree | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `BT_MixedCombat` | BehaviorTree | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

## AI/Dummy

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_AI_AssassDummy` | Blueprint | BP_AI_Dummy_C | 0 | 1 | 0 | 0 | 1 | 7 | 0 |
| `BP_AI_BlockDummy` | Blueprint | BP_AI_Parent_C | 1 | 2 | 0 | 0 | 1 | 60 | 0 |
| `BP_AI_Dummy` | Blueprint | BP_AI_Parent_C | 0 | 1 | 0 | 0 | 1 | 2 | 0 |
| `BP_AI_InvinsibleDummy` | Blueprint | BP_AI_Dummy_C | 0 | 2 | 1 | 0 | 1 | 9 | 0 |
| `BP_AI_RangedAssassDummy` | Blueprint | BP_AI_Dummy_C | 0 | 1 | 1 | 0 | 1 | 6 | 0 |

## AI/EQS

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `EQS_AroundTarget` | EnvQuery | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `EQS_TestingPawn` | Blueprint | EQSTestingPawn | 0 | 1 | 0 | 0 | 1 | 4 | 0 |

## AI/Tasks/GeneralCombat

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `T_AIFalseAlarm` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 20 | 0 |
| `T_ClearFocus` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 8 | 0 |
| `T_DeactivateAI` | Blueprint | BTTask_BlueprintBase | 3 | 0 | 0 | 0 | 1 | 4 | 0 |
| `T_GetNoiseLocation` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 7 | 0 |
| `T_ResetComponents` | Blueprint | BTTask_BlueprintBase | 3 | 0 | 0 | 0 | 1 | 40 | 0 |
| `T_ReturnToLocation` | Blueprint | BTTask_BlueprintBase | 2 | 0 | 0 | 0 | 1 | 8 | 0 |
| `T_RollBackwards` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 22 | 0 |
| `T_RunToEnemyUpdated` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 26 | 0 |
| `T_SendDrawWeapon` | Blueprint | BTTask_BlueprintBase | 2 | 0 | 0 | 0 | 1 | 45 | 0 |
| `T_SendSheatheWeapon` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 11 | 0 |
| `T_SendToLastNoiseLocation` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 18 | 0 |
| `T_SetEnemyAsFocus` | Blueprint | BTTask_BlueprintBase | 4 | 2 | 0 | 0 | 1 | 81 | 0 |
| `T_SetMovementSpeed` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 8 | 0 |
| `T_ShouldSwitchAttackStyle` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 39 | 0 |
| `T_StopMovement` | Blueprint | BTTask_BlueprintBase | 3 | 0 | 0 | 0 | 1 | 5 | 0 |
| `T_StoreAITransform` | Blueprint | BTTask_BlueprintBase | 3 | 0 | 0 | 0 | 1 | 13 | 0 |

## AI/Tasks/MagicSpecificCombat

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `D_ShouldTeleport` | Blueprint | BTDecorator_BlueprintBase | 3 | 1 | 0 | 0 | 1 | 35 | 0 |
| `T_ChargeSpell` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 12 | 0 |
| `T_DetermineMagicSpell` | Blueprint | BTTask_BlueprintBase | 14 | 4 | 0 | 0 | 1 | 200 | 0 |
| `T_FireChannelSpell` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 13 | 0 |
| `T_FireSpell` | Blueprint | BTTask_BlueprintBase | 3 | 0 | 0 | 0 | 1 | 14 | 0 |
| `T_ReleaseSpell` | Blueprint | BTTask_BlueprintBase | 2 | 0 | 0 | 0 | 1 | 20 | 0 |

## AI/Tasks/MeleeSpecificCombat

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `D_IsInRange` | Blueprint | BTDecorator_BlueprintBase | 3 | 1 | 0 | 0 | 1 | 15 | 0 |
| `S_DetermineAttackType` | Blueprint | BTService_BlueprintBase | 5 | 2 | 0 | 0 | 1 | 62 | 0 |
| `T_LowerBlock` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 10 | 0 |
| `T_RemoveBlock` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 6 | 0 |
| `T_ResetBlock` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 10 | 0 |
| `T_SendLungeAttack` | Blueprint | BTTask_BlueprintBase | 2 | 0 | 0 | 0 | 1 | 29 | 0 |
| `T_SendOffAttack` | Blueprint | BTTask_BlueprintBase | 6 | 1 | 0 | 0 | 1 | 75 | 0 |
| `T_SendOffBlockChance` | Blueprint | BTTask_BlueprintBase | 3 | 0 | 0 | 0 | 1 | 37 | 0 |

## AI/Tasks/RangedSpecificCombat

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `T_ArrowLoadedCheck` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 6 | 0 |
| `T_AttemptShot` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 4 | 0 |
| `T_DrawArrow` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 11 | 0 |
| `T_FireArrow` | Blueprint | BTTask_BlueprintBase | 0 | 0 | 0 | 0 | 1 | 13 | 0 |
| `T_ProjectileShotAvailable` | Blueprint | BTTask_BlueprintBase | 2 | 0 | 0 | 0 | 1 | 29 | 0 |
| `T_SheatheArrow` | Blueprint | BTTask_BlueprintBase | 1 | 0 | 0 | 0 | 1 | 15 | 0 |

## AI/Tasks/Service

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `S_MageStrafe` | Blueprint | BTService_BlueprintBase | 10 | 2 | 0 | 0 | 1 | 131 | 0 |
| `S_SettingAnimalBehaviour` | Blueprint | BTService_BlueprintBase | 14 | 10 | 0 | 2 | 1 | 369 | 1 |
| `S_SettingCompanionBehaviour` | Blueprint | BTService_BlueprintBase | 18 | 12 | 0 | 2 | 1 | 278 | 1 |
| `S_SettingEnemyHumanBehaviour` | Blueprint | BTService_BlueprintBase | 18 | 11 | 0 | 2 | 1 | 329 | 1 |
| `S_Strafe` | Blueprint | BTService_BlueprintBase | 9 | 4 | 0 | 0 | 1 | 227 | 0 |

## Abilities

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_AbilityChannel` | Blueprint | BP_AbilityParent_C | 13 | 14 | 1 | 0 | 1 | 441 | 0 |
| `BP_AbilityMelee` | Blueprint | BP_AbilityParent_C | 10 | 6 | 1 | 0 | 1 | 287 | 0 |
| `BP_AbilityMovingChannel` | Blueprint | BP_AbilityParent_C | 10 | 2 | 0 | 0 | 1 | 152 | 0 |
| `BP_AbilityParent` | Blueprint | Actor | 26 | 16 | 2 | 0 | 1 | 468 | 0 |
| `BP_AbilityPlacementParent` | Blueprint | BP_AbilityParent_C | 3 | 1 | 0 | 0 | 1 | 172 | 0 |
| `BP_AbilityProjectile` | Blueprint | BP_AbilityParent_C | 4 | 4 | 1 | 0 | 1 | 167 | 0 |
| `BP_AbilityRanged` | Blueprint | BP_AbilityParent_C | 9 | 6 | 3 | 0 | 1 | 385 | 0 |
| `BP_AbilitySelfCast` | Blueprint | BP_AbilityParent_C | 1 | 1 | 1 | 0 | 1 | 59 | 0 |
| `BP_AbilitySkyChannel` | Blueprint | BP_AbilityParent_C | 10 | 4 | 0 | 0 | 1 | 250 | 0 |
| `BP_AbilitySpawnMagicWeapon` | Blueprint | BP_AbilityParent_C | 1 | 4 | 1 | 0 | 1 | 95 | 0 |

## Abilities/Children

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_AreaAttackSingleParent` | Blueprint | BP_AbilityPlacementParent_C | 4 | 1 | 0 | 0 | 1 | 59 | 0 |
| `BP_AreaAttackTickParent` | Blueprint | BP_AbilityPlacementParent_C | 6 | 1 | 0 | 0 | 1 | 92 | 0 |
| `BP_BuffParent` | Blueprint | BP_AbilitySelfCast_C | 1 | 1 | 0 | 0 | 1 | 15 | 0 |
| `BP_HealChannelAllyParent` | Blueprint | BP_AbilityChannel_C | 0 | 1 | 0 | 0 | 1 | 48 | 0 |
| `BP_HealChannelParent` | Blueprint | BP_AbilityChannel_C | 0 | 1 | 0 | 0 | 1 | 39 | 0 |
| `BP_HealSingleParent` | Blueprint | BP_AbilitySelfCast_C | 1 | 1 | 0 | 0 | 1 | 65 | 0 |
| `BP_NSTriggerSpellAttackParent` | Blueprint | BP_AbilityPlacementParent_C | 4 | 1 | 0 | 0 | 1 | 69 | 1 |
| `BP_Polymorph` | Blueprint | BP_AbilityProjectile_C | 0 | 1 | 1 | 0 | 1 | 8 | 0 |
| `BP_RuneParent` | Blueprint | BP_AbilityPlacementParent_C | 0 | 1 | 0 | 0 | 1 | 19 | 0 |
| `BP_SummonAlly` | Blueprint | BP_AbilityPlacementParent_C | 7 | 1 | 0 | 0 | 1 | 50 | 0 |
| `BP_Teleport` | Blueprint | BP_AbilityPlacementParent_C | 2 | 3 | 0 | 0 | 1 | 91 | 0 |

## Abilities/SpawnedItems

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_MovingDamagingSpell` | Blueprint | Actor | 14 | 1 | 2 | 0 | 1 | 115 | 0 |
| `BP_PhysicalRune` | Blueprint | Actor | 10 | 4 | 3 | 0 | 1 | 140 | 0 |
| `BP_PlacementCircle` | Blueprint | Actor | 5 | 1 | 2 | 0 | 1 | 26 | 0 |

## AnimNotifies

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `AN_InputBufferSwitch` | Blueprint | AnimNotifyState | 0 | 2 | 0 | 0 | 0 | 14 | 0 |
| `AN_RollAdjustment` | Blueprint | AnimNotifyState | 0 | 2 | 0 | 0 | 0 | 26 | 0 |
| `AN_ThrowItem` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_ThrowItemCollision` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |

## AnimNotifies/Magic

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `AN_MeleeChargeTrigger` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_RangedAbilityBowString` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_RangedAbilityFire` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_RangedAbilityHandToString` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_ToggleKnockbackAndCollision` | Blueprint | AnimNotifyState | 0 | 2 | 0 | 0 | 0 | 16 | 0 |
| `AN_ToggleRangedCamera` | Blueprint | AnimNotifyState | 0 | 2 | 0 | 0 | 0 | 26 | 0 |
| `AN_TriggerAndStopAbility` | Blueprint | AnimNotifyState | 0 | 2 | 0 | 0 | 0 | 12 | 0 |
| `AN_TriggerMagicAbility` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_TriggerNextAbility` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |

## AnimNotifies/Melee

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `AN_CinematicApplyDmg` | Blueprint | AnimNotify | 2 | 1 | 0 | 0 | 0 | 7 | 0 |
| `AN_ExecuteCombatText` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 5 | 0 |
| `AN_LungeMovementMode` | Blueprint | AnimNotifyState | 0 | 2 | 0 | 0 | 0 | 10 | 0 |
| `AN_MotionWarping_Attack` | Blueprint | AnimNotifyState_MotionWarping | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `AN_Ragdoll` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 8 | 0 |
| `AN_SetMovementMode` | Blueprint | AnimNotify | 4 | 1 | 0 | 0 | 0 | 7 | 0 |
| `AN_ToggleAttackRotate` | Blueprint | AnimNotifyState | 0 | 3 | 0 | 0 | 0 | 24 | 0 |
| `AN_TriggerNextAttack` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_WeaponHitToggle` | Blueprint | AnimNotifyState | 0 | 3 | 0 | 0 | 0 | 45 | 0 |
| `AN_WeaponTrail` | Blueprint | AnimNotifyState | 0 | 3 | 0 | 0 | 0 | 55 | 0 |

## AnimNotifies/Ranged

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `AN_ArrowLoaded` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_AttachArrowString` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_DrawBowString` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_ExecuteCameraSwap` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_FireArrow` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_ReleaseBowString` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |
| `AN_SetupArrow` | Blueprint | AnimNotify | 0 | 1 | 0 | 0 | 0 | 6 | 0 |

## AttackCombos

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ComboResolver` | Blueprint | Object | 3 | 12 | 0 | 0 | 1 | 137 | 0 |

## Components

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `ArrowComponent` | Blueprint | ActorComponent | 33 | 16 | 0 | 1 | 1 | 620 | 0 |
| `AttackMotionWarpingComponent` | Blueprint | ActorComponent | 8 | 2 | 0 | 0 | 1 | 172 | 0 |
| `BuffComponent` | Blueprint | ActorComponent | 23 | 9 | 0 | 1 | 1 | 470 | 0 |
| `CombatStatusComponent` | Blueprint | ActorComponent | 77 | 54 | 0 | 6 | 1 | 2,597 | 0 |
| `CraftingComponent` | Blueprint | ActorComponent | 12 | 5 | 0 | 0 | 1 | 324 | 0 |
| `DefensiveComponent` | Blueprint | ActorComponent | 21 | 16 | 0 | 0 | 1 | 431 | 1 |
| `DialogueComponent` | Blueprint | ActorComponent | 15 | 15 | 0 | 0 | 1 | 336 | 0 |
| `EquipmentComponent` | Blueprint | ActorComponent | 106 | 52 | 0 | 0 | 1 | 2,474 | 0 |
| `GaspComponent` | Blueprint | ActorComponent | 25 | 23 | 0 | 0 | 1 | 339 | 0 |
| `HitStopComponent` | Blueprint | ActorComponent | 12 | 6 | 0 | 0 | 1 | 166 | 1 |
| `InputBufferComponent` | Blueprint | ActorComponent | 16 | 9 | 0 | 0 | 1 | 316 | 0 |
| `InventoryComponent` | Blueprint | ActorComponent | 74 | 55 | 0 | 3 | 1 | 2,457 | 0 |
| `LootComponent` | Blueprint | ActorComponent | 11 | 13 | 0 | 0 | 1 | 377 | 0 |
| `MagicComponent` | Blueprint | ActorComponent | 45 | 29 | 0 | 1 | 1 | 857 | 0 |
| `MapTrackingComponent` | Blueprint | ActorComponent | 12 | 7 | 0 | 0 | 1 | 282 | 0 |
| `MeleeComponent` | Blueprint | ActorComponent | 51 | 33 | 0 | 1 | 1 | 1,202 | 1 |
| `MovementRotationStatus` | Blueprint | ActorComponent | 5 | 4 | 0 | 0 | 1 | 66 | 1 |
| `MovementStatusComponent` | Blueprint | ActorComponent | 31 | 14 | 0 | 0 | 1 | 662 | 0 |
| `NPC-RPC-Assist` | Blueprint | ActorComponent | 1 | 0 | 0 | 0 | 1 | 38 | 0 |
| `OutlineComponent` | Blueprint | ActorComponent | 14 | 15 | 0 | 1 | 2 | 518 | 0 |
| `PhysicalReactComponent` | Blueprint | ActorComponent | 13 | 0 | 0 | 0 | 1 | 147 | 0 |
| `PlayerWorldTriggers` | Blueprint | ActorComponent | 1 | 0 | 0 | 0 | 1 | 25 | 0 |
| `QuestGiverComponent` | Blueprint | ActorComponent | 22 | 16 | 0 | 0 | 1 | 562 | 0 |
| `QuestReceiverComponent` | Blueprint | ActorComponent | 33 | 17 | 0 | 0 | 1 | 787 | 0 |
| `RangedComponent` | Blueprint | ActorComponent | 69 | 48 | 0 | 1 | 1 | 1,590 | 0 |
| `ResourceComponent` | Blueprint | ActorComponent | 12 | 12 | 0 | 0 | 1 | 357 | 0 |
| `SkillTrainerComponent` | Blueprint | ActorComponent | 12 | 0 | 0 | 0 | 1 | 27 | 0 |
| `SwimmingComponent` | Blueprint | ActorComponent | 34 | 20 | 0 | 0 | 1 | 871 | 1 |
| `TakingDamage` | Blueprint | ActorComponent | 80 | 46 | 0 | 3 | 1 | 1,989 | 1 |
| `TargetingComponent` | Blueprint | ActorComponent | 17 | 9 | 0 | 0 | 1 | 423 | 1 |
| `TrajectoryDataReplication` | Blueprint | ActorComponent | 2 | 2 | 0 | 0 | 1 | 8 | 0 |
| `TraversalMovementComponent` | Blueprint | ActorComponent | 13 | 10 | 0 | 0 | 1 | 437 | 0 |
| `WeaponCollision` | Blueprint | ActorComponent | 23 | 4 | 0 | 0 | 1 | 168 | 1 |

## Crafting

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_BlacksmithHammer` | Blueprint | BP_CraftingToolParent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_BlacksmithMetal` | Blueprint | BP_CraftingToolParent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_CraftingBenchParent` | Blueprint | Actor | 6 | 1 | 7 | 0 | 1 | 34 | 3 |
| `BP_CraftingBenchPlayer` | Blueprint | BP_CraftingBenchParent_C | 0 | 1 | 0 | 0 | 1 | 4 | 0 |
| `BP_CraftingCooking` | Blueprint | BP_CraftingBenchParent_C | 0 | 2 | 2 | 0 | 1 | 11 | 0 |
| `BP_CraftingSmithing` | Blueprint | BP_CraftingBenchParent_C | 0 | 2 | 2 | 0 | 1 | 12 | 0 |
| `BP_CraftingToolParent` | Blueprint | Actor | 2 | 2 | 1 | 0 | 1 | 9 | 1 |

## GodOfRuin

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_LevelEndTrigger` | Blueprint | Actor | 2 | 1 | 2 | 0 | 1 | 8 | 0 |

## Interfaces

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `I_ABP-InteractionTransform` | Blueprint | Interface | 0 | 2 | 0 | 0 | 0 | 4 | 0 |
| `I_ABP-Revive` | Blueprint | Interface | 0 | 5 | 0 | 0 | 0 | 5 | 0 |
| `I_AI` | Blueprint | Interface | 0 | 33 | 0 | 0 | 0 | 43 | 0 |
| `I_AI-ABP` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 1 | 0 |
| `I_AIController` | Blueprint | Interface | 0 | 27 | 0 | 0 | 0 | 34 | 0 |
| `I_Character` | Blueprint | Interface | 0 | 43 | 0 | 0 | 0 | 71 | 0 |
| `I_CharacterAnimBP` | Blueprint | Interface | 0 | 30 | 0 | 0 | 0 | 31 | 0 |
| `I_DefensiveComponent` | Blueprint | Interface | 0 | 4 | 0 | 0 | 0 | 5 | 0 |
| `I_GameMode` | Blueprint | Interface | 0 | 14 | 0 | 0 | 0 | 19 | 0 |
| `I_HitStop` | Blueprint | Interface | 0 | 2 | 0 | 0 | 0 | 2 | 0 |
| `I_Interact` | Blueprint | Interface | 0 | 2 | 0 | 0 | 0 | 3 | 0 |
| `I_ItemInterface` | Blueprint | Interface | 0 | 7 | 0 | 0 | 0 | 9 | 0 |
| `I_LootInterface` | Blueprint | Interface | 0 | 10 | 0 | 0 | 0 | 13 | 0 |
| `I_MapTracker` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 2 | 0 |
| `I_MeleeComponent` | Blueprint | Interface | 0 | 8 | 0 | 0 | 0 | 16 | 0 |
| `I_MontageCancel` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 1 | 0 |
| `I_MovementRotationManager` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 1 | 0 |
| `I_OutlineInterface` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 2 | 0 |
| `I_PlayerController` | Blueprint | Interface | 0 | 38 | 0 | 0 | 0 | 57 | 0 |
| `I_RangedWeaponInterface` | Blueprint | Interface | 0 | 10 | 0 | 0 | 0 | 16 | 0 |
| `I_SwimmingInterface` | Blueprint | Interface | 0 | 3 | 0 | 0 | 0 | 4 | 0 |
| `I_TakingDamage` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 1 | 0 |
| `I_TargetingComponent` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 2 | 0 |
| `I_WeaponInterface` | Blueprint | Interface | 0 | 19 | 0 | 0 | 0 | 20 | 0 |
| `I_WidgetInterface` | Blueprint | Interface | 0 | 4 | 0 | 0 | 0 | 4 | 0 |

## Items

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_PickupItem` | Blueprint | Actor | 29 | 20 | 5 | 0 | 1 | 801 | 3 |

## Items/Arrows

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ArrowParent` | Blueprint | Actor | 2 | 1 | 10 | 0 | 1 | 23 | 1 |
| `BP_ExplosiveArrow` | Blueprint | BP_ArrowParent_C | 2 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_FireArrow` | Blueprint | BP_ArrowParent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_FrostArrow` | Blueprint | BP_ArrowParent_C | 2 | 1 | 0 | 0 | 1 | 2 | 0 |
| `BP_IceArrow` | Blueprint | BP_ArrowParent_C | 2 | 1 | 0 | 0 | 1 | 2 | 0 |
| `BP_LightningArrow` | Blueprint | BP_ArrowParent_C | 13 | 1 | 0 | 0 | 1 | 2 | 0 |
| `BP_MadnessArrow` | Blueprint | BP_ArrowParent_C | 2 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_PoisonArrow` | Blueprint | BP_ArrowParent_C | 2 | 1 | 0 | 0 | 1 | 2 | 0 |
| `BP_RainOfArrows` | Blueprint | BP_ArrowParent_C | 0 | 1 | 0 | 0 | 1 | 2 | 0 |
| `BP_SharpArrow` | Blueprint | BP_ArrowParent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_UltimateArrow` | Blueprint | BP_ArrowParent_C | 0 | 1 | 1 | 0 | 1 | 2 | 0 |

## Items/Containers

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ContainerChest` | Blueprint | BP_ContainerParent_C | 1 | 2 | 0 | 0 | 1 | 23 | 0 |
| `BP_ContainerParent` | Blueprint | Actor | 14 | 3 | 8 | 0 | 1 | 133 | 4 |
| `BP_ContainerSetMesh` | Blueprint | BP_ContainerParent_C | 2 | 2 | 0 | 0 | 1 | 16 | 0 |

## Items/LootActors

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ChestBase` | Blueprint | BP_LootActorParent_C | 4 | 1 | 0 | 0 | 1 | 18 | 0 |
| `BP_LootActorAnim` | Blueprint | BP_LootActorParent_C | 1 | 2 | 0 | 0 | 1 | 26 | 0 |
| `BP_LootActorParent` | Blueprint | Actor | 4 | 5 | 7 | 0 | 1 | 134 | 3 |
| `BP_LootActorSetMesh` | Blueprint | BP_LootActorParent_C | 2 | 2 | 0 | 0 | 1 | 13 | 0 |

## Items/PickUpItems

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_AnimalAttackAttachment` | Blueprint | Actor | 10 | 3 | 3 | 0 | 1 | 101 | 1 |
| `BP_ArrowQuiverAI` | Blueprint | BP_ArrowQuiverParent_C | 0 | 1 | 7 | 0 | 1 | 18 | 0 |
| `BP_ArrowQuiverParent` | Blueprint | BP_WeaponParent_C | 9 | 11 | 21 | 0 | 1 | 174 | 0 |
| `BP_ConsumableParent` | Blueprint | BP_PickupItem_C | 5 | 5 | 0 | 0 | 1 | 160 | 0 |
| `BP_Fists` | Blueprint | Actor | 8 | 3 | 4 | 0 | 1 | 86 | 1 |
| `BP_OnUseItem` | Blueprint | BP_PickupItem_C | 1 | 3 | 0 | 0 | 1 | 27 | 0 |
| `BP_WeaponParent` | Blueprint | BP_PickupItem_C | 7 | 8 | 2 | 0 | 1 | 276 | 2 |

## Miscellaneous

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ChangeLevel` | Blueprint | Actor | 3 | 1 | 3 | 0 | 1 | 31 | 0 |
| `BP_CharacterInventory` | Blueprint | Actor | 7 | 8 | 19 | 0 | 1 | 404 | 0 |
| `BP_CheckPoint` | Blueprint | Actor | 2 | 1 | 4 | 0 | 1 | 33 | 0 |
| `BP_CooldownManager` | Blueprint | Object | 4 | 1 | 0 | 0 | 1 | 16 | 0 |
| `BP_LocationTrigger` | Blueprint | Actor | 6 | 2 | 4 | 0 | 1 | 56 | 1 |
| `BP_MapChanger` | Blueprint | Actor | 1 | 1 | 3 | 0 | 1 | 36 | 0 |
| `BP_QuestTrigger` | Blueprint | Actor | 3 | 1 | 5 | 0 | 1 | 33 | 1 |
| `TraversableComponent` | Blueprint | ActorComponent | 3 | 2 | 0 | 0 | 1 | 96 | 0 |

## Miscellaneous/ActivateActors

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ActivateActorParent` | Blueprint | Actor | 6 | 4 | 1 | 0 | 1 | 107 | 1 |
| `BP_ActivateByTouchParent` | Blueprint | BP_ActivateActorParent_C | 2 | 1 | 2 | 0 | 1 | 31 | 1 |

## Miscellaneous/ActivateActors/ActivatingActors(Switch)

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ActivatorEmpty` | Blueprint | BP_SwitchParent_C | 0 | 2 | 0 | 0 | 1 | 10 | 0 |
| `BP_ButtonSwitch` | Blueprint | Actor | 7 | 1 | 2 | 0 | 1 | 25 | 0 |
| `BP_LevelSwitch` | Blueprint | BP_SwitchParent_C | 0 | 3 | 2 | 0 | 1 | 18 | 0 |
| `BP_SwitchParent` | Blueprint | BP_ActivateByTouchParent_C | 3 | 3 | 0 | 0 | 1 | 33 | 0 |

## Miscellaneous/ActivateActors/ActorsToActivate/Message

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_MessageEmpty` | Blueprint | BP_ActivateActorParent_C | 1 | 2 | 0 | 0 | 1 | 15 | 1 |
| `BP_OpenDoorMessage` | Blueprint | BP_ActivateActorParent_C | 2 | 2 | 2 | 0 | 1 | 28 | 0 |
| `BP_SpinActorMessage` | Blueprint | BP_ActivateActorParent_C | 2 | 2 | 10 | 0 | 1 | 22 | 1 |
| `BP_VFXMessage` | Blueprint | BP_ActivateActorParent_C | 2 | 1 | 0 | 0 | 1 | 12 | 0 |

## Miscellaneous/ActivateActors/ActorsToActivate/Overlap

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_SlidingDoorEpic` | Blueprint | Actor | 3 | 2 | 4 | 0 | 1 | 62 | 0 |

## Miscellaneous/ActivateActors/ActorsToActivate/Touch

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_OpenDoorTouch` | Blueprint | BP_ActivateByTouchParent_C | 1 | 3 | 2 | 0 | 1 | 32 | 0 |
| `BP_SpinActorByTouch` | Blueprint | BP_ActivateByTouchParent_C | 2 | 3 | 10 | 0 | 1 | 26 | 0 |
| `BP_TouchEmpty` | Blueprint | BP_ActivateByTouchParent_C | 1 | 3 | 0 | 0 | 1 | 19 | 0 |

## Miscellaneous/DemoLevels

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_AIDummySpawner` | Blueprint | Actor | 5 | 1 | 5 | 0 | 1 | 34 | 2 |
| `BP_AIDummySpawnerSmall` | Blueprint | Actor | 6 | 1 | 5 | 0 | 1 | 39 | 2 |
| `BP_AI_DummySpawner-Assass` | Blueprint | Actor | 5 | 1 | 6 | 0 | 1 | 28 | 2 |
| `BP_AnimPreviewDummy` | Blueprint | Actor | 4 | 1 | 10 | 0 | 1 | 134 | 0 |
| `BP_ArenaGate` | Blueprint | BP_ActivateActorParent_C | 1 | 1 | 3 | 0 | 1 | 41 | 1 |
| `BP_ArenaSpawner` | Blueprint | Actor | 20 | 1 | 2 | 0 | 1 | 93 | 2 |
| `BP_FadeOutToMainMenu` | Blueprint | Actor | 3 | 1 | 3 | 0 | 1 | 25 | 0 |
| `BP_FloorButton` | Blueprint | Actor | 11 | 1 | 4 | 0 | 1 | 51 | 0 |
| `BP_LevelUper` | Blueprint | Actor | 0 | 1 | 1 | 0 | 1 | 5 | 1 |
| `BP_LoweringGate` | Blueprint | BP_ActivateActorParent_C | 3 | 2 | 1 | 0 | 1 | 12 | 1 |
| `BP_OpeningWall` | Blueprint | BP_ActivateActorParent_C | 3 | 2 | 2 | 0 | 1 | 27 | 1 |
| `BP_RangedPractise` | Blueprint | Actor | 7 | 1 | 1 | 0 | 1 | 74 | 1 |
| `BP_Target` | Blueprint | Actor | 7 | 1 | 12 | 0 | 1 | 82 | 0 |
| `BP_TargetingDummy` | Blueprint | Pawn | 0 | 1 | 6 | 0 | 1 | 6 | 1 |
| `BP_Teleporter` | Blueprint | Actor | 4 | 2 | 11 | 0 | 1 | 52 | 0 |
| `BP_TextSpawner` | Blueprint | Actor | 4 | 1 | 3 | 0 | 1 | 31 | 0 |
| `BP_TitleSpawner` | Blueprint | Actor | 2 | 1 | 3 | 0 | 1 | 17 | 0 |
| `BP_TorchLight` | Blueprint | Actor | 0 | 1 | 4 | 0 | 1 | 4 | 0 |
| `BP_UpDownButton` | Blueprint | Actor | 14 | 1 | 3 | 0 | 1 | 35 | 0 |
| `BP_WarRoomTeleporter` | Blueprint | Actor | 4 | 2 | 5 | 0 | 1 | 40 | 0 |
| `I_WidgetTrainingRoomInterface` | Blueprint | Interface | 0 | 1 | 0 | 0 | 0 | 1 | 0 |

## Miscellaneous/DragDropOperation

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_DragDropOperation` | Blueprint | DragDropOperation | 12 | 0 | 0 | 0 | 1 | 19 | 0 |

## Miscellaneous/MainMenu

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ArenaMapDummy` | Blueprint | SkeletalMeshActor | 0 | 1 | 5 | 0 | 1 | 1 | 0 |
| `BP_CameraPawn` | Blueprint | Pawn | 15 | 1 | 2 | 0 | 1 | 80 | 0 |
| `BP_CharacterCreatorPawn` | Blueprint | Actor | 3 | 3 | 15 | 0 | 1 | 120 | 0 |
| `BP_CharacterLevelSwitcher` | Blueprint | Actor | 8 | 1 | 2 | 0 | 1 | 71 | 0 |
| `BP_CombatMapDummy` | Blueprint | SkeletalMeshActor | 0 | 1 | 1 | 0 | 1 | 1 | 0 |
| `BP_MagicCharacter` | Blueprint | SkeletalMeshActor | 0 | 1 | 3 | 0 | 1 | 1 | 0 |
| `BP_MeleeCharacter` | Blueprint | Actor | 0 | 1 | 14 | 0 | 1 | 14 | 0 |
| `BP_QuestChar` | Blueprint | SkeletalMeshActor | 0 | 1 | 3 | 0 | 1 | 1 | 0 |
| `BP_RangedCharacter` | Blueprint | SkeletalMeshActor | 0 | 1 | 15 | 0 | 1 | 20 | 0 |
| `BP_SwimmingMapDummy` | Blueprint | SkeletalMeshActor | 0 | 1 | 1 | 0 | 1 | 1 | 0 |
| `BP_WarMapDummy` | Blueprint | SkeletalMeshActor | 0 | 1 | 9 | 0 | 1 | 1 | 0 |

## ResourceCollection

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_ResourceParent` | Blueprint | Actor | 8 | 1 | 2 | 0 | 1 | 89 | 0 |
| `BP_ResourcePickup` | Blueprint | BP_PickupItem_C | 1 | 1 | 0 | 0 | 1 | 23 | 0 |

## ResourceCollection/Mining

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_LargeSilverVein` | Blueprint | BP_ResourceParent_C | 0 | 1 | 0 | 0 | 1 | 5 | 0 |
| `BP_MediumSilverVein` | Blueprint | BP_ResourceParent_C | 0 | 1 | 0 | 0 | 1 | 5 | 0 |
| `BP_SmallSilverVein` | Blueprint | BP_ResourceParent_C | 0 | 1 | 0 | 0 | 1 | 5 | 0 |

## ResourceCollection/Woodchopping

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_LargeTree` | Blueprint | BP_ResourceParent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_MediumTree` | Blueprint | BP_ResourceParent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |
| `BP_SmallTree` | Blueprint | BP_ResourceParent_C | 0 | 1 | 0 | 0 | 1 | 6 | 0 |

## Swimming

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_CharacterBuoyancy` | Blueprint | Actor | 3 | 1 | 2 | 0 | 1 | 6 | 0 |
| `BP_LakeWater` | Blueprint | WaterBodyLake | 0 | 1 | 1 | 0 | 1 | 15 | 0 |
| `BP_OceanWater` | Blueprint | WaterBodyOcean | 0 | 2 | 1 | 0 | 1 | 37 | 0 |
| `BP_WaterVolume` | Blueprint | Actor | 0 | 1 | 4 | 0 | 1 | 15 | 0 |
| `GerstnerWaves_LakeCustom` | WaterWavesAsset | — | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

## WarLevel

| Asset | Class | Parent | V | F | C | M | EG | Nodes | I |
|---|---|---|---:|---:|---:|---:|---:|---:|---:|
| `BP_TeamManager` | Blueprint | Actor | 1 | 1 | 3 | 0 | 1 | 31 | 0 |
| `BP_UnitSpawner` | Blueprint | Actor | 22 | 1 | 2 | 0 | 1 | 156 | 0 |
| `BP_WarLevel` | Blueprint | GameModeBase | 0 | 1 | 1 | 0 | 1 | 10 | 0 |
| `BP_WarLevelCameraPawn` | Blueprint | DefaultPawn | 3 | 1 | 1 | 0 | 1 | 30 | 0 |

