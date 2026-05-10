# Spirit System — God of Ruin
## Status: Design — Implementation Pending
## Last Updated: 2026-05-10

---

## OVERVIEW
Spirit is Shinbi's god-tier resource: filled by skilled combat (parries,
grabs, pressure), spent on signature abilities (Dash, Surge, Volley,
Phantom). It's the GoW-style Rage analogue — separate from FCS Stamina/Mana,
layered on top of the existing FCS combat loop.

Implementation philosophy: **child Blueprint of FCS player, new float vars,
hook into existing FCS events**. Never modify core FCS components directly.

---

## METER DESIGN
| Property | Value | Notes |
|---|---|---|
| Base max | 100 | Default at game start |
| Per-puzzle bonus | +15 | One per optional Zone E puzzle |
| Total optional puzzles | 8 | One per level |
| Final cap | 220 | 100 base + 8 × 15 |
| Start of run | 0 | Empty on level start |
| Persists across levels | yes | Stored in custom save data |

---

## GAIN SOURCES
| Source | Spirit | When |
|---|---|---|
| Successful parry | +20 | DefensiveComponent — parry success path |
| Grab landed | +10 | Grapple Component — grab success path |
| Passive in-combat tick | +2/sec | While `CombatStatusComp.CombatStatus` is active |
| (No gain out of combat) | 0 | Hard rule: no idle farming |

### FCS Hook Points (from audit)

**`DefensiveComponent`** (parry events)
- Parry-related variables: `ParryUp`, `ParryOpening`, `ParryConsumesStaminaDeactivated`, `ParryImpactAnim`
- Parry-relevant graphs: `CanDefendHit`, `BreakBlockConditions`, `DefensiveReactionText`,
  `EnemyBlockHit`, `OnRep_BlockUp`, `PassDamageInfo`, `UpdateAnimations`
- **No explicit `ParrySuccess` event found in the function list** — the parry-success branch likely lives inside `EventGraph` or `CanDefendHit`.
- **Action item:** read `DefensiveComponent.EventGraph` and `CanDefendHit` to find the exact node where parry succeeds, then add a custom dispatcher (`OnParrySuccess`) and bind to it from the Spirit child Blueprint.

**Grapple Component** (grab events)
- Owned plugin, paths not yet audited — flag for next session.

**`CombatStatusComponent`** (combat-state gate for passive tick)
- Audit the `CombatStatus` variable + the event(s) that flip it.
- Tie a `Set Timer by Function` (every 0.5s, +1 spirit) gated on `CombatStatus == Active`.

---

## SPENDING
| Ability | Cost | Bind | Unlocks at |
|---|---|---|---|
| Spirit Dash | 15 | LB | L1 |
| Spirit Surge | 40 | LB+X | L3 |
| Spirit Volley | 25 | LB+Y | L5 |
| Phantom Strike | 50 | LB+B | L6 |

If `CurrentSpirit < cost` at input time → no-op + brief "insufficient" SFX (no error popup).

---

## UNLOCK ORDER
Per the locked GoR design:
1. **L1 — Spirit Dash** (granted by Ruined One after the scripted death)
2. **L2 — Wolf Fang + Grab** (built on top of FCS heavy + Grapple)
3. **L3 — Spirit Surge**
4. **L4 — Greatsword** (separate weapon system, see GreatswordSystem.md)
5. **L5 — Spirit Volley**
6. **L6 — Phantom Strike**
7. **L7 — Chain Phantom** (passive upgrade, no new bind)
8. **L8 — none** (final boss uses everything)

`UnlockedAbilities` array (see `BP_SaveGame.PlayerSaveData` → custom field) tracks which are available; ability bindings check the list before firing.

---

## HUD
| Element | Position | Notes |
|---|---|---|
| Spirit bar | Below stamina/mana on existing FCS HUD | Custom UMG widget added to `WB_HUD` |
| Spirit numerical (optional) | Right of bar | Shows `current/max` only when bar is non-zero |
| Cost preview on input | Brief flash | Shows the cost about to be spent when LB is held |

The bar widget (`WB_SpiritBar`) is added to the existing FCS HUD container — do NOT
replace `WB_HUD`. Same crash gotchas as `WB_PickedUpItemPopup` apply: keep the
widget self-contained with no required spawn params.

---

## IMPLEMENTATION APPROACH

### Child Blueprint
Create `BP_PlayerCharacter_GoR` as a child of `BP_PlayerCharacter`.
Add the Spirit logic there — never touch the FCS parent.

### New Variables (on the GoR child)
| Variable | Type | Replicated | Save | Default |
|---|---|---|---|---|
| `CurrentSpirit` | float | yes | yes | 0 |
| `MaxSpirit` | float | yes | yes | 100 |
| `SpiritGainParry` | float | no | no | 20 |
| `SpiritGainGrab` | float | no | no | 10 |
| `SpiritPassiveTick` | float | no | no | 2 |
| `UnlockedAbilities` | array<name> | yes | yes | [] |
| `OnSpiritChanged` | mcdelegate | — | — | — |

### Save Hook
The four already-listed `BP_SaveGame` fields handle player-state restore;
add two GoR-only fields to the save (or extend `PlayerSaveData` struct):
- `Saved_CurrentSpirit` (float)
- `Saved_MaxSpirit` (float)
- `Saved_UnlockedAbilities` (array<name>)

### Wire-up steps
1. **Parry hook:** In `DefensiveComponent`, find the parry-success node. In
   the GoR child, override or attach a custom event listener that adds
   `SpiritGainParry` when fired.
2. **Grab hook:** Same pattern with the Grapple Component once paths are audited.
3. **Passive tick:** On `BeginPlay` start a timer with 0.5s interval; tick
   adds `SpiritPassiveTick * 0.5` if `CombatStatusComp.CombatStatus` indicates active combat.
4. **Spend:** Each ability input handler subtracts the cost atomically, checks
   `>= cost` first, fires the ability on success, broadcasts `OnSpiritChanged`.
5. **HUD:** `WB_SpiritBar` binds to `OnSpiritChanged`. No tick polling.

### Compile + test order
1. Add variables, compile clean.
2. Wire passive tick first (testable in any combat encounter).
3. Add parry hook second (testable against L1 soldiers).
4. Add HUD bar third.
5. Add abilities one at a time as their level unlock approaches.

---

## OPEN QUESTIONS / NEEDS AUDIT
- **Exact parry-success node** in `DefensiveComponent.EventGraph` or `CanDefendHit` — read graph to confirm the dispatcher name to bind, or whether we add a new event.
- **Grapple Component grab-success event** — needs a separate audit pass.
- **`CombatStatusComp` "in combat" boolean name** — verify on next audit (the variable is referenced from `DefensiveComponent` as `CombatStatusComp` of type `CombatStatusComponent_C`, but we haven't read the inner state field name).
- Replication strategy: server-authoritative spirit (current default assumption) vs client-predicted (only matters if multiplayer is in scope; GoR is single-player so server-auth is fine).
