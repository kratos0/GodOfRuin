# God of Ruin — Game Design Reference
## Master Reference Doc — Read at Session Start
## Last Updated: 2026-05-10

> **Purpose:** Authoritative source-of-truth for every design decision.
> Claude Code reads this each session before touching anything.
> If a decision conflicts with code, this doc wins — flag the code.

---

## GAME IDENTITY
- **Title:** God of Ruin
- **World:** Tarragon — ancient planet of the Greek pantheon, corrupted by their own divine technology
- **Player:** Shinbi only (Paragon asset — no character select)
- **Antagonist:** The Ruined One (a fallen god; uses the Sevarog mesh from Boss AI Toolkit at L8)
- **Aesthetic:** Greek temple ruins fused with corrupted divine tech (broken statuary + biomechanical conduits + light-bleed cracks)
- **Design Blueprint:** God of War (2018) — quality over quantity, grounded combat, seamless transitions

---

## TECH STACK
| Layer | Asset | Status |
|---|---|---|
| Combat framework | FCS Full Bundle | ✅ owned, in project |
| Boss behaviour | Boss AI Toolkit | ✅ owned (Terra, Countess, Sevarog, Rampage prebuilt) |
| Grab + grapple | Grapple Component | ✅ owned, audit pending |
| Characters | Paragon Assets (free via Epic) | imported per-session, see ParagonCharacters.md |
| Skeleton | FCS Manny (`SK_UEFN_Mannequin`) | retarget all imports to this |
| Engine | Unreal Engine 5 (5.x) | — |

All asset paths live in `docs/FCS_SystemReference.md` — never hard-code paths in this doc when one is already there.

---

## ZONE FORMULA (per level — A through F)
GoW rule: **quality over quantity. Aggression beats headcount.**

| Zone | Role | Combat | Notes |
|---|---|---|---|
| **A** | Arrival | None | Required puzzle; gates entry to B |
| **B** | First Contact | 2 enemies (1 heavy + 1 light) | Tutorial-light; introduces level theme |
| **C** | Escalation | 3–4 enemies, mixed types, 1 ranged | Tests positioning |
| **D** | Mob Gauntlet | 6–8 enemies in 2 overlapping waves | **Not 12.** Capped at 8 active. |
| **E** | Breather | 0 enemies | Optional puzzle, chest, lore beat |
| **F** | Climax Arena | 1 boss OR 3–4 elites | Closes the level |

Checkpoints fire at A→B, D→E, pre-F transitions per level (3 per level, 24 total).

---

## 8 LEVELS

| # | Title | Setting | Climax | Puzzle Theme | Unlock |
|---|---|---|---|---|---|
| **L1** | The Fallen Prison | Arena + temple ruins | Khaimera mini-boss | Pressure plates ×3 | Spirit Dash |
| **L2** | Gates of the Underworld | Subterranean caverns | Greystone boss | Sequence terminals ×3 | Wolf Fang + Grab |
| **L3** | Temple of the Storm | Storm-cliff temple | Rampage (Boss AI Toolkit) | Light beam + mirror ×3 | Spirit Surge |
| **L4** | The Titan's Arena | Colosseum | Feng Mao elite | Conduit routing ×3 nodes | **Greatsword** (Legendary chest) |
| **L5** | Ascent to Olympus | Vertical temples | Serath ×3 elite | Timed plates + enemy pressure | Spirit Volley |
| **L6** | Throne of the Gods | Grand palace | Terra → Countess (2-phase) | Multi-room conduit (3 rooms) | Phantom Strike |
| **L7** | The Ruined Pantheon | Fractured reality | 4 elites simultaneous | All mechanics combined | Chain Phantom (passive) |
| **L8** | Seat of the Ruined One | The Throne | The Ruined One (Sevarog mesh) | None — pure boss | Two endings |

---

## ENEMY ROSTER (summary — see `docs/Characters/EnemyRoster.md` for full detail)
| Level | Enemies | Boss/Mini-boss |
|---|---|---|
| L1 | FCS soldiers | Khaimera |
| L2 | FCS soldiers + Kwang | Greystone (Boss AI Toolkit / Paragon) |
| L3 | Crunch + Morigesh | Rampage (Boss AI Toolkit) |
| L4 | Steel + Narbash | Feng Mao |
| L5 | Aurora + Kallari + Shinbi-clones + Serath | Serath wave |
| L6 | (boss-room only) | Terra → Countess (2-phase) |
| L7 | All previous types mixed | 4 elites simultaneous |
| L8 | (boss-only) | The Ruined One (Sevarog mesh, 2-phase) |

> **IMPORTANT — Sevarog mesh is reserved for L8 only.** It is the visual identity of The Ruined One. Do not use Sevarog as a regular enemy or earlier mini-boss.

---

## SHINBI COMBAT

## Controls (FINAL — 2026-05-16)

### Base Actions
A   = Jump (AAMS)
X   = Dodge (AAMS) 
Y   = Light Attack
B   = Interact (AAMS)
LT  = Lock On toggle — strafe when held
RT  = Ranged Attack
LB  = Block / Parry (precise timing window)
RB  = Grab (Grapple Component)
L3  = Sprint (AAMS)
R3  = Crouch / Sprint+Crouch = Slide (AAMS)
DPad Left = Weapon Swap
DPad Up   = Examine / Lore

### Hold Actions (1 second hold)
X hold  = Spirit Surge  (-40 spirit)
Y hold  = Heavy Attack
LB hold = Spirit Dash   (-15 spirit)
RT hold = Spirit Volley (-25 spirit)
RB hold = Phantom Strike(-50 spirit)

### Parry System
LB press → 0.2s parry window
  Hit lands in window → Parry success
    Spirit +20, enemy stagger
  No hit → transitions to Block stance
  Block hit → stamina drains on hit only
  Stamina 0 → guard break → Shinbi staggers

### Lock On System
LT Started  → lock on, enter strafe
LT Completed → unlock, free movement
Works with ranged for easy targeting

### Stamina (Guard Resource)
Drains on: blocked hits only
Passive regen: when not blocking
Empty: guard break

### Spirit Resource
Max: 100 (upgrades to 220 via puzzles)
Gains: parry +20, grab +10, hits passive
Abilities unlock per level:
  Dash L1, Surge L3, Volley L5, Phantom L6

### Greatsword (see `docs/Systems/GreatswordSystem.md`)
- Unlocks: L4 Zone E Legendary chest
- Swap: D-Pad Left
- Moveset: X = wide sweep, Y = heavy smash, RT = thrust lunge
- Spirit abilities have Greatsword-specific variants on the same binds

### Boss Grab (special)
Boss grab does NOT damage. It **staggers only**, then triggers a cinematic vault combo with handful of follow-up presses (the GoW boss-execution pattern).

---

## GoW ROADMAP DECISIONS

### Death + Respawn
- **System:** FCS native death handler + fade-to-black overlay
- **L1 scripted death:** A trigger volume early in L1 forces `HP → 0`, drops the player into the death sequence on cue (no random failure required).
- **Respawn:** at last checkpoint (see `docs/Systems/CheckpointSystem.md`). Invisible — no UI, no death screen, no menu.

### Ruined One Dialogue
- **Delivery:** Voice overlay played AFTER the death fade-to-black completes.
- **First line (L1):** *"You fight for me now."*

### Checkpoints
- 3 per level: Zone A→B, Zone D→E, pre-F.
- All invisible (mesh hidden, RotatingMovement disabled).
- See `docs/Systems/CheckpointSystem.md` for placement details.

### Win Condition
- Boss death → 1.5s slow-mo → arena barriers drop → exit door opens → player walks through → next level loads.
- **No win screen. No interruption. No "level complete" toast.** Seamless.

---

## PUZZLE ESCALATION
| Level | Puzzle |
|---|---|
| L1 | Pressure plates ×3 |
| L2 | Sequence terminals ×3 (input correct order) |
| L3 | Light beam + mirrors ×3 (redirect to receivers) |
| L4 | Conduit routing ×3 nodes (connect a circuit) |
| L5 | Timed plates + enemy pressure (combat + puzzle simultaneous) |
| L6 | Multi-room conduit across 3 rooms |
| L7 | All mechanics combined |
| L8 | None — pure boss |

Each level's Zone E **optional** puzzle (separate from the gating Zone A puzzle) awards +15 max Spirit on solve.

---

## CHEST SYSTEM (see `docs/Systems/ChestSystem.md`)
| Chest | Reward |
|---|---|
| Gold | EXP orb burst (auto-collect on proximity) |
| Silver | Full health restore (FCS `BuffComponent.Heal` — heals to full, no partial option) |
| Stat | Permanent stat upgrade (`EquipmentComponent.IncreaseLevelUpStats`) |
| Legendary | Weapon unlock + slow-mo + notification (used **only** in L4 Zone E for the Greatsword) |

Removed from earlier design: named God-of-War-themed weapons (Axe of Sparta, Apollo's Bow, Blade of Olympus). Greatsword is the only weapon unlock.

---

## STORY BEATS
| Level | Beat |
|---|---|
| L1 | "You fight for me now." — Ruined One after the scripted death |
| L2 | The gods fell because of the technology they worshipped |
| L3 | First vision of the Throne. *"It was mine once."* |
| L4 | Shinbi was chosen — no divine blood, immune to corruption |
| L5 | Ruined One betrayal hinted: he wants the Throne destroyed, not reclaimed |
| L6 | The Throne is a machine, not a seat of power |
| L7 | "You were never meant to reach this far." — Ruined One turns on Shinbi |
| L8 | Two endings — destroy the Throne, or claim it |

---

## WORKFLOW RULES (summary — see `WorkflowGuide.txt` if present)
1. **Read docs before building.** This file + `FCS_SystemReference.md` + `KnownIssues.md` first, every session.
2. **State the plan before writing nodes.** No surprise refactors.
3. **One compile attempt per approach.** If it fails, re-read and re-plan, do not retry blind.
4. **Never modify core FCS assets directly.** Always use a child Blueprint or a custom component layered on top.
5. **YELLOW escalation** at 5 tool calls with no progress → stop, summarise, ask.
6. **RED escalation** on crash, data loss, or destructive operation → stop immediately, report.

---

## DESIGN DOCS INDEX
| Topic | File |
|---|---|
| FCS audit + system reference | `docs/FCS_SystemReference.md` |
| Known bugs + workarounds | `docs/KnownIssues.md` |
| Enemy roster (per-level) | `docs/Characters/EnemyRoster.md` |
| Paragon import schedule | `docs/Assets/ParagonCharacters.md` |
| Wave system | `docs/Systems/WaveSystem.md` |
| Chest system | `docs/Systems/ChestSystem.md` |
| Checkpoint system | `docs/Systems/CheckpointSystem.md` |
| Spirit system | `docs/Systems/SpiritSystem.md` |
| Progression system | `docs/Systems/ProgressionSystem.md` |
| Greatsword system | `docs/Systems/GreatswordSystem.md` |
| FCS Blueprint inventory | `docs/FCS_Blueprints/Inventory.md` |
| FCS Blueprint deep refs | `docs/FCS_Blueprints/*.md` |
| Gap audits | `docs/FCS_Gaps/gap_audit_*.md` |
