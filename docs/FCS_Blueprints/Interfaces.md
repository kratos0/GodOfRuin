# Interfaces — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/Interfaces/`
> Total assets: 25 | All relevant — none skipped

### Key Concept
FCS uses interfaces as the primary cross-system communication layer. Instead of direct object references (which require casting), components and actors call interface messages — the receiver implements whichever functions it needs. This means **any custom Blueprint that participates in FCS systems must implement the correct interfaces**.

### Critical for Paragon Integration — Must Implement

| Interface | Implement on | Why |
|---|---|---|
| `I_TakingDamage` | Every damageable actor | `EventDamage` is the damage entry point |
| `I_Character` | Player character BP | 43 functions — the character contract |
| `I_CharacterAnimBP` | All character ABPs | 30 functions — ABP state update contract |
| `I_Interact` | All interactable actors | `Interact` is called by the interact system |
| `I_AI` | All BP_AI_Human children | 33 functions — AI actor contract |
| `I_AI-ABP` | All AI Animation BPs | `SetupAI-ABP` wires the ABP to FCS |
| `I_WeaponInterface` | All weapon BPs | Weapon event contract |
| `I_OutlineInterface` | All actors with OutlineComponent | `RetrieveOutlineInfo` for highlight system |

---

## CORE CHARACTER

### I_TakingDamage
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_TakingDamage`
- **Purpose:** The universal damage entry point — any actor that can receive damage must implement this interface.
- **Functions:**
  - `EventDamage` — called by `WeaponCollision`, `ArrowComponent`, and ability BPs when a hit is registered; the implementation routes to `TakingDamage.StoreDamageInfo`
- **GodOfRuin Use:** **IMPLEMENT on every damageable actor** — enemies, players, destructibles. Without this, weapon traces hit the actor but damage is silently ignored.
- **Notes:** This is the single most important interface in the system. If an enemy is taking zero damage and `WeaponHitToggle` is firing, the first check is whether this interface is implemented and whether `EventDamage` calls `TakingDamageComponent.StoreDamageInfo`.

---

### I_Character
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_Character`
- **Purpose:** The complete player character contract — 43 functions covering death, movement, companions, revive, camera, input, and UI that components and external actors use to communicate with the character without direct casting.
- **Key Functions:**
  - `SendDeath` — triggers death sequence; called by `TakingDamage` when health reaches zero
  - `IsDead` — death state query; polled by AI and systems before interactions
  - `IsWounded` / `Wounded` — wounded state queries/setters for companion revive flow
  - `IsMoving?` / `IsFacingEnemy?` — state queries used by combat conditions
  - `Initialize Player` — called on spawn to wire up all component references
  - `UpdateMovementMode` / `PassMovementMode` — movement mode change notifications
  - `PassCamera` / `UpdateCamera` / `UpdateCameraSprint` / `UpdateRangedAim` — camera state management
  - `PassPrimaryMesh` / `PassArmorMeshes` / `PassBody` — mesh reference passing to components
  - `PassAnimationSet` — sets the character's animation data asset
  - `PassFactionType` — broadcasts faction to nearby AI
  - `SetDisableControls` — locks/unlocks all input
  - `Toggle Collision to Characters` — toggles capsule collision during cinematics/death
  - `RespawnPlayer` — triggers respawn sequence
  - `ReviveStart` / `ReviveUnderway` / `ReviveFinished` / `ReviveCancelled` — companion revive state machine
  - `CharacterRevived` — cleanup after successful revive
  - `HasCompanion?` / `PassCompanions` / `UpdateCompanion` / `UpdateCompanionWounded` / `CompanionDied` / `DisbandRemoveCompanion` / `RemoveSpawnedCompanion` — companion management
  - `ToggleInventoryCamera!` — switches camera for inventory open/close
  - `UpdateGameplayStyle` / `UpdateControlSensitivity` / `UpdateGamepadControl` — settings application
  - `AllowAimSprint` / `ResolveAimSprintMovementState` — aim-sprint mode logic
  - `ProduceSaveTexture` — generates character portrait for save file
  - `RemoveEnemyHUD` — clears enemy health bar when target dies
  - `FP-ToggleGearVisibility` — first-person gear visibility toggle
  - `PassReplicatedControlRotation` — syncs control rotation to clients
- **GodOfRuin Use:** **IMPLEMENT on player character BP** — every function must be connected. Unimplemented functions silently do nothing, which causes subtle bugs (e.g. `SendDeath` not implemented = character never dies visually even though health reaches zero).
- **Notes:** `Initialize Player` is called once on BeginPlay to distribute component references — it must fire before any combat interaction.

---

### I_CharacterAnimBP
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_CharacterAnimBP`
- **Purpose:** The Animation Blueprint state update contract — 30 functions that components call to push state changes into the ABP without direct casting; covers every animated state in combat, movement, and traversal.
- **Key Functions:**
  - `SetupABP` — initial wiring call; ABP reads component refs on setup
  - `UpdateWeaponDrawn` — syncs weapon draw state to ABP blend tree
  - `UpdateBlock` / `UpdateBowAiming` / `UpdateSpellAiming` / `UpdateSpellChannel` / `UpdateSpellChannelType` — combat state flags
  - `UpdateArrowLoaded` / `UpdateArrowFiring` / `PassArrowLoaded` / `SendArrowExecute` — bow state
  - `UpdateTargetLocked` — lock-on state for strafe blending
  - `UpdateHasEnemyTarget` — enemy detected state for ABP transitions
  - `UpdateCrouching` — crouch blend
  - `Update Movement Stance` — stance change (unarmed/weapon/bow)
  - `UpdateExecuteOccuring` — execution animation trigger
  - `UpdateHandToString` — bow hand-to-string cosmetic state
  - `UpdateFootIK` — enables/disables foot IK
  - `PassBoneScaling` — passes bone scale data for equipment-driven bone scaling
  - `AttackRotate` / `AttackEndRotate` — rotation permission flags during attack animations
  - `BootsEquipped` — signals ABP to adjust foot IK offset for boot height
  - `SwimmingToggle` / `IsSwimmingOnSurface` / `Swimming Up /Down = Diving` / `SwimDeath` / `PassSwimSpeed` / `PassSwimLedgeClimbIK` / `JumpIntoWaterComplete` / `Diving OR Surface Swimming` — full swimming ABP state set
- **GodOfRuin Use:** **IMPLEMENT on every Paragon character's ABP** — this is what connects the ABP to the FCS component system. Each function must update the corresponding ABP variable. Missing implementations = animation states that never change (e.g. weapon always looks undrawn, block pose never activates).
- **Notes:** `SetupABP` is the first call — it runs before `Initialize Player` in some cases. Ensure ABP variable names match what these functions set to avoid silent failures.

---

### I_Interact
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_Interact`
- **Purpose:** Two-function contract for the interact system — confirmed in FCS_SystemReference.md as required on all interactable actors.
- **Functions:**
  - `Interact` — called when the player presses the interact key on this actor; triggers the primary interaction (open chest, start dialogue, use bench)
  - `MidInteract` — called during an ongoing interaction for mid-interaction events (e.g. chest opening partway, dialogue mid-line callbacks)
- **GodOfRuin Use:** **IMPLEMENT on every interactable world actor** — chests, NPCs, crafting benches, pickups, doors. The `OutlineComponent` activates on overlap; `Interact` fires when the player confirms.
- **Notes:** `I_Interact_C` is referenced by name in FCS_SystemReference.md — confirm the interface class name matches when checking interact-eligibility conditions.

---

### I_MontageCancel
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_MontageCancel`
- **Purpose:** Single-function interface for stopping montages on simulated proxies in multiplayer — prevents ghost animations on other clients.
- **Functions:**
  - `SimulatedProxies_StopMontage` — called on simulated proxy characters to stop any playing montage when the server determines it should have ended
- **GodOfRuin Use:** **IMPLEMENT on player and AI character BPs** — required for correct multiplayer animation behaviour. In single-player this is dormant.
- **Notes:** Forgetting this in multiplayer causes animations to keep playing on other players' screens after they've ended server-side.

---

## AI

### I_AI
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_AI`
- **Purpose:** The complete AI actor contract — 33 functions for activating/deactivating AI, managing companions, quest triggers, dialogue, trading, and patrol.
- **Key Functions:**
  - `ActivateAI` / `DeactivateAI` / `ServerActivateAI` — AI on/off with server authority variant
  - `ComponentsSetup` — called when all components are initialised; signals the AI is ready
  - `IsCombatActive` — combat state query
  - `PassAIData` / `Pass-AI-Weapons` — passes setup data to the AI on spawn
  - `PassHealthHUD` — passes health HUD widget reference
  - `PassWarningMesh` / `PassSpline` / `PassQuestMarkers` — environment data passing
  - `PassHasTarget` / `SetHasTarget` — enemy target state
  - `SetupCompanion` / `LoadCompanionInfo` / `LoadDisbandedCompanion` / `DisbandCompanion` — companion lifecycle
  - `DialogueStarted` / `EndConversation` / `EndConversationDamage` / `DialogueEndedServer` — dialogue state
  - `BeginTrading` / `EndTrading` — vendor interaction
  - `BeginTraining` / `EndTraining` — skill trainer interaction
  - `PlayAnimation` — direct animation trigger from external systems
  - `TriggerQuest` — fires quest activation on this NPC
  - `ContinuePatrol` / `StopPatrol` — patrol control
  - `TransitionCamera` — dialogue camera trigger
  - `ActivateActor` / `ActivateActorServer` / `PassActorActivateNo` — world actor chaining
- **GodOfRuin Use:** **IMPLEMENT on all BP_AI_Human children** — `BP_AI_Parent` already implements this; child Blueprints inherit the implementation. For custom NPC types, override specific functions as needed.
- **Notes:** `ActivateAI` vs `ServerActivateAI` — always call `ServerActivateAI` in multiplayer contexts. `ComponentsSetup` is the correct moment to trigger post-init logic on AI actors.

---

### I_AI-ABP
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_AI-ABP`
- **Purpose:** Single-function interface for setting up AI Animation Blueprints.
- **Functions:**
  - `SetupAI-ABP` — called when the AI spawns to wire the ABP to the AI's components; the ABP implementation reads component refs from the owning actor
- **GodOfRuin Use:** **IMPLEMENT on all Paragon enemy ABPs** — without this, the AI ABP never receives its component references and all animated states will be stuck at defaults.
- **Notes:** Called by `BP_AI_Parent.Deactivate Controller & ABP` and setup functions — ensure the ABP implements this before the AI is activated.

---

### I_AIController
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_AIController`
- **Purpose:** The AI Controller ↔ AI Actor communication contract — 27 functions that `BP_AIController` calls on the possessed AI pawn to drive combat decisions, crowd control, strafe, and weapon state.
- **Key Functions:**
  - `PassBehaviour` — pushes the current `AI_Behaviour` enum value from the Blackboard to the character
  - `PassEnemyTarget` — updates the AI's enemy target reference
  - `PassAttackGroup` / `SetAttackGroupCheck` — crowd control grouping (prevents all enemies attacking simultaneously)
  - `OutsideAttackGroup` tracking for BT spacing
  - `BeginAttackTimer` / `End Attack Timer` — attack cadence timing
  - `BlockedAttack` / `BlockBreak` — reacts to block/parry events
  - `PassBlockCD` — block cooldown management
  - `FindNextTarget` — triggers target search when current target dies
  - `IsAttackingFromRange` — query for current attack mode
  - `ToggleStrafe` — activates strafe movement
  - `TriggerDamageSense` — AI "hears/feels" damage and reacts
  - `PassInvestigatingNoise` — noise investigation state
  - `UpdateWeaponDrawn` / `Update Arrow Loaded` — weapon state sync
  - `SpellCharged` / `UpdateSpellCharged` / `UpdateAbilityCast` — magic state
  - `Update Combat Stance` — stance change notification
  - `UpdateCompanionBehaviour` / `UpdateCompanionRef` — companion mode
  - `MadnessActive` / `MadnessOver` / `MadnessTargetDead` — madness elemental state
  - `UpdateCrowdControlled` — stun/freeze state
  - `ResetAll` — full controller state reset
- **GodOfRuin Use:** **IMPLEMENT on BP_AIController** — already implemented in FCS; only relevant if extending the AI controller for custom behaviours.
- **Notes:** `PassAttackGroup` is the crowd control system — it ensures only N enemies attack at once. The group size is configured on `BP_AIController`, not here.

---

## COMBAT

### I_WeaponInterface
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_WeaponInterface`
- **Purpose:** The weapon actor contract — 19 functions covering weapon attachment, attack sequencing, bow string animation, arrow loading/firing, and death cleanup.
- **Key Functions:**
  - `Attach Weapon to Hand` / `Attach Weapon to Back` / `Attach Weapon - Swap Hand` / `AttachWeaponToUniqueBone` / `AttachToInventoryWeaponSwitch` — all weapon socket attachment variants
  - `StartAttack` / `StopAttack` — attack active window signals
  - `PassAttackInfo` — receives damage/type info from `MeleeComponent` before a swing
  - `PassItemInfo` — receives item data on equip
  - `PassWeaponHand` — which hand this weapon is in (main/off)
  - `BowPullString` / `DrawBowString` — bow string cosmetic events
  - `LoadArrow` / `UnloadArrow` / `ArrowLoaded` / `ArrowFired` / `FireArrow` — bow arrow lifecycle
  - `PlayerDied` — cleanup call on owner death
  - `Destroy Self & Attachments` — weapon cleanup
  - `PassWeaponTrailNS` — passes trail Niagara system ref to the weapon
- **GodOfRuin Use:** **IMPLEMENT on all weapon BPs** — weapon Blueprints in Items/ folder implement this; custom GodOfRuin weapons must implement all relevant functions.
- **Notes:** `PassAttackInfo` is called before the swing — the weapon uses this to set up `WeaponCollision` with the correct damage values. Missing this = weapon always deals zero damage regardless of stats.

---

### I_RangedWeaponInterface
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_RangedWeaponInterface`
- **Purpose:** Extended contract for bow/ranged weapons — 10 additional functions covering arrow management, quiver data, and string physics.
- **Key Functions:**
  - `PassArrowToFire` / `PassArrowRef` — passes the spawned arrow actor reference to the bow
  - `Get Arrow Name & Quantity` — queries current arrow type and remaining count
  - `GetBowAttachments` — returns bow mesh attachment points (string, grip, etc.)
  - `GetQuiverData` — returns quiver mesh and socket data
  - `LowerArrowQuantity` — decrements arrow count on fire
  - `PassTotalArrowSlots` — passes how many arrow types the bow supports
  - `PullString` — triggers bow string pull physics
  - `EventResetSingleArrow` — resets one arrow's display state
  - `RespawnArrowsNotify` — respawns cosmetic arrow in quiver after fire
- **GodOfRuin Use:** **IMPLEMENT on bow weapon BPs** — in addition to `I_WeaponInterface`; bow BPs need both interfaces.
- **Notes:** `LowerArrowQuantity` calls back to `InventoryComponent` to consume ammo — ensure arrow items are in inventory before attempting to fire.

---

### I_MeleeComponent
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_MeleeComponent`
- **Purpose:** Query interface for combo state — 8 functions that external systems (ABP, UI, other components) call to read `BP_ComboResolver` state without direct access.
- **Functions:**
  - `TryContinueCombo` — attempts to advance the combo; the main combo-chain call
  - `IsComboFinished` — returns true if the chain has ended
  - `HasAnyStartedCombo` — returns true if any combo is in progress
  - `HasAnyValidCombo` — returns true if the current weapon has any valid combos
  - `HasValidLungeAttack` — checks if a lunge combo exists
  - `HasValidHeavyAttackCombo` — checks if a heavy attack combo exists
  - `HasValidBlockBreakAttack` — checks if a guard-break attack exists
  - `GetAllComboNames` — returns all combo names for the current weapon
- **GodOfRuin Use:** **IMPLEMENT on character BPs** — already on `BP_AI_Parent`; ensures ABPs and UI can query combo state safely.
- **Notes:** `TryContinueCombo` is what `AN_TriggerNextAttack` notify calls — if combos never chain, this function's implementation is the first place to check.

---

### I_DefensiveComponent
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_DefensiveComponent`
- **Purpose:** Block state contract — 4 functions for querying and controlling the defensive state from outside the `DefensiveComponent`.
- **Functions:**
  - `SendBlock` — activates block state; called by input system
  - `ReleaseBlock` — deactivates block; called on input release
  - `ResetBlock` — emergency block reset (e.g. after stagger)
  - `IsBlockUp` — query: returns true if block is currently active
- **GodOfRuin Use:** **IMPLEMENT on character BPs** — `BP_AI_Parent` already implements; AI block decisions in `BT_MixedCombat` call `SendBlock` via this interface.
- **Notes:** `IsBlockUp` is polled frequently by `TakingDamage.CanBlockAttack` — ensure the implementation is a direct read of `DefensiveComponent.BlockUp`, not a complex condition.

---

### I_HitStop
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_HitStop`
- **Purpose:** Hit-stop communication contract between `WeaponCollision` and `HitStopComponent`.
- **Functions:**
  - `NotifyReceivedHit` — called on both attacker and target when a weapon trace lands; triggers the dilation timeline
  - `ResetHitStopCount` — resets the per-attack hit count; called at the start of each new attack swing
- **GodOfRuin Use:** **IMPLEMENT on character BPs** — already on `BP_AI_Parent`; the implementation calls `HitStopComponent.TriggerHitStop` and `HitStopComponent.Internal_ResetHitStopCount`.
- **Notes:** Both attacker and target receive `NotifyReceivedHit` — this is how the attacker "feels" the hit through controller vibration and the slight freeze.

---

## WORLD / ITEMS

### I_ItemInterface
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_ItemInterface`
- **Purpose:** Contract for spawned item actors (throwables, consumables, pickup items) — covers launch physics, collision activation, and component wiring.
- **Functions:**
  - `Spawn & Store` — spawned and stored in hand, not yet active
  - `Spawn & Use` — spawned and immediately used/thrown
  - `LaunchItem` — applies velocity to launch the item as a projectile
  - `ToggleCollision` — enables/disables item collision (used by `AN_ThrowItemCollision`)
  - `ThrowItemCollision` — activates throw collision at the correct animation frame
  - `PassComponents` — passes character component refs to the item actor on spawn
  - `PassWeaponTrailNS` — passes Niagara trail system to the item
- **GodOfRuin Use:** **IMPLEMENT on all throwable/usable item actors** — consumables, grenades, thrown weapons.
- **Notes:** `PassComponents` is called before `LaunchItem` — item needs character refs to calculate throw velocity and damage attribution.

---

### I_LootInterface
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_LootInterface`
- **Purpose:** Loot UI and server-authority contract for loot containers — 10 functions covering opening/closing the loot window, passing inventory data, and server-side container state.
- **Functions:**
  - `ServerOpenContainer` / `ServerCloseContainer` — server RPCs for container state
  - `ServerCloseUI` — server-side UI close
  - `ToggleLootMenu` (via `I_PlayerController`) complement — opens loot window on player
  - `RetrieveLootInfo` / `RetrieveInventoryInfo` — returns loot and inventory data to the requesting UI
  - `PassLooted` / `SetLooted` — marks container as looted
  - `AddLootToDroppedLoot` — adds items to a dropped loot container
  - `ToggleAnimation` / `ToggleNotifyForAnim` — chest open/close animation triggers
- **GodOfRuin Use:** **IMPLEMENT on all loot containers** — enemy bodies, chests, dropped bags. `BP_ChestBase` (FCS_SystemReference.md) implements this; any custom chest must too.
- **Notes:** `ToggleAnimation` drives the chest lid open/close — pair with `ToggleNotifyForAnim` to gate the interact until the animation reaches the right frame.

---

### I_OutlineInterface
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_OutlineInterface`
- **Purpose:** Single-function contract for actors with an `OutlineComponent` — provides mesh references for the outline highlight.
- **Functions:**
  - `RetrieveOutlineInfo` — called by `OutlineComponent` during setup; the implementation passes this actor's mesh components back so the outline system knows what to highlight
- **GodOfRuin Use:** **IMPLEMENT on all actors with OutlineComponent** — the outline won't appear on the correct mesh without this. `BP_CraftingCooking` and `BP_CraftingSmithing` both implement it; any custom interactable should too.
- **Notes:** The implementation is always the same pattern: pass `GetComponentByClass(StaticMeshComponent)` or the specific mesh ref to the `OutlineComponent.Setup Outline Variables` call.

---

### I_MapTracker
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_MapTracker`
- **Purpose:** Single-function contract for actors tracked on the minimap.
- **Functions:**
  - `PassMapTrackingInfo` — passes the actor's `MapTrackingComponent` reference and tracking config to the HUD minimap system
- **GodOfRuin Use:** **IMPLEMENT on any actor that should appear on the minimap** — enemies (via `BP_AI_Parent`), quest objectives, crafting benches.
- **Notes:** Called by `PlayerWorldTriggers.AddMapMarkers` broadcast — timing is correct automatically if the actor binds to that dispatcher on BeginPlay.

---

### I_SwimmingInterface
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_SwimmingInterface`
- **Purpose:** Swimming state query contract for water body actors and the SwimmingComponent.
- **Functions:**
  - `IsSwimming` — returns whether the character is currently swimming
  - `SwimmingInOcean` — returns whether swimming in an open ocean (affects current/drift simulation)
  - `Toggle Swimming Checks` — enables/disables swimming detection (used when entering/exiting water volumes)
- **GodOfRuin Use:** **IMPLEMENT on character BPs** — already on player/AI parents; implementation reads `SwimmingComponent` state vars.
- **Notes:** Water body actors call `Toggle Swimming Checks` on overlap/end-overlap — ensure water volumes in GodOfRuin levels use this pattern.

---

## SYSTEMS

### I_PlayerController
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_PlayerController`
- **Purpose:** The complete player controller UI/HUD contract — 38 functions for toggling every UI panel, passing widget references, and managing HUD state.
- **Key Functions:**
  - `PassHUDRef` / `SetupReferences` — initial HUD wiring
  - `ShowHUD` / `HideHUD` — master HUD visibility
  - `PassPlayerController` — passes controller ref to systems that need it
  - `PassInventory` / `PassPlayerInventoryComp` / `PassPlayerMenu` — inventory UI
  - `Toggle Player Menu` / `ToggleLootMenu` / `ToggleQuestLog` / `ToggleSpellBook` / `ToggleRadialMenu` / `ToggleSkillTrainer` / `TogglePauseMenu` / `ToggleWorldMap` / `ToggleSpecificMap` / `Toggle Exchange Menu` — all UI panel toggles
  - `CloseAllWidgets` / `CloseLootMenu` — bulk close
  - `PassExternalInventory` / `PassExchangeMenu` — vendor/exchange UI
  - `PassQuestLog` / `PassRadialMenu` / `PassSpellBook` / `PassOptionsMenu` / `PassWorldMap` / `PassSkillTrainer` — widget reference distribution
  - `PassSpawnedItems` / `AddSpawnedItemToStore` / `RemoveSpawnedItemFromStore` — spawned item tracking
  - `PassControlInput` / `UpdateControlInput` — input scheme updates
  - `SetCanUpdateWidget` — gates widget updates during transitions
  - `RecieveOptionsMenu` — options data receiver
  - `RespawnPlayer` — triggers respawn from controller
  - `UpdateGameplayStyle` — applies gameplay style settings
- **GodOfRuin Use:** **IMPLEMENT on PlayerController BP** — all UI management routes through here; missing functions mean UI panels that can't open/close.
- **Notes:** Note the typo `RecieveOptionsMenu` (should be `Receive`) — match this exactly when implementing.

---

### I_GameMode
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_GameMode`
- **Purpose:** The complete save system contract — 14 functions covering every save operation: game save, checkpoints, inventory, loot, quests, dialogue, runes, slain AI, and spawn location.
- **Key Functions:**
  - `SaveGame` — full game save trigger
  - `LoadSave` — loads save data on level start
  - `SaveCheckpoint` — saves at a checkpoint (partial save)
  - `UpdateSaveSlot` — switches active save slot
  - `UpdateSpawnLocation` — updates where the player respawns
  - `SaveInventoryData` — serialises inventory state
  - `SaveLootData` — records which containers have been looted
  - `SaveSlainAI` — records which enemies have been killed (for persistence)
  - `SavePickedUpItems` — records picked-up world items
  - `SaveActorActivated` — records which world actors have been triggered
  - `SaveRunePlaced` — records placed rune actors (`BP_PhysicalRune`)
  - `SaveDialogue` / `PassDialogueSaveInfo` — dialogue progress persistence
  - `RestartLevel` — triggers level restart
- **GodOfRuin Use:** **IMPLEMENT on GameMode BP** — FCS_SystemReference.md confirms GodOfRuin extends the FCS save system with `CompletedLevels`, `UnlockedAbilities`, `HasCompanion`, and `SelectedCharacterID`. These additions should be added to the GameMode's implementation of these functions.
- **Notes:** `SaveSlainAI` is how enemy kill persistence works — every slain enemy calls this. For GodOfRuin's wave system, decide whether wave enemies should be persisted or always respawn.

---

### I_TargetingComponent
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_TargetingComponent`
- **Purpose:** Single-function interface for querying the current lock-on target from outside `TargetingComponent`.
- **Functions:**
  - `GetLockedOnTarget` — returns the current `LockedOnTarget` actor reference
- **GodOfRuin Use:** **IMPLEMENT on character BPs** — ability BPs, AI, and UI call this to know what the player is locked onto without casting to the character.
- **Notes:** Simpler to call than casting to the character and then accessing `TargetingComponent` — use this everywhere a lock-on target reference is needed.

---

### I_MovementRotationManager
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_MovementRotationManager`
- **Purpose:** Single-function interface for triggering a rotation mode switch.
- **Functions:**
  - `SwitchRotationMode` — called by components that need to change the character's rotation mode (e.g. entering aim mode switches to camera-facing rotation)
- **GodOfRuin Use:** **IMPLEMENT on character BPs** — implementation calls `MovementRotationStatus.CheckRotationMode`.
- **Notes:** Always use this interface instead of calling `MovementRotationStatus` directly — it decouples the caller from the specific component.

---

### I_WidgetInterface
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_WidgetInterface`
- **Purpose:** Minimal widget contract for enemy health HUD widgets — 4 functions.
- **Functions:**
  - `InitializeEnemyHealth` — sets up the enemy health bar with max health value
  - `UpdateHealth` — updates the health bar display value
  - `ToggleVisibility` — shows/hides the widget
  - `ClosePopup` — closes a popup element within the widget
- **GodOfRuin Use:** **IMPLEMENT on enemy health HUD widget BPs** — if using custom health bars for Paragon enemies, implement these. May be affected by widget crash issues (see KnownIssues.md).
- **Notes:** `InitializeEnemyHealth` is called by `BP_AI_Parent.Initialize Health UI Functionality` — timing depends on component init order.

---

## ANIMATION BRIDGE

### I_ABP-Revive
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_ABP-Revive`
- **Purpose:** Animation Blueprint contract for the companion revive state machine — 5 functions drive the ABP into and out of wounded/revive animations.
- **Functions:**
  - `EventWounded` — triggers the wounded-crawl/down animation in the ABP
  - `EventRevived` — triggers the getting-up animation
  - `RevivingCharacter` — signals the ABP that a revive is being attempted (plays the "being revived" idle)
  - `ReviveFinished` — clears revive state in the ABP
  - `ReviveInterupted` — returns ABP to wounded state if revive was cancelled
- **GodOfRuin Use:** **IMPLEMENT on ABPs for any Paragon character that can be wounded/revived** — companion characters and any player character that supports the revive mechanic.
- **Notes:** Pairs with `I_Character.ReviveStart/Underway/Finished/Cancelled` — the character interface drives the gameplay logic; this interface drives the animations.

---

### I_ABP-InteractionTransform
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Interfaces/I_ABP-InteractionTransform`
- **Purpose:** Passes IK target transforms to the ABP for interaction animations (e.g. hand IK on a chest, crafting bench reach).
- **Functions:**
  - `SetInteractionTransform` — pushes a world-space transform to the ABP for the IK solver to target
  - `GetInteractionTransform` — reads the current IK target transform from the ABP
- **GodOfRuin Use:** **IMPLEMENT on character ABPs** — used during crafting and loot interactions for hand IK alignment. If Paragon animations have IK rigs, hook into this for polished interaction reach.
- **Notes:** `SetInteractionTransform` is called by `CraftingComponent` and `LootComponent` with the bench/container transform — ensure the ABP's IK system reads from the stored transform correctly.
