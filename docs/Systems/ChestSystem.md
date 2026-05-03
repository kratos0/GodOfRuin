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
## Last Updated: May 2, 2026
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
| 0 | GOLD | XP burst | `EquipmentComponent → ApplyXPIncrease(XPAmount)` |
| 1 | SILVER | Full heal | `BuffComponent → Heal` (heals to full) |
| 2 | STAT | Stat upgrade | `EquipmentComponent → IncreaseLevelUpStats` |
| 3 | LEGENDARY | Weapon unlock | `BP_WeaponHotkeys.UnlockSlot(WeaponSlot)` + 0.3s slow-mo |

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

| Level | Zone | ChestType | Reward |
|-------|------|-----------|--------|
| L1 | Arena 1 clear | GOLD | 1500 XP |
| L1 | Zone 3 traversal | SILVER | Full heal |
| L1 | After Khaimera | LEGENDARY | Axe of Sparta (slot 2) |
| L2 | Arena 1 clear | GOLD | 1500 XP |
| L2 | After Sevarog | STAT | HEALTH +25 |
| L3 | After Rampage | LEGENDARY | Apollo's Bow (slot 3) |
| L4 | Wave 1 clear | GOLD | 2000 XP |
| L4 | Wave 3 clear | STAT | ATTACK +10% |
| L5 | Statue puzzle | LEGENDARY | Blade of Olympus (slot 1 upgrade) |
| Boss | Post-boss | LEGENDARY | Divine Spear (slot 5) |

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

