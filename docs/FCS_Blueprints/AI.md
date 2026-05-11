# AI — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/AI/`
> Subfolders discovered: `AI-Spawners/`, `AI-Types/`, `BehaviourTrees/`
> Total assets in folder: 66 (documented: 8 | skipped: 58 dummies/tasks/services/decorators)

---

## BP_AISpawnMarker
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AI/AI-Spawners/BP_AISpawnMarker`
- **Purpose:** A minimal placement actor (1 variable, 1 component) that marks where an AI should spawn; used as a reference point by spawner Blueprints.
- **Key Variables:** 1 variable (name not exposed by API without deeper read)
- **Key Functions/Events:** EventGraph, UserConstructionScript
- **GodOfRuin Use:** **USE** — place in levels to define spawn positions fed into BP_Spawner-Set or BP_Spawner-Random
- **Notes:** Thin wrapper — logic lives in the spawner, not here.

---

## BP_Spawner-Set
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AI/AI-Spawners/BP_Spawner-Set`
- **Purpose:** Spawns a fixed, designer-defined set of AI at designated markers, then tracks whether all spawned enemies are dead before optionally despawning.
- **Key Variables:** 9 variables (names require deeper read — likely AI class refs + count + trigger bools)
- **Key Functions/Events:**
  - `SpawnAI` — spawns the configured set of enemies
  - `Are AI Dead Check` — polls whether all spawned enemies have died
  - `Should Despawn AI` — conditionally cleans up living AI
- **GodOfRuin Use:** **USE** — wire into wave/arena triggers for scripted enemy encounters with a known roster
- **Notes:** Prefer this over BP_Spawner-Random when enemy composition must be deterministic (boss rooms, story beats).

---

## BP_Spawner-Random
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AI/AI-Spawners/BP_Spawner-Random`
- **Purpose:** Spawns a randomised selection of AI from a configurable pool, tracking deaths and conditionally despawning survivors — same lifecycle as BP_Spawner-Set but with randomised picks.
- **Key Variables:** 13 variables (3 more than Set — likely an AI pool array + weight/count controls)
- **Key Functions/Events:**
  - `SpawnAI` — selects and spawns random AI from the pool
  - `Are AI Dead Check` — polls whether all spawned enemies have died
  - `Should Despawn AI` — conditionally cleans up living AI
- **GodOfRuin Use:** **USE** — use for open-world roaming groups or repeatable arena waves where variety is desirable
- **Notes:** Current wave system (BP_ArenaSpawner) uses 7 individual struct variables instead of this — see KnownIssues.md. Migrating to BP_Spawner-Random would fix that scalability issue.

---

## BP_PatrolPath
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AI/BP_PatrolPath`
- **Purpose:** Defines a patrol route as a sequence of spline points that AI with a `PatrolPath` variable assigned will walk between.
- **Key Variables:** 8 variables, 11 components (spline-based — names require deeper read)
- **Key Functions/Events:** EventGraph, UserConstructionScript (spline construction)
- **GodOfRuin Use:** **USE** — assign to `BP_AI_Parent.PatrolPath` on any NPC or enemy that should patrol; referenced by `PatrolConditions` function in BP_AI_Parent
- **Notes:** 90 nodes across 2 graphs — more logic than expected for a path actor; may handle debug visualisation at construction time.

---

## BP_AI_Parent
- **Type:** Blueprint
- **Parent:** Character
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AI/BP_AI_Parent`
- **Purpose:** The master base class for all FCS AI — handles initialisation of every combat component, companion logic, dialogue camera transitions, patrol, loot, quest linking, and death.
- **Key Variables:**
  - `AI Faction` (byte, Setup In Game) — which faction this AI belongs to; drives aggro rules
  - `AI Setup Data` (struct, Setup In Game) — primary config struct; sets stats, behaviour type, weapons
  - `AutoActivateCombat` (bool, Setup in Game) — whether combat starts on spawn without overlap trigger
  - `PatrolPath` (object, Setup in Game) — reference to BP_PatrolPath actor to walk
  - `CameraTransitionPath` (object, Setup in Game) — spline used during dialogue camera cuts
  - `ActorsToActivate` (object, Setup in Game) — actors triggered on interaction
  - `AIActive` (bool, AI Status) — master on/off for AI logic
  - `CombatActivated` (bool, AI Status) — true once combat has been triggered
  - `HasEnemyTarget` (bool, AI Status) — set when the AI has acquired a target
  - `RootMotionOff` (bool, AI Status) — replicated flag to disable root motion
  - `AI Weapons` (struct, AI Status) — weapon references used in combat
  - `Patroling` (bool, AI Status) — true while executing patrol behaviour
  - `CombatABP` (class, Initialize) — Animation Blueprint class to assign on init
  - `InitializeTimer` (struct, Initialize) — timer handle for deferred setup
  - `PlayerValidTimer` (struct, Initialize) — timer handle for player validity checks
  - `IsActiveCompanion` (bool, Companion) — true when this AI is a player companion
  - `IsSpawnedCompanion` (bool, Companion) — true when spawned as companion (not world-placed)
  - `SummonedCompanion` (bool, Companion) — true when summoned mid-combat
  - `CompanionInventoryType` (byte, Companion) — controls what loot companions carry
  - `CompanionIndex` (int, Companion) — which companion slot this fills
  - `Dead` (bool, Companion) — death state flag (replicated context)
  - `TakingDamageComponent` (object, Default) — **primary damage handler** — all enemy damage routes through this
  - `MeleeComponent` (object, Default) — melee attack logic
  - `RangedComponent` (object, Default) — ranged/arrow attack logic
  - `MagicComponent` (object, Default) — spell casting logic
  - `LootComponent` (object, Default) — loot table and drop logic
  - `InventoryComponent` (object, Default) — inventory management
  - `QuestComponent` (object, Default) — quest objective link
  - `DialogueComponent` (object, Default) — conversation system
  - `OutlineComponent` (object, Default) — interact highlight outline
  - `BuffComponent` (object, Default) — status effect handling
  - `EquipmentComponent` (object, Default) — equipped item slots
  - `CombatStatusComponent` (object, Default) — tracks combat state flags
  - `MovementStatusComponent` (object, Default) — tracks movement state flags
  - `ResourceComponent` (object, Default) — health/stamina/mana pool
  - `InputBufferComponent` (object, Default) — buffered attack inputs
  - `MapTrackerComponent` (object, Default) — minimap marker
  - `PhysicalReactComponent` (object, Default) — hit reactions
  - `CharacterRef` (object, References) — reference to the player character
  - `OriginalAIController` (class, References) — stores AI controller class for re-activation
  - `ComponentsSetupBool` (bool, Status) — guards against double-initialisation
- **Key Functions/Events:**
  - `InitializeCombatComponents` — master init; called on BeginPlay
  - `SetupComponents` / `CreateCoreCombatComponents` / `CreateOptionalCombatComponents` — component factory functions
  - `Setup Weapon /Range /Magic Combat` — arms the AI based on AI Setup Data
  - `ActivateCombatAI` — enables the AI controller and starts the Behavior Tree
  - `Death` — handles death sequence, loot spawning, component cleanup
  - `LootFunctionality` — activates loot drop on death
  - `PatrolConditions` — evaluates whether patrol should begin
  - `OverlapConditions` — handles aggro trigger on player overlap
  - `WoundedNeedRevive` / `Revived` — companion revive flow
  - `SetCompanion` — converts this AI to companion role
  - `DestroyIF Companion` — cleans up companion on dismissal
  - `OnRep_AIActive` / `OnRep_RootMotionOff` / `OnRep_CombatABP` — replicated state callbacks
  - `InteractFunctionality` / `InteractConditions` — NPC interact handling
  - `RunDialogue` / `SetupConversationCam` / `ReturnCameraAndSettings` — dialogue camera system
- **GodOfRuin Use:** **EXTEND** — all custom enemy and NPC Blueprints (BP_AI_Human children) derive from this; do not edit this directly — override in child classes
- **Notes:**
  - 58 graphs / 1,564+ nodes — do not read full graphs without a specific target function.
  - `TakingDamageComponent` is the confirmed damage routing point per FCS_SystemReference.md.
  - Component `EnemyFaction (Delete when Ready)` has a dev note in its name — flag for cleanup.
  - `ComponentsSetupBool` prevents double-init — do not call `InitializeCombatComponents` manually.

---

## BP_AI_Human
- **Type:** Blueprint
- **Parent:** BP_AI_Parent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AI/AI-Types/BP_AI_Human`
- **Purpose:** Thin child of BP_AI_Parent that marks human-type AI — currently a shell (0 own variables, 0 own components, 2 nodes) whose behaviour is entirely inherited.
- **Key Variables:** None own — all inherited from BP_AI_Parent
- **Key Functions/Events:** EventGraph (empty), UserConstructionScript (empty)
- **GodOfRuin Use:** **EXTEND** — create GodOfRuin enemy Blueprints as children of BP_AI_Human (not BP_AI_Parent directly); this is the correct layer to add human-specific overrides
- **Notes:** Being a shell is by design — FCS uses this as the branching point between human and animal AI trees. All Paragon character Blueprints should inherit from here.

---

## BT_MixedCombat
- **Type:** BehaviorTree
- **Parent:** N/A
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AI/BehaviourTrees/BT_MixedCombat`
- **Purpose:** The primary enemy combat Behavior Tree — drives melee, ranged, and magic attack selection, blocking, strafing, patrol, noise response, and lunge behaviour via 38 Blackboard keys.
- **Key Variables (Blackboard keys):**
  - `EnemyTarget` (Object) — current combat target
  - `AI_Behaviour` (Enum) — master state (patrol / combat / idle / etc.)
  - `AttackStyle` (Enum) — melee / ranged / magic switching
  - `MeleeAttackType` (Enum) — which melee combo to use
  - `SpellCastType` (Enum) — which spell to cast
  - `Block` (Bool) — currently blocking
  - `Rolling` (Bool) — currently rolling
  - `ShouldLunge` / `SendLunge` (Bool) — lunge attack trigger
  - `ArrowLoaded` (Bool) — ranged readiness
  - `SpellCharged` / `SpellSelected` (Bool) — magic readiness
  - `CurrentMeleeCombo` / `LastMeleeCombo` (Name) — combo chain tracking
  - `StrafeLocation` (Vector) — target strafe position
  - `StoredPlayerLocation` (Vector) — last known player position
  - `CanHearPlayer?` (Bool) — noise detection flag
  - `AtNoiseLocation?` (Bool) — reached noise investigation point
  - `OutsideAttackGroup` (Bool) — crowd control spacing flag
  - `TeleportOnCD` / `LungeOnCD` (Bool) — ability cooldowns
  - `StandStillChannel` (Bool) — channelling a spell
  - `ClearForShot` (Bool) — line-of-sight check for ranged
- **GodOfRuin Use:** **USE** — assigned to all `BP_AI_Human`-derived enemies; do not replace, extend via BT Services and Tasks in child trees if needed
- **Notes:** Blackboard asset is `BB_AI`. Both BTs share the same 38 keys — CompanionMixedCombat uses `CompanionBehaviour`, `CompanionOwner`, and `CompanionGoHereLocation` keys that this tree likely ignores.

---

## BT_CompanionMixedCombat
- **Type:** BehaviorTree
- **Parent:** N/A
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AI/BehaviourTrees/BT_CompanionMixedCombat`
- **Purpose:** Companion variant of BT_MixedCombat — same 38 Blackboard keys but adds companion-specific behaviour branches for follow, defend-owner, and go-to-location commands.
- **Key Variables (Blackboard keys — companion-specific):**
  - `CompanionBehaviour` (Enum) — companion state (follow / attack / stay / go-here)
  - `CompanionOwner` (Object) — the player this companion belongs to
  - `CompanionGoHereLocation` (Vector) — destination when player issues a move command
  - (all other keys shared with BT_MixedCombat — see above)
- **Key Functions/Events:** Inherits all BT_MixedCombat task/service/decorator structure
- **GodOfRuin Use:** **USE** — automatically assigned when `SetCompanion` is called on a BP_AI_Human; do not manually assign
- **Notes:** Confirmed same blackboard (BB_AI, 38 keys). The companion keys are always present in BB_AI — the enemy tree simply never writes to them.

---

## Skipped Assets
The following assets exist in this folder but are excluded from documentation:

| Asset | Reason |
|---|---|
| BP_AI_Dummy, BP_AI_BlockDummy, BP_AI_AssassDummy, BP_AI_RangedAssassDummy, BP_AI_InvinsibleDummy | Test/training dummies — no production use |
| EQS_TestingPawn | Editor testing tool only |
| BP_AIController | FCS internal — do not subclass; override behaviour via BT and components |
| BP_AI_Animal | Not relevant to current GodOfRuin roster (human enemies only) |
| BT_AnimalCombat, BT_AnimalCompanionCombat | Not relevant until animal enemies are scoped |
| BB_AI | Data asset — documented via BT entries above |
| S_*, T_*, D_* (40 assets) | BT Services, Tasks, Decorators — internal FCS nodes; document individually only if custom versions are needed |
| E_AISpawnMethod | Enum used by spawners — document if spawn logic is extended |
| S_AISpawnData | Struct used by spawners — document if spawn logic is extended |
| I_SpawnMarker | Interface on BP_AISpawnMarker — document if custom markers are built |
| EQS_AroundTarget | EQS query for BT strafe logic — internal FCS |
| BP_AI-Jump-Link | NavLink proxy — place in levels for AI jump points; no subclassing needed |

---

## E_StatBuilds — Stat Preset Enum
Path: /Game/FlexibleCombatSystem/Enumerations/AI/E_StatBuilds

FCS uses preset stat builds instead of manual HP/damage values.
Set this enum on each enemy child BP via AI Setup Data → S_AI-StatData.

| Index | Internal Name | Display Name | GodOfRuin Use |
|-------|--------------|--------------|---------------|
| 0 | NewEnumerator0 | Tank (Defence Focused) | Steel, Narbash |
| 1 | NewEnumerator1 | Warrior (Attack Focused) | FCS Soldiers, Kwang, Crunch |
| 2 | NewEnumerator2 | Rogue (Crit Attack Focused) | Kallari, Feng Mao |
| 3 | NewEnumerator3 | Ranger (Range Attack Focus) | Aurora, Morigesh, FCS Ranged |
| 4 | NewEnumerator4 | Mage (Spell Damage Focus) | NOT USED in GodOfRuin |
| 5 | NewEnumerator5 | Default (Mixed) | Khaimera, Shinbi clones |
| 6 | NewEnumerator6 | Duelist (Attack/Crit Mix) | Serath, Greystone |

Boss tier enemies configured via Boss AI Toolkit — not E_StatBuilds.
Rampage, Terra, Countess, The Ruined One use Boss AI phase system.
