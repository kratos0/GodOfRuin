# AnimNotifies — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/`
> Subfolders: `Melee/`, `Ranged/`, `Magic/`
> Total assets: 30 | All relevant to GodOfRuin — none skipped

### Key Concept
**AnimNotify** fires once at a specific frame. **AnimNotifyState** has a Begin and End — it covers a window of frames.
When importing Paragon animations, these notifies must be manually placed in the animation's notify track to wire them into FCS combat logic.

### Paragon Import Checklist
For each imported attack animation, add these notifies to the track:

| Notify | Type | When to place |
|---|---|---|
| `AN_MotionWarping_Attack` | State | Full duration of lunge/step-in attacks |
| `AN_WeaponHitToggle` | State | Active hit frames only (not windup/recovery) |
| `AN_ToggleAttackRotate` | State | Early frames when character should rotate to face target |
| `AN_WeaponTrail` | State | Same window as WeaponHitToggle |
| `AN_TriggerNextAttack` | Notify | Frame where combo input becomes valid |
| `AN_InputBufferSwitch` | State | Same window as TriggerNextAttack |

---

## MELEE

### AN_MotionWarping_Attack
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState_MotionWarping
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_MotionWarping_Attack`
- **Purpose:** Defines the motion warping window for melee attacks — warps the character toward the enemy during the swing to close small gaps without root motion gaps.
- **Key Variables:** None (configured via UE5 MotionWarping target warp point)
- **Key Events:** Begin / End (window-based)
- **GodOfRuin Use:** **USE** — place on every Paragon melee attack montage over the lunge/step-in frames; requires a `MotionWarpingComponent` on the character (check BP_AI_Parent components)
- **Notes:** Extends UE5's built-in `AnimNotifyState_MotionWarping` — no custom logic, just a named subclass for FCS targeting.

---

### AN_LungeMovementMode
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_LungeMovementMode`
- **Purpose:** Sets the character's movement mode (e.g. Flying) at the start of a lunge and restores it at the end, allowing physics-free forward momentum during the lunge animation.
- **Key Variables:** None
- **Key Events:** NotifyBegin, NotifyEnd
- **GodOfRuin Use:** **USE** — place on any lunge or dash-attack animation; pairs with `AN_MotionWarping_Attack`
- **Notes:** Without this, CharacterMovement may fight the root motion and cause stuttering during lunge recovery.

---

### AN_ToggleAttackRotate
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_ToggleAttackRotate`
- **Purpose:** Enables character rotation toward the enemy during the early frames of an attack, then locks rotation once the swing commits.
- **Key Variables:** None
- **Key Events:** NotifyBegin (enable rotate), NotifyEnd (lock rotate)
- **GodOfRuin Use:** **USE** — place at the windup frames of every melee attack; without it, attacks always fire in the direction the character was facing when input registered
- **Notes:** Works with `MeleeComponent`'s rotation logic — do not combine with manual `SetActorRotation` nodes in the same frame.

---

### AN_WeaponHitToggle
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_WeaponHitToggle`
- **Purpose:** Activates the weapon's hit-detection traces (sweep/box traces on the weapon mesh) at Begin and deactivates them at End — defines the exact active damage window.
- **Key Variables:** None
- **Key Events:** NotifyBegin (enable hit traces), NotifyEnd (disable hit traces)
- **GodOfRuin Use:** **USE** — the most critical notify for any melee attack; if missing, the attack plays but deals no damage
- **Notes:** Only covers the hit window — windup and recovery frames must NOT include this state. Overlapping windows across combo hits can cause double-damage.

---

### AN_TriggerNextAttack
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_TriggerNextAttack`
- **Purpose:** Fires a single notification at the frame where the current attack can chain into the next combo hit — signals `InputBufferComponent` that a queued input can now execute.
- **Key Variables:** None
- **Key Events:** Received_Notify (single-fire)
- **GodOfRuin Use:** **USE** — place at the natural combo-link frame of every attack (usually just after the hit frames, before full recovery)
- **Notes:** Must be paired with `AN_InputBufferSwitch` state to function — TriggerNextAttack fires the signal, InputBufferSwitch defines the window during which input can be queued.

---

### AN_WeaponTrail
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_WeaponTrail`
- **Purpose:** Activates a Niagara weapon trail VFX on the weapon mesh between Begin and End — the trail follows the blade arc during the swing.
- **Key Variables:** None
- **Key Events:** NotifyBegin (spawn trail), NotifyEnd (stop trail)
- **GodOfRuin Use:** **USE** — place to match or slightly overlap `AN_WeaponHitToggle`; the trail is the visual cue that the weapon is active
- **Notes:** Trail asset is configured on the weapon mesh, not on this notify — if no trail appears, verify the weapon BP has the correct Niagara trail component socket configured.

---

### AN_Ragdoll
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_Ragdoll`
- **Purpose:** Triggers the ragdoll physics transition on the target character — used in death animations or heavy knockdown finishers.
- **Key Variables:** None
- **Key Events:** Received_Notify (single-fire)
- **GodOfRuin Use:** **USE** — add to any death or knockdown montage at the frame when the body should go limp; `BP_AI_Parent.Death` function manages the full death flow
- **Notes:** Ragdoll is irreversible mid-play — do not place on stagger animations where the character gets back up.

---

### AN_SetMovementMode
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_SetMovementMode`
- **Purpose:** Sets the character's movement mode at a specific animation frame, optionally applying a damage multiplier and directional VFX — used for special attacks that change physics mid-animation (slam, aerial, charge).
- **Key Variables:**
  - `Movement Mode` (byte) — target movement mode to set (Walking / Falling / Flying / etc.)
  - `DamageMultiplier` (real) — scales damage applied at this frame (used for charged/heavy hits)
  - `FinalHit` (bool) — when true, this is the last/decisive hit of a combo or ability
  - `VFX Direction` (byte) — which directional hit VFX to play (front / back / left / right)
- **Key Events:** Received_Notify
- **GodOfRuin Use:** **USE** — the most configurable damage notify; use on slam landings, charge releases, or any attack where damage amount and physics change at the hit frame
- **Notes:** `DamageMultiplier` stacks with the weapon's base damage — a value of `2.0` doubles damage at that frame. Set `FinalHit = true` on finishing blows to trigger execution VFX.

---

### AN_CinematicApplyDmg
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_CinematicApplyDmg`
- **Purpose:** Applies damage during a cinematic finisher or scripted attack sequence where normal hit-detection is bypassed.
- **Key Variables:**
  - `DamageMultiplier` (real) — damage scale for the cinematic hit
  - `VFX Direction` (byte) — directional hit VFX to play
- **Key Events:** Received_Notify
- **GodOfRuin Use:** **USE** — place in execution/finisher montages; guaranteed to deal damage regardless of weapon collision (important for scripted sequences where geometry may interfere)
- **Notes:** Subset of AN_SetMovementMode without the movement mode change — prefer this for pure damage application in cinematics.

---

### AN_ExecuteCombatText
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Melee/AN_ExecuteCombatText`
- **Purpose:** Triggers floating combat text display (damage numbers, status effects) at the hit frame of an attack.
- **Key Variables:** None
- **Key Events:** Received_Notify
- **GodOfRuin Use:** **USE** — place at the hit frame of heavy or ability attacks where text pop-ups are desired; light combo hits may not need this (FCS may handle it automatically via TakingDamageComponent)
- **Notes:** If widget crashes are occurring (see KnownIssues.md), combat text may be affected — test in PIE before relying on it.

---

## RANGED

> The ranged bow sequence requires all six notifies placed in order in a single draw-aim-fire animation.
> Sequence: `AN_SetupArrow` → `AN_DrawBowString` → `AN_AttachArrowString` → `AN_ArrowLoaded` → *(hold)* → `AN_FireArrow` → `AN_ReleaseBowString`

### AN_SetupArrow
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Ranged/AN_SetupArrow`
- **Purpose:** Spawns the arrow actor and attaches it to the bow at the start of the draw animation.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — frame 1 of any bow draw animation
- **Notes:** If the arrow doesn't appear, verify `BP_AbilityRanged.SetupArrow` is correctly referenced from `RangedComponent`.

---

### AN_DrawBowString
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Ranged/AN_DrawBowString`
- **Purpose:** Moves the bow string mesh to the drawn/pulled position at the corresponding animation frame.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place at the frame when the draw reaches full extension
- **Notes:** Cosmetic — won't affect gameplay if misaligned by a few frames, but aim for accuracy.

---

### AN_AttachArrowString
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Ranged/AN_AttachArrowString`
- **Purpose:** Attaches the arrow to the bow string during the draw phase so arrow and string move together.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place between DrawBowString and ArrowLoaded
- **Notes:** Order matters — must fire after AN_SetupArrow and AN_DrawBowString.

---

### AN_ArrowLoaded
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Ranged/AN_ArrowLoaded`
- **Purpose:** Sets the `ArrowLoaded` Blackboard key to true — signals the AI (or player) that the bow is fully drawn and a shot can be fired.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place at the frame where the draw animation reaches the "held at full draw" pose; the BT_MixedCombat `ArrowLoaded` key won't be set without this
- **Notes:** Missing this notify causes the AI to never fire — `T_ArrowLoadedCheck` BT task polls this key.

---

### AN_FireArrow
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Ranged/AN_FireArrow`
- **Purpose:** Launches the arrow actor with the calculated velocity at the release frame of the fire animation.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place at the exact release frame; early placement causes the arrow to launch before the animation visually releases it
- **Notes:** Pairs with AN_ReleaseBowString — fire these at the same frame.

---

### AN_ReleaseBowString
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Ranged/AN_ReleaseBowString`
- **Purpose:** Returns the bow string mesh to its resting position after the arrow is fired.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place at the same frame as AN_FireArrow
- **Notes:** Purely cosmetic — string snapping back gives the shot its visual punch.

---

### AN_ExecuteCameraSwap
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Ranged/AN_ExecuteCameraSwap`
- **Purpose:** Switches to the ranged aim camera at the frame when the bow is raised to aim position.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place in the bow raise/aim animation; without it, the player aims from the default camera perspective
- **Notes:** Pairs with `AN_ToggleRangedCamera` (Magic folder) — ExecuteCameraSwap triggers the swap, ToggleRangedCamera manages the state window.

---

## ROOT-LEVEL

### AN_InputBufferSwitch
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/AN_InputBufferSwitch`
- **Purpose:** Opens and closes the input buffer window — during this state, attack inputs are queued and executed at the next valid opportunity rather than dropped.
- **Key Variables:** None
- **Key Events:** NotifyBegin (open buffer), NotifyEnd (close buffer)
- **GodOfRuin Use:** **USE** — place on every attack and ability animation around the combo-link frame; the window should overlap with `AN_TriggerNextAttack`
- **Notes:** If the input buffer window is too long, the character feels "sticky" and chains attacks unintentionally; tune the window end point carefully.

---

### AN_RollAdjustment
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/AN_RollAdjustment`
- **Purpose:** Adjusts capsule collision and/or physics during a dodge roll — typically reduces capsule height so the character can roll under obstacles and enemies.
- **Key Variables:** None
- **Key Events:** NotifyBegin (shrink/adjust), NotifyEnd (restore)
- **GodOfRuin Use:** **USE** — add to all dodge/roll montages; without it, the character's collision stays full-height during the roll
- **Notes:** If Paragon dodge animations are imported, this notify must be added manually.

---

### AN_ThrowItem
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/AN_ThrowItem`
- **Purpose:** Detaches and launches the thrown item actor at the release frame of the throw animation.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place on any throw animation (grenade, potion, consumable) at the hand-release frame
- **Notes:** The item must already be held in the character's hand socket before this fires — ensure the equip/hold logic precedes the throw montage.

---

### AN_ThrowItemCollision
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/AN_ThrowItemCollision`
- **Purpose:** Enables collision on the in-flight thrown item at the frame when it becomes a physical projectile (separate from `AN_ThrowItem` which handles the launch).
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place 1-2 frames after `AN_ThrowItem`; enables hit detection after the item has visually left the hand
- **Notes:** Placing this too early causes the thrown item to immediately collide with the throwing character.

---

## MAGIC

### AN_TriggerMagicAbility
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_TriggerMagicAbility`
- **Purpose:** Fires the magic ability actor spawn at the precise animation frame — the ability is instantiated when this notify fires, not when input is pressed.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — the primary ability-trigger notify; place at the "cast release" frame of every spell animation
- **Notes:** Without this, the spell ability may spawn immediately on input press rather than at the correct animation moment.

---

### AN_TriggerNextAbility
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_TriggerNextAbility`
- **Purpose:** Advances a chained ability sequence to the next ability in the queue — equivalent to `AN_TriggerNextAttack` but for ability chains.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place in spell combo or ability-chain animations at the link frame; works with `AN_InputBufferSwitch`
- **Notes:** Required for ability combo sequences (e.g. multi-stage spells where each stage is a separate animation).

---

### AN_ToggleKnockbackAndCollision
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_ToggleKnockbackAndCollision`
- **Purpose:** Enables knockback physics and collision responses during a spell impact window — applies impulse forces to hit enemies within this state.
- **Key Variables:** None
- **Key Events:** NotifyBegin (enable knockback), NotifyEnd (disable knockback)
- **GodOfRuin Use:** **USE** — place on AoE and shockwave spell animations over the impact window; without it, spells damage but don't push enemies
- **Notes:** Knockback magnitude is configured on the ability Blueprint, not on this notify.

---

### AN_TriggerAndStopAbility
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_TriggerAndStopAbility`
- **Purpose:** Triggers ability activation at Begin and deactivates it at End — used for ability effects that should be active for exactly the duration of the notify window.
- **Key Variables:** None
- **Key Events:** NotifyBegin (start ability), NotifyEnd (stop ability)
- **GodOfRuin Use:** **USE** — use for abilities that must be time-bounded to an animation window (short AoE bursts, timed barriers, brief invulnerability)
- **Notes:** An alternative to `AN_TriggerMagicAbility` when the ability duration must match an animation window exactly.

---

### AN_MeleeChargeTrigger
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_MeleeChargeTrigger`
- **Purpose:** Fires at the charge-release frame of an ability-enhanced melee attack, triggering `BP_AbilityMelee.IncreaseCharge` to finalise the charge tier before damage is applied.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place in charged heavy attack animations at the moment the charge is committed; drives damage scaling in BP_AbilityMelee
- **Notes:** Must fire before `AN_WeaponHitToggle` activates on the same animation so the charge level is set before damage is calculated.

---

### AN_ToggleRangedCamera
- **Type:** Blueprint (AnimNotifyState)
- **Parent:** AnimNotifyState
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_ToggleRangedCamera`
- **Purpose:** Maintains the ranged aim camera for the duration of the state window — keeps the camera in aim mode while the character is drawn and aiming.
- **Key Variables:** None
- **Key Events:** NotifyBegin (hold aim cam), NotifyEnd (return to default cam)
- **GodOfRuin Use:** **USE** — cover the full aim/hold phase of bow and ranged ability animations; works with `AN_ExecuteCameraSwap`
- **Notes:** For ability-based ranged casts (not standard bow), this also applies — place over the aim-and-fire window.

---

### AN_RangedAbilityBowString
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_RangedAbilityBowString`
- **Purpose:** Bow string cosmetic notify for ranged ability animations — equivalent to `AN_DrawBowString` but specifically for the ability cast variant of the bow draw.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — add to ranged ability cast animations that involve a bow draw (separate from standard arrow shots)
- **Notes:** Distinction from `AN_DrawBowString` (Ranged folder) is the animation context — this is for ability-driven shots, that is for standard attacks.

---

### AN_RangedAbilityHandToString
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_RangedAbilityHandToString`
- **Purpose:** Attaches the ability arrow from the casting hand to the bow string during a ranged ability draw — equivalent to `AN_AttachArrowString` for ability shots.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place in ranged ability animations between the bow-raise and full-draw frames
- **Notes:** Cosmetic order: `AN_RangedAbilityBowString` → `AN_RangedAbilityHandToString` → `AN_RangedAbilityFire`.

---

### AN_RangedAbilityFire
- **Type:** Blueprint (AnimNotify)
- **Parent:** AnimNotify
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AnimNotifies/Magic/AN_RangedAbilityFire`
- **Purpose:** Fires the ranged ability projectile at the release frame — equivalent to `AN_FireArrow` but routes through `BP_AbilityRanged` instead of the standard arrow system.
- **Key Variables:** None
- **GodOfRuin Use:** **USE** — place at the release frame of all ranged ability animations; this is what actually launches the BP_AbilityRanged projectile
- **Notes:** Distinct from `AN_FireArrow` — use this for ability shots (BP_AbilityRanged), use AN_FireArrow for standard bow attacks. Using the wrong one will route to the wrong system.
