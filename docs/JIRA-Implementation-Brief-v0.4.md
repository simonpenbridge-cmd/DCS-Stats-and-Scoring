# DCS 3.0 — Player vs AI Air-to-Air Scoring

> Snapshot of [the v0.4 live JIRA implementation brief](https://docs.google.com/document/d/1vWaoJLwBNJmNQKArwjg1slQY7EOgR-pvnM7OnytzPtc/edit) on 9 November 2026.
> **Source of truth:** The linked Google Doc and Sheets. This GitHub file is an editable review snapshot, not an automatically synchronised copy.
> **Presentation:** Tables show no more than two decimal places for numeric factors; all final kill points are integers. Computations use full-precision source values and a single final rounding step.

## DCS 3.0 — Air-to-Air Kill Scoring

*Player vs AI | JIRA implementation brief | Review v0.4 — 9 November 2026*

Status: Version 0.4 review draft. The base formula and five AI skill levels remain unchanged. The current weapon source is Weapons Catalogue v1.1, with globally consistent Phoenix factors (AAM-076 = 0.84, AAM-077 = 0.79). The reference best weapon is resolved against the actual eligible loadout rather than blindly trusting a legacy platform mapping. All other catalogue weapon values and V7.0 aircraft ratings remain as in v0.3, pending an avionics-weighting audit. Section 6.5 remains pending approval. V0.3 and the original Weapons v1.0 are preserved as historical baselines.

## 1. Purpose

Introduce a single, understandable points system for credited air-to-air kills made by human players against AI aircraft in DCS. The system must work across World War II, Cold War and modern eras without making one era intrinsically more rewarding simply because of its weapon technology.

Each qualifying kill earns points from (1) the destroyed AI pilot's skill, (2) relative aircraft capability and (3) the difficulty of the weapon actually used relative to the best weapon available to the opponent. If the AI is unarmed or has only defensive armament, compare against the attacking aircraft's own best weapon instead. This prevents an automatic WWII-gun bonus and still rewards a difficult gun kill in the jet era.

#### Table 1 — Three factors determining kill points

| Factor | What it measures | Scoring principle |
| --- | --- | --- |
| AI pilot skill | The skill level of the destroyed AI pilot | More difficult AI opponents earn more |
| Aircraft capability | Relative air-to-air capability of player and AI aircraft | Defeating a more capable aircraft earns more |
| Weapon used | Actual gun or missile credited with the kill | Difficulty compared with the AI's best weapon (or player's best for unarmed targets); neutral when equal |

## 2. Scope and principle

In scope: points for a human player shooting down an AI aircraft, provided the simulator or statistics system has identified an eligible air-to-air kill.

Out of scope for this version: player-versus-player scoring, player ELO or skill ranks, and air-to-ground scoring.

This is a points-award calculation. Kill detection, attribution, assists and disputed kills require separate event-handling rules if they are not already defined elsewhere.

The calculation must be reproducible and auditable: the inputs and each applied factor should be recoverable for any scored kill.

## 3. Scoring model

Kill points = 100 × AI skill factor × aircraft factor × RELATIVE weapon factor. The relative weapon factor is not the catalogue multiplier itself.

#### Table 2 — Scoring factors and catalogue references

| Element | Rule / source | Interpretation |
| --- | --- | --- |
| Base score | 100 points | Before the three adjustments |
| AI skill factor | Cadet 0.60; Rookie 0.80; Trained 1.00; Veteran 1.20; Ace 1.40 | Applied to the destroyed AI aircraft |
| Aircraft factor | 2 ÷ [1 + 10^((player rating − AI rating)/400)] | Equals 1.00 when aircraft ratings are equal; >1 when AI aircraft is higher rated; <1 when player aircraft is higher rated |
| Relative weapon factor | 1 + 0.50 × (used scoring multiplier − reference scoring multiplier) | Reference = AI best weapon; if unarmed or defensive-only, player best. Equal weapons → 1.00. |
| Raw weapon difficulty | Weapons Catalogue v1.1 — Raw difficulty | The catalogue scoring multiplier is an input to the new relative comparison, not the final kill factor. |

#### Table 3 — Weapon multiplier examples (v1.1)

| Weapon | Raw difficulty | Scoring multiplier |
| --- | --- | --- |
| M61 Vulcan 20 mm cannon | 1.40 | 1.20 |
| AIM-9B Sidewinder | 2.18 | 1.59 |
| AIM-9L Sidewinder | 1.00 | 1.00 |
| AIM-9X Block I | 0.62 | 0.81 |
| AIM-7D Sparrow | 1.92 | 1.46 |
| AIM-7E Sparrow | 1.82 | 1.41 |
| AIM-7F Sparrow | 1.24 | 1.12 |
| AIM-120C AMRAAM | 0.43 | 0.72 |
| AIM-54A Phoenix (AAM-076) | 0.68 | 0.84 |
| AIM-54C Phoenix (AAM-077) | 0.58 | 0.79 |

### 3.1 — Relative weapon comparison (retained scoring rule)

Every distinct weapon variant/ID has one global catalogue scoring multiplier M, from Weapons Catalogue v1.1, independent of its launching aircraft. Smaller M indicates a more effective/easier-to-use weapon according to this scoring model. The AIM-9L catalogue anchor remains M = 1.000. The catalogue's 50% raw-to-scoring compression formula is unchanged; the AIM-54A and AIM-54C raw difficulty values have been revised to yield their new global scoring multipliers.

Define M_used = catalogue scoring multiplier of the weapon actually credited with the kill. Define M_reference = catalogue scoring multiplier of the reference best weapon selected for the aircraft comparison.

RELATIVE WEAPON FACTOR = 1 + 0.50 × (M_used − M_reference).

FINAL KILL POINTS = ROUND[100 × AI skill factor × aircraft factor × relative weapon factor]. Keep full precision until the final rounding step. Presentation standard: display numeric weapon multipliers and aircraft/relative factors in tables to two decimal places. This is DISPLAY ROUNDING ONLY; engine calculations use unrounded source multipliers and the full-precision aircraft factor, then round the final award once to an integer.

Reference selection (v0.4): if the AI target is armed offensively, identify its **actual mounted, period-valid offensive A2A weapons**; use the exact catalogue ID with the **lowest v1.1 scoring multiplier** as M_reference (that is, the most effective according to the catalogue). If the AI is unarmed or defensive-only, use the same rule against the attacking player's **actual equipped offensive A2A weapons**. The Aircraft V7.0 static BEST A2A WEAPON (CATALOGUE ID) is provisional, loadout-independent legacy metadata and may be used only where independently verified for the explicit test configuration. If an exact loadout cannot be established, flag the case for review; never silently substitute a generic weapon.

Interpretation: if both weapons are equally effective (same multiplier), relative weapon factor = 1.000 regardless of historical era. A harder-to-use weapon has a higher multiplier and earns a bonus; an easier-to-use weapon earns less. AI skill and aircraft capability remain the main determinants of the score.

Implementation: compare the already-compressed catalogue scoring multipliers, NOT the raw difficulty values. The 0.50 in this relative rule weights the difference between two weapons; it is not a second compression of the raw catalogue value. There is no separate WWII/Cold War/modern-era weapon or points multiplier.

Validation: resolve each exact AAM-### catalogue ID; do not infer a weapon from an unspecified generic family. Confirm applicable DCS aircraft variant, era and loadout. If either required weapon ID is missing, uncatalogued or only a proxy without an accepted mapping, flag the calculation for review rather than invent a neutral replacement. Section 6.4.3 documents the fallback and tests.

### 3.2 — Global weapon identity and loadout-sensitive reference (v0.4)

**One exact weapon variant, one catalogue multiplier worldwide.** AAM-044 AIM-9M has multiplier 0.94 whether fired from an F-14, F-15, F-16 or another supported aircraft. AAM-018 M61 is 1.20, AAM-076 AIM-54A is 0.84 and AAM-077 AIM-54C is 0.79 on every platform. Differences in *final points* arise from AI skill, relative aircraft ratings and the target's actual reference weapon, **not** from a platform-specific copy of the same missile.

**Reference is determined from loadout, not aircraft type alone.** For an F-14 AI carrying AIM-54C, AIM-54A and AIM-9M, its best weapon under catalogue v1.1 is AIM-54C (0.79). If carrying AIM-54A but not C, it is AIM-54A (0.84). If carrying only AIM-9M (and guns/other weapons with larger multipliers), it is AIM-9M (0.94). Historical/mission restrictions decide whether a weapon is eligible. Never select uncarried weapons.

**Computation sequence:** resolve credited weapon exact AAM ID → enumerate relevant AI loaded offensive A2A AAM IDs (or player IDs for unarmed/defensive-only AI) → choose lowest catalogue multiplier → compute full-precision relative weapon factor → compute aircraft factor from ratings → apply AI skill → round final nonnegative points once. If the loaded reference cannot be resolved, retain the kill event for audit and do not award a fabricated neutral value.

**Avionics versus weapon difficulty:** sensor detection, target discrimination, radar track capability, electronic-warfare resilience, fire-control and integration of a platform's era-appropriate weapons belong in the **aircraft capability audit**, not in multiple carrier-specific missile multipliers. Individual gun/missile multipliers express *relative kill difficulty* and retain global identity. A platform loadout may justify a future distinct **configuration rating**, but does not change an AAM ID's catalogue multiplier.

**Review boundary:** the values 830 (F-14A Phoenix-equipped), 780 (MiG-29A), 765 (original F-16A) remain **sensitivity-trial candidate ratings only**, not replacements for current V7.0. The comparison exposes a second inconsistency: V7.0 currently lists F-14B 805, so no arbitrary F-14A uplift can be adopted before adjacent aircraft are recalibrated.

## 4. AI skill factors — fixed mapping

#### Table 4 — AI skill multipliers

| AI skill level | Multiplier |
| --- | --- |
| Cadet | 0.60 |
| Rookie | 0.80 |
| Trained | 1.00 |
| Veteran | 1.20 |
| Ace | 1.40 |

## 5. First worked scenario — isolate AI skill

Keep every variable unchanged except AI skill. In this identical-aircraft/identical-weapon example, the catalogue factors cancel and the relative weapon factor is 1.000.

#### Table 5 — First worked example: fixed inputs

| Fixed input | Value |
| --- | --- |
| Player aircraft | Spitfire Mk IX |
| AI aircraft | Spitfire Mk IX (same aircraft rating) |
| Weapon used | Hispano Mk II 20 mm cannon (AAM-008) |
| Raw weapon difficulty | 1.80 |
| Used / AI best catalogue multipliers | 1.40 / 1.40 |
| Aircraft / relative weapon factors | 1.00 / 1.00 |
| Only variable across test cases | AI skill |

#### Table 6 — First worked example: points by AI skill

| AI skill | Skill multiplier | Points awarded |
| --- | --- | --- |
| Cadet | 0.60 | 60 |
| Rookie | 0.80 | 80 |
| Trained | 1.00 | 100 |
| Veteran | 1.20 | 120 |
| Ace | 1.40 | 140 |

Acceptance expectation: Cadet 60, Rookie 80, Trained 100, Veteran 120 and Ace 140 points. Only AI skill changes; both aircraft and relative weapon factors remain 1.000.

## 6. Proposed sequence for the rest of the JIRA specification

Baseline test: identical aircraft, identical weapon, five AI levels (Section 5).

Aircraft comparison: hold AI skill and weapon constant; vary relative aircraft capability.

Weapon comparison: hold player aircraft, AI aircraft and AI skill constant; change weapon.

Combined examples: test WWII, Cold War and modern kills using the complete formula.

Implementation checks: lookup by weapon ID, missing data, consistent rounding, event attribution and score breakdown for audit.

### 6.1 — Same aircraft, same weapon, different AI skill levels (SCENARIO APPROVED; RELATIVE SCORES FOR REVIEW)

Purpose: demonstrate the effect of AI pilot skill while holding player and AI aircraft, weapon and relative weapon factor fixed. Both Spitfires have Hispano Mk II as their best weapon, so the relative factor is 1.000.

#### Table 7 — Section 6.1: fixed inputs

| Fixed input | Value |
| --- | --- |
| Player aircraft | Spitfire Mk IX |
| AI aircraft | Spitfire Mk IX |
| Weapon | Hispano Mk II 20 mm cannon (AAM-008) |
| Aircraft factor | 1.00 |
| Used / AI best catalogue multipliers | 1.40 / 1.40 |
| Aircraft / relative weapon factors | 1.00 / 1.00 |

#### Table 8 — Section 6.1: scoring acceptance cases

| AI skill | Skill factor | Points awarded |
| --- | --- | --- |
| Cadet | 0.60 | 60 |
| Rookie | 0.80 | 80 |
| Trained | 1.00 | 100 |
| Veteran | 1.20 | 120 |
| Ace | 1.40 | 140 |

Acceptance test: the five scores must be 60, 80, 100, 120 and 140. The AI skill belongs to the destroyed AI pilot, not to the human player's rank. Aircraft factor = 1.000 and relative weapon factor = 1.000 throughout.

Decision: scenario design approved on 6 November 2026; numerical expectations recalculated against Weapons v1.0 on 9 November 2026, pending review.

### 6.2 — Different AI aircraft, fixed player weapon and AI skill (SCENARIO APPROVED; RELATIVE SCORES FOR REVIEW)

Tables 9 and 10. Purpose: show how the score varies when the same F-4E (rating 740) fires an AIM-7E against different Trained AI aircraft. The player, actual weapon and AI skill remain fixed; the AI aircraft rating AND its best available weapon necessarily change with the opponent. Both factors must be recalculated for each kill.

#### Table 9 — Section 6.2: fixed inputs

| Fixed input | Value |
| --- | --- |
| Player aircraft | F-4E Phantom II (late), aircraft rating 740 |
| Weapon | AIM-7E Sparrow (AAM-063) |
| Raw weapon difficulty | 1.82 |
| Catalogue multiplier (weapon used) | 1.41 |
| AI skill | Trained, 1.00 |
| Base score | 100 points |
| Aircraft factor formula | 2 ÷ [1 + 10^((player aircraft rating − AI aircraft rating)/400)] |

#### Table 10 — Section 6.2: F-4E firing AIM-7E against three AI aircraft

| AI aircraft / best weapon | AI rating | Aircraft × relative weapon factor | Points awarded |
| --- | --- | --- | --- |
| MiG-21bis; R-60M (1.04) | 710 | 0.91 × 1.19 | 108 |
| F-4E late; AIM-7M (1.05) | 740 | 1.00 × 1.18 | 118 |
| F-16C; AIM-120C (0.72) | 875 | 1.37 × 1.35 | 185 |

Acceptance test: F-4E firing AIM-7E (catalogue multiplier 1.410) must earn 108 points against MiG-21bis (best R-60M, 1.040), 118 against F-4E late (best AIM-7M, 1.045), and 185 against F-16C Block 50/52 (best AIM-120C, 0.715). AI skill stays Trained (1.00). Aircraft and relative weapon factors are recalculated for each AI type; use final-only rounding.

Aircraft-rating source: A2A V7.0 — MiG-25/31, Su-15 and Fitter Variants, Sheet1, A2A Rating column (MiG-21bis 710, F-4E late 740, F-16C Block 50/52 875).

Decision: scenario design approved on 6 November 2026; numerical expectations recalculated against Weapons v1.0 on 9 November 2026, pending review.

### 6.3 — Same aircraft, same AI skill, different weapons (SCENARIO APPROVED; RELATIVE SCORES FOR REVIEW)

Tables 11 and 12. Purpose: isolate actual weapon choice against one fixed AI opponent. Player MiG-21bis, AI F-4E late (best AIM-7M, 1.045), and Trained AI remain fixed. Only the weapon used changes; its catalogue multiplier is compared with the AI best to obtain the relative weapon factor.

#### Table 11 — Section 6.3: fixed inputs

| Fixed input | Value |
| --- | --- |
| Player aircraft | MiG-21bis, rating 710 |
| AI aircraft | F-4E Phantom II (late), rating 740 |
| AI skill | Trained, 1.00 |
| AI best A2A weapon | AIM-7M Sparrow (1.05) |
| Base score | 100 points |

#### Table 12 — Section 6.3: weapon comparison - Player in MiG-21

| Weapon / ID | Raw difficulty | Catalogue multiplier | Relative factor → points |
| --- | --- | --- | --- |
| R-60M (AAM-045) | 1.08 | 1.04 | 1.00 → 108 |
| K-13 / R-3S Atoll (AAM-035) | 2.20 | 1.60 | 1.28 → 139 |
| GSh-23L 23 mm cannon (AAM-014) | 1.58 | 1.29 | 1.12 → 122 |

Acceptance test: the R-60M, K-13/R-3S and GSh-23L kills must score 108, 139 and 122 respectively. The AI best weapon is AIM-7M (1.045), unchanged across tests. Use full-precision aircraft/weapon calculations and round the final score once.

Decision: scenario design approved on 6 November 2026; numerical expectations recalculated against Weapons v1.0 on 9 November 2026, pending review.

### 6.4 — Complete scoring across three eras (SCENARIO APPROVED; RELATIVE SCORES FOR REVIEW)

Purpose: apply the relative-weapon formula to one WWII, one Cold War and one modern engagement. All AI opponents are Trained (1.00). In each case the player flies the lower-rated aircraft; the AI's best weapon is used as the reference, irrespective of the era.

Table 13 shows the aircraft and weapon choices, including the separately calculated relative weapon factor. Each AI's best weapon is: Bf 109K-4 → MK 108 (1.500); F-4E late → AIM-7M (1.045); F-16C Block 50/52 → AIM-120C (0.715).

#### Table 13 — Complete scoring across three eras

|  | WWII | Cold War | Modern |
| --- | --- | --- | --- |
| Player aircraft | Spitfire IX | MiG-21bis | Su-27 |
| Player aircraft rating | 417 | 710 | 830 |
| AI aircraft | Bf 109K-4 | F-4E late | F-16C Block 50/52 |
| AI aircraft rating | 424 | 740 | 875 |
| Weapon | Hispano Mk II | R-60M | R-27ER |
| AI skill | Trained | Trained | Trained |
| Skill multiplier | 1.00 | 1.00 | 1.00 |
| Aircraft multiplier | 1.02 | 1.09 | 1.13 |
| Relative weapon factor | 0.95 | 1.00 | 1.20 |
| Points awarded | 97 | 108 | 135 |

Acceptance test: WWII 97 points; Cold War 108 points; Modern 135 points. These derive from relative weapon factors 0.9500, 0.9975 and 1.1975 respectively. No independent WWII, Cold War or modern-era multiplier is permitted.

These are aircraft capability ratings, not human-player skill or ELO ratings. Only the destroyed AI aircraft carries an AI skill multiplier.

Decision: scenario design and aircraft ratings retain prior approval; all displayed numerical expectations are recalculated for relative weapon scoring (v0.3 review).

### 6.4.1 — Reversed engagements: best available weapons (SCENARIO APPROVED; RELATIVE SCORES FOR REVIEW)

Purpose: reverse the original player and AI aircraft in all five examples. Keep AI skill at Trained (1.00). Use the specified actual player weapon and the best weapon recorded for the newly destroyed AI aircraft. Aircraft ratings measure platform capability, not player skill; all scores now include the relative weapon factor.

#### Table 14 — Within-era reversed match-ups

|  | WWII | Cold War | Modern |
| --- | --- | --- | --- |
| Player aircraft | Bf 109K-4 | F-4E late | F-16C Block 50/52 |
| Player aircraft rating | 424 | 740 | 875 |
| AI aircraft | Spitfire IX | MiG-21bis | Su-27 baseline |
| AI aircraft rating | 417 | 710 | 830 |
| Best weapon | MK 108 30 mm | AIM-7M Sparrow | AIM-120C AMRAAM |
| AI skill | Trained | Trained | Trained |
| Skill multiplier | 1.00 | 1.00 | 1.00 |
| Aircraft multiplier | 0.98 | 0.91 | 0.87 |
| Relative weapon factor | 1.05 | 1.00 | 0.92 |
| Points awarded | 103 | 92 | 80 |

#### Table 15 — Original and reversed match-ups, including cross-era comparisons

| Match-up | Earlier direction (points) | Reversed direction (points) | Reversed weapon |
| --- | --- | --- | --- |
| Bf 109K-4 / Spitfire IX | Spitfire IX → Bf 109K-4: 97 | Bf 109K-4 → Spitfire IX: 103 | MK 108 |
| F-4E / MiG-21bis | MiG-21bis → F-4E: 108 | F-4E → MiG-21bis: 92 | AIM-7M |
| F-16C / Su-27 | Su-27 → F-16C: 135 | F-16C → Su-27: 80 | AIM-120C |
| MiG-21bis / F-16C | MiG-21bis → F-16C: 168 | F-16C → MiG-21bis: 47 | AIM-120C |
| F-4E / Su-27 | F-4E → Su-27: 159 | Su-27 → F-4E: 77 | R-27ER |

Programming interpretation: there is no separate era adjustment. With Trained AI, MiG-21bis using R-60M against F-16C earns 168 versus 47 for F-16C using AIM-120C against MiG-21bis. F-4E using AIM-7E against Su-27 earns 159 versus 77 for Su-27 using R-27ER against F-4E. Both relative aircraft ratings and the target's best weapon drive these results.

Acceptance tests: with Trained AI, reversed within-era scores are 103 (Bf 109K-4 vs Spitfire IX), 92 (F-4E vs MiG-21bis) and 80 (F-16C vs Su-27). Reversed cross-era scores are 47 (F-16C vs MiG-21bis) and 77 (Su-27 vs F-4E). Round only the final product.

Decision: the reversed engagement definitions remain approved; their relative weapon scores are recalculated in the v0.3 review. AI skill and aircraft ratings are unchanged.

### 6.4.2 — Aircraft-independent weapon catalogue comparison (REFERENCE ONLY)

Purpose: compare the v1.1 catalogue multipliers directly, regardless of aircraft. Tables 16 and 17 are REFERENCE INDICES, not kill points, and deliberately omit the opposing aircraft's weapon. They must never be used as final PvAI kill awards. Real kills use the relative weapon factor in Section 3.

#### Table 16 — Weapon comparison: cannon, Sidewinder and Sparrow

Reference-only calculation: catalogue index = 100 × catalogue scoring multiplier. This is a visual comparison of weapon difficulty, not an award for a kill. AI skill and aircraft values are intentionally omitted.

| Weapon used | Weapon ID | Scoring multiplier | Catalogue index (NOT kill points) |
| --- | --- | --- | --- |
| M61 Vulcan cannon | AAM-018 | 1.20 | 120 |
| AIM-9B Sidewinder | AAM-032 | 1.59 | 159 |
| AIM-9L Sidewinder | AAM-043 | 1.00 | 100 |
| AIM-9X Block I | AAM-054 | 0.81 | 81 |
| AIM-7D Sparrow | AAM-062 | 1.46 | 146 |
| AIM-7E Sparrow | AAM-063 | 1.41 | 141 |
| AIM-7F Sparrow | AAM-069 | 1.12 | 112 |

#### Table 17 — Weapon comparison: modern missile options

The same catalogue-only reference convention applies. AIM-9L remains the 1.000 raw catalogue reference; this is not a claim that all modern kills earn fewer points than WWII kills.

| Weapon used | Weapon ID | Scoring multiplier | Catalogue index (NOT kill points) |
| --- | --- | --- | --- |
| M61 Vulcan cannon | AAM-018 | 1.20 | 120 |
| AIM-9L Sidewinder | AAM-043 | 1.00 | 100 |
| AIM-9X Block I | AAM-054 | 0.81 | 81 |
| AIM-120C AMRAAM | AAM-080 | 0.72 | 71.5 |

Reference-index checks (NOT kill points): M61 120, AIM-9B 159, AIM-9L 100, AIM-9X Block I 81, AIM-7D 146, AIM-7E 141, AIM-7F 112 and AIM-120C 71.5. These numbers are a catalogue scale only; actual kill awards require the AI's best weapon (or the unarmed/defensive-only fallback).

Implementation interpretation: the current Weapons Catalogue v1.1 defines the catalogue scoring multiplier as 1 + 0.50 × (raw weapon difficulty − 1). The NEW relative factor is separately 1 + 0.50 × (used catalogue multiplier − reference catalogue multiplier). The second 0.50 weights the difference between two weapons; it is not a second compression of the same raw catalogue value. Tables 16–17 are deliberately aircraft-independent references; for a DCS kill apply the relative factor after the AI skill and aircraft factors, and confirm the actual weapon ID and aircraft fit.

Decision: catalogue comparison values are retained for reference, but were never intended as awarded kill points. Final v0.4 kill awards use the approved relative comparison principle; acceptance scores remain for review.

### 6.4.3 — Unarmed and defensive-only AI targets (NEW ACCEPTANCE TEST)

An unarmed target has no best offensive weapon to compare. For both unarmed and defensive-only AI targets, set M_reference to the lowest multiplier among the attacking player's actually equipped eligible offensive A2A weapons instead. This preserves the value of the AI's aircraft rating and skill, while offering an extra reward for using a less effective weapon than the player's best. Defensive turret weapons are not treated as offensive fighter armament under this rule.

#### Table 18 — Selecting the reference weapon by AI target armament

| AI target armament | Reference weapon used | Scoring / exception rule |
| --- | --- | --- |
| Armed (offensive A2A) | Lowest-multiplier eligible weapon actually mounted on AI | Compare credited player weapon against the selected exact AI AAM ID; do not infer uncarried weapons |
| Unarmed | Lowest-multiplier eligible weapon actually mounted on player | Never invent a weapon for target; normal aircraft and AI-skill factors still apply |
| Defensive turrets only | Lowest-multiplier eligible weapon actually mounted on player | Defensive guns do not become an offensive best A2A weapon |
| Armed, but target's offensive loadout or exact weapon ID unresolved | No automatic reference | Flag for review; do NOT assume the aircraft is unarmed or default multiplier = 1 |
| Credited player weapon ID unresolved | No valid used-weapon factor | Retain kill event for later reconciliation; do NOT assign a guessed factor |

Worked test: a Trained F-111F AI (rating 640; unarmed) is destroyed by an F-16C Block 50/52 player (rating 875; best AIM-120C, AAM-080, multiplier 0.715). Aircraft factor = 0.410836170. The F-111F has no weapon ID and contributes no numerical weapon multiplier; the attacking F-16C's AIM-120C is the fallback reference.

#### Table 19 — F-16C versus unarmed F-111F: relative-weapon points

| F-16C weapon used | Catalogue multiplier | Relative weapon factor | Kill points |
| --- | --- | --- | --- |
| M61 Vulcan (AAM-018) | 1.20 | 1.24 | 51 |
| AIM-9L (AAM-043) | 1.00 | 1.14 | 47 |
| AIM-9X Block I (AAM-054) | 0.81 | 1.05 | 43 |
| AIM-120C (AAM-080) | 0.72 | 1.00 | 41 |

Acceptance test: M61 Vulcan 51 points, AIM-9L 47, AIM-9X Block I 43 and AIM-120C 41. All assume Trained AI. The AIM-120C kill is neutral in weapon terms (relative factor 1.000), and using the M61 cannon earns a modest premium (relative factor 1.2425).

Edge-case rules: defensive-only bomber aircraft use the same fallback. A missing catalogue ID on an otherwise armed aircraft is NOT an unarmed classification; leave the score pending review, retaining the event for later reconciliation. Do not substitute 0, 1.00 or an unrelated gun/missile multiplier. A missing credited player weapon is likewise an unresolved event, not an automatic neutral weapon factor.

Decision: unarmed and defensive-only fallback uses the player's best weapon; all numerical examples and exception paths in this section are proposed v0.4 programmer acceptance tests.

## 7. Source of truth and open implementation decisions

- A2A Weapons Catalogue v1.1 (authoritative scoring for this v0.4 draft): https://docs.google.com/spreadsheets/d/1k89dwrlw7CEEDyPfPrMp6X3TM9Pu0PX_nT7xDa2PUBc/edit — Weapon Catalogue tab, fields “Raw difficulty” and “Scoring multiplier” (50%-compressed).

- DCS scoring notes: https://github.com/simonpenbridge-cmd/DCS-Stats-and-Scoring (background and decision history).

- Aircraft rating and best-weapon source: A2A V7.0, Sheet1, fields “A2A Rating” and “BEST A2A WEAPON (CATALOGUE ID)”: https://docs.google.com/spreadsheets/d/1Whm0B302ITesvfZ9S_Z-5VWFTclyoMdodv5AnfAAb6s/edit. Treat the static AAM-### best-weapon mapping only as a provisional legacy hint; v0.4 must enumerate the target's actual available weapons and choose the lowest v1.1 multiplier (with player-loadout fallback for unarmed/defensive-only AI). Exact DCS IDs and loadout observability still need implementation confirmation.

- Implementation convention for review: retain full floating-point precision through the catalogue lookup, relative weapon comparison and aircraft factor. Round nonnegative final kill points once to the nearest integer; award once per qualifying kill. Missing/unmapped/uncatalogued weapon IDs must be reported for review, not silently given a fabricated neutral factor. Event attribution and final rounding behaviour still require implementation sign-off.

- This v0.4 review succeeds v0.3 while preserving it separately; v0.3 itself replaced earlier v0.2 calculations after approval. The complete pre-change v0.2 brief is preserved at https://docs.google.com/document/d/1k44JNBZC8XG1ma6XtEB5mSW9J9Y4Prr_gGj8IUeYJF4/edit. Section 6.5 remains draft/unapproved.

## 8. Annexes — Era-by-era scoring acceptance tests

Planned sequence: Annex A — World War II; Annex B — Korea; Annex C — Vietnam War (Early Cold War); Annex D — Late Cold War; Annex E — Early Modern; Annex F — Modern. Each annex tests representative aircraft, weapon choices and the five AI skill levels. Annex A covers Spitfire IX versus Bf 109K-4; Annex B covers F-86F and P-51D versus MiG-15bis; Annex C covers all seven catalogue-listed Vietnam-era F-4E air-to-air weapons against MiG-21bis. Annex D is populated; Annexes E–F remain to be selected.

### Annex A — World War II: Spitfire IX versus Bf 109K-4

Method: Points = ROUND(100 × AI skill factor × aircraft factor × relative weapon factor). Relative weapon factor = 1 + 0.50 × (catalogue multiplier of weapon used − catalogue multiplier of AI best weapon). Human player skill does not enter this PvAI calculation. Round only once at the end.

#### Annex A.1 — Human Spitfire IX shoots down AI Bf 109K-4

Fixed inputs: Human player Spitfire IX (aircraft rating 417), using Hispano Mk II 20 mm cannon (AAM-008; catalogue multiplier 1.400). AI target Bf 109K-4 (rating 424); its best weapon is the MK 108 30 mm cannon (AAM-019; catalogue multiplier 1.500). Aircraft factor 1.020144894; relative weapon factor 0.9500. Only AI pilot skill changes.

#### Table 20 — Annex A.1: Spitfire IX human versus Bf 109K-4 AI

| AI pilot skill | Skill factor | Aircraft factor | Relative weapon factor | Points awarded |
| --- | --- | --- | --- | --- |
| Cadet | 0.60 | 1.02 | 0.95 | 58 |
| Rookie | 0.80 | 1.02 | 0.95 | 78 |
| Trained | 1.00 | 1.02 | 0.95 | 97 |
| Veteran | 1.20 | 1.02 | 0.95 | 116 |
| Ace | 1.40 | 1.02 | 0.95 | 136 |

#### Annex A.2 — Human Bf 109K-4 shoots down AI Spitfire IX

Fixed inputs: Human player Bf 109K-4 (aircraft rating 424), using MK 108 30 mm cannon (AAM-019; catalogue multiplier 1.500). AI target Spitfire IX (rating 417); its best weapon is the Hispano Mk II 20 mm cannon (AAM-008; catalogue multiplier 1.400). Aircraft factor 0.979855106; relative weapon factor 1.0500. Only AI pilot skill changes.

#### Table 21 — Annex A.2: Bf 109K-4 human versus Spitfire IX AI

| AI pilot skill | Skill factor | Aircraft factor | Relative weapon factor | Points awarded |
| --- | --- | --- | --- | --- |
| Cadet | 0.60 | 0.98 | 1.05 | 62 |
| Rookie | 0.80 | 0.98 | 1.05 | 82 |
| Trained | 1.00 | 0.98 | 1.05 | 103 |
| Veteran | 1.20 | 0.98 | 1.05 | 123 |
| Ace | 1.40 | 0.98 | 1.05 | 144 |

Acceptance test: Spitfire IX human scores Cadet 58, Rookie 78, Trained 97, Veteran 116, Ace 136 points. Bf 109K-4 human scores Cadet 62, Rookie 82, Trained 103, Veteran 123, Ace 144 points. The five skill multipliers apply to the destroyed AI only; weapon/aircraft factors remain constant within each direction. No separate era or human-skill multiplier is permitted.

Status: Annex A is proposed for programmer acceptance testing under the v0.3 relative-weapon model. Aircraft ratings and catalogue weapon factors are unchanged; Annexes B and C follow below, while Annexes D–F remain unpopulated.

### Annex B — Korea: F-86F Sabre, MiG-15bis and P-51D Mustang

Four test directions use the same v0.3 relative-weapon formula and five AI skill factors. Aircraft ratings: F-86F Sabre 610; MiG-15bis 605; P-51D Mustang 410. Best weapon catalogue IDs and multipliers: F-86F and P-51D — AAM-005 Browning M2 .50 machine gun, 1.475; MiG-15bis — AAM-015 NR-23 23 mm cannon, 1.375.

Catalogue qualification: the aircraft sheet currently maps the Sabre's six .50-calibre guns to AAM-005 Browning M2 as a scoring proxy. Exact F-86F gun-model differences should be checked separately; this annex intentionally follows the existing approved-for-testing catalogue mapping rather than silently altering weapon values. The MiG-15bis's 37 mm N-37 is a second cannon, but AAM-015 NR-23 is the best-weapon selection in the present aircraft sheet.

#### Annex B.1 — Human F-86F Sabre shoots down AI MiG-15bis

Fixed inputs: player F-86F (rating 610) uses the catalogue .50 machine gun (AAM-005; multiplier 1.475). AI target MiG-15bis (rating 605) has NR-23 (AAM-015; multiplier 1.375) as its best weapon. Aircraft factor = 0.985609837; relative weapon factor = 1.0500. Only the destroyed AI's skill changes.

#### Table 22 — Annex B.1: F-86F human versus MiG-15bis AI

| AI pilot skill | Skill factor | Aircraft factor | Relative weapon factor | Points awarded |
| --- | --- | --- | --- | --- |
| Cadet | 0.60 | 0.99 | 1.05 | 62 |
| Rookie | 0.80 | 0.99 | 1.05 | 83 |
| Trained | 1.00 | 0.99 | 1.05 | 103 |
| Veteran | 1.20 | 0.99 | 1.05 | 124 |
| Ace | 1.40 | 0.99 | 1.05 | 145 |

#### Annex B.2 — Human MiG-15bis shoots down AI F-86F Sabre

Fixed inputs: player MiG-15bis (rating 605) uses NR-23 (AAM-015; multiplier 1.375). AI target F-86F (rating 610) has its catalogue .50 machine gun (AAM-005; multiplier 1.475) as best weapon. Aircraft factor = 1.014390163; relative weapon factor = 0.9500. Only AI skill changes.

#### Table 23 — Annex B.2: MiG-15bis human versus F-86F AI

| AI pilot skill | Skill factor | Aircraft factor | Relative weapon factor | Points awarded |
| --- | --- | --- | --- | --- |
| Cadet | 0.60 | 1.01 | 0.95 | 58 |
| Rookie | 0.80 | 1.01 | 0.95 | 77 |
| Trained | 1.00 | 1.01 | 0.95 | 96 |
| Veteran | 1.20 | 1.01 | 0.95 | 116 |
| Ace | 1.40 | 1.01 | 0.95 | 135 |

#### Annex B.3 — Human P-51D Mustang shoots down AI MiG-15bis

Fixed inputs: player P-51D (rating 410) uses Browning M2 .50 machine gun (AAM-005; multiplier 1.475). AI target MiG-15bis (rating 605) has NR-23 (AAM-015; multiplier 1.375) as best weapon. Aircraft factor = 1.508908084; relative weapon factor = 1.0500. Only AI skill changes.

#### Table 24 — Annex B.3: P-51D human versus MiG-15bis AI

| AI pilot skill | Skill factor | Aircraft factor | Relative weapon factor | Points awarded |
| --- | --- | --- | --- | --- |
| Cadet | 0.60 | 1.51 | 1.05 | 95 |
| Rookie | 0.80 | 1.51 | 1.05 | 127 |
| Trained | 1.00 | 1.51 | 1.05 | 158 |
| Veteran | 1.20 | 1.51 | 1.05 | 190 |
| Ace | 1.40 | 1.51 | 1.05 | 222 |

#### Annex B.4 — Human MiG-15bis shoots down AI P-51D Mustang

Fixed inputs: player MiG-15bis (rating 605) uses NR-23 (AAM-015; multiplier 1.375). AI target P-51D (rating 410) has Browning M2 .50 machine gun (AAM-005; multiplier 1.475) as best weapon. Aircraft factor = 0.491091916; relative weapon factor = 0.9500. Only AI skill changes.

#### Table 25 — Annex B.4: MiG-15bis human versus P-51D AI

| AI pilot skill | Skill factor | Aircraft factor | Relative weapon factor | Points awarded |
| --- | --- | --- | --- | --- |
| Cadet | 0.60 | 0.49 | 0.95 | 28 |
| Rookie | 0.80 | 0.49 | 0.95 | 37 |
| Trained | 1.00 | 0.49 | 0.95 | 47 |
| Veteran | 1.20 | 0.49 | 0.95 | 56 |
| Ace | 1.40 | 0.49 | 0.95 | 65 |

Acceptance tests, listed Cadet / Rookie / Trained / Veteran / Ace: F-86F → MiG-15bis: 62 / 83 / 103 / 124 / 145; MiG-15bis → F-86F: 58 / 77 / 96 / 116 / 135; P-51D → MiG-15bis: 95 / 127 / 158 / 190 / 222; MiG-15bis → P-51D: 28 / 37 / 47 / 56 / 65. Keep the aircraft and relative weapon factors constant within each direction, use the full-precision formula, and round only at the end.

Status: Annex B is a proposed Korea-era programmer acceptance test under the v0.3 relative-weapon model. Annexes A and the main specification remain unchanged. The Sabre .50-calibre gun mapping is provisional, as described above. Annex C follows below; Annexes D–F remain unpopulated.

### Annex C — Vietnam War (Early Cold War): F-4E versus MiG-21bis

Scope: calculate a human F-4E Phantom II using all seven Vietnam-era air-to-air weapons represented in Weapons Catalogue v1.0 against a MiG-21bis AI at each of the five skill levels. The comparison concerns a credited air-to-air kill only; bombs, unguided rockets, air-to-ground missiles and weapon combinations do not count as separate A2A kill weapons.

Historical qualification: use the existing catalogue aircraft F-4E late (rating 740) and MiG-21bis (rating 710) as a deliberately standardised model matchup. The MiG-21bis and its R-60M best-weapon reference are later than the MiG-21PF/PFM/MF equipment characteristic of North Vietnamese combat; these are acceptance-test inputs, NOT a claim of a historically exact Vietnam engagement. The F-4E aircraft rating and its later AIM-7M best-weapon field are not a reason to permit postwar missiles in this Vietnam-era test.

Weapon selection: the F-4E's internal M61 cannon; AIM-4D Falcon (compatible, of limited/uncertain combat use on F-4E in Vietnam); AIM-9B, AIM-9E and AIM-9J Sidewinders; AIM-7D and AIM-7E Sparrows. The AIM-9J belongs to the late war. AIM-7E-2 is an improved AIM-7E family variant without its own v1.0 catalogue record, so AIM-7E is used as the provisional family scoring entry rather than inventing an eighth multiplier. Exclude later AIM-7F/M, AIM-9L/M and AIM-9P from this 1968–1973 scope.

#### Annex C.1 — Fixed inputs and relative weapon factors

Player aircraft: F-4E late (rating 740). AI target: MiG-21bis (rating 710), whose catalogue best weapon is R-60M (AAM-045, source multiplier 1.040). AI pilot skill varies: Cadet 0.60, Rookie 0.80, Trained 1.00, Veteran 1.20, Ace 1.40. Aircraft factor = 0.9138670159555764 internally (0.91 displayed). For each weapon, relative factor = 1 + 0.50 × (weapon source multiplier − 1.040). Tables display two decimals but calculations use unrounded catalogue values.

#### Table 26 — Annex C.1: F-4E Vietnam-era weapons and relative factors

| F-4E weapon | Weapon ID | Catalogue multiplier | Relative weapon factor |
| --- | --- | --- | --- |
| M61 Vulcan | AAM-018 | 1.20 | 1.08 |
| AIM-4D Falcon | AAM-037 | 1.83 | 1.39 |
| AIM-9B Sidewinder | AAM-032 | 1.59 | 1.28 |
| AIM-9E Sidewinder | AAM-034 | 1.43 | 1.19 |
| AIM-9J Sidewinder | AAM-038 | 1.28 | 1.12 |
| AIM-7D Sparrow | AAM-062 | 1.46 | 1.21 |
| AIM-7E Sparrow | AAM-063 | 1.41 | 1.19 |

#### Annex C.2 — Awarded points by AI pilot level

Each row holds the player aircraft, target aircraft and weapon fixed, changing only the destroyed MiG-21bis AI pilot's skill. Points are whole numbers computed using 100 × AI skill × full-precision aircraft factor × full-precision relative weapon factor, then rounded once at the end. Columns in the table display no more than two decimal places.

#### Table 27 — Annex C.2: F-4E human kills MiG-21bis AI, all weapons

| F-4E weapon | Cadet | Rookie | Trained | Veteran | Ace |
| --- | --- | --- | --- | --- | --- |
| M61 Vulcan | 59 | 79 | 99 | 118 | 138 |
| AIM-4D Falcon | 76 | 102 | 127 | 153 | 178 |
| AIM-9B Sidewinder | 70 | 93 | 117 | 140 | 163 |
| AIM-9E Sidewinder | 65 | 87 | 109 | 131 | 153 |
| AIM-9J Sidewinder | 61 | 82 | 102 | 123 | 143 |
| AIM-7D Sparrow | 66 | 88 | 111 | 133 | 155 |
| AIM-7E Sparrow | 65 | 87 | 108 | 130 | 152 |

Acceptance-test interpretation: the gun earns 99 points against Trained AI; AIM-7E earns 108; AIM-4D earns 127. The AIM-4D has the highest points because the catalogue rates it as the more difficult weapon, not because it is superior in combat. These awards are conditional on an actual recorded kill and do not imply equal probability of achieving that kill with every weapon.

Weapon-fit note: historical compatibility does not guarantee carriage in the particular DCS mission or module build. Use the actual weapon ID credited by DCS and validate the specific F-4E block and loadout. AIM-4D was technically compatible with F-4E but its Vietnam combat use was limited/uncertain; optional testing may be disabled by a historically constrained mission profile.

Historical source checks: USAF F-4E internal cannon — https://www.163atkw.ang.af.mil/About-Us/Fact-Sheets/Display/Article/452913/mcdonnell-douglas-f-4e-phantom-ii/ ; Vietnam-era F-4E illustrative loads — https://www.cybermodeler.com/aircraft/f-4/f-4e_loads.shtml ; AIM-4D compatibility — https://www.ausairpower.net/Falcon-Evolution.html ; Vietnamese 1972 MiG-21PFM service — https://vietnam.warbirdsresourcegroup.org/mig21_history_vietnam.html . These are historical cross-checks; scoring numbers come from the two project catalogues.

Status: Annex C is a Vietnam-era equipment comparison using existing F-4E late and MiG-21bis catalogue ratings, for v0.3 programmer acceptance review. Annexes A and B and the main specification remain unchanged. Annex D follows; Annexes E–F remain unpopulated.

## Annex D — Late Cold War (mid-1980s): F-14A and F-16A versus MiG-29A

Scope: test human F-14A Tomcat and original F-16A Block 15 (infrared-only) against a mid-1980s MiG-29A Fulcrum 9.12 AI at Cadet, Rookie, Trained, Veteran and Ace. One table per player aircraft lists all selected period-appropriate weapons, followed by the scored awards across five AI skills. This is a model acceptance comparison, not a claim that every period loadout was fitted to one mission.

Aircraft and catalogue qualification: A2A V7.0 has F-14A 790 and MiG-29 early 760. The aircraft snapshot does **not** contain a separate F-16A row. The earlier Cold War scoring discussions used **F-16A (IR-only) 780**; that number is carried into this annex as a **provisional legacy rating pending formal addition to V7.0**, rather than silently borrowing F-16C early 805. The ADF/Sparrow-equipped F-16A (earlier model figure 795) is excluded. The MiG-29 AI best weapon is **R-73 Archer (AAM-052, catalogue multiplier 0.880)**, for the **specific MiG-29A test loadout assumed to carry R-73**. Its R-27R has a higher multiplier and is not selected in this case; mission loadouts that omit R-73 require a new loadout-sensitive reference.

Method: ROUND[100 × AI skill factor × aircraft factor × {1 + 0.50 × (catalogue multiplier of credited player weapon − 0.880)}]. Source precision is retained internally; displayed weapon and aircraft factors use **no more than two decimal places**, final points are integers. Human player rank does not enter this PvAI calculation.

### Annex D.1 — Human F-14A Tomcat kills MiG-29A AI

Fixed inputs: player F-14A, rating 790; AI MiG-29A, rating 760, best weapon R-73 (AAM-052, 0.880). Aircraft factor 0.9138670159555764 (display **0.91**). Available selection: M61, AIM-9L/M, AIM-7F/M, AIM-54A/C. These are **alternative credited kill weapons**, not an assertion they must all be carried simultaneously. AIM-54C is a later mid-1980s alternative: NAVAIR notes 1985 fleet deployment while a US Navy training plan records formal IOC in December 1986. If a mission is tightly set before C availability, use AIM-54A and omit that row.

#### Table 28 — Annex D.1: F-14A weapon factors against MiG-29A

| F-14A weapon | Weapon ID | Catalogue multiplier | Relative weapon factor |
| --- | --- | ---: | ---: |
| M61 Vulcan 20 mm | AAM-018 | 1.20 | 1.16 |
| AIM-9L Sidewinder | AAM-043 | 1.00 | 1.06 |
| AIM-9M Sidewinder | AAM-044 | 0.94 | 1.03 |
| AIM-7F Sparrow | AAM-069 | 1.12 | 1.12 |
| AIM-7M Sparrow | AAM-070 | 1.05 | 1.08 |
| AIM-54A Phoenix | AAM-076 | 0.84 | 0.98 |
| AIM-54C Phoenix (later fit) | AAM-077 | 0.79 | 0.96 |

#### Table 29 — Annex D.1: F-14A human kills MiG-29A AI; points

| Credited F-14A weapon | Cadet | Rookie | Trained | Veteran | Ace |
| --- | ---: | ---: | ---: | ---: | ---: |
| M61 Vulcan | 64 | 85 | 106 | 127 | 148 |
| AIM-9L Sidewinder | 58 | 77 | 97 | 116 | 136 |
| AIM-9M Sidewinder | 56 | 75 | 94 | 113 | 132 |
| AIM-7F Sparrow | 61 | 82 | 102 | 123 | 143 |
| AIM-7M Sparrow | 59 | 79 | 99 | 119 | 138 |
| AIM-54A Phoenix | 54 | 72 | 90 | 107 | 125 |
| AIM-54C Phoenix (later fit) | 52 | 70 | 87 | 105 | 122 |

### Annex D.2 — Human F-16A Block 15 kills MiG-29A AI

Fixed inputs: player F-16A Block 15, original non-ADF infrared-only fit, **provisional aircraft rating 780** based on the earlier scoring project discussion; AI MiG-29A rating 760, best R-73 (AAM-052, 0.880). Aircraft factor 0.9424988722154628 (display **0.94**). Period-appropriate player weapon alternatives: M61, AIM-9L and AIM-9M. Do not add AIM-7 Sparrow, AIM-120 AMRAAM, or F-16A ADF/MLU equipment to this baseline.

#### Table 30 — Annex D.2: F-16A weapon factors against MiG-29A

| F-16A weapon | Weapon ID | Catalogue multiplier | Relative weapon factor |
| --- | --- | ---: | ---: |
| M61 Vulcan 20 mm | AAM-018 | 1.20 | 1.16 |
| AIM-9L Sidewinder | AAM-043 | 1.00 | 1.06 |
| AIM-9M Sidewinder | AAM-044 | 0.94 | 1.03 |

#### Table 31 — Annex D.2: F-16A human kills MiG-29A AI; points

| Credited F-16A weapon | Cadet | Rookie | Trained | Veteran | Ace |
| --- | ---: | ---: | ---: | ---: | ---: |
| M61 Vulcan | 66 | 87 | 109 | 131 | 153 |
| AIM-9L Sidewinder | 60 | 80 | 100 | 120 | 140 |
| AIM-9M Sidewinder | 58 | 78 | 97 | 116 | 136 |

Acceptance tests, listed Cadet / Rookie / Trained / Veteran / Ace:
- F-14A M61: **64 / 85 / 106 / 127 / 148**; F-14A AIM-54A: **54 / 72 / 90 / 107 / 125**; F-14A AIM-54C: **52 / 70 / 87 / 105 / 122**.
- F-16A M61: **66 / 87 / 109 / 131 / 153**; F-16A AIM-9L: **60 / 80 / 100 / 120 / 140**; F-16A AIM-9M: **58 / 78 / 97 / 116 / 136**.

Implementation notes: import/confirm the original F-16A IR-only 780 entry into the aircraft catalogue before treating the F-16A cases as production acceptance tests; track exact year, country, flight model and available DCS loadouts. Do not override the selected MiG-29 best-weapon reference or round intermediate multipliers. Earlier Annexes A–C and Tables 1–27 remain unchanged. Annexes E–F remain planned.

Historical source checks: F-16A initial service and 1981 growth provisions — https://www.af.mil/News/Article/104505/f-16-fighting-falcon/ ; 1980s F-16A with AIM-9L — https://www.airandspaceforces.com/app/uploads/1994/10/1094_Oct1994.pdf ; US Navy Phoenix dates — https://www.navair.navy.mil/node/12701 ; US Navy training plan (AIM-54C formal IOC December 1986) — https://www.globalsecurity.org/military/library/policy/navy/ntsp/aim54.htm .

Status: Annex D is recalculated for v0.4 with globally consistent Weapons Catalogue v1.1 Phoenix factors. Existing F-14A 790 and MiG-29A 760 aircraft ratings remain in force for these test tables; legacy F-16A IR-only 780 is provisional until entered in V7.0. The earlier avionics sensitivity trials are not implemented.


### Annex D.3 — Loadout-sensitive AI F-14A reference test (new Table 32)

Purpose: validate that the *same* MiG-29A human, using R-73 AAM-052 (0.88), receives different scores when the **F-14A AI's actual weapons differ**, while the R-73 missile's **global multiplier stays 0.88**. Fixed player aircraft MiG-29A rating 760; F-14A AI rating 790; aircraft factor 1.086132984 (display 1.09). All tests assume the F-14A carries an AIM-9M (0.94) and M61 (1.20), with Phoenix loadout varied as specified. A value is not selected from an uncarried weapon.

#### Table 32 — Annex D.3: Human MiG-29A/R-73 against F-14A AI under three weapon loadouts

| F-14A AI weapons available | Selected AI reference | Reference multiplier | Cadet | Rookie | Trained | Veteran | Ace |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| AIM-54C + AIM-9M + M61 | AIM-54C (AAM-077) | 0.79 | 68 | 91 | 114 | 136 | 159 |
| AIM-54A + AIM-9M + M61 | AIM-54A (AAM-076) | 0.84 | 66 | 89 | 111 | 133 | 155 |
| AIM-9M + M61 (no Phoenix) | AIM-9M (AAM-044) | 0.94 | 63 | 84 | 105 | 126 | 147 |

Interpretation: a more capable equipped enemy earns more points for an R-73 kill because its effective reference multiplier is lower. This is **not** a per-carrier weapon-factor change. The fixed F-14A aircraft rating in this test has not yet been made loadout-sensitive; any future aircraft-configuration rating changes would require a separate full-catalogue audit.

## 9. Version 0.4 changes, avionics review and remaining decisions

### 9.1 — Amendments accepted in v0.4

- One exact AAM catalogue ID has one global multiplier regardless of launching platform.
- Weapons Catalogue v1.1 applies AIM-54A scoring **0.84** from raw difficulty **0.68**, and AIM-54C scoring **0.79** from raw difficulty **0.58**. The 50% raw-to-scoring formula remains unchanged. AIM-9L **1.00**, AIM-9M **0.94**, M61 **1.20** are unchanged.
- A gun kill on the F-14A is more rewarding than AIM-9L/M and Phoenix kills, on otherwise identical AI skill, aircraft and target-loadout assumptions.
- Reference best weapon follows the opponent's actual period-valid and loaded offensive A2A weapons, rather than a static all-era aircraft-family label. For an unarmed/defensive-only AI target, use the player's actually equipped offensive A2A reference.
- Table 28, Table 29 and Annex D acceptance tests are recalculated; new Table 32 verifies the loadout rule. Tables 1–27 and 30–31 retain their previous results because their example weapon ratings and reference multipliers do not change.

### 9.2 — Aircraft avionics audit (not yet approved for rating changes)

V7.0 retains aircraft F-14A **790**, F-14B **805**, early MiG-29A **760**, F-16C early **805**; the annex's infrared-only F-16A **780** is a separate legacy provisional value not presently listed in V7.0. Candidate sensitivity values of F-14A Phoenix-equipped **830**, MiG-29A **780**, F-16A IR-only **765** were explored in an earlier analysis but are **not approved or substituted into Annex D**.

Before changing any 199-aircraft rating, use consistent historical fit and assess radar discovery range under target/altitude conditions, track/multi-target engagement, fire-control quality, IFF/data link and situational awareness, ECM/ECCM, weapon compatibility and control support, kinematics and energy manoeuvrability, and close-range off-boresight engagement. Rate the aircraft's actual historical fit rather than anachronistic later weapons. Compare F-14A against F-14B and other nearby aircraft; an isolated increase that reverses an upgraded variant without reason is not acceptable. Decide separately whether the platform rating will be fixed per subtype or variable per actual mission loadout.

For DCS reference evidence, Heatblur documents AWG-9 tracking up to 24 tracks and controlling up to six Phoenix engagements, subject to radar mode and limitations (https://f14.manuals.heatblur.se/f14ab/systems/radar/general_operation.html). Heatblur documents AIM-54A/C guidance and launch-mode dependencies (https://f14.manuals.heatblur.se/f14ab/stores/air_to_air/aim_54.html). NAVAIR describes the AIM-54C improvements and deployment history (https://www.navair.navy.mil/node/12701). These support the avionics-review priorities, **not** a numerically proven +40-point F-14A rating adjustment.

### 9.3 — Remaining implementation decisions

- Verify DCS can supply the exact credited weapon ID and **actual target loadout** at the event time. If not, define a verified mission-configured loadout mapping or leave the case pending review; do not guess a best weapon.
- Verify consistency of AAM-077 AIM-54C **0.79** compared with AAM-079 AIM-120B **0.80**. The presently selected Phoenix numbers are deliberately preserved for testing; catalogue-wide order/calibration remains open.
- Aircraft V7.0 currently maps F-14A/F-14B static best A2A weapon to AIM-9M despite carrying Phoenix in many configurations. This field is an unvalidated legacy default, not sufficient for v0.4 scoring. All 199 rows require a best-weapon/loadout audit.
- Add and verify F-16A IR-only (780 provisional) as a distinct platform/fit before using its test cases in a production implementation.
- Retain the full-precision internal values, two-decimal-place presentation, and final-only whole-point rounding across all six planned era annexes.
- Section 6.5 remains pending approval; annexes E (Early Modern) and F (Modern) remain planned.

**Version preservation:** JIRA v0.3 and Weapons Catalogue v1.0 remain unchanged as historic references. This v0.4 draft is paired with the separate live Weapons Catalogue v1.1. The GitHub copies are dated snapshots and do not automatically synchronise with the live Google files.

