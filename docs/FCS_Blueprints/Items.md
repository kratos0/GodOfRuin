# Items — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/Items/`
> Total assets: 26 | Subfolders: Arrows/, PickUpItems/, LootActors/, Containers/

### Inheritance Trees

```
BP_PickupItem  (Actor — ROOT for all pickup items)
├── BP_WeaponParent
│   ├── BP_ArrowQuiverParent
│   │   └── BP_ArrowQuiverAI
│   └── (your custom weapon BPs extend from here)
├── BP_ConsumableParent
└── BP_OnUseItem

BP_ArrowParent  (Actor — ROOT for all projectile arrows)
├── BP_SharpArrow         ← basic pierce
├── BP_FireArrow          ← fire VFX
├── BP_ExplosiveArrow     ← AoE blast
├── BP_UltimateArrow      ← particle effect
├── BP_IceArrow           ← freeze status
├── BP_FrostArrow         ← frost status
├── BP_PoisonArrow        ← poison status
├── BP_LightningArrow     ← chain lightning
├── BP_RainOfArrows       ← summon barrage
└── BP_MadnessArrow       ← madness status

BP_LootActorParent  (Actor — ROOT for world loot drops)
├── BP_ChestBase
├── BP_LootActorAnim
└── BP_LootActorSetMesh

BP_ContainerParent  (Actor — ROOT for world containers)
├── BP_ContainerSetMesh
└── BP_ContainerChest

BP_AnimalAttackAttachment  (Actor — standalone, NOT in pickup tree)
BP_Fists                   (Actor — standalone, NOT in pickup tree)
```

---

## BP_PickupItem
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/BP_PickupItem`
- **Purpose:** Base class for all pickable world items — handles pick-up detection, throw/arc physics, mesh replication, quest item dispatch, and save passthrough; all weapon and consumable BPs extend this.
- **Key Variables:**
  - `ItemInfo` (struct) — the item's data definition (name, type, stack size, icon, etc.) fed to InventoryComponent on pick-up
  - `DataSetup` (struct) — secondary item config (damage, weight, rarity)
  - `PickedUp` (bool, replicated) — flags item as collected; `OnRep_PickedUp` hides the mesh and triggers inventory logic
  - `ThrowingItem` (bool) — active while the item is in throw arc flight
  - `Velocity` / `ThrowStartLocation` / `Throw End Location` / `ThrowPathLocation` (struct) — arc throw trajectory data
  - `CurrentDistanceTraveled` / `DistanceTravelledThisFrame` / `RotationPerUnitDistance` (real) — throw arc math per-tick state
  - `WeaponCollision` (object, replicated) — collision component reference; `OnRep_WeaponCollision` wires it up client-side
  - `SkeletalMesh` (bool) — if true, uses `ItemSkeletal` instead of `ItemStatic` for the world display mesh
  - `CharacterRef` / `EquipmentComponent` / `CombatStatusComp` (object) — cached refs set on pick-up
  - `PathCounter` / `SpinTimer` (int/struct) — throw arc timing counters
  - `Hit` / `VelocityApplied` / `TargetUpdated` (bool) — throw state flags
- **Key Components:**
  - `ItemStatic` : StaticMeshComponent — default world mesh
  - `ItemSkeletal` : SkeletalMeshComponent — alternate skeletal world mesh
  - `Capsule` : CapsuleComponent — pick-up overlap trigger
  - `OutlineComponent` : OutlineComponent_C — interact highlight
  - `DefaultSceneRoot` : SceneComponent
- **Key Functions/Events:**
  - `SpawnSetup` — called after spawn to set mesh and initial state; pass `ItemInfo` here
  - `PickedUpCheck` — validates pick-up conditions before committing; called from Capsule overlap
  - `OnRep_PickedUp` — replication callback; hides mesh and calls `EquipmentComponent.AddItem` on owning client
  - `SetupThrow` / `ThrowHit` / `StopThrowMovement` — arc throw system; called by `EquipmentComponent` when throwing equipped items
  - `MagicThrownItem` — variant throw for magic-launched items (ability projectile style)
  - `CalculateThrowDamage` — computes throw damage at impact based on velocity/distance
  - `SpinWeapon` — tick-driven rotation during throw arc
  - `UpdateMesh` — swaps between static/skeletal mesh at runtime
  - `PassItemForSave` — routes item data to save system via `I_GameMode`
  - `PassQuestObjectiveData` / `SetupQuestDispatch` — feeds item pick-up to quest tracking
  - `Destroy Unused Mesh` — cleans up whichever mesh (static/skeletal) is not active
  - `PickupPopup` — triggers the HUD pick-up notification widget
- **GodOfRuin Use:** **EXTEND** — do not place this directly; extend `BP_WeaponParent` for weapons and `BP_ConsumableParent` for usable items. If creating a unique non-weapon/non-consumable pickup (key item, collectible), create a direct child of this BP and configure `ItemInfo`.
- **Notes:**
  - `SkeletalMesh = false` is the default — most weapons use static meshes in the world. Set to `true` only for items that need skeletal animation while lying in the world.
  - The throw system has 17 variables — it is fully self-contained; do not replicate this logic in child BPs.
  - `PickedUp` is the authoritative flag; server sets it, `OnRep_PickedUp` syncs clients. Do not call `AddItem` directly — let the rep callback do it.

---

## BP_WeaponParent
- **Type:** Blueprint
- **Parent:** BP_PickupItem_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/PickUpItems/BP_WeaponParent`
- **Purpose:** Extends the base pickup with weapon-specific features — weapon trail Niagara, execute camera, `WeaponComponent` reference binding, and off-hand support; this is the root all custom weapon BPs must extend.
- **Key Variables:**
  - `WeaponComponent` (object) — reference to the character's `WeaponComponent` once equipped; used to pass weapon data back to the combat system
  - `AttachWeapon` (byte, replicated) — notifies clients to attach the weapon mesh to the character socket; `OnRep_AttachWeaponBackNotify` handles the callback
  - `SpawnNiagaraNotify` (bool, replicated) — triggers the weapon trail Niagara spawn on equip; `OnRep_SpawnNiagaraNotify` handles it
  - `OffHandWeapon` (bool) — marks this weapon as an off-hand item (shield, dagger, etc.)
  - `WeaponDespawnTime` (real) — seconds before a dropped weapon despawns from the world
  - `System Template` (object) — the Niagara system asset for the weapon trail
  - `SetScale` (event dispatcher) — broadcasts when weapon scale should update
- **Key Components:**
  - `ExecuteCamera` : CameraComponent — camera used during execution/finisher animations
  - `WeaponTrail` : NiagaraComponent — weapon trail VFX (active during swings)
- **Key Functions/Events:**
  - `PassWeapon` — passes the weapon BP reference to `WeaponComponent` on equip; called by `EquipmentComponent`
  - `PassWeaponTrailNS` — passes the Niagara trail system to `WeaponComponent` for swing activation
  - `AttachToDummy /Character` — attaches the weapon mesh to the character's weapon socket during equip
  - `SpawnNiagara` — spawns the trail effect; driven by `SpawnNiagaraNotify` rep
  - `SetupMagicMaterials` — applies magic emissive materials to the weapon mesh when a magic buff is active
  - `ReplicationValidCheck` — guard function ensuring weapon references are valid before rep callbacks fire
- **GodOfRuin Use:** **EXTEND** — create one child BP per Paragon character weapon (e.g., `BP_Weapon_Sword_Gideon`, `BP_Weapon_Hammer_Khaimera`). Set `ItemInfo` with weapon stats, assign the static mesh, configure `WeaponDespawnTime`. The FCS weapon data asset (combo sets, damage values) is separate — set via `WeaponComponent` after equip.
- **Notes:**
  - `OffHandWeapon = true` tells `EquipmentComponent` to route this to the off-hand socket. Shields and dual-wield off-hands must set this.
  - `WeaponDespawnTime` should be set; weapons left at 0 may linger indefinitely in the level on death drops.
  - `ExecuteCamera` is used by the execution system — position it appropriately in child BPs to frame the execution animation.

---

## BP_ConsumableParent
- **Type:** Blueprint
- **Parent:** BP_PickupItem_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/PickUpItems/BP_ConsumableParent`
- **Purpose:** Extends the base pickup with consumable logic — defines restoration type/amount and buff activation; base class for all food, potions, and buff items.
- **Key Variables:**
  - `Consumable Type` (byte) — enum selecting the consumable category (health potion, stamina food, mana elixir, buff item, etc.)
  - `Consumable Power` (real) — restoration amount or buff strength
  - `Consumable Duration` (real) — duration of the effect in seconds (for timed buffs)
  - `Consume Anim Time` (real) — time in seconds for the consume animation; delays effect application to match animation
  - `BuffComponent` (object) — cached reference to the character's `BuffComponent` for buff application
- **Key Functions/Events:**
  - `ActivateBuff` — routes to `BuffComponent` to apply the buff effect on use
  - `RestoreHealth` — directly restores health via `ResourceComponent`
  - `RestoreStamina` — directly restores stamina
  - `RestoreMana` — directly restores mana
- **GodOfRuin Use:** **EXTEND** — create a child BP for each consumable type (health potion, buff food, mana crystal). Set `Consumable Type`, `Consumable Power`, and `Consume Anim Time`; populate `ItemInfo` with name/icon data. Note the known heal issue: `RestoreHealth` restores to full HP regardless of `Consumable Power` — do not use partial heal potions until this is fixed.
- **Notes:**
  - **Known Issue:** `RestoreHealth` is a full heal. `Consumable Power` does not currently control partial heal amounts. Partial restoration must be custom-implemented in a child BP. See KnownIssues.md.
  - `Consume Anim Time` is important for feel — set it to match the consume animation length so the effect doesn't pop before the animation ends.

---

## BP_ArrowQuiverParent
- **Type:** Blueprint
- **Parent:** BP_WeaponParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/PickUpItems/BP_ArrowQuiverParent`
- **Purpose:** Equippable quiver item that manages up to 20 arrow slots — tracks arrow types, counts, and passes arrow data to the bow system; this is the inventory-side of the bow/arrow system.
- **Key Variables:**
  - `ArrowSlots` (struct) — array/struct of up to 20 arrow slot entries (type + count)
  - `ArrowSpawnLocationsT` (struct) — transforms for each of the 20 ArrowComponent attachment points
  - `TotalArrowSlots` (int) — total number of slots this quiver supports (up to 20)
  - `AvailableArrowSlots` (int) — slots currently filled
  - `ArrowToPass` (object) — the arrow BP class to pass to the bow for the next shot
  - `ArrowIndexToPass` (int) — which slot index is the active arrow
  - `RemoveArrowNo` (int) — slot index of the arrow to decrement on fire
  - `ArrowsRespawn` (bool) — if true, arrows regenerate after use (infinite quiver mode)
  - `PlacedInLevel` (bool) — marks quiver as a world pickup rather than inventory item
- **Key Components:**
  - `Arrow0`–`Arrow19` : ArrowComponent (20 total) — visual attachment points for arrow models inside the quiver
  - `Arrow5Central` : ArrowComponent — central reference arrow
  - `ArrowAdjuster` : SceneComponent — root offset for quiver arrow positioning
- **Key Functions/Events:**
  - `SpawnArrowsIntoQuiver` — populates visual arrow components based on `ArrowSlots` data
  - `PassArrowToFire` — returns the `ArrowToPass` BP class to the bow system when a shot is triggered
  - `PassTotalArrowSlots` — returns slot count to UI
  - `GetArrowType` — looks up the BP class for a given slot index
  - `GetQuiverData` — returns full quiver state struct
  - `GetBowAttachments` — returns attachment socket info for bow system binding
  - `LoadSlots` / `SetSlot` — initialize and update slot data from save or pickup
  - `Remove Arrow Data` / `Reset Single Arrow` — decrement/reset a slot on fire or reload
  - `InfoValidCheck` — guard function; validates quiver data before passing to bow
- **GodOfRuin Use:** **USE** — equip on archer Paragon characters (Twinblast, Kallari with bow). Configure `ArrowSlots` with desired arrow types and counts. Set `ArrowsRespawn = true` for infinite ammo mode during gameplay testing. Child `BP_ArrowQuiverAI` is auto-equipped by AI archers — no config needed.
- **Notes:**
  - The 20 `ArrowComponent` slots are visual only — the actual arrow BP class fired is determined by `ArrowToPass`, not the ArrowComponent direction.
  - `PlacedInLevel` should be set on quivers placed as world pickups so the quiver system doesn't treat them as equipped inventory items at BeginPlay.

---

## BP_ArrowQuiverAI
- **Type:** Blueprint
- **Parent:** BP_ArrowQuiverParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/PickUpItems/BP_ArrowQuiverAI`
- **Purpose:** Simplified quiver for AI archer enemies — replaces the 20 ArrowComponents with 7 static mesh arrows (visual only) and inherits all functional quiver logic from the parent; auto-equipped by AI on spawn.
- **Key Variables:** None own (all inherited)
- **Key Components:**
  - `StaticMesh` / `StaticMesh1`–`StaticMesh6` : StaticMeshComponent — 7 decorative arrow meshes in the AI quiver visual
- **GodOfRuin Use:** **USE** — auto-handled by `EquipmentComponent` on AI archers; no configuration needed. Swap `StaticMesh` assets if a different quiver/arrow look is desired.
- **Notes:** Shell BP — identity and visual differentiation only. All quiver logic inherited.

---

## BP_OnUseItem
- **Type:** Blueprint
- **Parent:** BP_PickupItem_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/PickUpItems/BP_OnUseItem`
- **Purpose:** Pickup item that triggers a world action on use rather than going into inventory — currently implements campfire placement via `CampfireOnUse`.
- **Key Variables:**
  - `OnUseItemType` (byte) — selects which world action to trigger on use (e.g., 0 = campfire)
- **Key Functions/Events:**
  - `DetermineOnUseItem` — switch on `OnUseItemType`; dispatches to the correct use handler
  - `CampfireOnUse` — spawns a campfire actor at the player's location and destroys the item
- **GodOfRuin Use:** **EXTEND** — extend for deployable items (campfire kits, bear traps, portable forges). Add new cases to `DetermineOnUseItem` for each deployable type. Connect to `BP_CraftingBenchPlayer` via `SpawnConsumable` for portable bench deployment.
- **Notes:** Only 4 graphs and 27 nodes — very lean. All the deployable object logic lives in the spawned world actor, not here.

---

## BP_AnimalAttackAttachment
- **Type:** Blueprint
- **Parent:** Actor *(NOT in the pickup tree)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/PickUpItems/BP_AnimalAttackAttachment`
- **Purpose:** Invisible melee damage zone attached to animal/creature character meshes — drives hit detection for creature bite/claw attacks using a BoxComponent + WeaponCollision component; analogous to a weapon BP for non-humanoid enemies.
- **Key Variables:**
  - `CharacterRef` (object) — owning creature character reference
  - `EquipmentComponent` (object) — cached ref to creature's equipment component
  - `Damage Type` (byte) — damage type enum for this creature's attacks
  - `WeaponBaseDamage` (real) — base damage value per hit
  - `Special Damage Info` (struct) — additional damage properties (status effects, knockback)
  - `HitReady?` (bool) — whether the attack window is open (set by anim notify)
  - `AlreadyDamagedEnemy` (object) — tracks last hit actor to prevent multi-hit per swing
  - `NumberOfWeaponAttacks` (int) — attack count tracker for combo/multi-hit patterns
  - `DmgTextColour` (struct) — colour of damage numbers for this creature
  - `DmgTextFontSize` (int) — size of damage number text
- **Key Components:**
  - `DamageArea` : BoxComponent — the collision box for melee hit detection
  - `WeaponCollision` : WeaponCollision_C — the damage processing component (same as used in weapon BPs)
  - `DefaultSceneRoot` : SceneComponent
- **Key Functions/Events:**
  - `AttachDamageAttachment` — attaches this actor to the specified socket on the creature mesh at spawn
  - `LineTraceHit` — performs line trace from the box to confirm solid hit and determine hit actor
- **GodOfRuin Use:** **USE** — attach to animal enemies (wolves, panthers, boars) for melee damage zones. Configure `Damage Type`, `WeaponBaseDamage`, and `Special Damage Info`. Set `HitReady?` via `AN_WeaponHitToggle` on the creature's animation montages.
- **Notes:**
  - `AlreadyDamagedEnemy` prevents the box from hitting the same target multiple times per attack swing — reset it in `LineTraceHit` logic or on attack end.
  - This is a standalone Actor, not a PickupItem child — it is spawned at runtime and attached, not placed in the level.

---

## BP_Fists
- **Type:** Blueprint
- **Parent:** Actor *(NOT in the pickup tree)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/PickUpItems/BP_Fists`
- **Purpose:** Unarmed combat damage actor — two BoxComponent fists attached to character hand sockets, providing hit detection for punch/kick attacks when no weapon is equipped.
- **Key Variables:**
  - `CharacterRef` (object) — owning character reference
  - `EquipmentComponent` (object) — cached equipment component ref
  - `WeaponBaseDamage` (real) — base unarmed damage per strike
  - `HitReady?` (bool) — attack window flag; set by anim notify
  - `AlreadyDamagedEnemy` (object) — prevents double-hit per swing
  - `NumberOfWeaponAttacks` (int) — combo hit counter
  - `DmgTextColour` (struct) — damage number colour
  - `DmgTextFontSize` (int) — damage number size
- **Key Components:**
  - `Fist1Ref` / `Fist2Ref` : BoxComponent — left and right hand hit volumes
  - `WeaponCollision` : WeaponCollision_C — shared damage processing component
  - `DefaultSceneRoot` : SceneComponent
- **Key Functions/Events:**
  - `AttachFists` — attaches this actor's two box components to the character's hand sockets
  - `LineTraceHit` — hit confirmation trace
- **GodOfRuin Use:** **USE** — equip on unarmed Paragon characters (brawlers, martial artists). `EquipmentComponent` spawns and equips BP_Fists automatically when `Weapon Slot` has no weapon assigned. Configure `WeaponBaseDamage` for base unarmed damage.
- **Notes:** Structurally mirrors `BP_AnimalAttackAttachment` but with two collision boxes instead of one. Both parent as standalone Actors to keep them completely separate from the pickup/inventory system.

---

## BP_ArrowParent
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/Arrows/BP_ArrowParent`
- **Purpose:** Base class for all fired arrow projectiles — handles flight physics via ProjectileMovementComponent, impact collision, and houses the VFX/sound components shared by all arrow types.
- **Key Variables:**
  - `DamageType` (byte) — the damage type this arrow deals on impact; read by `TakingDamageComponent` on the target
  - `CombatStatusComp` (object) — cached reference to the archer's `CombatStatusComponent` for damage calculation
- **Key Components:**
  - `Projectile` : ProjectileMovementComponent — handles velocity, gravity, and trajectory
  - `CollisionComponent` : SphereComponent — impact detection
  - `Arrow` : StaticMeshComponent — the arrow shaft/head mesh
  - `ArrowGrabPoint` : SceneComponent — socket for the notch/grab point (used by bow string attachment)
  - `ArrowComponent` : ArrowComponent — debug direction indicator
  - `ArrowVFX` : NiagaraComponent — in-flight VFX (fire trail, magic glow, etc.)
  - `DynamiteSM` : StaticMeshComponent — secondary mesh for explosive arrows (the dynamite bundle)
  - `RadialForce` : RadialForceComponent — AoE physics impulse on explosive impact
  - `ExecuteCamLocation` : SceneComponent — camera point for execution-via-arrow scenarios
  - `ArrowExeCamera` : CameraComponent — execution camera
- **GodOfRuin Use:** **EXTEND** — all 9 typed arrows extend this. To create a custom arrow type (holy arrow, gravity arrow), create a child, set `DamageType`, configure `ArrowVFX`, and override `EventGraph` logic for special impact behavior.
- **Notes:**
  - `ArrowVFX` is always present but may be empty in shells like `BP_SharpArrow` — only activate it in types that need in-flight effects.
  - `RadialForce` is only relevant for `BP_ExplosiveArrow` — it is inactive on all other types unless explicitly triggered.
  - Arrow spawning is handled by `AN_FireArrow` anim notify → `RangedComponent` → arrow BP spawn. Never route through `AN_RangedAbilityFire` (that path is for `BP_AbilityRanged`, not arrows).

---

## Arrow Leaf BPs

All 9 arrow leaf BPs extend `BP_ArrowParent_C`. Unless noted, they are identity BPs — all behavior from the parent. Create children of these for custom variants.

| Blueprint | Path | Own Vars | Special Feature | GodOfRuin Use |
|-----------|------|----------|-----------------|---------------|
| `BP_SharpArrow` | `.../Arrows/BP_SharpArrow` | 0 | Basic pierce — no status | **USE** — default arrow type |
| `BP_FireArrow` | `.../Arrows/BP_FireArrow` | 0 | Fire VFX via `ArrowVFX` Niagara | **USE** — assign fire Niagara asset |
| `BP_UltimateArrow` | `.../Arrows/BP_UltimateArrow` | 0 | Extra `ParticleSystemComponent` for legacy particle effects | **USE** or replace with Niagara |
| `BP_ExplosiveArrow` | `.../Arrows/BP_ExplosiveArrow` | 2 (`HitEnemy`, `OverlappingActorsArray`) | AoE impact: uses `RadialForce` + sphere overlap to damage all nearby actors | **USE** — configure AoE radius in `RadialForce` |
| `BP_IceArrow` | `.../Arrows/BP_IceArrow` | 2 (`HitEnemy`, `OverlappingActorsArray`) | Ice/freeze status on impact | **USE** — ensure ice status type is in `CombatStatusComponent` |
| `BP_FrostArrow` | `.../Arrows/BP_FrostArrow` | 2 (`HitEnemy`, `OverlappingActorsArray`) | Frost slow status on impact | **USE** |
| `BP_PoisonArrow` | `.../Arrows/BP_PoisonArrow` | 2 (`HitEnemy`, `OverlappingActorsArray`) | Poison DoT status on impact | **USE** |
| `BP_MadnessArrow` | `.../Arrows/BP_MadnessArrow` | 2 (`HitEnemy`, `OverlappingActorsArray`) | Madness/confusion status | **USE** |
| `BP_LightningArrow` | `.../Arrows/BP_LightningArrow` | 13 | **Chain lightning** — arcs between up to 3 targets; uses Niagara lightning beams with `BeamStart`/`BeamEnd` targeting | **USE** — most complex arrow; review EventGraph before placing in levels |
| `BP_RainOfArrows` | `.../Arrows/BP_RainOfArrows` | 0 | Triggers arrow barrage at impact point | **USE** — configure barrage count/spread in EventGraph |

**Notes on status arrows:** `HitEnemy?` prevents the status from applying twice to the same target per arrow. `OverlappingActorsArray` stores the AoE hit list. Status effect magnitude is determined by `DamageType` byte passed to `CombatStatusComponent` on the target.

---

## BP_LootActorParent
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_LootActorParent`
- **Purpose:** Base class for world loot drop actors — owns `LootComponent` which manages the loot table, handles player approach detection via Sphere trigger, and drives the pick-up interaction outline; placed in levels or spawned on enemy death.
- **Key Variables:**
  - `LootSetupInfo` (struct) — the loot table config passed to `LootComponent` (item types, rarity weights, quantity range)
  - `Loot State` (byte, replicated) — current loot state enum (idle, being looted, empty); `OnRep_Loot State` updates visuals
  - `DroppedLoot` (bool, replicated) — true when this was spawned as an enemy death drop vs. a pre-placed pickup; `OnRep_DroppedLoot` adjusts positioning
  - `CharacterRef` (object) — player reference cached on interaction start
- **Key Components:**
  - `Item` / `Item2` : StaticMeshComponent — primary and secondary item meshes (set in child BPs)
  - `LootComponent` : LootComponent_C — manages the loot table and item generation
  - `OutlineComponent` : OutlineComponent_C — interact highlight
  - `Sphere` : SphereComponent — player proximity trigger (initiates interact prompt)
  - `Scene` : SceneComponent — pivot for mesh offset
  - `DefaultSceneRoot` : SceneComponent
- **Key Functions/Events:**
  - `InitializeDroppedLoot` — called when this actor is spawned as a death drop; sets `DroppedLoot = true` and initializes `LootSetupInfo` from the enemy's loot table
  - `OnRep_DroppedLoot` — adjusts mesh position/rotation for dropped-on-ground presentation
  - `OnRep_Loot State` — updates mesh visibility or open/closed state based on loot state enum
  - `BP_LootActorParent_AutoGenFunc` — auto-generated replication function
- **GodOfRuin Use:** **EXTEND** — use child BPs `BP_LootActorAnim` or `BP_LootActorSetMesh` for all loot actors. Extend directly only to create a new variant (e.g., a loot bag with cloth physics). Configure `LootSetupInfo` per enemy or chest type.
- **Notes:**
  - `LootComponent` drives what items are actually generated — `LootSetupInfo` is the config fed to it. The actual item BP classes come from the loot table data asset, not from this BP.
  - `DroppedLoot` is the key flag for runtime-spawned vs pre-placed actors; pre-placed actors leave this false.

---

## BP_ChestBase
- **Type:** Blueprint
- **Parent:** BP_LootActorParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_ChestBase`
- **Purpose:** Chest loot actor variant — adds chest-specific properties (type, XP reward, stat type, weapon slot) to the base loot actor for differentiated chest encounters.
- **Key Variables:**
  - `ChestType` (int) — chest category (wooden, iron, golden, boss)
  - `XPAmount` (int) — XP awarded to the player on opening
  - `StatType` (int) — stat bonus type granted on opening (strength, dexterity, etc.)
  - `WeaponSlot` (int) — if non-zero, guarantees a weapon in this slot tier from the loot table
- **GodOfRuin Use:** **USE** — place in levels for treasure rooms, dungeon rewards, and secret areas. Set `ChestType` to control mesh appearance (via `LootSetupInfo` → `LootComponent`), set `XPAmount` for progression reward, configure `LootSetupInfo` with the chest's item pool.
- **Notes:** Only 2 graphs (EventGraph + UserConstructionScript) — all opening and loot logic inherited from `BP_LootActorParent` and `LootComponent`.

---

## BP_LootActorAnim
- **Type:** Blueprint
- **Parent:** BP_LootActorParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_LootActorAnim`
- **Purpose:** Animated loot actor variant — adds `RetrieveOutlineInfo` for the interact highlight and a `Relative Rotation` variable to control the open animation orientation.
- **Key Variables:**
  - `Relative Rotation` (struct) — rotation offset for the animated lid/flap opening
- **Key Functions/Events:**
  - `RetrieveOutlineInfo` — passes mesh refs to `OutlineComponent` for interact highlight
- **GodOfRuin Use:** **USE** — use for chests/crates with an open animation (skeletal mesh with open anim). Configure `Relative Rotation` to match the animation's final open position to avoid a snap.
- **Notes:** Use `BP_LootActorAnim` when the item has an open/close animation; use `BP_LootActorSetMesh` when it is a static prop.

---

## BP_LootActorSetMesh
- **Type:** Blueprint
- **Parent:** BP_LootActorParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_LootActorSetMesh`
- **Purpose:** Static mesh loot actor variant — allows the loot prop mesh to be set at runtime via variables rather than hard-referencing a mesh in the BP.
- **Key Variables:**
  - `LootMesh` (object) — the static mesh asset to assign to the `Item` component
  - `LootMeshScale` (real) — uniform scale applied to the mesh
- **Key Functions/Events:**
  - `RetrieveOutlineInfo` — passes mesh refs to `OutlineComponent`
- **GodOfRuin Use:** **USE** — preferred for enemy death drops where the dropped item mesh should match the item type (e.g., a sword mesh for a weapon drop). Pass `LootMesh` and `LootMeshScale` via `InitializeDroppedLoot` or a custom init call.
- **Notes:** More flexible than `BP_LootActorAnim` for procedural loot drops since the mesh is a variable not a hard-reference.

---

## BP_ContainerParent
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/Containers/BP_ContainerParent`
- **Purpose:** Base class for world containers (chests, crates, barrels) with their own `InventoryComponent` — distinct from `BP_LootActorParent` in that containers have persistent, browsable inventory rather than immediate item generation.
- **Key Variables:**
  - `ContainerName` (text) — displayed name in the loot UI
  - `ContainerSize` (int) — number of inventory slots this container has
  - `ContainerRarity` (byte) — container tier (common, uncommon, rare, epic, legendary) affects auto-generated loot quality
  - `ContainerType` (byte) — container category (chest, barrel, crate, corpse)
  - `ContainerImage` (object) — UI texture displayed in the container inventory screen
  - `ContainerInventory` (struct) — the actual inventory data (items + quantities)
  - `ContainerDataTable` (object) — DataTable asset for guaranteed loot; used when `UseDataTable? = true`
  - `UseDataTable?` (bool) — if true, populates container from `ContainerDataTable` instead of random generation
  - `GuaranteedDTLoot` (bool) — if true + DataTable, all DataTable rows are guaranteed drops (no RNG roll)
  - `Open` (bool, replicated) — container open state; `OnRep_Open` plays open animation/sound
  - `TrackingInfo` (struct) — minimap marker config fed to `MapTrackerComponent`
  - `PlayerRef` / `Character` (object) — player refs cached on interact
  - `CharacterMesh` (object) — player mesh ref (for animation during loot screen)
- **Key Components:**
  - `Item` / `Item2` : StaticMeshComponent — container meshes
  - `Sphere` : SphereComponent — interact proximity trigger
  - `InventoryComponent` : InventoryComponent_C — the container's own inventory
  - `OutlineComponent` : OutlineComponent_C — interact highlight
  - `MapTrackerComponent` : MapTrackingComponent_C — minimap marker
  - `Scene` : SceneComponent — mesh pivot
  - `DefaultSceneRoot` : SceneComponent
- **Key Functions/Events:**
  - `OnRep_Open` — plays open animation/SFX and opens the loot UI on clients when `Open` flips true
  - `IsQuestContainer` — returns true if this container is linked to a quest objective; gates loot availability
- **GodOfRuin Use:** **EXTEND** — use child BPs `BP_ContainerSetMesh` or `BP_ContainerChest` for all placed containers. Configure `ContainerSize`, `ContainerRarity`, `ContainerName`, and either set `ContainerInventory` manually or set `UseDataTable?` + `ContainerDataTable` for data-driven loot. Set `TrackingInfo` to add minimap markers for important chests.
- **Notes:**
  - Key difference from `BP_LootActorParent`: containers use `InventoryComponent` (browsable, persistent) vs `LootComponent` (one-shot item generation). Use containers for named chests the player can fully browse; use LootActors for enemy drops.
  - `GuaranteedDTLoot = true` is useful for quest reward chests that must always contain specific items.
  - `MapTrackerComponent` — set `TrackingInfo` in editor for any container the player should see on the minimap (e.g., boss chests).

---

## BP_ContainerSetMesh
- **Type:** Blueprint
- **Parent:** BP_ContainerParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/Containers/BP_ContainerSetMesh`
- **Purpose:** Container variant where the mesh is set at runtime via variables — allows one BP to represent any container visual.
- **Key Variables:**
  - `ContainerMesh` (object) — static mesh asset to assign to `Item`
  - `ContainerScale` (struct) — scale vector for the mesh
- **Key Functions/Events:**
  - `RetrieveOutlineInfo` — passes mesh to `OutlineComponent`
- **GodOfRuin Use:** **USE** — preferred for procedurally placed containers or when many container variants share the same logic but different meshes.
- **Notes:** 3 graphs only — lean shell. All logic inherited.

---

## BP_ContainerChest
- **Type:** Blueprint
- **Parent:** BP_ContainerParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Items/Containers/BP_ContainerChest`
- **Purpose:** Animated chest container — adds `RetrieveLootInfo` to feed chest mesh data to the loot system, and `Relative Rotation` for the lid open animation.
- **Key Variables:**
  - `Relative Rotation` (struct) — lid open rotation offset
- **Key Functions/Events:**
  - `RetrieveLootInfo` — passes container info (name, image, inventory) to the loot UI on open
- **GodOfRuin Use:** **USE** — standard chest BP for all named treasure chests with open animations. Configure all `BP_ContainerParent` vars (size, rarity, loot table), then set `Relative Rotation` to match the chest lid open angle.
- **Notes:** This is the go-to container for most chest placement in levels. Use `BP_ContainerSetMesh` for non-chest containers (barrels, crates).
