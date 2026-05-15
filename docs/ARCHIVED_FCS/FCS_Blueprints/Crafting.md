# Crafting — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/Crafting/`
> Total assets: 7 | All relevant — none skipped

### Inheritance Tree

```
BP_CraftingBenchParent  (Actor — ROOT)
├── BP_CraftingCooking       ← cooking fire / pot bench
├── BP_CraftingSmithing      ← blacksmith anvil bench
└── BP_CraftingBenchPlayer   ← portable player-owned bench

BP_CraftingToolParent   (Actor — prop root)
├── BP_BlacksmithHammer      ← hammer prop during smithing animation
└── BP_BlacksmithMetal       ← metal ingot prop during smithing animation
```

---

## BP_CraftingBenchParent
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Crafting/BP_CraftingBenchParent`
- **Purpose:** Base class for all crafting stations — provides the interact outline, minimap marker, player positioning socket, dialogue-style camera, and the crafting data struct passed to `CraftingComponent` on use.
- **Key Variables:**
  - `CraftingInfo` (struct) — the recipes and material requirements for this bench; passed directly to `CraftingComponent.Crafting Data Info` on interact
  - `TrackingInfo` (struct, Setup In Game) — minimap marker config (icon, range) fed to `MapTrackingComponent`
  - `Actor Tag Type` (byte) — interact tag icon type for `OutlineComponent` (determines which prompt icon shows on approach)
  - `CraftingInUse` (bool) — locks the bench while a player is using it; prevents simultaneous access
  - `DestroyAfterUse` (bool) — if true, bench destroys itself after one use (single-use stations)
  - `Character` (object) — reference to the player character currently using the bench
- **Key Components:**
  - `Camera` : CameraComponent — the crafting camera (positioned to frame the animation)
  - `Cube` : StaticMeshComponent — root mesh (replace in child BPs with the actual bench mesh)
  - `PlayerTransform` : SkeletalMeshComponent — invisible mesh used as the player snap-to socket during crafting animation
  - `Box1` : BoxComponent — interact proximity trigger
  - `OutlineComponent` : OutlineComponent_C — interact highlight and floating tag
  - `MapTrackerComponent` : MapTrackingComponent_C — minimap marker
  - `DefaultSceneRoot` : SceneComponent
- **Key Functions/Events:**
  - EventGraph — handles `Box1` overlap → `OutlineComponent` activation → interact trigger → calls `CraftingComponent.InitializeCrafting` on the player
  - UserConstructionScript — sets up `MapTrackerComponent` with `TrackingInfo`
- **GodOfRuin Use:** **EXTEND** — never place this directly; use `BP_CraftingCooking` or `BP_CraftingSmithing`. For a custom bench type (alchemy table, enchanting table), create a new child of this BP, swap the `Cube` mesh, and position `PlayerTransform` + `Camera` to match.
- **Notes:**
  - `PlayerTransform` SkeletalMeshComponent is the snap target — `CraftingComponent.InitializeCrafting` teleports the player to this transform. Position it carefully in the child BP or crafting animations will look misaligned.
  - `CraftingInfo` struct is the only config that varies per bench type — all recipe data lives here.
  - `CraftingInUse` should be set/cleared by `CraftingComponent` — do not manage it manually.

---

## BP_CraftingCooking
- **Type:** Blueprint
- **Parent:** BP_CraftingBenchParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Crafting/BP_CraftingCooking`
- **Purpose:** Cooking station variant — adds ambient fire VFX and cooking audio to the base bench, and implements `RetrieveOutlineInfo` to feed its mesh data to `OutlineComponent`.
- **Key Variables:** None own — all inherited from BP_CraftingBenchParent
- **Key Components:**
  - `Niagara` : NiagaraComponent — fire/smoke VFX on the cooking fire
  - `Audio` : AudioComponent — ambient crackling/cooking sounds
- **Key Functions:**
  - `RetrieveOutlineInfo` — passes this BP's mesh references to `OutlineComponent` for the interact highlight
- **GodOfRuin Use:** **USE** — place in levels for cooking stations; populate `CraftingInfo` with food/potion recipes. The Niagara fire and audio are ambient and always playing — they are not triggered by crafting.
- **Notes:** `Cube` mesh from the parent should be replaced with a cooking fire / campfire mesh in the instance. `PlayerTransform` and `Camera` need repositioning to face the fire correctly.

---

## BP_CraftingSmithing
- **Type:** Blueprint
- **Parent:** BP_CraftingBenchParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Crafting/BP_CraftingSmithing`
- **Purpose:** Blacksmith anvil station variant — adds the anvil and stump static meshes, implements `RetrieveOutlineInfo`, and drives the smithing animation that spawns `BP_BlacksmithHammer` and `BP_BlacksmithMetal` props on the character.
- **Key Variables:** None own — all inherited
- **Key Components:**
  - `anvil_and_stump_Object002` / `anvil_and_stump_Object003` : StaticMeshComponent — the anvil and wooden stump meshes
- **Key Functions:**
  - `RetrieveOutlineInfo` — passes mesh refs to OutlineComponent
- **GodOfRuin Use:** **USE** — place for weapon/armor crafting stations; populate `CraftingInfo` with smithing recipes. The `BP_BlacksmithHammer` and `BP_BlacksmithMetal` tool props are spawned by the character's `CraftingComponent` during the animation, not by this BP.
- **Notes:** `anvil_and_stump` mesh names suggest a specific asset is hard-referenced — verify the mesh is present in the project. If substituting a different anvil, replace both mesh components and re-check `PlayerTransform` positioning.

---

## BP_CraftingBenchPlayer
- **Type:** Blueprint
- **Parent:** BP_CraftingBenchParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Crafting/BP_CraftingBenchPlayer`
- **Purpose:** Portable player-owned crafting bench — a minimal shell (no own variables or components, 4 nodes) that represents a bench the player deploys from their inventory rather than a world-placed station.
- **Key Variables:** None own
- **Key Components:** None own
- **GodOfRuin Use:** **USE** — relevant if portable crafting kits are in scope; deploy via `InventoryComponent.SpawnConsumable` targeting this BP class. `DestroyAfterUse` (inherited) should be set to `true` for one-use portable benches.
- **Notes:** Shell BP — all logic inherited. The portable bench concept implies it's spawned as a world actor at the player's feet, then acts like any other bench. Ensure `PlayerTransform` offset in the parent is tuned for ground-level placement.

---

## BP_CraftingToolParent
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Crafting/BP_CraftingToolParent`
- **Purpose:** Base class for crafting prop actors spawned during animation — attaches to a named socket on the character mesh during crafting and is destroyed when the animation ends.
- **Key Variables:**
  - `Socket Name` (name) — the character socket this prop attaches to (e.g. `"hand_r"` for a hammer)
  - `Character` (object) — reference to the owning character (replicated via `OnRep_Character`)
- **Key Components:**
  - `StaticMesh` : StaticMeshComponent — the tool prop mesh (set in child BPs)
- **Key Functions:**
  - `OnRep_Character` — replication callback: when `Character` is set, attaches this actor to the correct socket on the character mesh
- **GodOfRuin Use:** **EXTEND** — create new child BPs for any custom crafting tools (tongs, ladle, chisel); set `Socket Name` to match the character skeleton socket and assign the `StaticMesh`.
- **Notes:** `CraftingComponent.Attachment1` and `Attachment2` (from the Components doc) hold references to two spawned tool actors — `BP_BlacksmithHammer` fills `Attachment1`, `BP_BlacksmithMetal` fills `Attachment2` for smithing.

---

## BP_BlacksmithHammer
- **Type:** Blueprint
- **Parent:** BP_CraftingToolParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Crafting/BP_BlacksmithHammer`
- **Purpose:** Hammer prop spawned in the character's right hand during smithing animation — cosmetic only.
- **Key Variables:** None own
- **Key Components:** None own (mesh set via parent `StaticMesh` component)
- **GodOfRuin Use:** **USE** — spawned automatically by `CraftingComponent` when smithing begins; no configuration needed. Swap the `StaticMesh` asset if a different hammer model is preferred.
- **Notes:** Shell BP — identity only. All logic inherited from `BP_CraftingToolParent`.

---

## BP_BlacksmithMetal
- **Type:** Blueprint
- **Parent:** BP_CraftingToolParent_C
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Crafting/BP_BlacksmithMetal`
- **Purpose:** Metal ingot prop held in the character's off-hand during smithing animation — cosmetic only.
- **Key Variables:** None own
- **Key Components:** None own
- **GodOfRuin Use:** **USE** — spawned automatically alongside `BP_BlacksmithHammer`; no configuration needed. Swap the `StaticMesh` for a different ingot or material prop.
- **Notes:** Shell BP — identity only.
