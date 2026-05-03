# =====================================
# FILE 2: docs/KnownIssues.md
# =====================================
# INSTRUCTIONS: Add every bug, crash, and workaround
# here AS SOON AS you encounter it.
# This prevents Claude Code from hitting the same 
# crash twice.
# FORMAT: Add newest issues at the TOP.
# =====================================

```markdown
# Known Issues + Workarounds
## Gods of Ruin
Last Updated: May 2, 2026

---

## CRITICAL — Read Before Every Session

### CommonInput Plugin Missing
- **Symptom:** Project crashes on Play with `Assertion failed: ModuleName=CommonInput`
- **Cause:** FCS widgets use CommonUI which depends on CommonInput. Not declared in .uproject.
- **Fix:** On project open, click YES when prompted "Missing Plugin: CommonInput — disable and continue?"
- **Status:** Resolved — plugin disabled. Do not re-enable.

### BP_Gate Null Reference Crash
- **Symptom:** Crash on Play in any arena level
- **Cause:** BP_ArenaSpawner has a direct object reference to BP_Gate. If null → crash on BeginPlay when OpenGates is called.
- **Fix:** In every arena level, select BP_ArenaSpawner actor → Details panel → assign BP_Gate variable to the BP_ArenaGate actor in the level.
- **Status:** Ongoing — must check every new level.

---

## WIDGET CRASHES

### WB_PickedUpItemPopup Crashes on Create Widget
- **Symptom:** PIE crash immediately when chest is opened
- **Cause:** Widget requires two constructor params: `WB Popup Holder` (object) + `Item Info` (struct). Both null at chest-open time.
- **Fix:** Do not use WB_PickedUpItemPopup. Use PrintString for now.
- **Status:** Workaround in place.

### WB_XPGain Crashes on Create Widget
- **Symptom:** PIE crash when XP widget spawned
- **Cause:** Same parent widget class issue — exposes `WB Popup Holder` as required spawn param.
- **Fix:** Do not use WB_XPGain directly. Use PrintString for now.
- **Status:** Workaround in place. Proper widget wiring needed in polish pass.

---

## IMPORT ISSUES

### Bulk Animation Import Crashes UE5
- **Symptom:** UE5 runs out of memory and crashes mid-import
- **Cause:** 5300+ animations imported simultaneously exhausts RAM
- **Fix:** Import max 2-3 Paragon characters per session. Never bulk import the full animation pack.
- **Status:** Ongoing — follow per-session import plan.

### BP_LootActorParent Child Creation Fails
- **Symptom:** `Failed to create Blueprint with parent BP_LootActorParent` or `Asset already exists`
- **Cause:** Asset already exists at destination path from previous attempt
- **Fix:** Delete the existing asset at the path first, then create new child Blueprint
- **Status:** Resolved when path is clear.

---

## BLUEPRINT ISSUES

### BP_ChestBase Interact Not Triggering
- **Symptom:** Walk up to chest, no interact prompt, E does nothing
- **Cause:** Actor parent instead of BP_LootActorParent — missing OutlineComponent, SphereComponent, I_Interact_C
- **Fix:** BP_ChestBase must be child of BP_LootActorParent to inherit full interact system
- **Status:** In progress — rebuilding with correct parent.

### BP_ArenaSpawner Wave Options Not Scalable
- **Symptom:** Wave options are 7 loose struct variables (Option 1 through Option 1_6), not an array
- **Cause:** Stock FCS design
- **Fix:** For GodOfRuin levels, configure the existing option structs. Do not try to add more — add an array variable instead if more waves needed.
- **Status:** Known limitation — workaround in place.

---

## PLUGIN ISSUES

### CommonUI + CommonInput Not In .uproject
- **Symptom:** Missing Plugin warning on open
- **Cause:** FCS references CommonUI base classes but doesn't declare the plugin dependency
- **Fix Applied:** Both CommonUI + CommonInput added to .uproject. Then disabled on open via Yes prompt.
- **Status:** Resolved.

---

## ENVIRONMENT ISSUES

*(Add here as levels are built)*

---

## PERFORMANCE ISSUES

*(Add here as levels are built)*
```

