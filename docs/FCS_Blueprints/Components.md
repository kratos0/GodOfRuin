# Components — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/Components/`
> Total assets: 33 | All relevant — none skipped
> All components parent: `ActorComponent`

### Component Groups
- **Combat Core:** TakingDamage, MeleeComponent, RangedComponent, MagicComponent, DefensiveComponent, BuffComponent, ResourceComponent, CombatStatusComponent, InputBufferComponent, HitStopComponent, WeaponCollision
- **Movement / Physical:** MovementStatusComponent, MovementRotationStatus, PhysicalReactComponent, AttackMotionWarpingComponent, GaspComponent, TraversalMovementComponent, SwimmingComponent, TargetingComponent, TrajectoryDataReplication, ArrowComponent
- **NPC / World:** EquipmentComponent, InventoryComponent, LootComponent, OutlineComponent, DialogueComponent, QuestGiverComponent, QuestReceiverComponent, SkillTrainerComponent, CraftingComponent, MapTrackingComponent, NPC-RPC-Assist, PlayerWorldTriggers

---

## COMBAT CORE

### TakingDamage
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/TakingDamage`
- **Purpose:** The single entry point for all incoming damage on any FCS actor — routes hits through damage reduction, elemental effects (fire, frost, poison, lightning, madness, polymorph), stagger/poise, knockback, and death; the most complex component in the system at 61 graphs and 80 variables.
- **Key Variables:**
  - `DamageIncoming` (real, Incoming Attack Info) — raw damage value before reductions
  - `DamageType` (byte, Incoming Attack Info) — physical / magic / elemental etc.
  - `AttackType` (byte, Incoming Attack Info) — light / heavy / ranged / spell
  - `AttackingActor` (object) — who dealt the damage (for kill attribution)
  - `CriticalHit` (bool) — incoming crit flag
  - `HitResult` (struct) — world-space hit data (location, bone, surface)
  - `ImpactPoint` (struct) — world-space impact position for VFX
  - `NiagaraHitVFX` (object) — VFX asset to spawn at impact
  - `PoiseCurrent` / `PoiseMax` / `Poise Damage` / `PoiseRegenAmount` (real) — poise/stagger system
  - `Staggered` (bool, Stagger) — currently staggered
  - `ImmuneToStagger` (bool, Stagger) — boss/elite stagger immunity flag
  - `StaggeredDamageMultiplier` (real) — bonus damage taken while staggered
  - `AutoBlockChance` (real, Block) — chance to auto-block without player input
  - `StunTrue` / `Stun Duration` — stun state
  - `KnockbackTrue` / `Knockback Data` — knockback state
  - `IsFrozen` / `FreezeTimer` — freeze state
  - `Polymorphed` / `PolymorphTime` / `PolymorphAnim` / `PolymorphMesh` — polymorph state
  - `MadnessActive` / `MadnessDuration` — madness state (friendly fire)
  - `Damage Over Time` (bool) — active DoT flag
  - `BurnDuration` / `BurnDamageBase` / `BurnTick` — fire DoT
  - `PoisonDuration` / `PoisonTickRate` / `CurrentPoisonDmg` / `PoisonMaxDamage` — poison DoT
  - `FrostSlowDuration` / `FrostSlowSpeed %` — frost slow
  - `LightningSpreadDamage` — lightning chain damage
  - `ArrowBonusDamage` — bonus damage when hit by arrows
  - `CanBeRevived` (bool, Status) — companion revive eligibility
  - `GameplayTags` (struct) — gameplay tag container for this actor
  - Component refs: `CombatStatusComp`, `MeleeComponent`, `MagicComponent`, `DefensiveComponent`, `EquipmentComponent`, `BuffComponent`, `RangedComponent`, `HitStopComponent`, `InputBufferComp`, `PhysicalAnimComponent`
- **Key Functions:**
  - `Store Components` / `StoreComponentWithCheck` — init; called once on BeginPlay
  - `StoreDamageInfo` — caches incoming attack data before processing
  - `CanTakeDamage` — master guard (checks immunity, death, dialogue, etc.)
  - `CanBlockAttack` — evaluates block/parry eligibility
  - `CalculateDamageReduction` — applies armor, resistance, buff reductions
  - `DamageHealth` — applies final reduced damage to health pool
  - `DamageHealthUI` / `DamagePopup` — floating damage number display
  - `StaggerCalculation` — evaluates poise break and stagger trigger
  - `CalculateHitAnim` / `CalculateHitDirection` / `GenerateFrontHitAnim` — selects hit reaction
  - `Physical Hit` — melee hit processing pipeline
  - `Magic Damage` / `Fire Hit` / `Frost Damage` / `Ice Damage` / `Electric Damage` / `Energy Damage` / `Poison Damage` / `Madness Damage` — elemental pipelines
  - `CalculateKnockback` — applies knockback impulse
  - `Polymorph` / `PolymorphToggle` — transforms target to animal form
  - `Freeze` / `RemoveFreeze` / `FrostSlow` / `RemoveFrost` / `RemoveCold` / `RemoveMadness` — elemental state management
  - `BarrierAbsorb` / `BarrierDamageAbsorb` — barrier damage interception
  - `Stun & HitReact` — stun + animation combo
  - `AIReport` — notifies AI BT of damage received
  - `Play Shake & Remove Revive` — camera shake + revive state cleanup
- **GodOfRuin Use:** **USE** — never bypass; all damage on all actors routes here. `StoreDamageInfo` → `CanTakeDamage` → `CalculateDamageReduction` → `DamageHealth` is the pipeline. When debugging "enemy doesn't take damage," start at `CanTakeDamage`.
- **Notes:** `TakingDamageComponent` is confirmed in FCS_SystemReference.md as the routing point for enemy damage. The `Heal` full-heal-only bug (KnownIssues.md) originates in `BuffComponent.Heal`, not here.

---

### MeleeComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/MeleeComponent`
- **Purpose:** Drives all melee combat — combo sequencing via BP_ComboResolver, damage calculation, lunge attacks, dual-wield, assassination, and block/parry impact reactions; 36 graphs, 51 variables.
- **Key Variables:**
  - `Attacking` (bool, Attack Status) — currently in an attack
  - `AttackType` (byte, Attack Status) — current attack type (light/heavy/lunge/special)
  - `HeavyAttack` (bool) — heavy attack in progress
  - `LungeAttacking` (bool) — lunge in progress
  - `SaveAttack` (bool) — buffered next attack flag
  - `ComboNumber` (int) — current combo step count
  - `Root Motion Off` (bool) — replicated root motion disable flag
  - `Combo Resolver` (object, AttackCombo) — `BP_ComboResolver` instance
  - `CurrentComboAttack` (struct, Attack Combo) — active combo data
  - `LungeAttackAnim` (struct, Animations) — lunge montage data
  - `Main Hand Dmg` / `Off Hand Dmg` (real, Damage) — calculated per-hand damage
  - `Main Hand Crit` / `Off Hand Crit` (bool) — per-hand crit flags
  - `PairDamage` (real) — combined dual-wield damage
  - `AttackComponentRef` (object, Weapon Ref) — active weapon BP reference
  - `AssassOccuring` (bool, Assassination) — assassination in progress
  - `CanAssassinate` (bool) — assassination eligibility
  - `OverlappedActor` (object, Assassination) — assassination target
  - `ExecuteTarget` (object, Assassination) — execution target
  - `AssassinationToPlay` (struct) — assassination montage data
  - `MotionWarping` (object) — MotionWarpingComponent ref
  - `AttackMotionWarpingComponent` (object) — FCS warp coordinator ref
  - Component refs: `TakingDamageComponent`, `CombatStatusComp`, `InputBufferComponent`, `EquipmentComponent`, `ResourceComponent`, `DefensiveComponent`, `MovementComponent`, `CapsuleComponent`
- **Key Functions:**
  - `StoreComponents` — init
  - `AttackConditions` — master guard before executing an attack
  - `CalculateTotalAttackDamage` — sums base weapon + stats + buffs
  - `CalculateIndividualDamage` — per-weapon damage calc
  - `DualWieldUpdatedDamage` — recalculates for dual wield
  - `SetMeleeAnimOutput` — sends the combo montage to the ABP
  - `GetComboAnimations` — queries BP_ComboResolver for next animation
  - `PassAnimation` — sends montage to character for play
  - `SendAttackInfo` — broadcasts attack data to WeaponCollision
  - `SendLungeAttack` — triggers lunge with motion warping
  - `UpdateWarpTargets` — updates motion warp target before lunge
  - `ShouldSpecialAttack` — evaluates special attack conditions
  - `Reset Melee Attack` — clears attack state on animation end
  - `AssassinationConditions` / `AssassinationPrep` / `PlacePlayerForAssassination` — assassination pipeline
  - `SpawnAssassinationMarker` / `CheckAssassinateDetected` — stealth marker management
  - `PassExecuteAnim` — sends execution montage
  - `CombatText` / `ExecuteCombatText` — damage text display
  - `GlobalResetCheck` — emergency full reset
  - `OnRep_BlockUp` — replicated block state response
- **GodOfRuin Use:** **USE** — all Paragon melee attacks route through here. For new combo sets: bind weapon data to `Combo Resolver` via `BP_ComboResolver.SetWeaponCombos`. For assassination: set `CanAssassinate` on the target.
- **Notes:** `AttackConditions` checks stamina, movement state, and whether another attack is already playing — if attacks aren't triggering, this is the first place to inspect.

---

### RangedComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/RangedComponent`
- **Purpose:** Full bow combat system — draw/aim/fire cycle, arc prediction path visualisation, arrow type switching, execute shots, ranged parry, and camera management; 56 graphs, 69 variables.
- **Key Variables:**
  - `DrawBow` / `BowAiming` / `ArrowLoaded` / `FiringArrow` (bool) — bow state machine flags (replicated)
  - `AttachHandToString` (bool) — cosmetic: hand attached to string
  - `ArrowType` (class) — currently selected arrow Blueprint class
  - `E_ArrowType` (byte) — enum version of arrow type for UI
  - `SpawnedArrow` (object) — active spawned arrow actor
  - `ArrowInfo` (struct, Arrow) — current arrow stats/type data
  - `ArrowVelocity` (struct, Arrow Math) — calculated launch velocity
  - `ArrowSpeed` (real) — base arrow travel speed
  - `GravityScale` (real) — arrow gravity for arc calculation
  - `InstantCast` (bool) — skip aim phase for instant-fire arrows
  - `PredictionRunning` (bool, Prediction) — arc prediction active
  - `PredictionTraces` (struct) — prediction path trace data
  - `Prediction Beam Array` (object) — Niagara beam actors for arc visualisation
  - `ArrowSwitcherOpen` (bool) — radial arrow switcher UI open
  - `ArrowExecuteOccuring` / `ClearForExecute` (bool, Execute) — execute shot state
  - `BowMesh` (object, Bow) — bow skeletal mesh ref
  - `BowStringOrigin` (object) — bow string socket ref
  - `BowPredictionPath` (object) — prediction spline actor
  - `CustomAIArrows` (struct, AI) — AI-specific arrow config
  - Camera vars: `CameraLocationCurrent/Destination`, `CameraRotationCurrent/Destination`, `TargetArmLengthCurrent/Destination`
  - Component refs: `TargetingComponent`, `EquipmentComponent`, `InventoryComponent`, `InputBufferComponent`, `ResourceComponent`, `MovementRotationStatus`, `DefensiveComponent`, `CombatStatusComponent`
- **Key Functions:**
  - `StoreComponents` / `StoreComponentsWithCheck` — init
  - `LaunchConditions` — master fire guard
  - `AimSetup` / `ReleaseAimSetup` — camera transition into/out of aim mode
  - `SetArrowVelocity` — calculates launch velocity from camera angle and arrow speed
  - `CreateArrow` — spawns the arrow actor at bow socket
  - `DestroyArrow` — removes arrow on miss/expire
  - `FireArrowGroup` — sequences the fire events (notify-driven)
  - `PredictionPath` — calculates and renders the arc prediction spline
  - `SetPredictionPSs` / `SpawnPredictionPSs` / `RemovePredictionPath` — arc VFX lifecycle
  - `ArrowSwitcher` / `PopulateArrowSwitcher` / `UpdateArrowSwitcher` — arrow type UI
  - `Execute` / `ExecuteConditions` / `IsExecuteShot` — execute shot pipeline
  - `RangedParry` / `GetParryAnimation` — ranged parry system
  - `Setup_AI_Arrows` / `AI-UpdateEnemyTarget` — AI bow configuration
  - `CalculateRangedDamage` — bow damage formula
  - `TryConsumeStamina` — stamina cost on draw
  - `GlobalResetCheck` — emergency reset
  - `OnRep_ArrowLoaded` / `OnRep_DrawBow` / `OnRep_BowAiming` / `OnRep_AttachHandToString` — replication
- **GodOfRuin Use:** **USE** — handles all bow/ranged combat. For Paragon archer characters, `CustomAIArrows` configures which arrow type the AI uses. `PredictionPath` gives the parabolic aim arc visible to the player.
- **Notes:** Arrow type switching reads from `InventoryComponent` — arrows must be in the inventory to be selectable. `ArrowSpeed` + `GravityScale` together define the arc shape.

---

### MagicComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/MagicComponent`
- **Purpose:** Manages the full ability/spell system — spell selection, cooldown tracking, mana consumption, ability spawning, channel state, and radial spell menu; 32 graphs, 45 variables.
- **Key Variables:**
  - `AbilityFiring` (bool, Ability Status) — an ability is currently active
  - `MagicAiming` (bool, Spell Status) — player is in spell aim mode
  - `SpellChannel` (bool) — channeling is active (replicated)
  - `SpellChannelAnimType` (byte) — which channel animation to play (replicated)
  - `AbilityCharged` (bool) — charge threshold met for charged abilities
  - `InstantCast` (bool) — ability bypasses aim phase
  - `Magic Attempting Aim` (bool) — mid-transition to aim state
  - `SelectedAbility` (struct, Spell Status) — currently equipped ability data
  - `CurrentSpellSlot` (int) — active slot index
  - `SpawnedSpell` (object) — reference to the active spawned ability actor
  - `CurrentlyFiringSpell` (object) — ability BP currently executing
  - `SpellsOnCD` (struct, Cooldowns) — which spells are on cooldown
  - `CooldownTimer` / `SpellCDChecker` (struct) — cooldown tick timers
  - `CooldownDecrement` (real) — per-tick cooldown reduction amount
  - `MagicSpellsLearnt` (struct, Setup & Data) — all learned abilities
  - `StartingSpells` (struct) — default starting ability loadout
  - `NumberOfRadialSpells` (int) — how many spells on the radial menu
  - `RangedAbilityCamera` (bool) — camera override active for ranged abilities
  - `EnemyTarget` (object, References) — locked-on target for homing spells
  - `BP_SpellPlacement` (object) — spawned placement circle reference
  - Dispatchers: `UpdateSpellAiming`, `AbilityChargeUp`, `EventSpellChanneling`, `EventSpellChannelNumber`, `SpellFiredDispatch`, `CooldownUpdatedDispatch`, `CooldownEnded`
  - Component refs: `EquipmentComponent`, `ResourceComponent`, `InventoryComponent`, `TargetingComponent`, `InputBufferComponent`, `CombatStatusComp`, `BuffComponent`, `MovementComponent`, `MovementRotationComponent`
- **Key Functions:**
  - `StoreComponents` / `StoreComponentsWithCheck` — init
  - `Setup Spells & Spellbook` — initialises ability slots from `MagicSpellsLearnt`
  - `LoadDefaultSpells` / `LoadSaveData` / `LoadBuffs` — save/load pipeline
  - `SpawnAbility` — instantiates the correct ability BP child class
  - `SpawnRunes` — special path for rune placement abilities
  - `AbilitySwap` — switches active spell slot
  - `CreateAbilityConditions` — guard before ability spawn
  - `CheckMana` / `ReduceMana` — mana validation and consumption
  - `ReduceCooldown` — per-tick cooldown timer
  - `Calculate Ability Damage` — scales ability damage with stats
  - `AimingConditions` / `Update Aim Status & Move Speed` — aim mode management
  - `InstantCastCheck` / `InstantCastConditions` — bypass for instant abilities
  - `IsChannelSpell` / `IsTeleportSpell` / `AbilityIsPlace & Channel` / `AbilityIsInstantCastOnly` — ability type routing
  - `CanSpellUseLockOn` — whether the ability homes to target
  - `AI-SpellConditions` / `AI-UpdateEnemyTarget` — AI spell casting
  - `GlobalResetCheck` — emergency reset
  - `OnRep_MagicAiming` / `OnRep_SpellChannel` / `OnRep_SpellChannelAnimType` — replication
- **GodOfRuin Use:** **USE** — all ability usage for both player and AI routes here. To add a new ability: create a child of the appropriate BP_Ability* class, add it to the character's `MagicSpellsLearnt` struct, and `SpawnAbility` handles the rest. `AI_Spell` bool on the ability distinguishes player from AI casts.
- **Notes:** `CooldownDecrement` is a flat per-tick reduction — cooldown speed is sensitive to tick rate. `UnlockedAbilities` in the save system (per FCS_SystemReference.md) feeds into `LoadSaveData` here.

---

### DefensiveComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/DefensiveComponent`
- **Purpose:** Handles all blocking and parrying — evaluates whether an incoming attack can be blocked, applies stamina cost, plays the correct reaction animations, and can break the attacker's combo on a perfect parry.
- **Key Variables:**
  - `BlockUp` (bool, Block) — shield/weapon block is active (replicated)
  - `ParryUp` (bool, Parry) — parry window is open
  - `ParryOpening` (real, Parry) — duration of the parry window in seconds
  - `ParryConsumesStaminaDeactivated` (bool) — when true, parrying costs no stamina
  - `StaminaDamageScalar` (real, Block) — stamina cost multiplier per blocked hit
  - `AttackType` (byte) / `DamageInc` (real) — cached incoming attack data
  - `AttackingActor` (object) — ref to attacker for counter-attack
  - `ImpactPoint` (struct) — impact world location for block VFX
  - Animation refs: `BlockImpactAnim`, `BlockBreakAnim`, `AttackBlockedAnim`, `ParryImpactAnim`
  - Component refs: `TakingDamageComponent`, `ResourceComponent`, `CombatStatusComp`, `InputBufferComponent`, `EquipmentComponent`, `MovementComponent`
- **Key Functions:**
  - `CanDefendHit` — master block/parry eligibility check
  - `CheckAngleForBlock` — only blocks attacks within the block arc
  - `EnoughStaminaForBlock` / `StaminaCheck` — stamina gating
  - `PassDamageInfo` — routes blocked/partial damage back to TakingDamage
  - `BreakBlockConditions` / `BreakBlockTakeDamage` — guard-break handling
  - `EnemyBlockHit` — plays impact animation on the attacker when blocked
  - `FistBlockCheck` — special case for unarmed blocking
  - `CalculateKnockTime` — determines stagger duration from block impact
  - `UpdateAnimations` — sends current block state to ABP
  - `DefensiveReactionText` — triggers floating "BLOCKED" / "PARRIED" text
  - `Reset AIAttack` — resets AI attack after being blocked
  - `OnRep_BlockUp` — replication
- **GodOfRuin Use:** **USE** — add to any character that can block. `ParryOpening` controls the parry window size — tune per enemy type to control difficulty. `ParryConsumesStaminaDeactivated` is useful for shield-locked boss phases.
- **Notes:** Block arc check (`CheckAngleForBlock`) means attacks from behind always connect regardless of block — ensure enemy AI uses flanking to punish blocking.

---

### BuffComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/BuffComponent`
- **Purpose:** Applies and manages timed buffs, barriers, and periodic heals — supports stat modifiers (Strength, Agility, Armor, Intellect), damage barriers with a type filter, and over-time healing.
- **Key Variables:**
  - `Buff Type` (byte, BuffInfo) — which type of buff this instance represents
  - `BuffAmount` (real) — generic buff magnitude
  - `BuffDuration` (real, Time) — how long this buff lasts
  - `CustomTime` (bool) / `CustomTimeRemaining` (real) — override duration at runtime
  - `BarrierShieldAmount` (real) — total barrier HP
  - `Barrier Hit Damage` (real) — damage absorbed per hit
  - `Barrier Damage Type` (byte) — which damage type the barrier absorbs
  - `DelayBetweenHeal` (real) — seconds between HoT ticks
  - `HealTimer` (struct) — HoT tick timer handle
  - `StrengthBuffAmount` / `AgilityBuffAmount` / `ArmorBuffAmount` / `IntellectBuffAmount` (real, Temp Stat Changes) — per-stat modifier amounts
  - `Special Damage Info` (struct) — special attack damage override while buffed
  - `Buff Save Info` (struct) — persistence data for save system
  - `Niagara System` / `BuffPS` (object) — VFX refs
  - Component refs: `EquipmentComponent`, `InventoryComponent`
- **Key Functions:**
  - `StoreVariables` — init
  - `UpdateStats & Damage` — applies stat deltas to EquipmentComponent
  - `ReduceStatBeforeReapply` — removes old stats before stacking a new buff
  - `DoesBuffExist` — checks if this buff type is already active
  - `DestroyOldBuff` — removes expired or overwritten buff instance
  - `BuffDurationDelay` — countdown timer to auto-destroy
  - `Heal` — periodic heal tick (**known issue: heals to full only**)
  - `HealPopup` — floating heal number
  - `AI_DestroyDoubleBarrier` — AI-specific barrier cleanup
  - `Remove Spawned Weapon` — cleans up conjured weapon on buff expiry
- **GodOfRuin Use:** **USE** — buff Paragon characters via MagicComponent's `LoadBuffs`; barrier is the shield mechanic; `Special Damage Info` enables damage-type-switching buffs (e.g. blessed weapons).
- **Notes:** `Heal` full-heal bug (KnownIssues.md) lives here — `BuffDuration` still works correctly. Do not use `Heal` for partial heals until fixed.

---

### ResourceComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/ResourceComponent`
- **Purpose:** Manages health, stamina, and mana pools — tracks current values, drives regen timers, and broadcasts UI update dispatchers.
- **Key Variables:**
  - `DrainStaminaTick` / `DrainStaminaTimer` — stamina drain rate and timer
  - `IncreaseStaminaTick` / `RecoverStaminaTimer` — stamina regen rate and timer
  - `StaminaRegainDelay` (real) — seconds before stamina begins regenerating after use
  - `DrainManaTick` / `DrainManaTimer` — mana drain rate/timer (for channeled abilities)
  - `IncreaseManaTick` / `RecoverManaTimer` — mana regen rate/timer
  - Component refs: `EquipmentComponent`, `MovementComponent`, `PhysicalAnimComponent`
- **Key Functions:**
  - `StoreComponents` / `StoreComponentsWithCheck` — init
  - `ReduceStaminaByOne` / `DamageStaminaOnce` — stamina consumption
  - `IncreaseStamina` — stamina regen tick
  - `StaminaDrained` — fires when stamina hits zero (triggers exhaustion)
  - `SprintCheck` — validates stamina available for sprint
  - `ReduceMana` / `DamageManaOnce` — mana consumption
  - `IncreaseMana` — mana regen tick
  - `ManaDrainedCheck` — fires when mana hits zero
  - `ClientUpdateUI` / `UpdateStaminaDispatch` — UI update broadcasts
- **GodOfRuin Use:** **USE** — health, stamina, and mana are read from `EquipmentComponent` max values; this component drives the tick-based regen. Tune `StaminaRegainDelay` to control combat pacing — it's the most impactful feel variable.
- **Notes:** Actual current health/stamina/mana values are stored on `EquipmentComponent`, not here — this component only drives the regen/drain timers and dispatchers.

---

### CombatStatusComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/CombatStatusComponent`
- **Purpose:** The character's central state bus — manages weapon draw/sheathe, action bar slots (main hand / off hand / abilities / on-use items), movement stance, throwing items, and AI combat HUD display; 66 graphs, 77 variables. The largest component in the system.
- **Key Variables:**
  - `WeaponDrawn` (bool, Draw/Sheathe) — replicated weapon drawn state
  - `WeaponStance` (byte, Character Status) — current weapon stance (unarmed/one-hand/two-hand/bow/dual)
  - `MovementStance` (byte, Character Status) — replicated movement stance
  - `DisableControls` (bool) — locks all input (cutscenes, dialogue)
  - `AimHeldDown` (bool) — aim button currently held
  - `Instant Cast` (bool, Combat) — ability instant-fire override
  - Action bar data: `FSS-MainHandBarData`, `FSS-OffHandBarData`, `FSS-AbilityBarData`, `FSS-OnUseBarData`, `ActionBarData`, `SelectedOnUse`, `SelectedAbilitySlot`, `SelectedMainHandSlot`, `SelectedOffHandSlot`
  - Ammo: `MainHandAmmoCurrent/Total`, `OffHandAmmoCurrent/Total`
  - Throwing: `ThrowingItem`, `AttemptingThrow`, `ThrowSpeed`, `ThrowType`, `ThrowVelocity`, `SpawnedOnUseItem`, `ThrowEndLocation`
  - AI: `AI-Data` (struct), `AI Combat Stance` (int), `AIShowCombatHUD` (bool), `AI-WarningMeshVisibility` (bool)
  - Companion: `CompanionSaveData` (struct)
  - Dispatchers: `GlobalReset`, `OnMovementStanceUpdated`, `DoesActionBarSlotHaveItem`, `ResetWeaponScale`, `AmmoStatsUpdated`
  - Component refs: all major combat/inventory components
- **Key Functions:**
  - `StoreComponents` — init
  - `DrawSheatheWeapon` / `DrawWeaponSetup` / `SheatheWeaponSetup` — weapon draw/sheathe pipeline
  - `Equip Weapon` / `SendWeaponAttach` / `AttachWeaponToSocket` — weapon equip/socket
  - `DestroyWeapons` / `DeathWeaponDrop` — weapon cleanup on death
  - `Action Bar` / `LoadActionBarData` / `UpdateActionBarData` — action bar management
  - `SelectActionSlotConditions` / `CalculateSlotAction` — slot selection
  - `AttackConditions` / `AttackOrResourceCollect` — input routing
  - `Throwing Items` / `SetupThrowingObject` / `LaunchThrowItemFunctionality` — item throw pipeline
  - `CreatePredictionPath` / `UpdatePredictionPath` — throwing arc visualisation
  - `MovementStanceUpdateConditions` — stance transition logic
  - `AI-UpdateWeaponStyle` / `Update_AI_HUD` / `AI Reset Components` — AI state management
  - `Interact` / `Resource Collection` / `UseItem` — world interaction entry points
  - `LoadSaveData` — save/load
  - `OnRep_WeaponDrawn` / `OnRep_MovementStance` / `OnRep_AIShowCombatHUD` — replication
- **GodOfRuin Use:** **USE** — treat this as the character's control panel; most player input decisions pass through here. `GlobalReset` dispatcher is the emergency "reset everything" signal — subscribe to it from any component that can get stuck.
- **Notes:** `DisableControls` is the correct way to lock input during cinematics or dialogue — do not freeze the character via other means. Action bar save data (`FSS-*BarData`) feeds into the save system.

---

### InputBufferComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/InputBufferComponent`
- **Purpose:** Queues a single pending attack or ability input during the buffer window defined by `AN_InputBufferSwitch` notify state — fires the queued action when the window closes.
- **Key Variables:**
  - `BufferOpen` (bool) — buffer window currently accepting inputs
  - `ConsumeInputBuffer` (byte, Buffer Stored) — the primary queued input
  - `InputBufferHighPrio` / `InputBufferLowerPrio` (byte) — priority-tiered buffer slots
  - `LowPriorityInputs` (byte) — inputs that yield to higher-priority ones
  - `LoadingStored` (bool) — currently consuming a stored input
  - `ConsumeBufferSpamPrevention` (bool) — prevents rapid re-queue on same frame
  - `OnActionCalled` (mcdelegate) — dispatcher: fires when buffered action executes
  - Component refs: `MeleeComponent`, `RangedComponent`, `MagicComponent`, `DefensiveComponent`, `CombatStatusComponent`, `TakingDamageComp`, `MovementComponent`
- **Key Functions:**
  - `StoreComponents` — init
  - `StoreBufferFunction` — stores an input when buffer is open
  - `MainInputBufferEmpty` — checks if no input is buffered
  - `FirePrimaryAction` / `FireSecondaryAction` — executes the stored input
  - `CanAnimationPlay` — validates conditions before firing buffered input
  - `IsInputLowPriority` / `RemoveLowPriorityStatus` — priority management
  - `BlockChangeToAim` — special case: block input converts to aim input
- **GodOfRuin Use:** **USE** — do not call `MeleeComponent` or `MagicComponent` directly for player input; route through this. The `OnActionCalled` dispatcher is the hook for UI feedback (e.g. button flash on buffered input).
- **Notes:** High/low priority system means a queued heavy attack can be overridden by a dodge if dodge is higher priority — configure priorities in the buffer storage calls, not here.

---

### HitStopComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/HitStopComponent`
- **Purpose:** Triggers hit-stop freeze (brief animation pause on hit) and camera shake — plays a timeline-driven dilation effect on both attacker and target to reinforce hit impact.
- **Key Variables:**
  - `Hit Stop Info` (struct) — duration and intensity for this hit stop instance
  - `HitStopCurve` (object) — timeline curve asset driving the dilation amount
  - `hitStopTimeline` (object) — active timeline component
  - `maxHitStopCountPerAttack` (int) — maximum hits that trigger stop per single attack swing
  - `currentAttackHitStopCount` (int) — running count this swing
  - `cameraShakeCooldown` (real) — minimum time between camera shakes
  - `bIsCameraShakeInCooldown` (bool) — cooldown active flag
  - `affectedTargets` (object) — array of actors already hit-stopped this swing
  - `ownerPC` (object) — owning PlayerController for camera shake
  - `OnHitStopEnded` (mcdelegate) — fires when stop completes
- **Key Functions:**
  - `TriggerHitStop` — starts the dilation timeline
  - `StopCurrentHitStop` — cancels early (e.g. death)
  - `InterruptHitStop` — forceful interruption
  - `IsOwnedByPlayer` — routes camera shake only to player-owned characters
  - `Internal_NotifyReceivedHit` — registers a new hit against the count cap
  - `Internal_ResetHitStopCount` — resets count at start of new attack
  - `SelectAttackHitStopInfo` — in WeaponCollision: selects which stop config to use
- **GodOfRuin Use:** **USE** — present on `BP_AI_Parent` as `HitStopComponent`; `HitStopComponent` and `WeaponCollision` share `Hit Stop Info` struct to coordinate. Tune `HitStopCurve` in the curve asset to control the freeze feel.
- **Notes:** `maxHitStopCountPerAttack` prevents AoE attacks from freezing the game on multi-hit — keep this at 1-2 for large AoE abilities.

---

### WeaponCollision
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/WeaponCollision`
- **Purpose:** Performs the per-frame sweep traces that detect weapon hits — traces between previous and current socket positions, collects valid hit actors, and dispatches damage/VFX/hit-stop data to TakingDamage.
- **Key Variables:**
  - `WeaponMesh` (object) — the weapon mesh to trace sockets on
  - `SecondaryWeapon` (object) — off-hand weapon mesh for dual-wield traces
  - `TraceStartLocation` / `TraceEndLocation` — current frame trace endpoints
  - `TracePreviousStartLocation` / `TracePreviousEndLocation` — last frame endpoints (sweeps between frames)
  - `TraceTimer` (struct) — timer handle for trace ticks
  - `AlreadyHitEnemy` (object) — array of actors hit this swing (prevents double-damage)
  - `ObjectTypes` (byte) — collision channel mask
  - `AttackType` (byte, Attack Info) — type of attack driving this trace
  - `Damage Type` (byte) — elemental/physical type for TakingDamage routing
  - `WeaponOutputDamage` (real) — final damage value to pass
  - `CriticalHit` (bool) — crit flag for this trace
  - `PoiseDamage` (real) — poise damage per hit
  - `Special Damage Info` (struct) — elemental/special damage data
  - `NiagaraHitVFX` / `Attack Hit VFX` (object) — impact VFX assets
  - `Hit Stop Info` / `Attack Hit Stop Info` (struct) — hit stop config for this attack
  - `Stun Data` / `Knockback Data` (softobject) — stun/knockback config assets
  - `Tag` (struct) — gameplay tags for this attack
- **Key Functions:**
  - `BeginTrace` — activates the trace timer (called by `AN_WeaponHitToggle` Begin)
  - EventGraph — runs `StopTrace` when `AN_WeaponHitToggle` End fires; stops timer and clears `AlreadyHitEnemy`
  - `SelectAttackHitVFX` / `SelectAttackHitStopInfo` — selects correct VFX and stop config per attack type
  - `PlayShakeIFPlayer` — routes camera shake if owned by player
- **GodOfRuin Use:** **USE** — this is what `AN_WeaponHitToggle` activates. `AlreadyHitEnemy` being cleared on trace end is why multi-hit combos can re-hit the same enemy on the next swing. Never call `BeginTrace` directly — it's driven by the notify.
- **Notes:** `SecondaryWeapon` traces independently — dual-wield hits are separate WeaponCollision instances, one per weapon.

---

## MOVEMENT / PHYSICAL

### MovementStatusComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/MovementStatusComponent`
- **Purpose:** Manages all locomotion states — rolling, sprinting, crouching, encumbrance, movement speed levels, and rotation mode (movement-oriented vs camera-oriented vs GASP); 22 graphs, 31 variables.
- **Key Variables:**
  - `Sprint` (bool, Status) — currently sprinting
  - `PlayerCrouch` (bool, Status) — currently crouched (replicated)
  - `bIsRolling` (bool) — currently rolling
  - `Encumbered` (bool, Status) — over-weight flag (slows movement)
  - `IsMovingOnGround` / `WasMovingOnGroundLastFrame` (bool, Status) — ground state tracking
  - `LockedSpeed` (bool) — prevents speed changes (e.g. during ability cast)
  - `MovementSpeed` (byte, Movement Speed) — current speed tier enum
  - `DefaultMaxWalkSpeed` / `WalkSpeedTarget` (real) — base and target walk speeds
  - `CurrentSlowPercent` (real, Speed) — active slow percentage (from frost, etc.)
  - `RollStaminaCost` (real, Rotations) — stamina cost per dodge roll
  - `RollRotation` (struct) — cached rotation at roll start
  - `Locomotion System` (byte) — which locomotion mode (Standard / GASP / TopDown)
  - `DA_Animations` (object) — animation data asset for locomotion montages
  - Component refs: `MeleeComponent`, `EquipmentComponent`, `InputBufferComponent`, `ResourceComp`, `CombatStatusComp`, `GaspComponent`
- **Key Functions:**
  - `Initialize` — init with refs and defaults
  - `RollingConditions` / `PlayRollMontage` — roll execution
  - `AdjustRollCollision` — capsule size during roll (pairs with `AN_RollAdjustment`)
  - `GetRollRotation` — locks roll direction at input
  - `Sprint Functionality` / `SprintConditions` — sprint gating
  - `Crouch` / `CanStand` / `CompanionCrouch` — crouch pipeline
  - `Encumbered Functionality` — applies slow when over weight limit
  - `FootstepConditions` — surface detection for footstep audio
  - `SetSlowSpeed` / `UpdateSlowSpeed` — frost/debuff speed reduction
  - `DefaultChangeMovement` / `GaspChangeMovement` — locomotion mode switching
  - `Rotate To Mouse` / `Rotate To Movement` / `Rotate To Roll` — rotation mode functions
- **GodOfRuin Use:** **USE** — `DA_Animations` is the key config point per character — swap it to match each Paragon character's animation set. `RollStaminaCost` tunes dodge feel per character class.
- **Notes:** `Locomotion System` byte controls whether GASP or standard locomotion is used — GaspComponent must be present if GASP mode is selected.

---

### MovementRotationStatus
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/MovementRotationStatus`
- **Purpose:** Thin coordinator that switches the character's `UseControllerRotationYaw` / orient-to-movement settings between Default (strafe), GASP, and Hybrid modes based on combat state.
- **Key Variables:**
  - `Locomotion System` (byte) — mirrors MovementStatusComponent's mode
  - `GaspInput` (object) — GASP input component ref
  - `CharacterMovement` (object) — CharacterMovementComponent ref
  - `CombatStatusComponent` (object) — for combat state reads
  - `ComponentsWantingOrientationToCamera` (object) — array of components that need camera-facing rotation
- **Key Functions:**
  - `CheckRotationMode` — evaluates current state and calls appropriate switch
  - `SwitchRotationModeDefault` / `SwitchRotationModeGASP` / `SwitchRotationModeHybrid` — applies settings
- **GodOfRuin Use:** **USE** — do not modify `bOrientRotationToMovement` or `bUseControllerDesiredRotation` directly on the character — always go through this component
- **Notes:** If a Paragon character is rotating incorrectly in combat, check `Locomotion System` byte matches the intended mode.

---

### PhysicalReactComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/PhysicalReactComponent`
- **Purpose:** Drives procedural physics-based hit reactions — blends the skeletal mesh into physical simulation on hit, then blends back to animation; creates the floppy limb / ragdoll blend look on impacts.
- **Key Variables:**
  - `PhysicsBlendWeight` / `CurrentPhysicsBlendWeight` (real) — target and current blend amount (0 = full anim, 1 = full physics)
  - `HitPhysicsActive` / `MovementPhysicsActive` (bool) — which physics mode is running
  - `PhysicsHitCooldown` (bool) — prevents re-triggering while already blending
  - `HitReactionAim` (real) — blend-in speed for hit physics
  - `HitReactionTimeRemaining` (real) — countdown before blend-out begins
  - `HitTimer` / `MovementTimer` (struct) — timer handles
  - `Pelvis` (name) — bone name to apply physics below (usually "pelvis")
  - `PhysicalAnimComponent` (object) — UE5 PhysicalAnimationComponent ref
  - `Mesh` (object) — SkeletalMeshComponent ref
- **GodOfRuin Use:** **USE** — present on `BP_AI_Parent`; creates the physical hit feel on enemies. Tune `HitReactionAim` (blend-in speed) and `HitReactionTimeRemaining` for enemy weight feel.
- **Notes:** `Pelvis` bone name must match the skeleton — for Paragon characters this is usually `"Pelvis"` but verify per skeleton if reactions look wrong.

---

### AttackMotionWarpingComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/AttackMotionWarpingComponent`
- **Purpose:** Coordinates motion warp target updates during attacks — keeps the warp target aimed at the locked-on enemy even if they move mid-swing, and calculates relative direction for animation blending.
- **Key Variables:**
  - `MotionWarpingComponent` (object) — UE5 MotionWarpingComponent ref
  - `TargetingComponent` (object) — reads current lock-on target
  - `AttackWarpTarget` (name) — the named warp point on the attack montage
  - `ShouldAllowTargetRotationUpdate` (bool) — enables/disables mid-swing target tracking
  - `TargetRelativeDirection` (struct) — direction from attacker to target (used by ABP for directional attack blending)
  - `CurrentTarget` (struct) — cached target transform
- **Key Functions:**
  - `UpdateAttackWarpTarget` — refreshes the warp point position each frame
  - `UpdateTargetRotation` — rotates warp to face moving target
- **GodOfRuin Use:** **USE** — this is what makes melee attacks "chase" dodging enemies slightly; present on characters via `MeleeComponent.AttackMotionWarpingComponent` ref. `ShouldAllowTargetRotationUpdate` off = attack commits to initial direction (good for heavy, slow swings).
- **Notes:** Works together with `AN_MotionWarping_Attack` notify state — the notify defines the window; this component defines the target.

---

### GaspComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/GaspComponent`
- **Purpose:** FCS's integration layer for Unreal's GASP (Game Animation Sample Project) locomotion system — manages gait states (walk/run/sprint), strafe toggles, analog input scaling, and movement speed settings for GASP-driven characters.
- **Key Variables:**
  - `GASP-Movement` (bool) — GASP mode active (replicated)
  - `Gait` (byte, Movement) — current gait state (Walk/Run/Sprint)
  - `MovementStickMode` (byte, GASP Input) — analog stick input mode
  - `SpeedMultiplier` (real, Movement) — global speed scalar
  - `WalkSpeeds` / `RunSpeeds` / `SprintSpeeds` / `CrouchSpeeds` (struct) — per-gait speed configs
  - `StrafeSpeedMapCurve` (object) — curve for speed scaling during strafe
  - `InputState` (struct, GASP Input) — current processed input state
  - `MovementInputValue` (struct) — raw input vector
  - `JustLanded` / `LandVelocity` (bool/struct) — landing state for GASP transitions
  - `AnalogWalk/RunThreshold` (real) — stick magnitude threshold for walk vs run
- **Key Functions:**
  - `SetupReferences` / `CacheMovementSettings` — init
  - `UpdateMovement` — main per-frame locomotion update
  - `GetDesiredGait` / `CalculateMaxSpeed` — gait and speed computation
  - `ToggleStrafe` / `SetWantsToStrafe` — strafe mode
  - `SetWantsToSprint` / `CanSprint` — sprint gating
  - `SetWantsToAim` / `IsTargeting` — aim mode for GASP
  - `GetInputState` / `UpdateMovementInputValue` — input processing
  - `SetGaspMovementSettings` / `RestoreMovementSettings` / `SetupHybridSettings` — mode transitions
  - `UpdatedMovementSimulated` — simulated proxy update
- **GodOfRuin Use:** **USE** — required if using GASP locomotion for Paragon characters; configure `WalkSpeeds/RunSpeeds/SprintSpeeds` structs per character to match their animation root motion speeds
- **Notes:** Only active when `MovementRotationStatus.Locomotion System` is set to GASP mode. Standard FCS characters don't use this — it's opt-in.

---

### TraversalMovementComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/TraversalMovementComponent`
- **Purpose:** Handles traversal actions (vault, hurdle, mantle, climb) — traces for valid traversal surfaces, selects the correct montage via motion matching or legacy selection, and warps the character to the ledge.
- **Key Variables:**
  - `DoingTraversalAction` (bool) — traversal in progress (replicated via `OnRep_TraversalResult`)
  - `TraversalResult` (struct) — data from the traversal surface trace (height, depth, surface normal)
  - `UseMotionMatching` (bool, Motion Matching) — use motion matching vs legacy montage selection
  - `MotionWarping` (object) — MotionWarpingComponent ref
  - `OnTraversalActionCompleted` (mcdelegate) — fires when action finishes
  - `IgnoreCorrectionsDelay` (real) — delay before allowing position correction after traverse
  - Debug vars: `DrawDebugDuration`, `DrawDebugLedgeLocations`, `DrawDebugTraversableSearch`, `DrawDebugSpaceChecks`, `PrintDebugData`, `DebugSelectedMontage`
- **Key Functions:**
  - `TryTraversalAction` — entry point; called on jump input near traversable surfaces
  - `GetTraversalCheckInputs` — packages camera/movement data for the trace
  - `GetTraversalForwardTraceDistance` — calculates how far forward to check
  - `GetTraversalMontageStartingTime` — adjusts montage start frame for blending
  - `UpdateWarpTargets` — sets motion warp points for ledge position
  - `CalculateGaitForState` — matches traversal animation to current gait
  - `OnTraversalStart` / `OnTraversalEnd` — lifecycle hooks
  - `IsDoingTraversalAction` — state query
- **GodOfRuin Use:** **USE** — enables parkour/climbing in GodOfRuin levels; place `I_Climbable` interface on geometry that should be traversable. Enable `DrawDebugLedgeLocations` in PIE to verify detection.
- **Notes:** Works with `ClimbingComponent` (from FCS_SystemReference.md) — `TraversalMovementComponent` handles the jump/vault actions while `ClimbingComponent` handles sustained wall climbing.

---

### SwimmingComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/SwimmingComponent`
- **Purpose:** Complete swimming system — water entry/exit, surface swimming, diving, breath tracking, buoyancy, and ledge climb-out from water; 23 graphs, 34 variables. Configurable via "SET THESE" category variables.
- **Key Variables (SET THESE — designer-facing):**
  - `SwimSpeed` (real) — base underwater movement speed
  - `BreathSeconds` (real) — how long the character can hold breath
  - `EnterWaterLowerHeight` (real) — how far below water surface triggers swim mode
  - `CharacterWaterSurfaceLowerer` (real) — body offset below water surface when surface-swimming
  - `MaxHeightDifferenceForLedgeClimb` (real) — max ledge height climbable when exiting water
  - `UsingBuoyancy` (bool) — enable buoyancy simulation
- **Key Variables (state):**
  - `Swimming` / `Diving` / `SwimmingOnSurface` / `OceanSwimming` (bool, Swimming State) — swim state flags (replicated)
  - `Swimming Up / Down` (bool) — vertical direction
  - `CurrentBreathValue` / `MaxBreathValue` (real) — breath pool
  - `DrainBreathTimer` / `RecoverBreathTimer` (struct) — breath tick timers
  - `SwimmingLedge` (name) — bone name for ledge IK when exiting
  - Component refs: `Water Actor`, `Character`, `Camera`, `MotionWarpingComponent`, `CharacterBuoyancy`
- **Key Functions:**
  - `SetupReferences` — init
  - `Set Character to Swim Mode` / `ExitWaterResetVariables` — mode transitions
  - `DetermineWaterSurface` / `IsCharacterDeepEnoughToSwim` / `IsOceanSwimming` — surface detection
  - `Swimming Dive OR Surface` — toggles diving vs surface swimming
  - `LockToSurface` — keeps character at correct depth when surface-swimming
  - `Breath Control` — manages breath drain/recovery
  - `CanClimbOutOfWater` / `SetupClimbOut` — water exit ledge detection
  - `LedgeClimbHandIK Transforms` — IK for hands on ledge when climbing out
  - `JumpIntoWaterSlowDown` / `RunIntoWaterSlowDown` — entry deceleration
  - `DisableSwimModeIfLandExists` — prevents swim mode on shallow water
- **GodOfRuin Use:** **USE** — add to any level with water bodies; configure the "SET THESE" variables per water area. `BreathSeconds` is the key difficulty dial for underwater sections.
- **Notes:** Requires a `Water Actor` reference — set in the level. `CharacterBuoyancy` is UE5's buoyancy component; if not using physics-based buoyancy, set `UsingBuoyancy = false`.

---

### TargetingComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/TargetingComponent`
- **Purpose:** Lock-on target system — scans for enemies within `LockOnRadius`, selects the closest valid target, cycles left/right through nearby enemies, and updates the camera to orbit the locked target.
- **Key Variables:**
  - `LockOn` (bool) — lock-on active (replicated)
  - `LockedOnTarget` (object) — current locked target actor
  - `TargetFound` (bool) — a valid target is in range
  - `EnemyVulnerable` (object) — current target vulnerability ref (for execute prompt)
  - `CameraOnRight` (bool) — camera hemisphere for target cycling
  - `LockOnRadius` (real) — sphere radius for target scan
  - `LockOnTick` (struct) — timer for lock-on validation ticks
  - `AI_InRange` / `AI_OnLeft` / `AI_OnRight` (object) — sorted nearby enemy arrays
  - `ClosestAIToLockTarget` (object) — closest enemy to currently locked target (for cycling)
  - `UpdateTargetLocked` (mcdelegate) — fires on target change
  - Component refs: `EquipmentComponent`, `CombatStatusComponent`, `MovementRotationStatus`, `FollowCamera`
- **Key Functions:**
  - `StoreComponents` / `StoreComponentsWithCheck` — init
  - `Toggle Lock On` — activate/deactivate lock-on
  - `LockOnToTarget` — executes the target acquisition
  - `ScanActorsInArea` — sphere trace to collect candidates
  - `AnythingBlockingLockOn` — line-of-sight check to target
  - `CreateForwardTrace` — forward-direction trace to find targets in view
  - `SwitchTargetLeft` / `SwitchTargetRight` — cycle through nearby enemies
  - `OnRep_LockOn` — replication callback
- **GodOfRuin Use:** **USE** — `LockOnRadius` is the primary tuning variable; too large and it grabs distant enemies, too small and players miss nearby ones. `AnythingBlockingLockOn` prevents locking through walls — ensure geometry collision is set correctly.
- **Notes:** `EnemyVulnerable` feeds the execute prompt UI — set on `TakingDamage` when health drops below execute threshold.

---

### TrajectoryDataReplication
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/TrajectoryDataReplication`
- **Purpose:** Thin replication helper — syncs GASP trajectory prediction data (velocity trajectory for animation prediction) from server to clients.
- **Key Variables:**
  - `Trajectory` (struct, Trajectory Data) — movement trajectory data for animation system
  - `TrajectoryCollision` (struct) — collision-adjusted trajectory
- **Key Functions:**
  - `UpdateTrajectoryData` — writes new trajectory data
  - `GetServerTrajectoryData` — RPC to fetch from server
- **GodOfRuin Use:** **USE** — required for GASP locomotion multiplayer correctness; add alongside GaspComponent. If only using single-player or non-GASP locomotion, this component is effectively dormant.
- **Notes:** Only active when GASP locomotion is in use — no config needed, it operates automatically.

---

### ArrowComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/ArrowComponent`
- **Purpose:** Lives on the spawned arrow actor itself — handles arrow flight physics, hit detection, damage application on impact, special functions (burst, arrow rain, execute), VFX trail, and stick-to-target on hit; 18 graphs, 33 variables.
- **Key Variables:**
  - `ArrowFired` (bool, Arrow Status) — arrow has been launched
  - `ArrowNotchedArrow` (bool) — arrow is notched (pre-fire)
  - `AssassinationTrigger` / `Execute` (bool) — assassination / execute arrow flags
  - `ArrowDamage` / `BurstDamage` (real, Arrow Damage) — impact and burst damage values
  - `BurstDamageRadius` (real) — AoE radius for explosive arrows
  - `ArrowSpecialFunction` (byte, Arrow Info) — which special function (normal/burst/rain/ultimate)
  - `DamageType` (byte) — elemental damage type
  - `SpecialDamageInfo` (struct) — elemental damage data
  - `CriticalHit` (bool) — this arrow is a crit
  - `HitActor` (object) / `HitInfo` (struct) — hit result data
  - `BodyBones` (name) — bone name hit for ragdoll
  - `StoredVelocity` (struct) — cached velocity at fire time
  - `ArrowRainDuration` / `ArrowRainTickRate` (real) — arrow rain special config
  - `ArrowOpacity` / `ArrowMaterialColour` (real, Fade & Spin) — cosmetic fade/spin
  - VFX refs: `TrailPS`, `ArrowBurst NS`, `Niagara Hit VFX`, `ArrowBurstPS`, `SpawnedTrail_PS`
  - `CharacterRef` / `RangedComponent` / `BowRef` / `ArrowActor` — ownership refs
- **Key Functions:**
  - `SetupArrowVariables` — init on spawn from RangedComponent
  - `ApplyHitDamage` — standard impact damage
  - `ApplyBurstDamage` — AoE explosion damage
  - `ApplyKnockBack` — impulse on hit target
  - `ExecutePushback` — strong knockback for execute arrows
  - `Calculate Body Damage` — multiplier based on hit bone (headshot, etc.)
  - `Attach To Component Hit` — sticks arrow to target mesh on impact
  - `ArrowStuckInWall` — alternative stick-to-geometry logic
  - `IsExplodeOnImpactArrow` / `IsUltimateArrow` — special function routing
  - `LightArrow` — cosmetic lighting effect on flight
  - `DestroyArrowDelay` — cleanup with delay for stuck arrows
  - `SpawnGroundPS` / `SpawnHitVFX` / `RemoveArrowFlightFX` — VFX lifecycle
  - `PlaySoundFX` — impact sound
  - `TweakDamageForBurst` — adjusts burst damage for multiple targets
- **GodOfRuin Use:** **USE** — spawned automatically by RangedComponent; configure `ArrowSpecialFunction` and damage on the arrow data asset, not directly on this component. For headshot multipliers, `Calculate Body Damage` already handles bone-based scaling.
- **Notes:** `ArrowRainTickRate` and `ArrowRainDuration` are the arrow-rain ability parameters — they're read by `ArrowSpecialFunction` byte `== rain`.

---

## NPC / WORLD

### EquipmentComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/EquipmentComponent`
- **Purpose:** The character's stat sheet and gear manager — stores all character stats (Strength, Agility, Intellect, Vitality, Dexterity, Wisdom, Armor, etc.), current/max health/stamina/mana, level/XP, all equipped item slots (helmet, chest, legs, hands, boots, shoulders, back, rings, necklace, weapons), and calculates final damage values; 57 graphs, 106 variables — the largest variable count in the system.
- **Key Variables (Character Stats):**
  - `Strength` / `Agility` / `Intellect` / `Vitality` / `Dexterity` / `Wisdom` (real) — primary stats
  - `Armor` (int) — physical damage reduction
  - `MagicResist` (int) — magic damage reduction
  - `ShieldBlock` (int) — block effectiveness
  - `CritChance` / `CritBase` / `CritBuffValue` (real) — crit calculation
  - `ArmorBonusDamage` (int) — bonus damage from armor set bonuses
- **Key Variables (Character Values):**
  - `Health` / `MaxHealth` / `BaseHealth` (real) — health pool
  - `Stamina` / `MaxStamina` / `BaseStamina` (real) — stamina pool
  - `Mana` / `MaxMana` / `BaseMana` (real) — mana pool
  - `BaseFistDamage` (real) — unarmed damage base
  - `CharacterLevel` / `Current_XP` / `Max_XP` (int) — leveling
  - `PlayerName` (text) — character display name
- **Key Variables (Equipment):**
  - `EquippedMainHand` / `EquippedOffhand` (object) — spawned weapon actor refs
  - `EquippedAmmoMainHand` / `EquippedAmmoOffHand` (object) — loaded ammo refs
  - `MainHandWepDamage` / `OffHandWepDamage` / `MainRangedWepDamage` / `OffRangedWepDamage` (int) — weapon damage values
  - `MaxMainHandDmg` / `MaxOffHandDmg` / `MaxFistDmg` (real) — calculated max damage caps
  - Slot structs: `MainhandInfo`, `OffHandInfo`, `HelmetInfo`, `ChestInfo`, `Legs`, `Boots`, `Hands`, `Shoulders`, `Back`, `Ring1`, `Ring2`, `Necklace`, `MainHandAmmoInfo`, `OffHandAmmoInfo`
- **Key Variables (Leveling/Save):**
  - `StatPointsAvailable` / `StatPointsPerLevel` (int) — stat allocation
  - `XPIncrease` / `OriginalXPIncrease` (int) — XP gains
  - `PlayerStartingData` (struct, Save Game) — initial character config
  - `LeveledUpStats` (struct, Save Game) — cumulative level-up stat history
- **Key Functions:**
  - `StoreComponents` / `StoreComponentsWithCheck` — init
  - `LoadCharacterStats` / `LoadSaveData` / `LoadSaveRestoreResources` / `LoadSavedMeshColors` — save/load pipeline
  - `Begin Play Update Stats` — applies starting stats on first play
  - `IncreaseCharacterStats` / `DecreaseCharacterStats` — stat modification
  - `UpdateDamageValues` — recalculates weapon damage after stat/gear change
  - `SetResourcesMax` / `SetValuesMax` / `UpdateMaxValues` — updates max pools after stat change
  - `Character Level Up` / `Leveling Up` / `ApplyXPIncrease` / `AddLevelUpStats` / `IncreaseLevelUpStats` — level system
  - `SpawnWeapon` / `SetupWeapon` / `ReplaceWeaponActor` / `RemoveWeapon` — weapon actor lifecycle
  - `Equip Weapon` / `DetermineSlotStatus` / `DetermineSlotType` — equip pipeline
  - `HasWeaponEquipped` / `Weapon Valid Check & Set Armor` — validation
  - `GetBlockingWeaponData` / `GetParryData` — defensive data queries
  - `SetMaterials` / `UpdateMaterials` / `Retrieve Mesh` / `SetInitialWidgetValues` — visual updates
  - `AddEquipmentAddStats` / `DeleteEquipmentRemoveStats` — stat delta on equip/unequip
  - `OC_SetStats` — authority-only stat set RPC
  - Dispatchers: `UpdateHealth`, `UpdateStamina`, `UpdateMana`, `UpdateStats`, `UpdateXP`, `UpdateLevel`, `UpdateCharacterBarValues`
- **GodOfRuin Use:** **USE** — `PlayerStartingData` is the key config struct for each Paragon character's starting loadout. `StatPointsPerLevel` controls the stat allocation feel per level. All damage calculations in TakingDamage reference stats here via component chain.
- **Notes:** `SelectedCharacterID` (in GodOfRuin's custom save — FCS_SystemReference.md) maps to the `PlayerStartingData` struct loaded here. Stat change always goes through `IncreaseCharacterStats` / `DecreaseCharacterStats` — never set stat variables directly.

---

### InventoryComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/InventoryComponent`
- **Purpose:** Full grid inventory system — manages item slots, stacking, weight, coin, loot pickup, companion inventories, equipping from inventory, and save/load; 60 graphs, 74 variables.
- **Key Variables:**
  - `InventorySlots` (object) — the array of item slot data
  - `MaxInventorySize` (int, Setup) — total slot count
  - `InventorySize` (int) — current used slots
  - `MaxWeight` (int) — encumbrance limit
  - `CurrentWeight` (real) — current carried weight
  - `CurrentCoins` (int) — coin balance
  - `InventoryType` (byte) — player / chest / vendor / companion type
  - `StartingItems` (struct, Setup) — items granted on first spawn
  - `Is Companion Inventory` (bool, Companion) — companion inventory flag
  - `Companion Name` (text) — owner name for companion inventories
  - `TempCompanionData` (struct) — temp companion inventory for dismissal
  - `InventoryData` / `InventorySaveArray` (struct) — serialised inventory state
  - Data tracking: `FreeSlots`, `TakenSlots`, `StackableSlots`, `WeightSlots`, `ValueSlots`, `AllSlots`
  - Dispatchers: `FindInventoryItem`, `FindItem`, `FindAmmoInInventory`, `FindAnyItemOfName`, `FindWeaponForAbility`, `GetTotalAmmoOfType`, `ItemCanStackToSlot`
  - `ItemUpdatedInInventory`, `AmmoQuantityReduced`, `UpdateEquippedVisuals`, `ActionBarSlotEmpty`, `PassItemsForSkillsPage`
- **Key Functions:**
  - `StoreComponents` / `StoreComponentsWithCheck` — init
  - `AddToInventory` / `CreateItem` / `CreateSlot` — item addition pipeline
  - `RemoveCompanionInventorySave` / `SpawnDropLoot` — item removal
  - `EquipItem` / `SpecifcActionBarEquip` / `InstantEquipAmmoCheck` — equip from inventory
  - `UnEquipSlot` / `UnEquipSlotChecks` — unequip
  - `SwapSlots` / `MoveToSlot` / `AttemptToStackSlots` — slot management
  - `Save Inventory` / `LoadSaveData` — persistence
  - `Set AI Inventory Data` — populates AI enemy loot tables
  - `AddCoin` / `UpdateCoins` — economy
  - `AddWeight` / `Remove Weight` / `CalculateMaxWeight` — encumbrance
  - `SpawnConsumable` / `UseItem` — consumable use
  - `ResetArrowSwitcher` — syncs arrow switcher UI after inventory change
- **GodOfRuin Use:** **USE** — `BP_ChestBase` (FCS_SystemReference.md) uses `InventoryComponent` for loot; set `InventoryType` to chest type. `StartingItems` drives Paragon character starting gear. `Is Companion Inventory` enables companion item management.
- **Notes:** The `FindInventoryItem` delegate is the correct way to query inventory from other components — do not iterate `InventorySlots` directly.

---

### LootComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/LootComponent`
- **Purpose:** Manages enemy loot drops and looting interaction — sets up loot data on spawn, handles the loot window UI, tracks loot state, and persists looted state to the save system.
- **Key Variables:**
  - `LootSetup` (struct, Setup) — loot configuration (items, quantities, rarity ranges)
  - `Loot Data` (struct) — runtime loot data after randomisation
  - `LootState` (byte) — current loot state (unlooted / being looted / looted)
  - `Dropped Container` (bool) — true if this is a dropped container (not a dead body)
  - `Looter` (object) — the player currently looting
  - `WB_LootWindow` (object) — loot UI widget ref
  - `WB_ItemsCreated` (object) — created item widget refs
  - `IndexToRemove` (int) — slot index being removed
  - Component refs: `InventoryComponent`, `OutlineComponent`, `CharacterRef`
- **Key Functions:**
  - `RetrieveLootData` / `AddLootDataRemake` / `AddItemsToLoot` — loot setup pipeline
  - `LootConditions` / `SetFocus` — interact eligibility
  - `SetupOverlap` — configures proximity trigger for loot prompt
  - `SetActorLooted` — marks as looted and removes interact prompt
  - `SaveLootData` / `LoadLootSaveData` / `Remove Loot Save` — persistence
  - `CloseWindows` / `RemoveTooltips` — UI cleanup
  - `DestroyDroppedItems` — cleans up dropped item actors
  - `ComponentValidCheck` — runtime component validation
- **GodOfRuin Use:** **USE** — configure `LootSetup` struct on each enemy BP variant to define their loot table. `Dropped Container` flag reuses this component for chest-style drops without a body.
- **Notes:** `LootState` persistence means enemies stay looted across sessions — ensure `Remove Loot Save` is called correctly on level reset.

---

### OutlineComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/OutlineComponent`
- **Purpose:** Manages the interact/loot highlight outline on actors — activates on player proximity overlap, positions a floating "tag" widget above the actor, and handles multiple simultaneous actor tag states (own tag vs other nearby actors).
- **Key Variables:**
  - `Actor Tag Type` (byte, Tag) — what type of tag to show (loot / interact / NPC / enemy)
  - `ActorTagVisible` (bool) — tag currently shown
  - `ActorIsActive` (bool, Status) — this actor is eligible for interaction
  - `Ignore Outline` (bool, Status) — suppresses outline (e.g. in dialogue)
  - `DataUpdated` (bool, Status) — data ready flag
  - `LocationOfTag` (struct) — world position of the floating tag
  - `ItemMesh` (object, LootInfo) — mesh ref for loot items
  - `ItemSkeletalMesh` (object, LootInfo) — skeletal mesh ref
  - `Item Mesh 2` (object) — secondary mesh (dual-mesh items)
  - `TagCanvas` / `ActorPickupTag` (object) — UI widget refs
  - `SetTagLocationTick` (struct) — timer for position updates
  - Dispatchers: `Overlap`, `EndOverlap`
- **Key Functions:**
  - `Setup Outline Variables` — init
  - `OutlineConditions` — determines if outline should show
  - `LootableObject` — sets up loot-specific tag
  - `ShowActorTag` / `RemoveActorTag` / `ShowOwnTag` / `ShowOtherTags` / `RemoveOtherTags` — tag visibility management
  - `SetTagPosition` / `LoopLocationForHeight` / `LoopLocationForWidth` — floating tag positioning
  - `DetermineMesh` — selects the correct mesh ref for the tag
  - `DontOverlapEquipped` — suppresses outline on already-equipped items
  - `DataUpdatedCheck` — validates data before showing
  - `EndOverlapConditions` / `RemoveActorTagForLoot` / `RemoveSpecificActorTag` — cleanup
- **GodOfRuin Use:** **USE** — present on BP_AI_Parent and all interactable actors; `ActorIsActive` is the on/off switch. `Ignore Outline` is the correct way to suppress the outline during dialogue or special sequences.
- **Notes:** `Actor Tag Type` controls what icon/text appears — ensure this is set correctly per actor type in BP_AI_Parent SetupComponents.

---

### DialogueComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/DialogueComponent`
- **Purpose:** Drives NPC dialogue sequences — reads lines from a DataTable (`NPC Dialogue DT`), manages line progression, triggers camera transitions, activates quest/event outcomes, and updates the dialogue widget.
- **Key Variables:**
  - `NPC Dialogue DT` (object, DialogueInfo) — DataTable with all dialogue lines for this NPC
  - `DialogueLine` (int, Dialogue Info) — current line index
  - `DialogueLineReturnNumber` (int) — line to return to after a branch
  - `DialogueLineInfo` (struct) — current line data (text, responses, events)
  - `QuestStartedDialogueLine` / `QuestCompleteDialogueLine` / `QuestConditionFailed` (int, Condition) — line indices for quest state branches
  - `QuestActivatedActor` (bool, Event) — this NPC has triggered a quest activation
  - `MidCameraTransition` (bool, Event) — dialogue camera transition in progress
  - `Trigger Event` (byte, Event) — which event to fire at line end
  - `Activate Camera Transition` (int) — line index that triggers camera swap
  - `Previous Dialogue Line` (text) — last spoken line (for response context)
  - Component refs: `QuestComponent`, `PlayerRef`, `WB Dialogue`
- **Key Functions:**
  - `GetDialogueData` — reads line from DataTable
  - `PassDialogueInfo` — sends line data to the widget
  - `UpdateDialogueData` / `UpdateDialogueTextWB` — advances conversation
  - `AddResponses` — populates player response buttons
  - `PlaySound & Animation` — triggers VO and NPC animation per line
  - `QuestAdjustments` — evaluates quest state to branch dialogue
  - `CameraTransition` / `CameraTransitionUnderway` — dialogue camera pipeline
  - `ActivateActor` — fires world actor triggers defined by `Trigger Event`
  - `TriggerEvent` — routes to the correct event handler
  - `LoadSaveData` — restores dialogue progress from save
  - `DialogueWorthSaving` — determines if this dialogue state should persist
- **GodOfRuin Use:** **USE** — assign a DataTable to `NPC Dialogue DT` per NPC; line branching is index-based. `QuestStartedDialogueLine` and `QuestCompleteDialogueLine` are the integration points with QuestGiverComponent.
- **Notes:** The duplicate `PlaySound&Animationn` (double-n) function graph may be a dev artefact — verify which one is called in the EventGraph before using either.

---

### QuestGiverComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/QuestGiverComponent`
- **Purpose:** NPC-side quest system — defines quests this NPC can offer, tracks acceptance/completion status, updates the `!` and `?` icons above the NPC, and communicates with `QuestReceiverComponent` on the player.
- **Key Variables:**
  - `AllQuestsInfo` (struct, Quest Info) — all quests this NPC can give
  - `QuestInfo` (struct) — current active quest data
  - `QuestConditions` / `QuestObjectiveStatus` (struct) — objective tracking
  - `QuestStorage Info` (struct) — persisted quest state
  - `QuestNumber` (int) — current quest index
  - `NumberOfQuests` (int) — total quests this NPC has
  - `QuestLogNumber` (int) — player's quest log slot
  - `QuestGiverType` (byte) — quest giver archetype (main / side / companion)
  - `QuestAccepted` / `QuestCompleted` / `AllQuestsComplete` / `QuestConditionsMet` (bool) — quest state flags
  - `NPCIsObjective` (bool) — this NPC is itself a quest objective
  - `ExclamationMarkSM` / `QuestionMarkSM` (object) — `!` and `?` mesh refs (from BP_AI_Parent)
- **Key Functions:**
  - `CreateQuest` / `ResetQuestVariables` — quest initialisation
  - `CheckQuestConditions` — evaluates all objective completion conditions
  - `Quest Complete Check` / `QuestObjectivesFinished` — completion detection
  - `Update Quest Giver Dialogue` — updates dialogue lines based on quest state
  - `Update Quest Mesh` / `Adjust Mesh & Tracker` / `UpdateNPCIcon` — `!/? ` icon management
  - `LoadQuest` / `PassQuestLog` — save/load and log handoff to player
  - `QuestGiverDialogueBacktrack` — resets dialogue to pre-accept state if conditions change
  - `AdjustQuestNumber` — increments to next quest after completion
  - `SetupComponentDispatchers` — binds quest update events
  - `RemoveItemFromInventory` — removes hand-in items on quest complete
  - `PassHandInTrackingInfo` — sends tracking data to QuestReceiverComponent
- **GodOfRuin Use:** **USE** — populate `AllQuestsInfo` struct on each NPC; `NPCIsObjective` for "speak to X" objectives. `QuestGiverType` affects UI sorting in the quest log.
- **Notes:** The `!` and `?` meshes are actual static mesh components on `BP_AI_Parent` (confirmed in components list) — `UpdateNPCIcon` toggles their visibility.

---

### QuestReceiverComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/QuestReceiverComponent`
- **Purpose:** Player-side quest tracker — stores accepted quests, manages the on-screen tracker (waypoints, objective text), broadcasts objective updates, handles save/load, and communicates completion back to QuestGiverComponent.
- **Key Variables:**
  - `QuestLogStorage` (struct, Quest Data) — full quest log data
  - `QuestSaveStorage` (struct) — serialised quest data for save
  - `AddedQuests` (object) — array of accepted quest refs
  - `AddedTrackedQuests` (object) — quests currently displayed on tracker
  - `QuestNumber` (int, Quest Data) — current quest being processed
  - `QuestLoadNumber` (int) — quest index during save load
  - `QuestTrackerVisible` (bool, Tracking) — tracker UI visible
  - `QuestAdded` (bool) — new quest was just accepted
  - Objective locations: `ObjectiveLocations_0/1/2/3/4` (struct) — world positions for objective markers (hardcoded to 5 objectives per quest)
  - `TrackedObjectivesCounter` (int) — number of objectives being tracked
  - `CompleteQuests` (text) — serialised completed quest IDs
  - `WB_QuestTracker` / `WB Quest Log` (object) — UI widget refs
  - Dispatchers: `QuestCheckConditions`, `QuestComplete`, `SetupQuestGivers`, `QuestObjectiveLink`, `AdjustQuestNumber`, `QuestAccepted`, `QuestAbandoned`, `QuestUpdate`, `RemoveButtonSelect`
- **Key Functions:**
  - `UpdateQuestObjective` / `QuestObjectiveUpdated` — objective progress update
  - `AddQuestComplete` / `UpdateQuestStorageObjective` — completion recording
  - `Add/RemoveQuestFromTracker` — tracker UI management
  - `AdjustObjectiveTracking` / `SetupObjectiveTracking` — objective marker setup
  - `UpdateObjectiveDataLocations` / `AverageObjectiveLocations` — waypoint position calculation
  - `AddQuestHandInTracker` — adds hand-in marker
  - `ProvideObjectiveColor` — colour-codes objectives by type/status
  - `LoadSaveData` / `LoadSaveRemoveCompletedTracker` — persistence
  - `UpdateQuestTrackerData` / `UpdateTrackQuestBool` — tracker display refresh
  - `RemoveSingleTrackingObjective` — cleans up completed objectives
- **GodOfRuin Use:** **USE** — present on the player character; `CompletedLevels` and `UnlockedAbilities` from GodOfRuin's custom save (FCS_SystemReference.md) should link into `QuestSaveStorage` for cross-session tracking.
- **Notes:** Objectives are hardcoded to 5 locations (`ObjectiveLocations_0/1/2/3/4`) — this is the "7 individual struct variables" pattern noted in KnownIssues.md applied to quests too. Plan for 5 objectives max per quest unless this is refactored.

---

### SkillTrainerComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/SkillTrainerComponent`
- **Purpose:** Handles the skill trainer NPC interaction — tracks which spells/skills this trainer can teach, manages a training quest, and broadcasts when training is available or completed.
- **Key Variables:**
  - `TrainerInfo` (struct) — this trainer's identity and available skills
  - `SpellsToTrain` (struct) — list of abilities this NPC can teach
  - `QuestStatus` / `QuestTotalStatus` (int) — training quest progress counters
  - `QuestNumber` / `QuestID` (int) — associated quest identifiers
  - `QuestType` (byte) — type of training quest
  - `QuestReward` (struct) — reward for completing training
  - `QuestAccepted` / `QuestFinished` (bool) — training quest state
  - `QuestActor` (object) — the trainer's parent actor ref
  - `CanQuestUpdate` (mcdelegate) — fires when training conditions change
- **GodOfRuin Use:** **USE** — place on skill trainer NPCs with `SpellsToTrain` populated; this feeds into `MagicComponent.LoadSaveData` via `UnlockedAbilities` in GodOfRuin's custom save.
- **Notes:** `UnlockedAbilities` in GodOfRuin's save system (FCS_SystemReference.md) maps to what this component teaches — ensure the ability names match.

---

### CraftingComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/CraftingComponent`
- **Purpose:** Drives the crafting bench interaction — teleports the player to the bench, sets up a dialogue-style camera, plays a looping craft animation, and manages the crafting UI.
- **Key Variables:**
  - `PlayerCraftingInfo` / `Crafting Data Info` (struct, Crafting Info) — crafting recipes and materials data
  - `CraftingMenuOpen` (bool, Status) — crafting UI is open
  - `LoopAnimPlaying` (bool, Status) — craft loop animation active (replicated)
  - `CraftingBench` (object, References) — the crafting actor this component belongs to
  - `CharacterOriginalTransform` (struct, Status) — player transform before teleport (for return)
  - `Attachment1` / `Attachment2` (object) — prop attachments during crafting animation
  - `WB_CraftingHUD` (object) — crafting widget ref
  - `ControlRotationZ Target` (real) — target yaw for crafting camera
  - `DialogueDamageTaken` (bool) — blocks damage during crafting (reused from dialogue system)
- **Key Functions:**
  - `InitializeCrafting` — entry point; teleports player and opens UI
  - `SetupConversationCam` / `ReturnCameraAndSettings` — camera transition in/out
  - `DestroyAttachments` — cleans up prop attachments on exit
  - `OnRep_LoopAnimPlaying` — replication for anim state
- **GodOfRuin Use:** **USE** — attach to crafting bench actors; `Crafting Data Info` struct holds recipes. `DialogueDamageTaken` is reused here — the character is invulnerable while crafting.
- **Notes:** Uses the same camera transition system as `BP_AI_Parent.SetupConversationCam` — this component essentially borrows the dialogue camera logic for crafting.

---

### MapTrackingComponent
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/MapTrackingComponent`
- **Purpose:** Places and manages map/minimap markers for NPCs, enemies, and objectives — handles marker icon, size, tracking range, and companion-specific marker updates.
- **Key Variables:**
  - `ObjectiveTrackingType` (byte, Objective Info) — marker type (enemy / NPC / companion / objective)
  - `Objective Texture` / `OriginalTexture` (object) — marker icon textures
  - `Objective Texture Size` / `OriginalTextureSize` (struct) — marker display sizes
  - `Track Distance (0 Unlimited Range)` (real) — 0 = always show; > 0 = only within this radius
  - `Active` (bool, Ref) — marker is currently displayed
  - `TrackingFinished` (bool) — tracking has been permanently stopped
  - `CombatAbilityAI` (bool) — this is a combat-capable AI (affects icon)
  - `OwnerCollision` (object) — collision component for range checks
  - `HUD Ref` / `LinkMapTrackerTimer` — HUD reference and link timer
- **Key Functions:**
  - `StoreVariables` — init
  - `AddMarkerConditions` — validates conditions before adding marker
  - `UpdatePlayerTexture` — updates marker for player character
  - `UpdateTrackingTexture - AI` / `Update Companion Texture - AI` — AI/companion marker updates
  - `AI-Tracker-Adjust` — adjusts AI marker on state change (combat start, death)
  - `StopTrackingPermanently` — removes marker on death/despawn
- **GodOfRuin Use:** **USE** — configure `ObjectiveTrackingType` and `Objective Texture` per NPC type. `Track Distance` set to 0 gives infinite range — useful for quest objectives, bad for generic enemies.
- **Notes:** Present on `BP_AI_Parent` as `MapTrackerComponent` — the naming differs by one character (`MapTrackerComponent` on AI vs `MapTrackingComponent` filename).

---

### NPC-RPC-Assist
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/NPC-RPC-Assist`
- **Purpose:** Multiplayer helper — provides server RPCs for NPC dialogue state so that dialogue-triggered effects (camera transitions, actor activations) can be authoritative in a networked session.
- **Key Variables:**
  - `DialogueOccuring` (bool) — dialogue is active (used by RPC authority checks)
- **GodOfRuin Use:** **USE** — present automatically on BP_AI_Parent NPCs; no configuration needed. In single-player, it's effectively dormant.
- **Notes:** The hyphen in the asset name (`NPC-RPC-Assist`) means it cannot be directly subclassed in Blueprint — it's intended as a utility component only.

---

### PlayerWorldTriggers
- **Type:** Blueprint (ActorComponent)
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Components/PlayerWorldTriggers`
- **Purpose:** Thin dispatcher component on the player — broadcasts an `AddMapMarkers` event that world actors (spawners, quest markers, level triggers) bind to for initialisation when the player is ready.
- **Key Variables:**
  - `AddMapMarkers` (mcdelegate) — broadcast dispatcher; world actors bind to this
- **GodOfRuin Use:** **USE** — bind world-level actors to `AddMapMarkers` for deferred initialisation that depends on the player being loaded. For example, `BP_ArenaSpawner` would bind here to set up its minimap marker when the player enters the level.
- **Notes:** Single-variable, single-dispatcher component — its entire value is as a safe initialisation signal. Do not add logic here; keep it as a pure broadcaster.
