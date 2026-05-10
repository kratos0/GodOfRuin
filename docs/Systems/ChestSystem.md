# =====================================
# FILE 3: docs/Systems/ChestSystem.md
# =====================================
# INSTRUCTIONS: Update this as BP_ChestBase
# is completed. Fill in the blank fields
# after Claude Code finishes building it.
# =====================================

```markdown
# Chest System
## Status: In Progress
## Last Updated: 2026-05-10
## Session: 1

---

## WHAT IT DOES
Replaces FCS loot system with God of War style instant rewards.
Player presses E on chest → explosion FX → reward auto-applies → chest destroys.
No loot window. No pickup screen. No inventory interaction.

---

## HOW IT WORKS
1. Player enters SphereComponent radius → OutlineComponent shows interact prompt
2. Player presses E → `CombatStatusComponent.Interact()` fires
3. `Event Interact` on BP_ChestBase receives `Character` reference
4. Cast to BP_PlayerCharacter
5. Switch on ChestType → apply reward
6. Spawn Niagara explosion FX
7. Print "Chest Opened!" (2s) — temporary until widget system wired
8. Delay 2s → DestroyActor

---

## BLUEPRINT PATH
```
BP_ChestBase: /Game/FlexibleCombatSystem/Blueprints/Items/LootActors/BP_ChestBase
```

## PARENT CLASS
`BP_LootActorParent` — inherits:
- `I_Interact_C` interface implementation
- `OutlineComponent` (shows interact prompt)
- `SphereComponent` (sets ActorTagVisible on player overlap)
- `LootComponent`

**CRITICAL:** Must be child of BP_LootActorParent — NOT Actor.
Actor parent = no interact prompt, no E key trigger.

---

## CHEST TYPES

| ChestType Value | Type | Reward | Notes |
|----------------|------|--------|-------|
| 0 | GOLD | EXP orb burst | Spawns auto-collect orbs near chest; orbs feed `EquipmentComponent → ApplyXPIncrease` |
| 1 | SILVER | Full health restore | `BuffComponent → Heal` — heals to full only, no partial heal option |
| 2 | STAT | Permanent stat upgrade | `EquipmentComponent → IncreaseLevelUpStats` |
| 3 | LEGENDARY | Greatsword unlock | **L4 Zone E only** — `BP_WeaponHotkeys.UnlockSlot(2)` + 0.3s slow-mo + toast |

**Removed from earlier design:** Named God-of-War-themed weapons (Axe of Sparta,
Apollo's Bow, Blade of Olympus, Divine Spear). The Greatsword is the only
weapon unlock — see `docs/Systems/GreatswordSystem.md`.

---

## VARIABLES

| Variable | Type | Default | Purpose |
|----------|------|---------|---------|
| ChestType | E_ChestType enum | GOLD | Which reward to give |
| XPAmount | int | 1000 | XP for Gold chests |
| StatType | E_StatType enum | HEALTH | Which stat for Stat chests |
| WeaponSlot | int | 2 | Which slot for Legendary chests |

All variables: Edit Anywhere + Expose on Spawn — configurable per instance in Details panel.

---

## CHEST PLACEMENT PER LEVEL

One chest in each level's Zone E (the "Breather"). Type rotates between Gold,
Silver, and Stat across levels — Legendary appears **only at L4**.

| Level | Zone | ChestType | Reward |
|-------|------|-----------|--------|
| L1 | Zone E | GOLD | EXP orb burst |
| L2 | Zone E | SILVER | Full heal |
| L3 | Zone E | STAT | +1 stat point (any) |
| **L4** | **Zone E** | **LEGENDARY** | **Greatsword unlock (slot 2)** |
| L5 | Zone E | GOLD | EXP orb burst |
| L6 | Zone E | SILVER | Full heal |
| L7 | Zone E | STAT | +1 stat point (any) |
| L8 | (none) | — | Pure boss level, no chest |

**Legendary chest is L4-only** — gating the Greatsword unlock. All other
levels rotate Gold/Silver/Stat to avoid telegraphing the L4 reward.

---

## ENUM PATHS
```
E_ChestType: /Game/FlexibleCombatSystem/Blueprints/Items/LootActors/E_ChestType
E_StatType:  /Game/FlexibleCombatSystem/Blueprints/Items/LootActors/E_StatType
```

---

## EXACT FUNCTION NAMES
| Function | Component | Notes |
|----------|-----------|-------|
| ApplyXPIncrease(int) | EquipmentComponent | Pass XPAmount variable |
| Heal | BuffComponent | Heals to full — no amount |
| IncreaseLevelUpStats | EquipmentComponent | Same as level-up stat bump |
| UnlockSlot(int) | BP_WeaponHotkeys | Pass WeaponSlot variable |

---

## NIAGARA EFFECT
```
NS_ExplosionGroundBig: /Game/FlexibleCombatSystem/SoundFX/General/SW_Explosion01
```
*(Confirm exact path after build — fill in here)*

---

## TEST PROCEDURE
1. Place BP_ChestBase in L_ArenaLevelBattleArena
2. Set ChestType = 0 (GOLD) in Details panel
3. Hit Play
4. Walk up to chest — interact prompt must appear
5. Press E
6. Verify: Niagara explosion fires
7. Verify: "Chest Opened!" prints on screen
8. Verify: XP increases on player
9. Verify: Chest actor disappears after 2 seconds

---

## GOTCHAS
- Must be child of BP_LootActorParent — Actor parent breaks interact system
- WB_PickedUpItemPopup crashes — use PrintString until widget system wired
- WB_XPGain also crashes — same issue
- Delete existing BP_ChestBase before recreating if parent class is wrong

---

## STATUS CHECKLIST
- [ ] BP_ChestBase created with BP_LootActorParent parent
- [ ] Variables added (ChestType, XPAmount, StatType, WeaponSlot)
- [ ] Event Interact override wired
- [ ] GOLD branch working (XP applies)
- [ ] SILVER branch working (heal applies)
- [ ] STAT branch working (stat applies)
- [ ] LEGENDARY branch working (weapon unlocks)
- [ ] Niagara FX spawning on open
- [ ] Chest destroys after 2 seconds
- [ ] Tested in PIE — interact prompt shows
- [ ] Tested in PIE — E key triggers reward
```

