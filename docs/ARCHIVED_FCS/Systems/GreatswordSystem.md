# Greatsword System — God of Ruin
## Status: Design — Implementation Pending
## Last Updated: 2026-05-10

---

## OVERVIEW
The Greatsword is the second weapon Shinbi unlocks. It enters the
inventory at the L4 Zone E Legendary chest and is the only swap-target
weapon in the game (Spirit abilities work on either weapon, but the moveset
changes based on which is active).

---

## UNLOCK
| Aspect | Value |
|---|---|
| Source | L4 Zone E Legendary chest |
| Function called | `BP_WeaponHotkeys.UnlockSlot(2)` |
| Slot | 2 (`int WeaponSlot` on `BP_ChestBase`) |
| Notification | 0.3s slow-mo + screen flash + "Greatsword Acquired" toast |
| Permanence | Stored in save game; available in all subsequent levels |

---

## SWAP CONTROL
| Bind | Action |
|---|---|
| **D-Pad Left (Xbox)** | Toggle between primary weapon and Greatsword |
| (No keyboard bind yet) | Mirror to a number key in input pass |

The swap respects the existing FCS weapon-equip flow — see EquipmentComponent
audit:
- `EquippedWeaponSlots` (array<S_EquippedWeapons>) holds the slot bindings
- `SwapWeaponToggle` (bool) is the existing FCS swap flag
- `ApplyWeaponEquipSlotAction`, `DrawNewWeaponSetup`, `SwapSlots`, `SetupWeapon`
  are the relevant function graphs

We hook D-Pad Left into the existing swap pipeline rather than building a
parallel swap path.

---

## MOVESET
| Input | Move | Notes |
|---|---|---|
| X (light) | Wide horizontal sweep | Slower than primary, hits arc |
| Y (heavy) | Overhead heavy smash | Knockdown on hit, slow startup |
| RT | Thrust lunge (forward dash + stab) | Closes gap; chains into next attack |
| LT | Parry (same window as primary) | Same `DefensiveComponent` parry path |
| B | Grab (same as primary) | Same Grapple Component path |

Build via `BP_AbilityMelee` children (audit shows BP_AbilityMelee parents
the existing melee abilities); each move is a montage + hit-window animation
notify pair, same pattern as the primary weapon's combo set.

---

## SPIRIT ABILITY VARIANTS
The four Spirit abilities have a Greatsword-specific behaviour when the
Greatsword is the active weapon. Same input bind, different visual + impact.

| Ability | Primary variant | Greatsword variant |
|---|---|---|
| Dash (LB) | Quick teleport-step | Heavy slide + ground-shake |
| Surge (LB+X) | Outward spirit pulse | Anchored ground-cleave (wider AOE, slower) |
| Volley (LB+Y) | Ranged spirit projectiles | Ground spike line ahead of player |
| Phantom (LB+B) | Backstab teleport | Overhead phantom slam |

Implementation: each ability's BP checks `EquipmentComponent.MainhandInfo` (or
the primary/greatsword identifier flag) at activation and branches to the
correct montage + VFX. Spirit cost is the same regardless of variant.

---

## FCS WEAPON-SWAP HOOKS (from EquipmentComponent audit)
The audit confirmed 52 functions on `EquipmentComponent`. The ones we touch:

| Function | Purpose |
|---|---|
| `ApplyWeaponEquipSlotAction` | Top-level "equip this slot" entry point |
| `DrawNewWeaponSetup` | Spawns/attaches the weapon actor |
| `RemoveWeapon` | Cleans up the previous weapon |
| `ReplaceWeaponActor` | Swaps the spawned actor in-place |
| `SetupWeapon` | Final wire-up of the equipped weapon |
| `Weapon Slots` | Routes by slot (the function name has a space — preserve it on calls) |
| `SwapWeaponToggle` (variable) | Bool the swap path checks |
| `EquippedMainHand` (variable, replicated) | The currently active main-hand actor |

`BP_WeaponHotkeys` (custom GoR Blueprint at `/Game/FlexibleCombatSystem/Blueprints/GodOfRuin/BP_WeaponHotkeys`) sits on top of these and exposes `UnlockSlot(int)` for the chest unlock path. Its full audit is **not yet done** — flag for next session if the swap behaviour misbehaves.

### NEEDS MANUAL AUDIT
- The exact slot-index mapping in `BP_WeaponHotkeys` (does slot 2 = Greatsword
  consistently across all GoR levels, or per-level override?).
- Whether the existing `SwapWeaponToggle` covers a 2-weapon swap or only the FCS dual-wield case.
- Animation montage compatibility for a Greatsword on the FCS Manny skeleton
  (hand grip socket, anim notify alignment).

---

## ASSETS NEEDED
| Asset | Source | Status |
|---|---|---|
| Greatsword mesh | Paragon (TBD which character — Sevarog/Steel/etc.) or Marketplace | ⬜ Not chosen |
| Greatsword anim montages × 5+ | Hand-built or retargeted from a Paragon heavy moveset | ⬜ Not built |
| Niagara hit FX | Reuse FCS heavy hit FX | ⬜ Not wired |
| Slow-mo notification widget | Extend `WB_HUD` toast widget | ⬜ Not built |

---

## TEST PLAN
1. Place a Legendary chest in L1 test arena with `WeaponSlot = 2`.
2. Open chest → confirm slow-mo, "Greatsword Acquired" toast.
3. Press D-Pad Left → confirm weapon swaps in hand mesh.
4. Combo X-X-X → wide sweeps play.
5. Combo Y → heavy smash.
6. RT → thrust lunge.
7. LT against an attacking enemy → parry triggers (using the same `DefensiveComponent` path).
8. LB+X with Spirit ≥ 40 → confirm Greatsword variant of Surge plays.
9. Quit + reload → Greatsword still unlocked (save persistence).
