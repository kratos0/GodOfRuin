# Abilities — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/Abilities/`
> Subfolders: `Children/`, `SpawnedItems/`
> Total assets: 24 (documented: 24 | none skipped — no dummies or test assets in this folder)

### Ability Inheritance Tree

```
BP_AbilityParent  (Actor — ROOT)
├── BP_AbilityChannel              ← channeled beam/drain spells
│   ├── BP_HealChannelParent
│   └── BP_HealChannelAllyParent
├── BP_AbilitySkyChannel           ← overhead/AoE channeled spells
├── BP_AbilityMovingChannel        ← moving channeled spells (caster moves while casting)
├── BP_AbilityPlacementParent      ← ground-targeted/placed spells
│   ├── BP_Teleport
│   ├── BP_RuneParent
│   ├── BP_SummonAlly
│   ├── BP_AreaAttackTickParent    ← AoE that ticks damage repeatedly
│   ├── BP_AreaAttackSingleParent  ← AoE that deals damage once
│   └── BP_NSTriggerSpellAttackParent
├── BP_AbilitySelfCast             ← instant self-targeted spells
│   ├── BP_BuffParent
│   └── BP_HealSingleParent
├── BP_AbilityProjectile           ← fired projectile spells
│   └── BP_Polymorph
├── BP_AbilityMelee                ← ability-enhanced melee strikes
├── BP_AbilityRanged               ← ability-enhanced arrow shots
└── BP_AbilitySpawnMagicWeapon     ← conjure a magic weapon

SpawnedItems (spawned by abilities, not in hierarchy):
  BP_PlacementCircle               ← targeting decal shown during placement
  BP_PhysicalRune                  ← persistent rune trap placed on ground
  BP_MovingDamagingSpell           ← traveling spell actor (used by MovingChannel)
```

---

## BP_AbilityParent
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilityParent`
- **Purpose:** Root base class for every ability — handles targeting, velocity calculation, damage setup, VFX/audio refs, and replicated activation for both player and AI spells.
- **Key Variables:**
  - `AI_Spell` (bool) — when true, ability is controlled by AI; skips player input checks
  - `AbilityInfo` (struct, Ability Info) — primary config struct passed from the character on spawn
  - `AbilityVelocity` (struct, Ability Info) — calculated launch/travel velocity (replicated via OnRep_SpellVelocity)
  - `SpellDamage` (real) — base damage value for this ability instance
  - `ShieldDamageAmount` (real) — separate damage applied to shields
  - `CriticalHit` (bool) — whether this cast was a crit
  - `Activated` (bool) — replicated activation flag (OnRep_Activated drives client-side FX)
  - `SpellFired` (bool) — set after the spell projectile/effect is released
  - `InstantCast` (bool) — skips placement phase; fires immediately
  - `ChannelPressed` (bool) — tracks whether player is holding the channel input
  - `SpellPlacementLocation` (struct) — world location of placement targeting
  - `SpellPlacementRotation` (struct) — rotation of placement targeting
  - `ProjectileHitLocation` (struct) — stores hit point for non-homing projectiles
  - `EnemyTarget` (object) — target actor for homing/AI abilities
  - `CharacterRef` (object, References) — the character that cast this ability
  - `MagicComponent` (object) — reference to caster's MagicComponent (manages mana/cooldown)
  - `TargetingComponent` (object) — reference to caster's TargetingComponent
  - `InputBufferComponent` (object) — reference to caster's InputBufferComponent
  - `EquipmentComponent` (object) — reference to caster's EquipmentComponent
  - `FollowCamera` (object, References) — caster's camera for direction calculation
  - `CharacterMesh` (object, References) — caster's mesh for socket attachment
  - `Camera Boom` (object) — spring arm ref for camera-relative calculations
  - `Bow Mesh` (object) — bow mesh ref used by ranged abilities
  - `Niagara System Ref` (object) — active Niagara FX ref for runtime control
  - `SpellEnd` (object) — sound cue played on spell end/expiry
  - `Mouse Location` (struct) — cursor world location for mouse-aimed placement
- **Key Functions/Events:**
  - `SetupVariables & References` — initialises all component refs from CharacterRef on spawn
  - `Calculate Ability Velocity` — computes launch velocity from camera/target data
  - `Calculate Ability Hit Location` — raycast to find world-space hit target
  - `Get Camera Forward` / `Get Camera Location` / `GetCameraRelative` — camera helpers
  - `CalculateCharacterRotation` — rotates caster to face spell direction
  - `SpellFiredAuthorityOR Local` — authority-safe spell fire entry point
  - `Update Ability Placement` — tick-driven placement decal/circle update
  - `RemovePlacementMarker` — destroys the BP_PlacementCircle actor
  - `ActivateNS` — activates the Niagara system at cast time
  - `AI-SpellPlacement` — AI-specific placement calculation (bypasses camera)
  - `BowValidCheck` — validates bow is equipped before ranged ability fires
  - `CorrectAuthority` — RPC authority correction for multiplayer
  - `OnRep_Activated` — replicates activation state to clients
  - `OnRep_SpellVelocity` — replicates velocity to clients for projectile sync
- **GodOfRuin Use:** **EXTEND** — never use directly; all custom player and enemy abilities must inherit from one of the typed children below
- **Notes:**
  - `AI_Spell` bool is the key to dual player/enemy use — always set on enemy-spawned abilities.
  - `AbilityInfo` struct is passed from the character at spawn time; inspect it when debugging wrong damage values.
  - Components are the same ones on `BP_AI_Parent` — the ability reads them from `CharacterRef` on spawn, not from its own set.

---

## BP_AbilityChannel
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilityChannel`
- **Purpose:** Channeled spell base — fires a sustained beam/drain effect, ticks damage on a timer, and plays loop audio until the channel is released or broken.
- **Key Variables:**
  - `Channeling` (bool) — true while hold input is maintained
  - `ChannelTickTime` (real) — seconds between each damage tick
  - `SpellDamage_` (real) — per-tick damage amount
  - `SpellTickTimer` (struct) — timer handle for the damage tick loop
  - `SpellStart` (object) — sound cue on channel start
  - `ChargeLoopSound` (object) — looping audio during charge phase
  - `ChannelLoopSound` (object) — looping audio during active channel
  - `SpellLoop` (object) — active audio component ref for stopping
  - `NiagaraChannel` (object) — active Niagara beam/VFX ref
  - `DamageActor` (object) — current hit target for damage application
  - `UpdateChannelRotation` (struct) — replicated rotation for beam direction sync
  - `HitInfo` (struct) — last hit result for FX placement
  - `CriticalHit_0` (bool) — crit state for this tick
- **Key Functions/Events:**
  - `SpawnNiagaraChannel` — spawns and attaches the beam Niagara system
  - `ApplyTraceDamage` / `ApplyAITraceDamage` — line trace + damage application per tick
  - `ApplyTraceDamageServer` — server-authoritative trace RPC
  - `SpellChannelDmgAdjust` — adjusts damage mid-channel (e.g. for ramp-up)
  - `DrainLife` — variant that heals the caster equal to damage dealt
  - `DeactivateChannel` — cleans up on release
  - `GetAbilityTraceEnd` — calculates far end of the trace beam
  - `SFXChannelCharge` / `SFXChannelLoop` / `StopAudio` — audio lifecycle
  - `OnRep_UpdateChannelRotation` — syncs beam direction on clients
- **GodOfRuin Use:** **EXTEND** — parent for any sustained beam spell; for heal beams use BP_HealChannelParent instead
- **Notes:** `DrainLife` is already implemented here — if a lifesteal ability is needed, extend this and call DrainLife rather than rebuilding.

---

## BP_AbilitySkyChannel
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilitySkyChannel`
- **Purpose:** Overhead channeled attack — places a targeting decal on the ground, then performs a sphere trace from above to deal AoE damage in a column while channeling.
- **Key Variables:**
  - `Channeling` (bool) — active channel state
  - `DamageChannel` (struct) — damage configuration for the sphere trace
  - `SpellPlacementTimer` (struct) — timer for placement update ticks
  - `SpellLoop` (object) — active loop audio component
  - `SpellStart` (object) — sound on cast start
  - `SpellAnimSound` (object) — sound tied to cast animation
  - `SpellChannelSound` (object) — loop audio during channel
  - `StopLoop` (bool) — flag to halt the loop sound
  - `SpellPS` (object) — Niagara/particle system ref
  - `HitInfo` (struct) — last trace hit result
- **Key Functions/Events:**
  - `SetupMarker` — spawns BP_PlacementCircle at target location
  - `SphereTraceAttack` — executes the downward sphere trace and applies damage
  - `DeactivateChannel` — cleans up decal, audio, VFX on release
- **GodOfRuin Use:** **EXTEND** — use for lightning-strike type spells or rain-of-arrows; the placement decal visualises the AoE radius automatically
- **Notes:** Decal size is driven by `BP_PlacementCircle.DecalSize` — set this on the spawned circle via `AbilityInfo` to match the sphere trace radius.

---

## BP_AbilityMovingChannel
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilityMovingChannel`
- **Purpose:** Channeled spell where the caster moves while casting — spawns a BP_MovingDamagingSpell actor that travels forward and deals damage along its path.
- **Key Variables:**
  - `DamagingSpell` (object) — reference to the spawned BP_MovingDamagingSpell actor
  - `ClientLocationPassTimer` (struct) — timer for passing updated caster location to the spell actor
  - `SpellPlacementTimer` (struct) — timer handle
  - `SpellLoop` (object) — active loop audio component
  - `SpellStart` / `SpellEnd_0` / `SpellAnimSound` / `LoopSound` (object) — audio lifecycle refs
  - `HitInfo` (struct) — hit result
  - `StopLoop` (bool) — halt loop flag
- **Key Functions/Events:**
  - `DeactivateSpell` — stops movement, destroys spawned actor, clears audio
- **GodOfRuin Use:** **EXTEND** — use for dash-cast spells where the ability travels with or ahead of the player
- **Notes:** Requires a BP_MovingDamagingSpell instance — configure `MoveSpeed` on that actor to control travel speed.

---

## BP_AbilityPlacementParent
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilityPlacementParent`
- **Purpose:** Ground-targeted placement spell base — shows a BP_PlacementCircle decal while the player aims, then triggers on confirm input; all AoE, rune, teleport, and summon spells extend this.
- **Key Variables:**
  - `AnimationSound` (object) — sound played on confirm/cast animation
  - `SpellPlacementTimer` (struct) — timer driving the placement update tick
  - `AnimationPlayed` (bool) — prevents double-playing the cast animation
- **Key Functions/Events:**
  - EventGraph — drives the placement loop (Update Ability Placement from parent) and confirm trigger
- **GodOfRuin Use:** **EXTEND** — all ground-targeted player abilities (AoE, traps, teleport, summons) should inherit from here
- **Notes:** 172 nodes in EventGraph despite only 3 own variables — the bulk of placement logic lives in `BP_AbilityParent.Update Ability Placement` and is called through here.

---

## BP_AbilitySelfCast
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilitySelfCast`
- **Purpose:** Instant self-targeted spell base — no placement phase; fires immediately on the caster with a Niagara VFX component.
- **Key Variables:**
  - `SpellPlacementTimer` (struct) — timer handle (used for spell duration)
- **Key Components:**
  - `Niagara_0` : NiagaraComponent — self-cast VFX
- **GodOfRuin Use:** **EXTEND** — for any instant self-buff, self-heal, or personal shield ability
- **Notes:** Shortest inheritance chain to a working ability — good starting point for new utility spells.

---

## BP_AbilityProjectile
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilityProjectile`
- **Purpose:** Fired projectile spell — uses ProjectileMovementComponent for physics-driven travel; applies damage on hit and plays a hit sound.
- **Key Variables:**
  - `SpellDamage_0` (real) — damage on hit
  - `HitInfo` (struct) — hit result
  - `LoopSound` (object) — sound while travelling
  - `StopCharge` (bool) — replicated flag; triggers `OnRep_StopCharge` to halt charge VFX on clients
- **Key Components:**
  - `ProjectileMovement` : ProjectileMovementComponent — drives travel velocity and gravity
- **Key Functions/Events:**
  - `ApplyDamage` — applies damage to hit actor
  - `PlayHitSound` — plays impact audio
  - `OnRep_StopCharge` — client-side: stops charge VFX when projectile fires
- **GodOfRuin Use:** **EXTEND** — for any physically-travelling magic bolt, fireball, or arrow-style ability; set `AbilityVelocity` from BP_AbilityParent to control launch speed
- **Notes:** `BP_Polymorph` extends this — the only child is cosmetic (adds a SkeletalMesh to show a morphed creature projectile).

---

## BP_AbilityMelee
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilityMelee`
- **Purpose:** Ability-enhanced melee strike — sweeps a trace arc, supports charge levels, applies scaled damage, and calculates Niagara rotation for directional VFX.
- **Key Variables:**
  - `Melee Ability Data` (struct) — sweep arc config (angle, range, trace shape)
  - `ObjectTypes` (byte) — collision channel mask for the trace
  - `AlreadyHitEnemy` (object) — prevents hitting the same enemy twice per swing
  - `ChargeLevel` (int) — current charge tier (drives damage scaling via IncreaseCharge)
  - `SpellDamage_0` (real) — calculated damage after charge scaling
  - `Z Rotation` (real) — VFX rotation around Z axis
  - `NS_Charge` (object) — Niagara charge-up VFX ref
  - `HitInfo` (struct) — hit result
  - `LoopSound` / `StopCharge` (object/bool) — audio and charge state
- **Key Components:**
  - `ProjectileMovement` : ProjectileMovementComponent — used for forward-lunge movement on release
- **Key Functions/Events:**
  - `ApplyDamageTraces` — executes the sweep traces and collects hits
  - `ApplyDamage` — applies scaled damage to each hit
  - `IncreaseCharge` — bumps ChargeLevel while hold input is maintained
  - `CalculateNiagaraRotation` — aligns VFX to swing direction
  - `PlayHitSound` — impact audio
- **GodOfRuin Use:** **EXTEND** — for charged heavy attacks, ability-driven combos, or special attacks that need trace-based hit detection rather than animation notify traces
- **Notes:** `AlreadyHitEnemy` is an object array cleared each swing — important to not share it across multi-hit abilities without resetting it.

---

## BP_AbilityRanged
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilityRanged`
- **Purpose:** Ability-enhanced arrow shot — spawns an arrow mesh on the bow socket, validates the draw via `BowValidCheck`, supports charge levels, and attaches or destroys the arrow on hit.
- **Key Variables:**
  - `Ranged Ability Data` (struct) — arrow config (speed, gravity, arc)
  - `SpawnedArrow` (object) — the spawned arrow actor ref
  - `ObjectTypes` (byte) — trace collision channels
  - `AlreadyHitEnemy` (object) — hit-once guard
  - `ChargeLevel` (int) — charge tier for draw strength
  - `SpellDamage_0` (real) — damage after charge scaling
  - `HitInfo` (struct) — hit result
  - `LoopSound` / `StopCharge` (object/bool) — audio and state
- **Key Components:**
  - `StaticMesh` : StaticMeshComponent — arrow head visual
  - `ProjectileMovement` : ProjectileMovementComponent — arrow flight physics
  - `ArrowComponent` : ArrowComponent_C — direction indicator
- **Key Functions/Events:**
  - `SetupArrow` — spawns and attaches arrow to bow socket
  - `ApplyDamage` / `ApplyDamageTraces` — hit detection and damage
  - `Attach OR Destroy` — on hit: attaches arrow to target or destroys it
  - `Remove Arrow Data` — clears arrow ref after flight
  - `PlayHitSound` — impact audio
- **GodOfRuin Use:** **EXTEND** — for special arrow abilities (explosive, piercing, multi-shot); ensure `BowValidCheck` passes before calling SpellFiredAuthorityOR Local
- **Notes:** Shares structure with BP_AbilityMelee — `ChargeLevel`, `AlreadyHitEnemy`, and `StopCharge` work identically. Arrow mesh is set via `StaticMesh` component — swap material/mesh in child BP.

---

## BP_AbilitySpawnMagicWeapon
- **Type:** Blueprint
- **Parent:** BP_AbilityParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/BP_AbilitySpawnMagicWeapon`
- **Purpose:** Conjures a magic weapon — spawns a weapon actor, displays a duration bar UI, and destroys it when the duration expires.
- **Key Variables:**
  - `Weapon` (struct) — weapon data (class to spawn, socket, duration)
- **Key Components:**
  - `Niagara` : NiagaraComponent — conjure VFX
- **Key Functions/Events:**
  - `SpawnMagicItem` — spawns the weapon actor at the appropriate socket
  - `CreateSpellDurationBar` — creates an in-world duration UI widget
  - `OnRep_Weapon` — replicates weapon spawn to clients
- **GodOfRuin Use:** **EXTEND** — for summoned weapon abilities (divine sword, spectral bow, etc.); the `Weapon` struct drives what gets spawned
- **Notes:** Duration bar uses FCS widget system — if WB_* widgets are crashing (see KnownIssues.md), this ability's UI element may also fail; test before relying on it.

---

## BP_Teleport
- **Type:** Blueprint
- **Parent:** BP_AbilityPlacementParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_Teleport`
- **Purpose:** Placement-based teleport — player aims with a decal then confirms to instantly move to the targeted location.
- **Key Variables:**
  - `TeleportLocation` (struct) — confirmed destination
  - `SpellPlacementTimer_0` (struct) — timer handle
- **GodOfRuin Use:** **USE** — directly usable as a player ability; assign to ability slot via character's MagicComponent
- **Notes:** 91 nodes despite only 2 own vars — teleport execution logic lives here, not in the parent.

---

## BP_RuneParent
- **Type:** Blueprint
- **Parent:** BP_AbilityPlacementParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_RuneParent`
- **Purpose:** Placement-based rune trap — places a BP_PhysicalRune actor at the targeted location that persists and triggers on enemy contact.
- **Key Variables:** None own — configuration driven entirely by parent placement system and BP_PhysicalRune
- **GodOfRuin Use:** **EXTEND** — create a child with the rune's damage type and VFX assigned; all trap-style abilities should use this lineage
- **Notes:** The physical rune actor (`BP_PhysicalRune`) handles persistence and save info independently.

---

## BP_BuffParent
- **Type:** Blueprint
- **Parent:** BP_AbilitySelfCast_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_BuffParent`
- **Purpose:** Self-cast buff ability base — applies a buff effect to the caster with a configurable Niagara particle system.
- **Key Variables:**
  - `BuffPS` (object) — Niagara particle system for the buff VFX
- **GodOfRuin Use:** **EXTEND** — parent for all player self-buff abilities (damage boost, speed boost, damage reduction, etc.)
- **Notes:** Buff application logic routes through `BP_AI_Parent.BuffComponent` — ensure the buff data struct is correct when creating child abilities.

---

## BP_HealSingleParent
- **Type:** Blueprint
- **Parent:** BP_AbilitySelfCast_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_HealSingleParent`
- **Purpose:** Instant self-heal — applies a one-shot heal to the caster with a Niagara VFX.
- **Key Variables:**
  - `NiagaraPS` (object) — heal VFX reference
- **GodOfRuin Use:** **EXTEND** — for potion-style ability heals or passive triggered heals; note KnownIssues.md: the `Heal` function currently operates at full capacity only — partial heals are broken
- **Notes:** **Known issue:** Heal is full-heal only — do not expose heal amount as a variable to designers until this is fixed.

---

## BP_HealChannelParent
- **Type:** Blueprint
- **Parent:** BP_AbilityChannel_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_HealChannelParent`
- **Purpose:** Sustained self-heal channel — holds the cast button to continuously regenerate health over time.
- **Key Variables:** None own — uses `ChannelTickTime` and `SpellDamage_` from BP_AbilityChannel for heal rate
- **GodOfRuin Use:** **EXTEND** — for meditation/prayer-style sustained healing abilities
- **Notes:** Inherits `DrainLife` from BP_AbilityChannel — if you want heal-on-hit rather than self-heal, use DrainLife on a damage channel instead.

---

## BP_HealChannelAllyParent
- **Type:** Blueprint
- **Parent:** BP_AbilityChannel_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_HealChannelAllyParent`
- **Purpose:** Sustained channel heal targeting an ally — same as BP_HealChannelParent but the heal applies to `EnemyTarget` (used as ally target) instead of self.
- **Key Variables:** None own
- **GodOfRuin Use:** **USE** — relevant if companion healing abilities are scoped; also usable for NPC healer enemies
- **Notes:** Uses `EnemyTarget` object reference for the ally — ensure companion targeting sets this correctly.

---

## BP_SummonAlly
- **Type:** Blueprint
- **Parent:** BP_AbilityPlacementParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_SummonAlly`
- **Purpose:** Ground-targeted summon — places a companion or temporary ally AI at the chosen location using the placement decal system.
- **Key Variables:**
  - `Units Spawned` (object) — reference to the spawned ally actor
  - `SpawnLocation` (struct) — confirmed spawn world location
  - `SpellDamage_0` (real) — damage value passed to the summoned unit
  - `ChannelTickTime` (real) — tick rate if the summon has an upkeep cost
  - `SpellTickTimer` (struct) — timer for upkeep ticks
  - `Hit Actor` (object) — last overlap/hit for targeting
  - `CriticalHit_0` (bool) — whether the summon roll was a crit
- **GodOfRuin Use:** **EXTEND** — wire to Paragon character classes for summoning specific allies; `Units Spawned` ref allows the caster to track and despawn the ally
- **Notes:** Connects into the companion system via `BP_AI_Parent.SetCompanion` on the spawned unit — ensure the spawned BP inherits from BP_AI_Human.

---

## BP_AreaAttackTickParent
- **Type:** Blueprint
- **Parent:** BP_AbilityPlacementParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_AreaAttackTickParent`
- **Purpose:** Placed AoE that deals repeated damage ticks to all enemies within the radius until it expires.
- **Key Variables:**
  - `SpellDamage_0` (real) — per-tick damage
  - `ChannelTickTime` (real) — seconds between ticks
  - `SpellTickTimer` (struct) — tick timer handle
  - `NiagaraFX` (object) — persistent AoE VFX ref
  - `Hit Actor` (object) — current tick target
  - `CriticalHit_0` (bool) — per-tick crit state
- **GodOfRuin Use:** **EXTEND** — for fire puddles, poison clouds, holy ground, or any lingering AoE
- **Notes:** Prefer this over BP_AreaAttackSingleParent when the ability should tick (DoT, slow zone, etc.).

---

## BP_AreaAttackSingleParent
- **Type:** Blueprint
- **Parent:** BP_AbilityPlacementParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_AreaAttackSingleParent`
- **Purpose:** Placed AoE that deals a single burst of damage to all enemies in the radius on detonation.
- **Key Variables:**
  - `SpellTickTimer` (struct) — detonation delay timer
  - `Hit Actor` (object) — primary hit target
  - `ChannelTickTime` (real) — delay before detonation
  - `CriticalHit_0` (bool) — crit state for the burst
- **GodOfRuin Use:** **EXTEND** — for instant-burst AoE (explosion, shockwave, gravity pull); pair with a Niagara charge-up VFX for telegraphing
- **Notes:** `ChannelTickTime` doubles as the detonation delay here — name is inherited and slightly misleading.

---

## BP_NSTriggerSpellAttackParent
- **Type:** Blueprint
- **Parent:** BP_AbilityPlacementParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_NSTriggerSpellAttackParent`
- **Purpose:** Placement ability that triggers damage via a Niagara System event rather than a trace — damage fires when the NS emits a collision/trigger event.
- **Key Variables:**
  - `SpellDamage_0` (real) — damage on NS trigger
  - `CriticalHit_0` (bool) — crit state
  - `Hit Actor` (object) — target from NS event
  - `Test` (object) — **dev leftover variable — flag for removal**
- **GodOfRuin Use:** **EXTEND** — use when the ability visual (Niagara) should drive the hit timing (e.g. a projectile spawned inside the NS that reports back on hit)
- **Notes:** The `Test` variable is a leftover from development — remove it before shipping any child ability.

---

## BP_Polymorph
- **Type:** Blueprint
- **Parent:** BP_AbilityProjectile_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/Children/BP_Polymorph`
- **Purpose:** Projectile that visually transforms the target on hit — extends BP_AbilityProjectile with a SkeletalMesh component to display the polymorph creature form.
- **Key Variables:** None own
- **Key Components:**
  - `SkeletalMesh` : SkeletalMeshComponent — creature mesh shown on the projectile
- **GodOfRuin Use:** **EXTEND** — if a polymorph/petrify/transform mechanic is needed; the actual transform effect needs to be implemented in a child
- **Notes:** Cosmetic shell only — damage and hit logic are entirely inherited from BP_AbilityProjectile.

---

## BP_PlacementCircle
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/SpawnedItems/BP_PlacementCircle`
- **Purpose:** The targeting decal shown on the ground while a placement ability is being aimed — fades in on spawn and scales its decal to the configured spell radius.
- **Key Variables:**
  - `DecalSize` (real) — radius of the decal in world units; must match the ability's trace/sphere radius
  - `DecalMaterial` (object) — base decal material asset
  - `DecalMaterialInstance` (object) — runtime MID for colour/opacity control
  - `AI Spell` (byte) — when set, may suppress or recolour the decal for AI-cast abilities
  - `FadedIn` (bool) — set after fade-in completes
- **Key Components:**
  - `Decal` : DecalComponent — the ground projection
  - `DefaultSceneRoot` : SceneComponent
- **GodOfRuin Use:** **USE** — spawned automatically by the placement system; set `DecalSize` via AbilityInfo to match each ability's AoE radius
- **Notes:** No manual spawning needed — `BP_AbilityParent.Update Ability Placement` handles lifecycle.

---

## BP_PhysicalRune
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/SpawnedItems/BP_PhysicalRune`
- **Purpose:** Persistent ground rune that sits at the placed location, detects enemy overlap via sphere collision, deals spell damage on trigger, and can be saved/loaded as part of the game save.
- **Key Variables:**
  - `Spell Info` (struct) — damage configuration passed from the caster
  - `SpellDamage` (real) — flat damage on trigger
  - `RuneSpellDuration` (real) — how long before the rune expires
  - `RuneOwner` (string) — identifies which player owns this rune
  - `EnemyAI` (bool) — true if this rune was placed by an AI
  - `Decoration` (bool) — when true, rune is cosmetic only (no damage trigger)
  - `CriticalHit` (bool) — whether the trigger counts as a crit
  - `Damage Causer` (object) — actor ref for kill attribution
  - `Hit Actor` (object) — current triggered actor
  - `RuneSaveInfo` (struct) — serialised state for the save system
- **Key Components:**
  - `Sphere` : SphereComponent — overlap trigger
  - `Niagara` : NiagaraComponent — rune VFX
- **Key Functions/Events:**
  - `IsCorrectTarget` — validates the overlapping actor is a valid enemy
  - `DestroyRune` — cleans up after trigger or expiry
  - `SaveInfo` — writes state to `RuneSaveInfo` for persistence
- **GodOfRuin Use:** **EXTEND** — any trap, sigil, or proximity mine ability; the `Decoration` flag enables purely visual runes for environment dressing
- **Notes:** Integrates with the save system — `RuneSaveInfo` struct must be included in save data if runes should persist across sessions.

---

## BP_MovingDamagingSpell
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Abilities/SpawnedItems/BP_MovingDamagingSpell`
- **Purpose:** A traveling spell actor spawned by BP_AbilityMovingChannel — moves at a configured speed, deals damage to enemies along its path, and self-destructs on expiry or reaching its destination.
- **Key Variables:**
  - `MoveSpeed` (real) — units/second travel speed
  - `FinalDestination` (struct) — target world location to travel toward
  - `SpellParent` (object) — reference back to the BP_AbilityMovingChannel that spawned it
  - `MagicComponent` (object) — caster's MagicComponent for mana checks
  - `CharacterActorRef` (object) — caster reference for damage attribution
  - `SpellInfo` (struct) — damage configuration
  - `SpellDamage` (real) — damage per hit
  - `CriticalHit` (bool) — crit state
  - `ManaSuccess` (bool) — whether mana was available on spawn
  - `DamageEnemy` (object) — current damage target
  - `Deactivating` (bool) — prevents double-deactivation
  - `EmptyLocation` (bool) — true if cast into empty space (no target)
  - `Timer` (struct) — lifetime timer handle
  - `NiagaraPS` (object) — travel VFX ref
- **Key Components:**
  - `Niagara` : NiagaraComponent — travel VFX
- **GodOfRuin Use:** **USE** — spawned automatically by BP_AbilityMovingChannel; configure `MoveSpeed` and `SpellInfo` through the caster ability
- **Notes:** `ManaSuccess` is checked on spawn — if false, the actor immediately deactivates; ensure MagicComponent has mana before spawning.
