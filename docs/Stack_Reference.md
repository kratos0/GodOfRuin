# God of Ruin — New Stack Reference
Project: God_Of_Ruin (clean project)
Decision date: 2026-05-11
Reason: FCS dropped due to skeleton lock-in and RPG overhead

## Confirmed Stack
- AAMS → traversal, climbing, movement, health, stamina, interact, save, checkpoint
- Enemy AI Toolkit (same creator as Boss AI) → grunt/elite enemy behavior
- Boss AI Toolkit → all boss phases + abilities
- Grapple Component → grab system
- Paragon Assets → all enemy + hero meshes (native skeletons)
- Custom BPs → hit detection, spirit meter, level end trigger

## Why FCS Was Dropped
- Manny skeleton lock-in caused major deformation on all Paragon characters
- Character creator system fought custom mesh setup constantly
- RPG overhead (inventory, crafting, quests, magic) added complexity with no value
- Retarget sessions consumed 2+ full sessions with no clean resolution
- AAMS handles everything FCS provided for our use case, without the overhead

## Skeleton Strategy
- AAMS uses SK_UEFN_Mannequin (UE5 Manny)
- Boss AI Toolkit uses UE4_Mannequin skeleton
- Paragon characters use UE4 Mannequin skeleton
- Boss AI + Paragon enemies = zero retarget needed (skeletons match)
- Hero (Shinbi) uses AAMS retarget workflow via ABP_GenericRetarget

---

## AAMS — Full Reference
Path: /Game/AAMS/

### Core Components
- AC_Stats_AAMS → health + stamina pools
- AC_Interact_AAMS → interact system
- AC_Climb_AAMS → climbing + ledge traversal
- AC_Action_AAMS → dodge, roll, actions
- AC_Adventure_AAMS → traversal (zipline, rope, swing pole, vault, wall run)
- AC_State_AAMS → character state management
- AC_Footsteps_AAMS → footstep system
- AC_Compass_AAMS → compass/map (not used in GodOfRuin)

### Key Blueprints
- BP_ExampleCharacter_MotionMatching → AAMS motion matching player character
- BP_RetargetedExample_MM → retargeted character example (Shinbi workflow)
- BP_RetargetedExample_SM → state machine retarget example
- BP_PlayerController_AAMS → player controller
- BP_ExampleGameMode → game mode
- BP_SaveGame → save system
- BP_BaseInteractable → interact base class
- BP_Pickup_Health → health pickup
- BP_Pickup_Stamina → stamina pickup
- BP_Pickup_Stats → stat pickup
- BP_TriggerButton → puzzle trigger (button)
- BP_TriggerFloor → puzzle trigger (floor plate)
- BP_TriggerLever → puzzle trigger (lever)
- BP_Door → door actor
- BP_SlidingDoor → sliding door
- BP_Teleporter → level teleporter
- BP_MoveablePlatform → moving platform

### Animation Blueprints
- ABP_AAMS_MotionMatching → main motion matching ABP
- ABP_AAMS_Locomotion → state machine locomotion ABP
- ABP_GenericRetarget → retarget ABP for custom characters
- ABP_AAMS_ActionLayer_MM → action layer (motion matching)
- ABP_AAMS_ClimbLayer → climb layer
- ABP_AAMS_AdventureLayer → adventure traversal layer

### Retarget Workflow (Shinbi)
Pattern from BP_RetargetedExample_MM:
  Main Mesh = SKM_UEFN_Mannequin (drives AAMS logic)
  Child mesh = custom character mesh (Shinbi)
  Anim Class = ABP_GenericRetarget

ABP_GenericRetarget reads Component Tag from child mesh:
  Tag name must match IK Retargeter asset name exactly
  Looks up retargeter from IKRetargeter_Map
  Automatically applies correct retarget per character

To add Shinbi:
  1. Child BP of BP_RetargetedExample_MM → BP_GodOfRuin_Player
  2. Find "Manny" sub-mesh → change to Shinbi SKM
  3. Add Component Tag to Shinbi mesh = retargeter asset name
  4. Add retargeter to IKRetargeter_Map in ABP_GenericRetarget
  5. Retarget direction: Manny (source) → Shinbi (target)

### Traversal Actors
- BP_Ladder → ladder climbing
- BP_VineWall → vine wall climbing
- BP_WallRun → wall run surface
- BP_SwingPole_Parent → swing pole
- BP_Zipline_Base → zipline
- BP_GrappleHookLocation → grapple hook point
- BP_GrindRail → rail grind
- BP_NarrowPassage → squeeze through gap
- BP_Chain_500/750/1000 → rope/chain lengths

### Struct/Data Reference
- S_BaseStats → health/stamina base values
- S_ClimbSettings → climb configuration
- S_DodgeSettings → dodge configuration
- S_CameraSettings → camera configuration
- DT_BracedClimbSettings → braced climb data table
- DT_VaultSettings → vault data table
- DT_TPPCamera → third person camera settings

---

## Boss AI Toolkit — Full Reference
Path: /Game/BossAIToolkit/

### Core System
- BP_Boss_Base → parent class for all bosses
- BPC_Boss_Behavior → behavior component (attach to any boss)
- BT_BossAI → behavior tree
- BB_BossAI → blackboard
- BP_Boss_Controller → AI controller
- DT_SamplePhases → phase data table example
- AnimBP_Boss_Template → template ABP for new bosses

### Pre-built Bosses (God of Ruin)
- BP_Boss_Rampage → L3 boss ✅
- BP_Boss_Terra → L6 boss phase 1 ✅
- BP_Boss_Countess → L6 boss phase 2 ✅
- BP_Boss_Sevarog → L8 The Ruined One ✅
- BP_Boss_Twinblast → NOT USED
- BP_Boss_Mimic → NOT USED
- BP_Boss_Howizter → NOT USED

### Ability System
- BP_Ability_Area → AoE ground zone
- BP_Ability_Beam → directional beam
- BP_Ability_Charge → charge attack
- BP_Ability_Charge_Circular → circular charge
- BP_Ability_Cone → cone AoE
- BP_Ability_Direct → direct melee hit
- BP_Ability_Dive → dive bomb
- BP_Ability_Donut → ring AoE
- BP_Ability_Flamethrower → sustained beam
- BP_Ability_Grab → grab attack
- BP_Ability_Hitbox → melee hitbox
- BP_Ability_Leap → leap attack
- BP_Ability_Projectile → standard projectile
- BP_Ability_Projectile_Homing → homing projectile
- BP_Ability_Wave_Ring → expanding ring
- BP_Ability_Summon → summon allies
- BP_Ability_DodgeAway → reactive dodge
- BP_Ability_Heal_Self → self heal

### Notifies
- Notify_Damage → hit detection (use on all attack montages)
- Notify_Combo → combo chain window
- Notify_Execute → finisher/execute trigger
- Notify_HyperArmor → armor frames during attack
- Notify_Dodge → reactive dodge trigger

### HUD Widgets
- WB_PlayerHealth → player health bar
- WB_BossHealth → boss health bar
- WB_PlayerUI → full player UI
- WB_LockOn → lock on indicator

### Phase System
- F_BossPhase → phase data struct
- F_BossPhaseDesign → phase design config
- F_BossPhaseIntro → phase intro sequence
- E_PhaseTriggers → phase trigger enum (HP%, time, etc.)
- BPC_StatusEffects → status effect component

### Damage Types
- BP_DamageType_Default → standard damage
- BP_DamageType_Bleed → bleed DOT
- BP_DamageType_Fire → fire DOT
- BP_DamageType_Poison → poison DOT
- BP_DamageType_Unblockable → bypass armor

### Skeleton
Boss AI uses UE4_Mannequin skeleton
Paragon characters use UE4 Mannequin skeleton
= Boss AI + Paragon = ZERO retarget needed

---

## Grapple Component — Reference
Path: /Game/GrappleComponent/

### Core
- GrappleComponent → main component (add to player BP)
- GrappleComponent_AnimBP → anim BP (Manny skeleton)
- GrappleComponent_AnimBP_AlternateSkeleton → skeleton agnostic version ✅

### Key Assets
- GrappleSequenceTutorial → tutorial BP for setup
- AnimNotify_GrappleAttempt → notify for grab attempt

### Integration Notes
- User Command pattern — Spirit +10 hooks on button press
- GrappleComponent_AnimBP_AlternateSkeleton works with any skeleton
- Add GrappleComponent actor component to BP_GodOfRuin_Player
- Wire B button → Send User Command → GrappleComponent
