# AAMS — God of Ruin Reference
Creator: Travis (TyGames)
Full docs: https://travistygames.com/docs-category/aams/
Skeleton: SK_UEFN_Mannequin

## Overview
Advanced Action Movement System. Handles all non-combat player systems.
Used for: traversal, climbing, health, stamina, save, interact, motion matching.

## Key Components (add to BP_GodOfRuin_Player)
AC_Stats_AAMS → health + stamina pools
AC_Interact_AAMS → interact system (chests, doors, terminals)
AC_Climb_AAMS → climbing + ledge traversal
AC_Action_AAMS → dodge, roll, actions
AC_Adventure_AAMS → traversal (zipline, rope, swing pole, wall run, vault)
AC_State_AAMS → character state management
AC_Footsteps_AAMS → footstep audio + FX

## Retarget Workflow — Shinbi Setup
Pattern from BP_RetargetedExample_MM:
  Main Mesh (CharacterMesh0): SK_UEFN_Mannequin (drives AAMS logic)
  Child mesh "Manny": Shinbi SKM (visible character)
  Anim Class on child mesh: ABP_GenericRetarget

ABP_GenericRetarget reads Component Tag from child mesh:
  Tag name must exactly match IK Retargeter asset name
  Auto-selects correct retargeter from IKRetargeter_Map
  Retarget direction: Manny (source) → Shinbi (target) ← CORRECT DIRECTION

To add Shinbi:
  1. Child BP of BP_RetargetedExample_MM → BP_GodOfRuin_Player
  2. Sub-mesh "Manny" → swap to Shinbi SKM
  3. Add Component Tag to Shinbi mesh = retargeter asset name
  4. Add retargeter to IKRetargeter_Map in ABP_GenericRetarget
  5. Retarget direction: Manny → Shinbi (not Shinbi → Manny)

## Key Animation Blueprints
ABP_AAMS_MotionMatching → main motion matching (locomotion)
ABP_AAMS_Locomotion → state machine locomotion
ABP_GenericRetarget → retarget for custom character meshes ← USE THIS
ABP_AAMS_ActionLayer_MM → action layer (attacks, interactions)
ABP_AAMS_ClimbLayer → climbing animations
ABP_AAMS_AdventureLayer → traversal animations

## Key Blueprints — Asset Paths
BP_ExampleCharacter_MotionMatching: /Game/AAMS/AAMS/Blueprints/Character/MotionMatching/BP_ExampleCharacter_MotionMatching
BP_RetargetedExample_MM: /Game/AAMS/AAMS/Blueprints/Character/MotionMatching/RetargetedCharacters/BP_RetargetedExample_MM
ABP_GenericRetarget: /Game/AAMS/AAMS/Blueprints/Character/MotionMatching/RetargetedCharacters/ABP_GenericRetarget
BP_PlayerController_AAMS: /Game/AAMS/AAMS/Blueprints/Character/BP_PlayerController_AAMS
BP_ExampleGameMode: /Game/AAMS/AAMS/Blueprints/Character/BP_ExampleGameMode
BP_SaveGame: /Game/AAMS/AAMS/Blueprints/Actors/TutorialLevelActors/BP_SaveGame
BP_BaseInteractable: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/BP_BaseInteractable
BP_Pickup_Health: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/Pickups/BP_Pickup_Health
BP_Pickup_Stamina: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/Pickups/BP_Pickup_Stamina
BP_Pickup_Stats: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/Pickups/BP_Pickup_Stats
BP_TriggerButton: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/Triggers/BP_TriggerButton
BP_TriggerFloor: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/Triggers/BP_TriggerFloor
BP_TriggerLever: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/Triggers/BP_TriggerLever
BP_Door: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/BP_Door
BP_SlidingDoor: /Game/AAMS/AAMS/Blueprints/Actors/Interactables/BP_SlidingDoor

## Key Traversal Actors — Asset Paths
BP_Ladder: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/Ladder/BP_Ladder
BP_VineWall: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/VineWall/BP_VineWall
BP_WallRun: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/WallRun/BP_WallRun
BP_SwingPole_Parent: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/SwingPole/BP_SwingPole_Parent
BP_Zipline_Base: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/Ziplines/BP_Zipline_Base
BP_GrappleHookLocation: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/GrapplingHook/BP_GrappleHookLocation
BP_NarrowPassage: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/NarrowPassage/BP_NarrowPassage
BP_Rope_500: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/Ropes/BP_Rope_500
BP_Rope_750: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/Ropes/BP_Rope_750
BP_Rope_1000: /Game/AAMS/AAMS/Blueprints/Actors/AAMS_Actors/Ropes/BP_Rope_1000

## Level Design — Climbable Surfaces
Any surface Shinbi can climb must have AAMS climbing configured.
Check AAMS docs for exact surface tagging approach.
Full docs: https://travistygames.com/docs-category/aams/

## Save System
BP_SaveGame handles all save/load.
AAMS save persists: health, checkpoint position, unlocks.
Place BP_SaveGame in each level.
Checkpoints: place at Zone A→B transition, Zone D exit, pre-Zone F.

## Motion Matching Data
PSS_Default: /Game/AAMS/AAMS/Animations/Locomotion/MotionMatchingData/Schemas/PSS_Default
PSS_Idle: /Game/AAMS/AAMS/Animations/Locomotion/MotionMatchingData/Schemas/PSS_Idle
PSS_Jump: /Game/AAMS/AAMS/Animations/Locomotion/MotionMatchingData/Schemas/PSS_Jump
PSS_Stop: /Game/AAMS/AAMS/Animations/Locomotion/MotionMatchingData/Schemas/PSS_Stop
PSS_Traversal: /Game/AAMS/AAMS/Animations/Locomotion/MotionMatchingData/Schemas/PSS_Traversal
