# =====================================
# FILE 6: docs/Assets/ParagonCharacters.md
# =====================================
# INSTRUCTIONS: Update after each import session.
# Check boxes as steps complete.
# Add animation notes from your testing.
# =====================================

```markdown
# Paragon Character Assets
## Gods of Ruin
Last Updated: May 2, 2026

---

## IMPORT RULES
1. Max 2-3 characters per session — never bulk import
2. Always use FCS Manny skeleton on import
3. Import at START of the session before building
4. Verify animations work before building level content

## FCS Manny Skeleton Path
```
/Game/FlexibleCombatSystem/GASP/UEFN-Manny/Meshes/SK_UEFN_Mannequin.SK_UEFN_Mannequin
```

## Import Template Prompt (reuse each session)
```
Import [CHARACTER NAME(S)] into GodOfRuin with FCS Manny skeleton.

FCS MANNY SKELETON:
/Game/FlexibleCombatSystem/GASP/UEFN-Manny/Meshes/SK_UEFN_Mannequin.SK_UEFN_Mannequin

Source folder: [PATH IN CONTENT FOLDER]
Destination: /Game/Characters/Paragon/[CHARACTER NAME]/

Use batch import script with:
- automated = True
- skeleton = FCS Manny skeleton loaded via load_asset()
- import_animations = True
- import_mesh = True
- replace_existing = True

After import: verify FCS Manny skeleton assigned on all skeletal meshes.
Report any failures.
```

---

## SESSION 4 IMPORTS — Khaimera + Grux

### Khaimera
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Khaimera/
- **Source Folder:** *(fill in)*
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Khaimera)
- **Test Result:** *(fill in after test)*
- **Issues:** *(fill in)*

### Grux
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Grux/
- **Source Folder:** *(fill in)*
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Grux)
- **Test Result:** *(fill in after test)*
- **Issues:** *(fill in)*

---

## SESSION 5 IMPORTS — Kwang

### Kwang
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Kwang/
- **Source Folder:** *(fill in)*
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Kwang)
- **Test Result:** *(fill in after test)*
- **Issues:** *(fill in)*

---

## SESSION 6 IMPORTS — Crunch + Morigesh

### Crunch
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Crunch/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Crunch)
- **Test Result:** *(fill in after test)*
- **Issues:** *(fill in)*

### Morigesh
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Morigesh/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Morigesh)
- **Test Result:** *(fill in after test)*
- **Issues:** *(fill in)*

---

## SESSION 7 IMPORTS — Steel + Narbash + Feng Mao

### Steel
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Steel/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Steel)
- **Test Result:** *(fill in after test)*

### Narbash
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Narbash/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Narbash)
- **Test Result:** *(fill in after test)*

### Feng Mao
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/FengMao/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_FengMao)
- **Test Result:** *(fill in after test)*

---

## SESSION 8 IMPORTS — Aurora + Kallari + Shinbi + Serath

### Aurora
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Aurora/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Aurora)
- **Test Result:** *(fill in after test)*

### Kallari
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Kallari/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Kallari)
- **Test Result:** *(fill in after test)*

### Shinbi
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Shinbi/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Shinbi)
- **Test Result:** *(fill in after test)*

### Serath
- **Import Status:** ⬜ Not started
- **Import Path:** /Game/Characters/Paragon/Serath/
- **Skeleton Set:** ⬜
- **Animations Working:** ⬜
- **BP_AI_Parent Child Created:** ⬜ (name: BP_Enemy_Serath)
- **Test Result:** *(fill in after test)*
- **Special Note:** Has wing bones — skip during chain mapping if manual retarget needed

---

## ANIMATION NOTES

*(Fill in as you discover which animations work well for each character)*

| Character | Idle | Walk/Run | Attack | Death | Special | Notes |
|-----------|------|----------|--------|-------|---------|-------|
| Khaimera | | | | | Leap attack | |
| Grux | | | | | Charge | |
| Kwang | | | | | | |
| Crunch | | | | | | |
| Morigesh | | | | | Magic cast | |
| Steel | | | | | Shield bash | |
| Narbash | | | | | Hammer AOE | |
| Feng Mao | | | | | Dash | |
| Aurora | | | | | Ice magic | |
| Kallari | | | | | Stealth | |
| Shinbi | | | | | Spirit summon | |
| Serath | | | | | Aerial | |
```
