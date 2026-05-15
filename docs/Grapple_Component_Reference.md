# Grapple Component — God of Ruin Reference
Creator: Jon Van Dam
Full docs: https://jonvandam.com/grapple-component/grapple-component/

## Overview
Skeleton-agnostic grab/grapple system. Used for Shinbi grab attacks on enemies.
Not a grappling hook. A character-to-character grab/throw/slam/execute system.

## Key Assets — Paths
GrappleComponent (main component): /Game/GrappleComponent/BP_GrappleComponent/GrappleComponent
GrappleComponent_AnimBP: /Game/GrappleComponent/BP_GrappleComponent/AnimBP/GrappleComponent_AnimBP
GrappleComponent_AnimBP_AlternateSkeleton: /Game/GrappleComponent/ContentExamples/SampleContent/AnimBP/GrappleComponent_AnimBP_AlternateSkeleton
GrappleSequenceTutorial: /Game/GrappleComponent/BP_GrappleComponent/GrappleSequenceTutorial

## Integration Pattern — User Command
The component uses a User Command pattern — NOT event-driven.
Shinbi sends a command to GrappleComponent.
GrappleComponent handles the sequence internally.

Wire grab (B button) in BP_GodOfRuin_Player:
  B Pressed → Get GrappleComponent reference → Send User Command
  Simultaneously → add Spirit +10 on button press (not on success)

## Skeleton Compatibility
GrappleComponent_AnimBP_AlternateSkeleton = skeleton agnostic ✅
Works with Shinbi's skeleton without retargeting.
Use this ABP on Shinbi — NOT the standard Mannequin ABP.

## Grab Outcomes (wire in GrappleSequenceTutorial)
Normal enemy → throw / slam / execute below 20% HP
Boss enemy → stagger only → triggers cinematic vault combo
Both outcomes wired via GrappleSequence custom events

## Grab Sequences — Example Files
CS_GrappleDamage_Default: /Game/GrappleComponent/ContentExamples/SampleContent/GrappleSequence/Common/CS_GrappleDamage_Default
CS_Impact_Kick: /Game/GrappleComponent/ContentExamples/SampleContent/GrappleSequence/Combo/CS_Impact_Kick
CS_Charge_Success: /Game/GrappleComponent/ContentExamples/SampleContent/GrappleSequence/charge/CS_Charge_Success

## Spirit Integration
Hook Spirit +10 on B button press (before command sends).
Do NOT wait for grab success event.
GrappleComponent handles success/fail internally.
Spirit deduction only on successful grab sequence if desired.
