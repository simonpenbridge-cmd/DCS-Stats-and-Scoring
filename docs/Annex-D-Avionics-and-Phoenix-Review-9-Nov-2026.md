# Annex D review — avionics weighting and AIM-54 Phoenix (9 November 2026)

**Status: OPEN / NON-AUTHORITATIVE SCENARIO. No changes have been approved to the Aircraft V7.0 or Weapons v1.0 catalogues, nor to JIRA Implementation Brief v0.3 or Annex D Tables 28–31.** This is a what-if analysis generated from user feedback on DCS F-14/Phoenix matchups.

## Issue

Mid-1980s F-14A AWG-9/AIM-54 combination can initiate long-range engagements well outside an infrared-only F-16A's missile envelope and outside many R-27R MiG-29A engagements. Current catalogue F-14A 790 / early MiG-29 760 and legacy F-16A infrared-only 780 put the F-14A only 10 above F-16A, potentially underweighting avionics, radar/fire-control and weapon *integration* for BVR air combat. The AIM-54A 1.175 and AIM-54C 1.085 catalogue difficulty multipliers currently award more points for a Phoenix kill than an AIM-9M 0.940 kill against the same target. User questions this result.

## Separate concepts

- Platform capability: detection and targeting systems, radar/fire-control, electronic protection, target acquisition, manoeuvrability, ability to employ an actual loadout. Its rating should reflect a defined historical configuration and chosen operational context, not just an airframe name.
- Weapon kill-difficulty multiplier: relative difficulty of achieving a *credited kill* with the missile/gun in the scenario. Smaller means more effective or easier, **hence fewer points**. This is not a ranking of destructive power alone.
- Do not count the same advantage twice. Review both factors together, and separate **F-14A with Phoenix** and **F-14A without Phoenix** in future loadout-aware implementation if necessary.
- MiG-29A 9.12 R-73 with helmet-mounted sight is potent at close range; F-14A's BVR advantage does not make it categorically superior in every geometry. Original infrared-only F-16A and later F-16A ADF (Sparrow) must remain distinct.

## Rating sensitivity scenario — analyst trial values, not accepted ratings

| Aircraft / loadout | Current project rating | Illustrative review rating | Change | Reason for review |
| --- | ---: | ---: | ---: | --- |
| F-14A with AIM-54 (AWG-9) | 790 | **830** | +40 | BVR detection, tracking and Phoenix reach not sufficiently differentiated |
| MiG-29A early (R-27R, R-73) | 760 | **780** | +20 | Combined BVR access plus R-73 and helmet sight WVR strengths |
| F-16A Block 15, original IR-only | 780 (legacy provisional) | **765** | −15 | No radar-guided BVR engagement capability, strong WVR kinematics retained |

These values are **illustrative** and not obtained from a new universal 199-aircraft weighting formula. They must be calibrated to neighbouring eras and variants (notably current F-14B 805, F-16C early 805, Mirage 2000C 785, Su-27 baseline 830 and F-15C AMRAAM 850) before considering adoption.

For completeness, a loadout with no Phoenix cannot automatically use the same 830 rating in a true loadout-sensitive model. A separate no-Phoenix F-14A configuration would need assessment. The entire rating catalogue and DCS-equipped configurations should be reconciled before production.

## Weapon sensitivity scenario — analyst trial values

| Weapon | Current catalogue multiplier | Illustrative review multiplier | Interpretation |
| --- | ---: | ---: | --- |
| AIM-9L Sidewinder | 1.00 | 1.00 | unchanged |
| AIM-9M Sidewinder | 0.94 | 0.94 | unchanged |
| AIM-54A Phoenix | 1.18 | **0.90** | now marginally easier than AIM-9M in reference BVR context |
| AIM-54C Phoenix | 1.09 | **0.85** | improved seeker relative to AIM-54A |
| M61 Vulcan | 1.20 | 1.20 | unchanged |

These are **trial assumptions**, not evidence-derived kill probabilities; use paired controlled DCS tests at varied range, aspect, altitude, target manoeuvre and countermeasure settings before adopting. Phoenix advantages depend on AWG-9 support and launch conditions.

## Points effect — Trained MiG-29A AI, existing v0.3 formula

Relative weapon reference remains MiG-29A R-73 (AAM-052) with original source multiplier 0.880. Old column reproduces approved Annex D Tables 29 and 31. Trial column changes aircraft ratings and Phoenix multipliers simultaneously, isolating no other variables.

| Human aircraft | Credited weapon | Approved current score | Trial score |
| --- | --- | ---: | ---: |
| F-14A | M61 | 106 | 99 |
| F-14A | AIM-9L | 97 | 91 |
| F-14A | AIM-9M | 94 | 88 |
| F-14A | AIM-7F | 102 | 96 |
| F-14A | AIM-7M | 99 | 93 |
| F-14A | AIM-54A | 105 | 87 |
| F-14A | AIM-54C | 101 | 84 |
| F-16A | M61 | 109 | 121 |
| F-16A | AIM-9L | 100 | 111 |
| F-16A | AIM-9M | 97 | 107 |

## Full five-skill acceptance *comparison only* — trial scenario

These are **not** new approved programmer acceptance values.

| Human aircraft | Credited weapon | Cadet | Rookie | Trained | Veteran | Ace |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| F-14A | M61 | 60 | 80 | 99 | 119 | 139 |
| F-14A | AIM-9L | 55 | 73 | 91 | 109 | 127 |
| F-14A | AIM-9M | 53 | 71 | 88 | 106 | 124 |
| F-14A | AIM-7F | 58 | 77 | 96 | 115 | 134 |
| F-14A | AIM-7M | 56 | 74 | 93 | 111 | 130 |
| F-14A | AIM-54A | 52 | 69 | 87 | 104 | 121 |
| F-14A | AIM-54C | 51 | 68 | 84 | 101 | 118 |
| F-16A | M61 | 73 | 97 | 121 | 145 | 169 |
| F-16A | AIM-9L | 66 | 88 | 111 | 133 | 155 |
| F-16A | AIM-9M | 64 | 86 | 107 | 129 | 150 |

Source-precision weapon values must be used in implementation, not the two-decimal display values. Aircraft factor in trial F-14A vs MiG-29A = 0.85707377 (display 0.86), F-16A vs MiG-29A = 1.04314667 (display 1.04).

## Potential downstream effects requiring an audit

1. Current F-14A and F-14B BEST A2A WEAPON (CATALOGUE ID) points to **AAM-044 AIM-9M**. If Phoenix multipliers move below 0.94, a Phoenix-equipped, mid-1980s F-14A target may instead reference AIM-54A (AAM-076) or AIM-54C (AAM-077) depending on loadout and date. This changes scores when F-14 is **the AI opponent**, not just the player.
2. Relative-best rule might be conditioned on the *actual AI loadout*, not a generic all-era best-weapon label. Do not assume the Phoenix is loaded on every F-14.
3. Future recalibration should include the F-14B and F-15C, and avoid making F-14A 830 while leaving the more capable F-14B 805 without justification.
4. Historical era constraint: the intended mid-1980s MiG-29A uses R-27R/R-73, not an assumed R-27ER fit. DCS user-selectable loadouts may span multiple upgrade eras.
5. Weapon scoring is a difficulty/ease proxy, not an empirical hit probability. Validate in controlled DCS cases against fighter targets rather than extrapolating missile brochure range.

## External checks

- Heatblur F-14 radar manual: https://f14.manuals.heatblur.se/f14ab/systems/radar/general_operation.html (24 tracks, up to 6 concurrent Phoenix engagements).
- Heatblur AIM-54 manual: https://f14.manuals.heatblur.se/f14ab/stores/air_to_air/aim_54.html (50–60 NM fighter-target BVR examples, guidance implementation and limitations).
- Eagle Dynamics MiG-29A announcement: https://www.digitalcombatsimulator.com/en/news/newsletters/becffd86caa2144cbf38545ccb163bdd/ (R-73/HMS and BVR R-27 capabilities).
- Oregon ANG F-16 ADF history: https://www.142wg.ang.af.mil/News/Article-Display/Article/3663270/oregons-viper-years/ (ADF retrofit and radar-guided Sparrow integration, distinct from original Block 15).

**No source-of-truth changes:** this review does not modify the live Google Docs, Google Sheets, aircraft CSV or weapons CSV. If the user approves a candidate model, update sources of truth and revise Annex D and acceptance tables together.


## Refinement 2 — user-selected Phoenix trial values and universal weapon IDs (9 November 2026)

**This refinement supersedes only the earlier illustrative AIM-54A 0.90 and AIM-54C 0.85 *trial proposals* above. Those earlier trial calculations remain preserved as an audit record. Aircraft and weapon authoritative catalogues and approved JIRA Annex D are unchanged.**

User feedback: (i) an F-14 gun kill should earn more points than an AIM-9L or AIM-9M kill with otherwise identical inputs; (ii) AIM-9M multiplier 0.94 is close enough to AIM-9L's 1.00; (iii) AIM-54A should be 0.10 better (= **0.84**) than AIM-9M and AIM-54C a further 0.05 better (= **0.79**); (iv) **one universal catalogue multiplier per exact weapon variant/ID, independent of carrier aircraft**.

### Refined candidate global weapon values

| Exact weapon | Catalogue ID | Current approved multiplier | Refined candidate | Reason |
| --- | --- | ---: | ---: | --- |
| M61 Vulcan | AAM-018 | 1.20 | 1.20 | Gun kill still hardest among these choices |
| AIM-9L | AAM-043 | 1.00 | 1.00 | Anchor unchanged |
| AIM-9M | AAM-044 | 0.94 | 0.94 | Accepted distinction from AIM-9L |
| AIM-54A | AAM-076 | 1.18 (source 1.175) | **0.84** | 0.10 below AIM-9M |
| AIM-54C | AAM-077 | 1.09 (source 1.085) | **0.79** | 0.05 below AIM-54A |

These multipliers represent proposed *global weapon catalogue* changes, **not F-14-specific values**. Example: AIM-9M (AAM-044) uses 0.94 when carried by an F-14A, F-16A, F-15C or any other aircraft. Final kill **points** may still differ between aircraft and AI targets because aircraft capability, AI skill, and the AI target's reference weapon change. That is correct under existing v0.3 methodology.

The proposed 0.79 for AIM-54C is slightly below current AIM-120B 0.80; before a universal catalogue update, review the complete inter-missile ordering for consistency, not just the F-14. Catalogue v1.0 raw-difficulty field and scoring multiplier have a defined relationship: if the refined values are accepted, update the raw value or explicitly revise catalogue derivation consistently rather than leaving contradictory columns.

### Resulting F-14A against MiG-29A scores — *existing* aircraft ratings 790 vs 760

AI MiG-29A best-weapon reference remains R-73 (AAM-052), source multiplier 0.88. This first comparison changes **only Phoenix weapon multipliers**; original aircraft factor is 0.91 displayed.

| Player F-14A weapon | Relative weapon factor | Cadet | Rookie | Trained | Veteran | Ace |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| M61 (1.20) | 1.16 | 64 | 85 | 106 | 127 | 148 |
| AIM-9L (1.00) | 1.06 | 58 | 77 | 97 | 116 | 136 |
| AIM-9M (0.94) | 1.03 | 56 | 75 | 94 | 113 | 132 |
| AIM-54A (0.84) | 0.98 | 54 | 72 | 90 | 107 | 125 |
| AIM-54C (0.79) | 0.96 | 52 | 70 | 87 | 105 | 122 |

### Resulting F-14A against MiG-29A scores — *earlier proposed* aircraft ratings 830 vs 780

This second comparison **also** assumes the provisional aircraft-rating review (not approved); aircraft factor is 0.86 displayed.

| Player F-14A weapon | Cadet | Rookie | Trained | Veteran | Ace |
| --- | ---: | ---: | ---: | ---: | ---: |
| M61 | 60 | 80 | 99 | 119 | 139 |
| AIM-9L | 55 | 73 | 91 | 109 | 127 |
| AIM-9M | 53 | 71 | 88 | 106 | 124 |
| AIM-54A | 50 | 67 | 84 | 101 | 118 |
| AIM-54C | 49 | 65 | 82 | 98 | 115 |

The F-14 gun scores exceed those of both Sidewinders and Phoenix missiles in either comparison. The sole base multiplier per exact weapon remains fixed regardless of platform. Final scores are produced from full-precision catalogue input and aircraft factor, rounding only the final award.

**Next calibration action:** check the entire 101-weapon catalogue ordering and 199-aircraft baseline for consistency (including F-14B current rating 805) before replacing approved values or acceptance tests. Do not silently overwrite the previous trials or the live Google Docs and Sheets.
