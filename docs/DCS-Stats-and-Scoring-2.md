# DCS Stats and Scoring 2

**Source:** current ChatGPT discussion on 6 November 2026.  
**Status:** project notes, not a full conversation export.

## Problem identified

The user observed that the weapon multipliers were having too much influence: WWII kills would be far more rewarding than many modern-aircraft kills, even before accounting for the aircraft factors and AI level.

The previously proposed response was to reduce the *distance from 1.0* of every weapon multiplier by **90%**:

`Revised Weapon Factor = 1 + 0.10 × (Original Weapon Factor − 1)`

This keeps the original weapon ordering but sharply reduces its impact.

## Weapon multiplier comparison

| Weapon | Original factor | Proposed revised factor |
|---|---:|---:|
| Hispano Mk II 20 mm | 1.80 | 1.080 |
| MK 108 30 mm | 1.95 | 1.095 |
| R-60M | 1.15 | 1.015 |
| AIM-7E Sparrow | 1.60 | 1.060 |
| AIM-120C AMRAAM | 0.73 | 0.973 |
| R-27ER Alamo-C | 1.20 | 1.020 |

For this example set, the weapon range compresses from 0.73–1.95 to 0.973–1.095.

## Six example results against Trained AI

The earlier discussion gave the following approximate comparison for the same opponent aircraft, AI level and weapon.

| Player aircraft → AI aircraft | Weapon | Original points | Proposed revised points |
|---|---|---:|---:|
| Spitfire IX → Bf 109K-4 | Hispano Mk II | 184 | 110 |
| Bf 109K-4 → Spitfire IX | MK 108 | 191 | 107 |
| MiG-21bis → F-4E | R-60M | 125 | 110 |
| F-4E → MiG-21bis | AIM-7E | 146 | 97 |
| F-16C Block 50 → Su-27 | AIM-120C | 64 | 85 |
| Su-27 → F-16C Block 50 | R-27ER | 135 | 115 |

Revised values are approximate because earlier points were rounded. For example, the original Spitfire score of 184 becomes approximately `184 × (1.08 / 1.80) = 110.4`.

## Elements deliberately unchanged

- Basic score = 100.
- Five AI levels, Cadet through Ace, with factors 0.60–1.40.
- Aircraft capability continues to be reflected in the aircraft factor.
- Focus remains exclusively player-versus-AI, not player-versus-player.

## Current status / open points

The weapon-factor compression was **proposed as a provisional revision**, not locked as an immutable final specification. The next step is to confirm whether this degree of compression gives the desired balance across the complete set of weapons and aircraft, not just six examples.

## Visible dialogue excerpts (not a complete transcript)

**User:** “I suppose the only thing that really bothers me is that because of the weapons multipliers. World War Two is going to be considerably more rewarding than. anything modern so it could be. We need to tighten up the weapons multiply is quite considerably because they are having a big effect”

**Assistant, summary of response:** Recommended a 90% compression of weapon multipliers toward 1.0; retained the existing points formula, aircraft factors and AI skill factors; compared six revised example scores.

**User:** “OK first of all can you. make a Github for this conversation and also the one that it is linked to called Chess elo scoring explained and can you rename this and call it DCS Stats and Scoring 2.”
