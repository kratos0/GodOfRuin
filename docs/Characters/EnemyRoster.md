

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
## God of Ruin — Locked Roster
Last Updated: 2026-05-10

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

### Khaimera (Mini-boss)
- **Source:** Paragon
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 4
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Role:** Fast feral mini-boss, leap attacks, flanking
- **Import Path:** /Game/Characters/Paragon/Khaimera/
- **Health Pool:** 300% of standard FCS soldier
- **Death Reward:** Gold chest (XP orb burst)
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

### Greystone (Boss — Divine Gate Guardian)
- **Source:** Paragon (free)
- **Import Status:** ⬜ Not imported
- **Import Session:** Session 5 (alongside Kwang)
- **Skeleton:** FCS Manny (assign on import)
- **Parent Class:** BP_AI_Parent (create child)
- **Path:** /Game/Characters/Paragon/Greystone/
- **Role:** Divine gate guardian, 2-phase boss
- **Phase 1 (100-50%):** Melee + sword combos, summons FCS soldiers at 75%
- **Phase 2 (50-0%):** Enrage + AOE shockwaves
- **Hazard:** Soul-fire floor damage zones during fight
- **Notes:** **Replaces Sevarog at L2.** Sevarog is reserved for L8 (The Ruined One). Greystone fits the L2 "Gates of the Underworld" guardian theme.

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
- **Death Reward:** Gold chest (XP orb burst)
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

### Shinbi-clones (Regular Enemy — Elite Wave)
- **Source:** Reuses player character mesh (Shinbi is the player — these are corrupted echoes)
- **Import Status:** ✅ Mesh already in project (player rig)
- **Import Session:** Not a separate import — Session 8 derives from player asset
- **Skeleton:** FCS Manny (already assigned on player)
- **Parent Class:** BP_AI_Parent (create child `BP_Enemy_ShinbiClone`)
- **Role:** Spirit wolf summons, AOE magic bursts; visually identical to the player as a story beat
- **Notes:** Tint material to a desaturated/corrupted variant so the player can distinguish self from clones

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

## LEVEL 6 — THRONE OF THE GODS

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
- **Death:** Victory cinematic → next level loads (no item drop — see ChestSystem.md, named weapons removed)
- **Notes:** Load from Boss Toolkit, configure phases

---

## LEVEL 7 — THE RUINED PANTHEON

### Mixed Elite Wave (4 simultaneous)
- **Source:** Reuses already-imported Paragon characters
- **Composition:** 1 from each previous level — e.g., Aurora + Kallari + Steel + Crunch
- **Role:** Final test of every system before the L8 boss
- **Notes:** No new imports needed; pull existing BP_Enemy_* children

---

## LEVEL 8 — SEAT OF THE RUINED ONE

### The Ruined One (Final Boss)
- **Source:** Boss AI Toolkit ✅ Already in project (Sevarog mesh + behaviour)
- **Import Status:** ✅ Pre-built in Boss Toolkit
- **Path:** /Game/BossAIToolkit/Demo/Sevarog/
- **Phases:** 2-phase fight, see GodOfRuin_GameDesign.md for narrative beats
- **Phase 1 (100-50%):** Melee + summons thralls; corruption builds
- **Phase 2 (50-0%):** Enrage + arena-wide AOE; the Throne reveals its true nature
- **Hazard:** Reality fractures — floor damage zones move
- **Notes:** **Sevarog mesh is reserved for L8 only.** This is the visual identity of The Ruined One. Two endings — destroy the Throne or claim it.

---

## IMPORT TRACKING SUMMARY

| Character | Level | Session | Status | Skeleton Set |
|-----------|-------|---------|--------|-------------|
| FCS Soldiers | L1 | Built-in | ✅ Ready | N/A |
| Khaimera | L1 | 4 | ⬜ Pending | ⬜ |
| Kwang | L2 | 5 | ⬜ Pending | ⬜ |
| Greystone | L2 | 5 | ⬜ Pending | ⬜ |
| Crunch | L3 | 6 | ⬜ Pending | ⬜ |
| Morigesh | L3 | 6 | ⬜ Pending | ⬜ |
| Rampage | L3 | Built-in | ✅ Boss Toolkit | N/A |
| Steel | L4 | 7 | ⬜ Pending | ⬜ |
| Narbash | L4 | 7 | ⬜ Pending | ⬜ |
| Feng Mao | L4 | 7 | ⬜ Pending | ⬜ |
| Aurora | L5 | 8 | ⬜ Pending | ⬜ |
| Kallari | L5 | 8 | ⬜ Pending | ⬜ |
| Shinbi-clones | L5 | Built-in (player asset) | ✅ Reuse | N/A |
| Serath | L5 | 8 | ⬜ Pending | ⬜ |
| Terra | L6 | Built-in | ✅ Boss Toolkit | N/A |
| Countess | L6 | Built-in | ✅ Boss Toolkit | N/A |
| The Ruined One | L8 | Built-in | ✅ Boss Toolkit (Sevarog mesh) | N/A |
```

