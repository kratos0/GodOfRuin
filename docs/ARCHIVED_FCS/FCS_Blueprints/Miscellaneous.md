# Miscellaneous — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/`
> Total assets: 55 | Documented: 23 | Skipped (Demo/MainMenu): 32

### Skipped Assets (IGNORE for GodOfRuin)

**DemoLevels/** (21 assets) — training room props and demo level scaffolding; replace entirely with GodOfRuin-specific level design:
`BP_Target`, `BP_UpDownButton`, `BP_RangedPractise`, `BP_FloorButton`, `BP_OpeningWall`, `BP_AnimPreviewDummy`,
`BP_Teleporter`, `BP_TargetingDummy`, `BP_TitleSpawner`, `I_WidgetTrainingRoomInterface`, `BP_TextSpawner`,
`BP_FadeOutToMainMenu`, `BP_AI_DummySpawner-Assass`, `BP_AIDummySpawner`, `BP_AIDummySpawnerSmall`,
`BP_LevelUper`, `BP_ArenaGate`, `BP_LoweringGate`, `BP_ArenaSpawner`, `BP_WarRoomTeleporter`, `BP_TorchLight`

**MainMenu/** (11 assets) — FCS demo main menu characters and camera pawns; GodOfRuin has its own menu system:
`BP_ArenaMapDummy`, `BP_CombatMapDummy`, `BP_MagicCharacter`, `BP_SwimmingMapDummy`, `BP_MeleeCharacter`,
`BP_QuestChar`, `BP_WarMapDummy`, `BP_CharacterCreatorPawn`, `BP_CameraPawn`, `BP_CharacterLevelSwitcher`,
`BP_RangedCharacter`

---

### ActivateActors System Overview

The ActivateActors subfolder implements a **two-part switch-and-target** system for world interactables:

```
Activating Actors (player-side triggers):
  BP_ActivateByTouchParent  ←  player walks up and interacts
    └── BP_SwitchParent
          ├── BP_ActivatorEmpty   ← invisible interact trigger (no mesh)
          └── BP_LevelSwitch      ← lever/switch with mesh

  BP_ButtonSwitch  ←  pressure plate (separate from the touch tree, standalone Actor)

Actors to Activate (world-side respondents):
  Touch-triggered (player direct):
    BP_ActivateByTouchParent
      ├── BP_SpinActorByTouch   ← puzzle wheel / spinning prop
      ├── BP_OpenDoorTouch      ← single-panel door
      └── BP_TouchEmpty         ← blank template

  Message-triggered (called by a switch via ActorToActivate ref):
    BP_ActivateActorParent
      ├── BP_OpenDoorMessage    ← double-panel door
      ├── BP_SpinActorMessage   ← spinning prop
      ├── BP_VFXMessage         ← Niagara effect trigger
      └── BP_MessageEmpty       ← blank template

  Overlap-triggered (autonomous proximity):
    BP_SlidingDoorEpic          ← standalone sliding double door
```

**Wiring pattern:** Place a `BP_LevelSwitch` (or `BP_ActivatorEmpty`) in the level. Set its `ActorToActivate` to point at a `BP_OpenDoorMessage` (or other message target). Player interacts with the switch → `InteractOn` flips → switch calls Activate on the target → target performs its action.

---

## BP_CooldownManager
- **Type:** Blueprint
- **Parent:** Object *(not Actor — sub-object logic class)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/BP_CooldownManager`
- **Purpose:** Reusable cooldown timer object — tracks a single cooldown duration, exposes `OnCooldownStart` / `OnCooldownEnd` event dispatchers, and can be queried with `IsCooldownActive`; used as a sub-object by ability and skill components.
- **Key Variables:**
  - `CooldownDurationTime` (real) — the cooldown length in seconds
  - `CooldownActive` (bool) — true while the timer is running
  - `OnCooldownStart` (event dispatcher) — fires when cooldown begins; bind UI elements here
  - `OnCooldownEnd` (event dispatcher) — fires when cooldown expires; bind ability re-enable logic here
- **Key Functions/Events:**
  - `IsCooldownActive` — returns `CooldownActive`; call this before allowing ability use
- **GodOfRuin Use:** **USE** — this is the cooldown primitive for all ability and item cooldowns. Do not reinvent cooldown timers; use this. To add a cooldown to a custom ability, create a `BP_CooldownManager` sub-object, set `CooldownDurationTime`, and bind `OnCooldownEnd` to your re-enable logic.
- **Notes:** Parent is `Object` — it is instantiated as a sub-object, not placed in the world. 2 graphs, 16 nodes — very lean by design.

---

## TraversableComponent
- **Type:** Blueprint
- **Parent:** ActorComponent
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/TraversableComponent`
- **Purpose:** Actor component that maps ledge data for vault/climb traversal — finds the nearest ledge to an actor and returns ledge transforms for the FCS traverse/mantle system.
- **Key Variables:**
  - `Ledges` (object) — array of ledge spline/transform references registered to this component
  - `OppositeLedges` (object) — ledge references on the opposite side (for two-sided vault surfaces)
  - `MinLedgeWidth` (real) — minimum width threshold before a ledge is considered traversable
- **Key Functions/Events:**
  - `FindLedgeClosestToActor` — iterates ledge refs, returns the nearest traversable ledge to the owning actor's position
  - `GetLedgeTransforms` — returns the transform array for a specific ledge (used by the mantle/vault animation system to set warp targets)
- **GodOfRuin Use:** **USE** — add to any world geometry actor (wall, crate, cliff edge) that players should be able to vault over or climb. Populate `Ledges` with the geometry's ledge spline actors. Pairs with `AN_MotionWarping_Attack` / `AttackMotionWarpingComponent` for the vault animation alignment.
- **Notes:** 3 graphs, 96 nodes — moderately complex for a component, suggesting the ledge-finding math is non-trivial. `MinLedgeWidth` is the key tuning variable — too small and the character tries to vault tiny props.

---

## BP_CheckPoint
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/BP_CheckPoint`
- **Purpose:** Save point actor — player walks into the `Box` trigger to save their position and game state; the spinning floppy disk mesh is the visual indicator.
- **Key Variables:**
  - `PlayerLocation` (struct) — the transform saved when the player activates this checkpoint
  - `Load?` (bool) — if true at game start, this checkpoint will load the saved state (used to differentiate the active/most-recent checkpoint)
- **Key Components:**
  - `Box` : BoxComponent — player overlap trigger
  - `FloppyDisk` : StaticMeshComponent — spinning save indicator prop (replace for GodOfRuin with a shrine/bonfire mesh)
  - `RotatingMovement` : RotatingMovementComponent — drives the floppy disk spin
  - `DefaultSceneRoot` : SceneComponent
- **GodOfRuin Use:** **USE** — place in levels as save/respawn points. Swap `FloppyDisk` mesh for a GodOfRuin-appropriate shrine prop. The save call routes through `I_GameMode` interface — confirm `GodOfRuin_GameMode` implements the save interface before placing checkpoints.
- **Notes:** `Load? = true` should only be set on the checkpoint the player most recently activated — manage this flag via the save system to avoid multiple checkpoints all trying to load on game start.

---

## BP_DragDropOperation
- **Type:** Blueprint
- **Parent:** DragDropOperation *(UMG drag-drop data carrier)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/DragDropOperation/BP_DragDropOperation`
- **Purpose:** UMG drag-drop payload — carries all the data needed during an inventory/ability/equipment drag operation (which slot was dragged, what type it is, what it contains) so the drop target knows what to do with it.
- **Key Variables:**
  - `SlotType` (byte) — which type of slot is being dragged (inventory, ability, equipment, ammo, spellbook)
  - `InventorySlotContent` (struct) — item data if dragging from an inventory slot
  - `AbilitySlotContent` (struct) — ability data if dragging from an ability slot
  - `WB_InventorySlot` (object) — reference to the source inventory slot widget
  - `WB_AbilitySlot` (object) — reference to the source ability slot widget
  - `WB_ActionBarSlot` (object) — reference to the source action bar slot widget
  - `SlotLock` (byte) — lock state of the source slot (prevents dropping locked items)
  - `SlotEquipmentLock` (byte) — equipment-specific lock state
  - `InventoryComponent` (object) — the owning character's inventory component ref
  - `AbilitySlot` (bool) — true if the drag source was an ability slot
  - `Ammo Slot` (bool) — true if dragging ammo
  - `SpellBookSlot` (bool) — true if dragging from the spellbook
- **GodOfRuin Use:** **USE** — this is created automatically by the FCS inventory UI widgets during drag. Do not create manually; the widget system instantiates it. If extending the UI with new slot types, add new variables here for the new payload data.
- **Notes:** Single graph only (EventGraph). All logic lives in the widget BPs that create and consume this operation — this is purely a data carrier.

---

## BP_LocationTrigger
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/BP_LocationTrigger`
- **Purpose:** Zone trigger that fires a location-entry event — shows a location name on screen, optionally logs a quest objective when the player enters the `Box` volume, and displays a minimap marker.
- **Key Variables:**
  - `LocationName` (text) — the display name shown on the HUD when the player enters (e.g. "Ironhold Mines")
  - `QuestObjective?` (bool) — if true, entering this zone advances a linked quest objective
  - `ObjectiveTexture` (object) — icon shown in the zone entry notification
  - `ObjectiveTextureSize` (struct) — display size of the objective icon
  - `ObjectiveTrackingType` (byte) — how the quest tracker registers this objective
  - `ObjectiveTrackDistance` (real) — distance at which the minimap marker appears
- **Key Components:**
  - `Box` : BoxComponent — the zone trigger volume
  - `MapTrackerComponent` : MapTrackingComponent_C — minimap marker for this location
  - `Billboard` : BillboardComponent — editor placement indicator
  - `DefaultSceneRoot` : SceneComponent
- **Key Functions/Events:**
  - `PassObjectiveData` — sends objective data to the quest system when the player enters the zone
- **GodOfRuin Use:** **USE** — place at dungeon entrances, area transitions, and key locations. Set `LocationName` for all named areas. Set `QuestObjective? = true` and configure `ObjectiveTrackingType` for zones that are quest destinations.
- **Notes:** The `Box` size determines the exact trigger zone — scale it to match the geography of the entrance or area boundary.

---

## BP_CharacterInventory
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/BP_CharacterInventory`
- **Purpose:** 3D character preview actor for the inventory/equipment screen — renders a full character model with all equipment slots visible, driven by a `SceneCaptureComponent2D` for the inventory UI portrait; this is the world-space character mannequin shown when the player opens their equipment screen.
- **Key Variables:**
  - `CharacterRef` (object) — the player character this preview mirrors
  - `EquipmentComponent` (object) — cached ref to read equipped item meshes from
  - `CombatStatusComponent` (object) — cached ref for status-driven visual changes
  - `CharacterOriginalBody` (object) — the base body mesh reference (without armor)
  - `MovingLocation` (struct) — off-screen spawn location for the preview actor
  - `StartingCameraTransform` (struct) — initial camera position for the equipment view
  - `CameraSnapTransition` (bool) — whether to smoothly transition to the inventory camera
- **Key Components:**
  - `SkeletalMesh` : SkeletalMeshComponent — base character body
  - `Shoulder`, `Back`, `Hands`, `Legs`, `Boots`, `Chest`, `Head-Constant`, `Helmet`, `Hair` : SkeletalMeshComponent — individual armor/equipment layer meshes (one per equipment slot)
  - `SceneCaptureComponent2D` : SceneCaptureComponent2D — renders the 3D preview to a render texture for the UI portrait
  - `CameraPositionQuest`, `CameraPositionSkills`, `CameraPositionMap`, `CameraPositionCrafting` : CameraComponent — pre-set camera angles for each inventory sub-screen
  - `RectLight` / `RectLight1` : RectLightComponent — studio lighting for the character preview
  - `DefaultSceneRoot` / `MeshPlaceholder` : SceneComponent
- **Key Functions/Events:**
  - `SetArmor` / `SetArmorPiece` — updates the appropriate armor SkeletalMeshComponent when equipment changes
  - `SetWeapons` — updates weapon visuals on the preview model
  - `SetDefaultMeshes` — resets all mesh slots to the character's base appearance
  - `AddCustomCharacterDefaults` — applies GodOfRuin-specific default appearance overrides (if any)
  - `AdjustLighting` — tweaks `RectLight` intensity/colour for the current inventory sub-screen
  - `AdjustHeightForBoots` — offsets the preview model when tall boot armor changes the character height
- **GodOfRuin Use:** **USE** — spawned automatically by `EquipmentComponent` when the player opens their inventory. The 9 individual armor mesh components map directly to FCS equipment slots — ensure Paragon character skeletal meshes support the same socket names for correct equipment attachment. The 4 camera positions may need repositioning for GodOfRuin's character proportions.
- **Notes:**
  - `SceneCaptureComponent2D` renders the preview to a `RenderTarget` asset referenced by the inventory UI widget — if the portrait appears black, check the render target is assigned and the capture component is pointing at the character.
  - This actor is spawned at `MovingLocation` (off-camera) and only the SceneCapture output is shown in the UI — the actor itself is never visible in the game world.
  - `AdjustHeightForBoots` is a nice detail — if Paragon characters have tall armor, this function may need tuning per character.

---

## BP_MapChanger
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/BP_MapChanger`
- **Purpose:** Zone trigger that changes the active map when the player enters — uses a DataTable row name to look up the destination map rather than a hard-coded level name string.
- **Key Variables:**
  - `MapName (MapData DT)` (name) — row key into the MapData DataTable; the DataTable row defines the actual level asset to load
- **Key Components:**
  - `Box` : BoxComponent — trigger volume
  - `Billboard` : BillboardComponent — editor indicator
  - `DefaultSceneRoot` : SceneComponent
- **GodOfRuin Use:** **USE** — place at level transitions. The `MapData` DataTable must exist and contain a row for your destination. For GodOfRuin, populate the MapData DT with all world areas and use `BP_MapChanger` at every zone boundary. Differs from `BP_ChangeLevel` in that it uses a DataTable registry rather than a raw level name.
- **Notes:** The variable name `MapName (MapData DT)` is a designer hint — the `(MapData DT)` suffix indicates it must match a row in the `MapData` DataTable asset. Find that DataTable to see all registered maps.

---

## BP_QuestTrigger
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/BP_QuestTrigger`
- **Purpose:** World actor that offers one or more quests when the player enters its `Box` trigger — owns a `QuestGiverComponent` and a minimap marker, making it a standalone NPC-less quest offer point (used for notice boards, ancient ruins, discovered locations).
- **Key Variables:**
  - `QuestsToOffer` (struct) — the quest data struct(s) defining which quests this trigger offers
  - `TrackingInfo` (struct) — minimap marker config (icon, range)
  - `Actor to Activate` (object) — optional linked actor to activate when quests are offered (e.g., open a door, start a cutscene)
- **Key Components:**
  - `QuestGiverComponent` : QuestGiverComponent_C — manages the quest offer/accept logic
  - `MapTrackerComponent` : MapTrackingComponent_C — minimap marker
  - `Box` : BoxComponent — player proximity trigger
  - `Billboard` : BillboardComponent — editor indicator
  - `DefaultSceneRoot` : SceneComponent
- **GodOfRuin Use:** **USE** — place for environmental quest triggers (discover a body → receive investigation quest, find ancient text → unlock lore quest). Set `QuestsToOffer` with GodOfRuin quest data. `Actor to Activate` is useful for chaining with the ActivateActors system (e.g., quest trigger opens a gate).
- **Notes:** This offers quests without a dialogue NPC — if the quest requires a conversation, use the FCS dialogue system and a character BP with `QuestGiverComponent` instead.

---

## BP_ChangeLevel
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/BP_ChangeLevel`
- **Purpose:** Direct level transition trigger — loads a level by raw name string when the player enters the overlap box; supports spawning at a PlayerStart or at a specific transform.
- **Key Variables:**
  - `Level Name` (name) — the exact level asset name to load (must match exactly)
  - `Spawn at Player Start` (bool) — if true, player spawns at the default PlayerStart in the destination level
  - `Spawn Specific Transform` (struct) — if `Spawn at Player Start = false`, player spawns at this transform
- **Key Components:**
  - `ChangeOverlap` : BoxComponent — transition trigger volume
  - `TextRender` : TextRenderComponent — shows `Level Name` in the editor viewport for easy identification
  - `DefaultSceneRoot` : SceneComponent
- **GodOfRuin Use:** **USE** for simple direct transitions; prefer `BP_MapChanger` for DataTable-managed map registries. Use `BP_ChangeLevel` for one-off transitions (intro cutscene to gameplay, tutorial to hub) where a DataTable entry is overkill.
- **Notes:** `Level Name` is a raw name — typos cause silent load failures. `TextRender` showing the level name in the editor is a helpful debug aid; it does not appear in-game.

---

## ActivateActors System — Detailed Documentation

### BP_ActivateActorParent
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/ActivateActors/BP_ActivateActorParent`
- **Purpose:** Root base class for all activatable world actors — defines the activation state, quest linkage, single-use guard, and save integration; both Switch (trigger) and Target (respondent) BPs derive from this.
- **Key Variables:**
  - `CanBeActivated` (bool) — master enable flag; if false, all activation attempts are ignored
  - `SingleActivate` (bool) — if true, actor can only be activated once; `CanBeActivated` is set to false after first use
  - `AnimationUnderway` (bool) — set true while an activation animation plays; prevents re-activation mid-animation
  - `SaveActivated` (bool) — if true, activated state is persisted to save data
  - `IsQuestActor` (bool) — if true, activation feeds back to the quest system
  - `ActivateActorName` (text) — identifier used by the quest system to track this actor's state
- **Key Functions/Events:**
  - `SetupObjectiveLink` — registers this actor with the quest system if `IsQuestActor = true`
  - `PassQuestObjectiveData` — sends quest state update when activated
  - `IsQuestObjectiveCheck` — guard function; validates quest linkage before activation
- **GodOfRuin Use:** **EXTEND** — all interactive world props should descend from this (doors, levers, chests, puzzles). The `SingleActivate` + `SaveActivated` combination is critical for non-repeatable events — set both to true for boss gates, story triggers, and reward chests.
- **Notes:** `AnimationUnderway` is the most commonly missed variable — if a door opens too quickly or re-activates during animation, check that this flag is set and cleared correctly in the child BP's `OnRep_InteractComplete`.

---

### BP_ActivateByTouchParent
- **Type:** Blueprint
- **Parent:** BP_ActivateActorParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/ActivateActors/BP_ActivateByTouchParent`
- **Purpose:** Adds player-proximity detection to the base activatable actor — the player approaches the `Box` trigger and can then interact (or the touch itself activates); all touch-activated objects and interact switches extend this.
- **Key Variables:**
  - `Mesh 1` / `Mesh 2` (object) — the two mesh references passed to `OutlineComponent` for the interact highlight
- **Key Components:**
  - `Box` : BoxComponent — player proximity/interact trigger
  - `OutlineComponent` : OutlineComponent_C — interact highlight glow
- **GodOfRuin Use:** **EXTEND** — do not place directly; use `BP_SwitchParent` children for interact switches, or the Touch subfolder BPs for touch-activated props. Extend this directly only for a new category of touch-activated object.

---

### BP_SwitchParent
- **Type:** Blueprint
- **Parent:** BP_ActivateByTouchParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/ActivateActors/ActivatingActors(Switch)/BP_SwitchParent`
- **Purpose:** Interact-switch base — adds the `ActorToActivate` reference and `InteractOn` replicated bool; when the player interacts, `InteractOn` flips and sends an activation message to `ActorToActivate`.
- **Key Variables:**
  - `ActorToActivate` (object) — the world actor this switch controls; must be a `BP_ActivateActorParent` child
  - `Character` (object) — cached reference to the interacting player character
  - `InteractOn` (bool, replicated) — the interact trigger state; `OnRep_InteractOn` fires the target activation
- **Key Functions/Events:**
  - `RetrieveOutlineInfo` — passes mesh to OutlineComponent for the interact highlight
  - `OnRep_InteractOn` — fires on all clients when the switch is triggered; calls Activate on `ActorToActivate`
- **GodOfRuin Use:** **EXTEND** — use `BP_LevelSwitch` for most switches. Extend `BP_SwitchParent` directly only for a custom switch type with unique behavior beyond `BP_LevelSwitch` (e.g., a switch that requires a key item first).

---

### BP_LevelSwitch
- **Type:** Blueprint
- **Parent:** BP_SwitchParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/ActivateActors/ActivatingActors(Switch)/BP_LevelSwitch`
- **Purpose:** Standard lever/switch prop — a static mesh cube + lever with an interact animation; the go-to switch for dungeon puzzles, gate triggers, and mechanism activators.
- **Key Variables:** None own
- **Key Components:**
  - `Cube` : StaticMeshComponent — switch housing mesh
  - `Lever` : StaticMeshComponent — the lever arm mesh
- **Key Functions/Events:**
  - `MidInteract` — plays the lever-pull animation mid-interaction; fires before `InteractOn` flips
  - `RetrieveOutlineInfo` — passes mesh refs to OutlineComponent
- **GodOfRuin Use:** **USE** — standard dungeon lever. Replace `Cube` and `Lever` static meshes with GodOfRuin art assets. Set `ActorToActivate` to the door/gate/mechanism this switch controls. Set `SingleActivate = true` for one-way switches.

---

### BP_ActivatorEmpty
- **Type:** Blueprint
- **Parent:** BP_SwitchParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/ActivateActors/ActivatingActors(Switch)/BP_ActivatorEmpty`
- **Purpose:** Invisible interact trigger — no mesh, triggers `ActorToActivate` when the player interacts within the `Box` volume; use when the interact point should be invisible (e.g., pressing against a hidden door).
- **Key Variables:** None own
- **GodOfRuin Use:** **USE** — hidden trigger zones, secret passages, and interactable surfaces that don't need a visible switch prop.

---

### BP_ButtonSwitch
- **Type:** Blueprint
- **Parent:** Actor *(standalone, not in the switch inheritance tree)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/ActivateActors/ActivatingActors(Switch)/BP_ButtonSwitch`
- **Purpose:** Pressure plate / floor button — activates `ActorToActivate` when the player (or an object) overlaps the button mesh; optionally requires the player to stay on the button (`StayOnButtonForActivate`).
- **Key Variables:**
  - `ActorToActivate` (object) — the target actor to activate on press
  - `Button Mesh` (object) — the button static mesh asset
  - `Button Material` (object) — material applied when the button is active vs inactive
  - `StayOnButtonForActivate` (bool) — if true, activation only lasts while the player is on the button (pressure plate mode)
  - `Moving` (bool) — tracks whether the button is in its press animation
  - `As Character` (object) — cached character ref on overlap
  - `Relative Location` (struct) — button press offset (how far down the button depresses)
- **GodOfRuin Use:** **USE** — floor pressure plates, weight triggers, timed hold buttons. `StayOnButtonForActivate = true` for puzzles requiring the player to hold position.
- **Notes:** Parent is standalone `Actor`, not `BP_SwitchParent` — this means `SingleActivate` and `SaveActivated` from `BP_ActivateActorParent` are NOT available. If you need save-persistent button state, either reparent this or use `BP_ActivatorEmpty` instead.

---

### Touch-Activated Target BPs

All three extend `BP_ActivateByTouchParent_C` and are activated by the player directly touching/interacting (no switch required).

#### BP_OpenDoorTouch
- **Path:** `.../ActorsToActivate/Touch/BP_OpenDoorTouch`
- **Purpose:** Single-panel door that opens when the player interacts with it directly.
- **Key Variables:** `InteractComplete` (bool, replicated)
- **Key Components:** `DoorLeft` : StaticMeshComponent — the door panel; `Scene` : SceneComponent — door pivot
- **Key Functions/Events:** `OnRep_InteractComplete` — plays door open animation on all clients; `RetrieveOutlineInfo`
- **GodOfRuin Use:** **USE** — simple room doors, storage doors. Swap `DoorLeft` mesh. Pair `SingleActivate = true` for one-way-only doors.

#### BP_SpinActorByTouch
- **Path:** `.../ActorsToActivate/Touch/BP_SpinActorByTouch`
- **Purpose:** Spinning puzzle prop (10 cube segments) activated by player touch.
- **Key Variables:** `InteractComplete` (bool, replicated); `SpinTimer` (struct) — rotation timing
- **Key Components:** `Cube`–`Cube9` : StaticMeshComponent (10 segments)
- **GodOfRuin Use:** **USE/EXTEND** — replace Cube meshes with puzzle piece art. The 10-segment design implies a rotating cipher ring or gear puzzle.

#### BP_TouchEmpty
- **Path:** `.../ActorsToActivate/Touch/BP_TouchEmpty`
- **Purpose:** Blank touch-activated template — no mesh, no action beyond flipping `InteractComplete`.
- **GodOfRuin Use:** **EXTEND** — starting point for any custom touch-activated behavior.

---

### Message-Activated Target BPs

All four extend `BP_ActivateActorParent_C` directly and are activated by a switch sending them an activation message via `ActorToActivate`.

#### BP_OpenDoorMessage
- **Path:** `.../ActorsToActivate/Message/BP_OpenDoorMessage`
- **Purpose:** Double-panel door opened by a remote switch.
- **Key Variables:** `InteractComplete` (bool, replicated); `Opening` (bool) — prevents re-trigger mid-animation
- **Key Components:** `DoorLeft` / `DoorRight` : StaticMeshComponent
- **GodOfRuin Use:** **USE** — gate doors, castle doors, portcullis equivalents. Wire to a `BP_LevelSwitch` via `ActorToActivate`.

#### BP_SpinActorMessage
- **Path:** `.../ActorsToActivate/Message/BP_SpinActorMessage`
- **Purpose:** 10-segment spinning prop triggered by a remote switch — puzzle mechanism activated at a distance.
- **Key Variables:** `InteractComplete` (bool, replicated); `SpinTimer` (struct)
- **GodOfRuin Use:** **USE/EXTEND** — remote-triggered puzzle mechanisms.

#### BP_VFXMessage
- **Path:** `.../ActorsToActivate/Message/BP_VFXMessage`
- **Purpose:** Plays a Niagara VFX system at this actor's location when triggered by a switch.
- **Key Variables:** `Niagara VFX` (object) — the Niagara system asset to spawn; `Activated` (bool)
- **GodOfRuin Use:** **USE** — trigger magic circle reveals, explosion VFX, environmental effects on puzzle solve. Assign `Niagara VFX` in the editor.

#### BP_MessageEmpty
- **Path:** `.../ActorsToActivate/Message/BP_MessageEmpty`
- **Purpose:** Blank message-activated template.
- **GodOfRuin Use:** **EXTEND** — starting point for any custom switch-triggered behavior.

---

### BP_SlidingDoorEpic
- **Type:** Blueprint
- **Parent:** Actor *(standalone — separate from the ActivateActors tree)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Miscellaneous/ActivateActors/ActorsToActivate/Overlap/BP_SlidingDoorEpic`
- **Purpose:** Automatic proximity sliding double-door — opens when any actor overlaps the `DoorTrigger` box and closes when the trigger is empty; no interact required.
- **Key Variables:**
  - `DoorEnabled?` (bool) — master enable; set false to lock the door regardless of proximity
  - `InMotionRight` / `InMotionLeft` (bool) — prevents animation interruption while a panel is moving
- **Key Components:**
  - `Door_R` / `Door_L` : StaticMeshComponent — left and right door panels
  - `DoorTrigger` : BoxComponent — proximity detection volume
  - `DefaultSceneRoot` : SceneComponent
- **Key Functions/Events:**
  - `SetDoorEnabled` — toggles `DoorEnabled?`; call this from a switch or scripted event to lock/unlock the door
  - `Open/Close Door` — the movement logic; interpolates panel positions based on overlap state
- **GodOfRuin Use:** **USE** — auto-opening doors for NPC buildings, taverns, safe zones. Set `DoorEnabled? = false` initially for doors locked until a quest condition is met, then call `SetDoorEnabled(true)` from the quest system.
- **Notes:** This is standalone (not in the `BP_ActivateActorParent` tree) — `SingleActivate` and `SaveActivated` don't apply. It always reopens on proximity unless `DoorEnabled? = false`. For one-way permanent opens, use `BP_OpenDoorMessage` instead.
