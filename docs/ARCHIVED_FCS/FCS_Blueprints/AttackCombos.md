# AttackCombos — FCS Blueprint Reference
## Last Updated: May 3, 2026

> Assets under `/Game/FlexibleCombatSystem/Blueprints/AttackCombos/`
> Total assets: 1

---

## BP_ComboResolver
- **Type:** Blueprint
- **Parent:** Object *(not Actor — pure logic class, no world presence)*
- **Path:** `/Game/FlexibleCombatSystem/Blueprints/AttackCombos/BP_ComboResolver`
- **Purpose:** Stateful combo sequencer — tracks the current position in an attack chain, validates transitions, and returns the correct montage/action for each hit in the combo when queried by `MeleeComponent`.
- **Key Variables:**
  - `CurrentWeaponCombos` (object) — reference to the active weapon's combo data asset; set via `SetWeaponCombos` on equip
  - `NextComboAttackIndex` (int) — current position in the chain (0 = not started, increments each valid input)
  - `PossibleCombos` (name) — name identifier of the valid next combo branch
- **Key Functions/Events:**
  - `SetWeaponCombos` — called by MeleeComponent on weapon equip; binds this resolver to a weapon's combo dataset
  - `ResetCombo` — resets `NextComboAttackIndex` to 0; called on combo timeout or end of chain
  - `HasStartedCombo` — returns true if `NextComboAttackIndex > 0`; used to determine if a chain is in progress
  - `IsComboFinished` — returns true when the index has reached the last attack; triggers reset
  - `HasValidComboData` — validates `CurrentWeaponCombos` is assigned; guard before any combo query
  - `HasValidDataForCombo` — validates that a specific combo name exists in the dataset
  - `NextComboAttack` — advances the index and returns the action for the next attack; called when `AN_TriggerNextAttack` fires and input is buffered
  - `PeekNextComboAttack` — returns what the next attack *would* be without advancing the index; used for UI or animation blending preview
  - `Internal_CheckNextComboAttack` — internal validation called by NextComboAttack before advancing
  - `GetActionForNextAttackInCombo` — returns the montage or action data for the current index position
  - `GetAllComboNames` — returns all combo chain names available on the current weapon
  - `GetNumberOfAttacksInCombo` — returns the total length of the active combo chain
- **GodOfRuin Use:** **USE** — this is the engine of all melee combo sequencing; do not modify directly. To add new combos for Paragon characters, create new weapon combo data assets and pass them via `SetWeaponCombos`. The resolver handles sequencing automatically.
- **Notes:**
  - Parent is `Object`, not `Actor` — this is instantiated as a sub-object owned by `MeleeComponent`, not placed in the world.
  - `NextComboAttackIndex` is the single variable to watch when debugging broken combos — if it's not advancing, `AN_TriggerNextAttack` + `AN_InputBufferSwitch` are not firing correctly in the animation.
  - `PeekNextComboAttack` is safe to call at any time without side effects — useful for anticipatory animation blending (pre-loading the next attack's blend space).
  - 13 graphs for 3 variables means this is logic-dense — all combo branching logic lives here, not in the weapon data.
