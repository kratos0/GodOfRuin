

# =====================================
# FILE 5: docs/Characters/EnemyRoster.md
# =====================================
# INSTRUCTIONS: Update import status column
# as you import each character per session.
# Fill in skeleton path column after each import.
# Add animation notes as you discover what works.
# =====================================

```markdown
# Enemy Roster
## Gods of Ruin — Locked Roster
Last Updated: May 2, 2026

---

## ROSTER RULES
- No gun-based characters
- No character appears in more than one level
- Boss Toolkit characters (Terra, Countess, Sevarog, Rampage) exclusive to their level
- All Paragon characters use FCS Manny skeleton on import

## FCS Manny Skeleton Path (use for every import)
```
/Game/FlexibleCombatSystem/GASP/UEFN-Manny/Meshes/SK_UEFN_Mannequin.SK_UEFN_Mannequin
```

---

## LEVEL 1 — THE FALLEN RUINS

### FCS Soldiers (Regular Enemies)
- **Source:** FCS built-in
- **Import Status:** ✅ Already in project
- **Parent Class:** BP_AI_Parent
- **Role:** Standard melee fodder
- **Scale:** 1.0 standard, 1.3 for Grux-style variants

### Grux (Regular Enemy)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 4
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Heavy beast enemy, scaled 1.3
- **Import Path:** /Game/Characters/Paragon/Grux/
- **Notes:** *(fill in after import)*

### Khaimera (Mini-boss)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 4
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Fast feral mini-boss, leap attacks, flanking
- **Import Path:** /Game/Characters/Paragon/Khaimera/
- **Health Pool:** 300% of standard FCS soldier
- **Death Reward:** Legendary chest spawns (Axe of Sparta)
- **Notes:** *(fill in after import)*

---

## LEVEL 2 — GATES OF THE UNDERWORLD

### Kwang (Regular Enemy)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 5
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Heavy swordsman, slow + high damage + charge attack
- **Import Path:** /Game/Characters/Paragon/Kwang/
- **Notes:** *(fill in after import)*

### Sevarog (Mini-boss)
- **Source:** Boss AI Toolkit ✅ Already in project
- **Import Status:** ✅ Pre-built in Boss Toolkit
- **Path:** /Game/BossAIToolkit/Demo/Sevarog/
- **Role:** Soul collector mini-boss, 2 phases
- **Phase 1 (100-50%):** Melee + summons thralls at 75%
- **Phase 2 (50-0%):** Enrage + AOE soul pulse every 15s
- **Hazard:** Hellfire floor damage zones during fight
- **Notes:** Load from Boss Toolkit demo, configure phases

---

## LEVEL 3 — TEMPLE OF THE STORM

### Crunch (Regular Enemy)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 6
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Slow heavy melee, charges on sight
- **Import Path:** /Game/Characters/Paragon/Crunch/
- **Notes:** *(fill in after import)*

### Morigesh (Regular Enemy — Caster)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 6
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Ranged dark magic, keeps distance
- **Import Path:** /Game/Characters/Paragon/Morigesh/
- **Notes:** *(fill in after import)*

### Rampage (Mini-boss)
- **Source:** Boss AI Toolkit ✅ Already in project
- **Import Status:** ✅ Pre-built in Boss Toolkit
- **Path:** /Game/BossAIToolkit/Demo/Rampage/
- **Role:** Beast titan mini-boss, 2 phases
- **Phase 1 (100-60%):** Ground pounds + summons 2x Crunch
- **Phase 2 (60-0%):** Constant ground pounds, frequent charges
- **Hazard:** Lightning hazard active during fight
- **Death Reward:** Legendary chest (Apollo's Bow slot 3)
- **Notes:** Load from Boss Toolkit demo, configure phases

---

## LEVEL 4 — TITAN'S ARENA

### Steel (Regular Enemy)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 7
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Shield tank, slow, high defense
- **Import Path:** /Game/Characters/Paragon/Steel/
- **Notes:** *(fill in after import)*

### Narbash (Regular Enemy)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 7
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Tribal heavy, hammer AOE attacks
- **Import Path:** /Game/Characters/Paragon/Narbash/
- **Notes:** *(fill in after import)*

### Feng Mao (Mini-boss)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 7
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Fast assassin mini-boss, dash attacks, combo resets
- **Import Path:** /Game/Characters/Paragon/FengMao/
- **Health Pool:** 400% of standard FCS soldier
- **Death Effect:** Champion gate shatters (Niagara debris)
- **Notes:** *(fill in after import)*

---

## LEVEL 5 — ASCENT TO OLYMPUS

### Aurora (Regular Enemy — Elite)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 8
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Ice magic ranged, slows player
- **Import Path:** /Game/Characters/Paragon/Aurora/
- **Notes:** *(fill in after import)*

### Kallari (Regular Enemy — Elite)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 8
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Shadow assassin, flank attacks, brief stealth
- **Import Path:** /Game/Characters/Paragon/Kallari/
- **Notes:** *(fill in after import)*

### Shinbi (Regular Enemy — Elite)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 8
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Spirit wolf summons, AOE magic bursts
- **Import Path:** /Game/Characters/Paragon/Shinbi/
- **Notes:** *(fill in after import)*

### Serath (Elite Wave)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 8
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Fallen angel, aerial approach, land to melee, retreat when low HP
- **Import Path:** /Game/Characters/Paragon/Serath/
- **Notes:** *(fill in after import)*

---

## BOSS — THRONE OF THE GODS

### Terra (Boss Phase 1)
- **Source:** Boss AI Toolkit ✅ Already in project
- **Import Status:** ✅ Pre-built in Boss Toolkit
- **Path:** /Game/BossAIToolkit/Demo/Terra/
- **Health Pool:** 60% of total boss HP
- **Attacks:** Ground slam (AOE indicator), charge, rock throw at 40% HP
- **At 75%:** Spawns 4x FCS soldiers from wall alcoves
- **At 50%:** Enrage — +30% damage, +20% speed, vignette darkens
- **At 0%:** Collapse cinematic → 3s pause → Countess rises
- **Notes:** Load from Boss Toolkit, configure in our arena

### Countess (Boss Phase 2)
- **Source:** Boss AI Toolkit ✅ Already in project
- **Import Status:** ✅ Pre-built in Boss Toolkit
- **Path:** /Game/BossAIToolkit/Demo/Countess/
- **Health Pool:** 40% of total boss HP
- **Attacks:** Homing orbs x3, teleport blink, shadow clones
- **At 40%:** Floor rune zones activate (2s warning, 35% damage)
- **At 20%:** Desperate — 8 orbs, teleport every 3s, persistent clones
- **Death:** Victory cinematic → Divine Spear chest spawns
- **Notes:** Load from Boss Toolkit, configure phases

---

## IMPORT TRACKING SUMMARY

| Character | Level | Session | Status | Skeleton Set |
|-----------|-------|---------|--------|-------------|
| FCS Soldiers | L1 | Built-in | ✅ Ready | N/A |
| Grux | L1 | 4 | ⬜ Pending | ⬜ |
| Khaimera | L1 | 4 | ⬜ Pending | ⬜ |
| Kwang | L2 | 5 | ⬜ Pending | ⬜ |
| Sevarog | L2 | Built-in | ✅ Boss Toolkit | N/A |
| Crunch | L3 | 6 | ⬜ Pending | ⬜ |
| Morigesh | L3 | 6 | ⬜ Pending | ⬜ |
| Rampage | L3 | Built-in | ✅ Boss Toolkit | N/A |
| Steel | L4 | 7 | ⬜ Pending | ⬜ |
| Narbash | L4 | 7 | ⬜ Pending | ⬜ |
| Feng Mao | L4 | 7 | ⬜ Pending | ⬜ |
| Aurora | L5 | 8 | ⬜ Pending | ⬜ |
| Kallari | L5 | 8 | ⬜ Pending | ⬜ |
| Shinbi | L5 | 8 | ⬜ Pending | ⬜ |
| Serath | L5 | 8 | ⬜ Pending | ⬜ |
| Terra | Boss | Built-in | ✅ Boss Toolkit | N/A |
| Countess | Boss | Built-in | ✅ Boss Toolkit | N/A |
```

