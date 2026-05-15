# Boss AI Toolkit — God of Ruin Reference
Creator: Drix Studios (same as AI Behavior Toolkit)
Version: v1.3.0
Full docs: check Fab page

## Overview
Boss fight creation system. Phase-based ability system.
Inspired by Dark Souls, Monster Hunter, Final Fantasy XIV.
Used for: Khaimera (mini), Greystone, Rampage, Terra+Countess, The Ruined One.

## Pre-Built Bosses in Project (use these directly)
- BP_Boss_Rampage → L3 boss ✅
- BP_Boss_Terra → L6 Phase 1 ✅
- BP_Boss_Countess → L6 Phase 2 ✅
- BP_Boss_Sevarog → base for The Ruined One L8 ✅

## Player Integration (DO ONCE on BP_GodOfRuin_Player)
These steps make bosses attack + react to Shinbi.

Step 1 — Actor Tag (if not already done for AIBT):
  Actor Tags → add "Player"

Step 2 — Interfaces:
  Class Settings → Add interfaces:
    BI_CrowdControl
    BI_Ability

Step 3 — Implement BI_CrowdControl functions:
  Get Priority Score → use distance to boss (copy from BP_Player example)
  Get Crowd Control State → return current CC state variable
  Receive Crowd Control → apply CC effect to player
  Receive Debuff → apply debuff effect
  Begin Grab / End Grab → handle grab state
  Is Grabbable → return true (bosses can grab Shinbi)

Step 4 — Add Component:
  BPC_StatusEffects → add to player
  Bind events: OnEffectChanged, OnCrowdControlApplied, OnCrowdControlRemoved

Step 5 — Receive Damage:
  Event Point Damage → drag DamageType pin → call Process Damage
  → call TakeDamage (reduce AAMS health)

Step 6 — Deal Damage to Boss:
  On Shinbi attack hit → Apply Point Damage to boss actor
  Implement Get Status Effect Info (can return none/default)
  Implement Get Stagger Info → return stagger value for parry

Step 7 — Boss Dodge Trigger:
  On Shinbi attack input → call Send Attack Message on boss

NOTE: Copy code from BP_Player in toolkit for all BI_CrowdControl implementations.

## Creating A New Boss (Khaimera + Greystone)
1. Create child BP of BP_Boss_Base
   Name: BP_Boss_Khaimera, BP_Boss_Greystone etc.
2. Assign Paragon skeletal mesh (UE4 Mannequin skeleton matches ✅ zero retarget)
3. Select Boss Behavior component → configure settings
4. Open DT_SamplePhases → add new row for each phase
5. In Boss BP → Boss Behavior → Phases array → set PhaseID = row name

## Phase Data Table (DT_SamplePhases) — Row Format
Each row = one phase. Fill these key fields:
  PhaseID: [unique name — referenced by boss BP]
  PhaseHealthTrigger: 0.5 (fires at 50% HP)
  PhaseTrigger: Health (or OnSpawn for Phase 1)
  Abilities: [array of F_BossAbility structs]
  HitReactions: [4 directional + headshot montages]
  DodgeAnimations: [optional dodge montages]
  MovementMode: Walking (or Flying for aerial bosses)

## Ability Setup — Per Attack
In DT_SamplePhases Abilities array, add element:
  AbilityName: "Ground Slam"
  AbilityClass: BP_Ability_Area
  Damage:
    DamageMultiplier: 1.0
    DamageType: BP_DamageType_Default
  Modifier: X=300 (radius for Area ability)
  Visuals:
    AnimMontage: [assign boss attack montage]
  Duration: 1.5
  Cooldown: 3.0

## Ability Classes — God of Ruin Boss Mapping
Khaimera:
  Leap Attack → BP_Ability_Leap (Modifier X = radius 300)
  Heavy Swipe → BP_Ability_Hitbox (ComponentTag = weapon socket)
  Combo Rush → BP_Ability_Hitbox with Notify_Combo

Greystone Phase 1:
  Shield Charge → BP_Ability_Charge (Max Distance 600)
  Shield Bash → BP_Ability_Direct
  Heavy Overhead → BP_Ability_Hitbox

Greystone Phase 2 (50% HP trigger):
  All Phase 1 + Divine Slam → BP_Ability_Area (larger radius)
  Empowered Rush → BP_Ability_Charge (longer distance)

Rampage Phase 1 (use BP_Boss_Rampage):
  Ground Pound → BP_Ability_Area
  Charge → BP_Ability_Charge
  Roar → BP_Ability_Direct (zero damage, stagger only)

Rampage Phase 2 (60% HP trigger):
  All Phase 1 + Lightning Trail → add particle to Charge ability
  Rapid Charge → BP_Ability_Charge with Combo

Terra Phase 1 (use BP_Boss_Terra):
  Ground Slam → BP_Ability_Area
  Rock Throw → BP_Ability_Projectile
  Stomp → BP_Ability_Direct

Countess Phase 2 (use BP_Boss_Countess, triggers at Terra 0 HP):
  Teleport Strike → BP_Ability_TweenTo + Hitbox combo
  Homing Orbs → BP_Ability_Projectile_Homing (3 projectiles)
  Shadow Clone → BP_Ability_Summon (2 clones at 50% HP each)
  Floor Runes → BP_Ability_RangedArea (at 40% HP trigger)

The Ruined One Phase 1 (child of BP_Boss_Sevarog):
  Soul Projectile → BP_Ability_Projectile_Homing
  Heavy Combo → BP_Ability_Hitbox with Notify_Combo
  Ground Slam → BP_Ability_Area + Wave_Ring

The Ruined One Phase 2 (50% HP):
  All Phase 1 + scale up (set via On Phase Change event)
  Throne Beam → BP_Ability_Beam
  Corruption Floor → BP_Ability_RangedArea (persistent zones)
  Area Denial → BP_Ability_Donut (expanding ring)

## Key Events on BPC_Boss_Behavior — Wire These
On Phase Change → play slow-mo + phase transition FX + scale change
On Phase Change End → resume combat
On Boss Death → barriers drop → level end door opens → load next level
On Use Ability → show telegraph decal
On Dodge → play dodge reaction

## Stagger System (Parry → Cinematic Vault)
Posture = separate from health. Depletes on parry hits.
In player Get Stagger Info: return stagger value per parry hit.
When posture depletes → boss enters Stun state.
Wire Stun state → play cinematic vault montage on Shinbi.

## Hyper Armor
Add Notify_HyperArmor to boss attack montage.
Length of notify = duration of armor.
During armor: parry still registers but no stagger.
Use on Rampage charge, Ruined One heavy combo.

## Boss Behavior Component Key Settings
Show Telegraphs: true (show ground indicators for AoE)
Can Block: false (bosses don't block in GoW style)
Can Dodge: true (reactive to player attack message)
Friendly Fire: false (bosses don't damage each other)
Desired Range: 0 (melee bosses) or 500+ (ranged phases)
Target Tags Enemy: "Player"
