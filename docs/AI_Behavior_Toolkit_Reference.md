# AI Behavior Toolkit — God of Ruin Reference
Creator: Drix Studios (same as Boss AI Toolkit)
Version: v1.9.7
Full docs: https://[check Fab page for URL]

## Overview
Component-based AI behavior system. Plug-and-play.
Used for: all grunt and elite enemies in God of Ruin.
Not used for: bosses (Boss AI Toolkit handles those).

## Setup Pattern for Paragon Enemies (Option B)
Do NOT use NPC_Base. Use Option B — add component to existing BP.

Steps per enemy:
1. Open Paragon character BP (e.g. Khaimera)
2. Add Component → search "Behavior Component" → add it
3. Class Settings → AI Controller Class → set to AIC
4. Auto Possess AI → Placed in World or Spawned

## Player Registration (DO ONCE on BP_GodOfRuin_Player)
1. Add Component → AIStorageComponent
2. Actor Tags → add tag: "Player"
These two steps make ALL enemies (grunt + boss) recognize Shinbi.

## Behavior Chain — Standard Grunt (GoW pattern)
Configure on BehaviorComponent Details panel:

Idle Settings:
  IdleType: Stationary (or Patrol if patrol route assigned)
  OnSightTriggers → add element:
    ActorTag: Player
    BehaviorTo: Follow

Follow Settings:
  TargetTags: Player
  FollowDistance: 200
  WithinDistanceTriggers → add element:
    ActorTag: Player
    Distance: 200
    BehaviorTo: AttackMelee

AttackMelee Settings:
  Animations: [assign Paragon attack montage]
  AttackMeleeDistance: 150
  AttackMeleeTransition: Follow
  LoseSightTriggers → add element:
    ActorTag: Player
    BehaviorTo: Idle
  DisengageByDistance: true
  DisengageByLostSight: true

## Behavior Chain — Ranged Enemy
Same as grunt but use AttackRanged instead of AttackMelee.
AttackRangedDistance: 800
AttackSpeed: 2.0 (seconds between shots)
NumAttacksBeforeReload: 3

## Hit Detection — Animations
Toolkit uses Anim Montages + Anim Notifies.
On each Paragon attack montage:
  Add notify "OnDamage" at impact frame → enemy deals damage
  Add notify "SpawnProjectile" for ranged attacks

For custom mesh — add DefaultSlot node to AnimBP anim graph.
Required so montages can play on top of locomotion.

## Key Components
BehaviorComponent — main AI brain, holds all behavior configs
AIStorageComponent — registers actor to global AI storage
BehaviorTriggerComponent — add to player to send messages to AI

## Key Blueprints
NPC_Base — base AI character (health + damage built in) — NOT used for Paragon
AIC — AI Controller class — assign to all enemy BPs
BT_GenericAI — generic behavior tree — duplicate for custom behaviors
BP_PatrolRoute — spline patrol path actor
BP_Spawner — wave spawner with respawn + activation options
BP_GroupCoordinator — limit how many enemies attack at once (Zone D waves)

## Key Enums
Enum_BehaviorTypes — all behavior states
Enum_MessageTag — custom message tags for player→AI communication

## Damage System
AI takes damage via standard UE Apply Point Damage / Apply Radial Damage.
Health and MaxHealth live on NPC_Base variables.
If using Option B (Paragon BP) — wire damage manually:
  Event Point Damage → reduce health variable → call CheckHealthTriggers

## Health Triggers (for enemy phasing)
Add HealthTriggers in BehaviorComponent:
  HealthPercentage: 0.3 (30% HP)
  Condition: LessThan
  BehaviorTo: Flee (or any state)
  
Note: Must call CheckHealthTriggers manually if not using NPC_Base.

## Group Coordination (Zone D waves)
Add BP_GroupCoordinator to level.
Assign GroupMembers = array of enemy BPs in zone.
GroupBehaviors → set AttackMelee limit = 2 (only 2 attack at once, others circle)
This creates GoW-style combat pacing — enemies surround but don't all attack.

## God of Ruin Enemy Behavior Chains

### FCS Soldiers (Grunt)
Idle(Patrol) → OnSight → Follow → WithinDistance 200 → AttackMelee → LoseSight → Idle

### Kwang (Heavy)
Idle(Stationary) → OnSight → Follow → WithinDistance 250 → AttackMelee → LoseSight → Idle
AttackMeleeDistance: 250 (larger swing radius)

### Crunch (Berserker)
Idle(Stationary) → OnSight → AttackMelee (charges directly, no follow state)
AttackMeleeDistance: 400 (charge distance)

### Morigesh (Ranged Caster)
Idle → OnSight → Seek(get distance) → AttackRanged → LoseSight → Idle
AttackRangedDistance: 800, stays at range, repositions

### Aurora (Ice Ranged)
Same as Morigesh pattern, AttackRangedDistance: 700

### Kallari (Stealth Assassin)
Idle(Stationary/Hidden) → WithinDistance 300 → AttackMelee (ambush)
Use Investigate state for searching after losing sight

### Steel (Shield Tank)
Idle(Patrol) → OnSight → Follow → Defend(block) → OnMessage "Threat" → AttackMelee
Requires parry to break defend state

### Narbash (Heavy AoE)
Idle → OnSight → Follow → WithinDistance 300 → AttackMelee
AttackMeleeDistance: 300 (hammer AoE range)

### Feng Mao (Elite Assassin)
Idle → OnSight → Follow → WithinDistance 200 → AttackMelee
Fast attack speed, high damage, disengages and repositions

### Serath (Aerial Elite)
Idle → OnSight → Follow → WithinDistance 350 → AttackMelee
At 30% HP → HealthTrigger → Flee briefly → return to AttackMelee

### Shinbi Clone (Mirror Warrior)
Idle → OnSight → Follow → AttackMelee
Same attack patterns as player (use Shinbi attack montages)

## FAQ Gotchas
- AI not recognizing player: Add AIStorageComponent to player BP
- AI not avoiding each other: NavMesh Runtime Generation must be Dynamic
- More than 50 AI: Increase Max Agents in Project Settings → Crowd Manager
- Custom animations not playing: Add DefaultSlot to AnimBP anim graph
- EQS errors on open: Edit → Editor Preferences → Experimental → check EQS
