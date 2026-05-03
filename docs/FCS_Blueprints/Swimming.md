# Swimming — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/Swimming/`
> Total assets: 5 | All relevant — none skipped

### System Overview

The FCS swimming system layers two concerns:

1. **Water body visuals** — `BP_LakeWater` / `BP_OceanWater` extend Unreal's Water plugin (`WaterBodyLake` / `WaterBodyOcean`) and add a `SurfaceMarker` component so FCS code can query the water surface Z.
2. **Swim mode trigger + buoyancy** — `BP_WaterVolume` is a separate placement actor whose `SwimmingCollision` box fires the character's swim mode transition; `BP_CharacterBuoyancy` is spawned and attached to the character to apply buoyancy physics during swimming.

**Placement pattern:**
- Place `BP_LakeWater` for the visual water surface.
- Place `BP_WaterVolume` aligned over/inside the water volume — its `SwimmingCollision` box drives the enter/exit swim transition.
- `BP_CharacterBuoyancy` is not placed in the level — it is spawned by the character's swim component and destroyed on exit.

---

## BP_WaterVolume
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Swimming/BP_WaterVolume`
- **Purpose:** The swim-mode trigger and visual wrapper — `SwimmingCollision` overlap puts the character into swim mode; `WaterMesh` is the water surface mesh; `PostProcess` applies the underwater screen effect.
- **Key Variables:** None own — configured entirely via components
- **Key Components:**
  - `SwimmingCollision` : BoxComponent — overlap trigger; entering starts swim mode, exiting ends it
  - `WaterMesh` : StaticMeshComponent — the flat water surface mesh (assign a water material with opacity/refraction)
  - `PostProcess` : PostProcessComponent — underwater visual effect (colour grading, depth fog, blur)
  - `SurfaceMarker` : SceneComponent — marks the exact water surface Z for buoyancy calculations
- **GodOfRuin Use:** **USE** — place in every swimmable body of water in GodOfRuin levels. Scale `SwimmingCollision` to fill the lake/river/pool volume. Position `SurfaceMarker` exactly at the water surface Z. Assign a GodOfRuin water material to `WaterMesh`. Tune `PostProcess` for the visual depth/tint of each body of water (dark underground pool vs. bright coastal sea).
- **Notes:**
  - `SwimmingCollision` box size is what determines where the character enters/exits swim mode — if the box is too shallow, the character exits swim mode while still visually in the water.
  - `PostProcess` is always active while the camera is inside the box — if the camera clips into the volume above the surface (on a sloped bank), the underwater effect will incorrectly apply. Offset the box slightly below the surface to prevent this.

---

## BP_CharacterBuoyancy
- **Type:** Blueprint
- **Parent:** Actor
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Swimming/BP_CharacterBuoyancy`
- **Purpose:** Spawned buoyancy actor attached to the swimming character — applies the `BuoyancyComponent` float force to keep the character at the water surface and tracks water level changes.
- **Key Variables:**
  - `WaterBaseLevel` (real) — the Z world position of the water surface; set from `BP_WaterVolume.SurfaceMarker` on spawn
  - `CharacterRef` (object) — the character this buoyancy actor is attached to
  - `PreviousValue` (real) — previous-frame buoyancy value; used to detect surface-level changes (wave bobbing)
- **Key Components:**
  - `Buoyancy` : BuoyancyComponent — Unreal Water plugin component that applies upward force based on water density and submersion depth
  - `Cube` : StaticMeshComponent — debug reference mesh (set to invisible in production; used to visualise the buoyancy attachment point in editor)
- **GodOfRuin Use:** **USE** — spawned automatically by the character's swim system; do not place manually. If buoyancy feels too floaty or the character sinks too deep, adjust the `BuoyancyComponent` parameters (pontoon offset, buoyancy coefficient) on this BP.
- **Notes:**
  - The `Cube` StaticMesh is a debug artifact — verify it is hidden (`bHiddenInGame = true`) before shipping.
  - `WaterBaseLevel` must be set immediately after spawn (before the first tick) for buoyancy to work correctly. If characters appear to clip through the water surface, check the value passed from `SurfaceMarker`.

---

## BP_LakeWater
- **Type:** Blueprint
- **Parent:** WaterBodyLake *(Unreal Water plugin)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Swimming/BP_LakeWater`
- **Purpose:** FCS-wrapped lake water body — extends the Unreal Water plugin's `WaterBodyLake` with a `SurfaceMarker` scene component so the FCS swimming system can query the exact surface Z from this actor.
- **Key Variables:** None own
- **Key Components:**
  - `SurfaceMarker` : SceneComponent — queried by `BP_CharacterBuoyancy` and `BP_WaterVolume` for surface Z position
- **GodOfRuin Use:** **USE** — place for all lake/pond/river bodies in GodOfRuin. Configure the `WaterBodyLake` water material, wave settings, and spline shape as normal UE Water plugin workflow. Always pair with a `BP_WaterVolume` placed at the same water level for the swim trigger.
- **Notes:** This replaces the raw `WaterBodyLake` actor — always use `BP_LakeWater` in GodOfRuin levels so the `SurfaceMarker` is available for the FCS buoyancy queries. Using raw `WaterBodyLake` will break buoyancy positioning.

---

## BP_OceanWater
- **Type:** Blueprint
- **Parent:** WaterBodyOcean *(Unreal Water plugin)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Swimming/BP_OceanWater`
- **Purpose:** FCS-wrapped ocean water body — extends `WaterBodyOcean` with `SurfaceMarker` and a `CheckIF OnShore` function that detects when the character has reached dry land (used to auto-exit swim mode at coastlines without requiring a manual `SwimmingCollision` exit).
- **Key Variables:** None own
- **Key Components:**
  - `SurfaceMarker` : SceneComponent — water surface Z reference
- **Key Functions/Events:**
  - `CheckIF OnShore` — line traces or landscape queries to determine if the character's position is above water level (on a beach/shore); triggers swim-mode exit when true
- **GodOfRuin Use:** **USE** — place for ocean/sea bodies. `CheckIF OnShore` is particularly important for open-water coastlines where a `BP_WaterVolume` box exit edge would be impractical. Tune the shore-detection threshold in `CheckIF OnShore` to match GodOfRuin's coastal terrain slope.
- **Notes:** For an ocean, the `BP_WaterVolume` approach still works for harbour/cove areas with defined boundaries. Reserve `CheckIF OnShore` for open coastlines where the player could exit the water anywhere along the beach.

---

## GerstnerWaves_LakeCustom
- **Type:** WaterWavesAsset *(data asset — not a Blueprint)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/Swimming/GerstnerWaves_LakeCustom`
- **Purpose:** Pre-configured Gerstner wave parameters for the FCS lake water surface — defines wave amplitude, frequency, steepness, and direction for `BP_LakeWater`'s surface simulation.
- **GodOfRuin Use:** **USE** — assign to `BP_LakeWater`'s wave settings for default lake wave behaviour. Duplicate and modify for different water moods (calm underground pool, choppy storm lake, fast river current). This is a data asset, not a Blueprint — edit via the Details panel in the asset editor.
- **Notes:** Gerstner waves affect both visual surface ripple and the buoyancy calculation (`BuoyancyComponent` samples wave height at the character's position). Increasing amplitude will cause more pronounced character bobbing at the surface.
