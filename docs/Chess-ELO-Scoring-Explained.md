# Chess ELO Scoring Explained — DCS scoring development

**Source:** prior ChatGPT conversation of this title, most recent scoring decisions on 6 November 2026.  
**Status:** reconstructed technical notes; **not a full transcript**.

## How the work evolved

Earlier discussions considered an ELO-style system combining pilot skill with an aircraft capability rating, then a common ranking for aircraft from WWII through Cold War and later jets. Prior work explored different K values (including 20, 30 and 50), human-vs-human outcomes and AI comparisons. Those historical approaches are *not* the currently active scoring model.

The project then narrowed deliberately to **player-versus-AI kill scoring only**. Player-versus-player scoring was abandoned for the present. There are now five AI levels: Cadet, Rookie, Trained, Veteran and Ace.

## Current player-versus-AI model

`Kill Points = 100 × AI Skill Factor × Aircraft Factor × Weapon Factor`

### AI skill level

| AI difficulty | Skill factor |
|---|---:|
| Cadet | 0.60 |
| Rookie | 0.80 |
| Trained | 1.00 |
| Veteran | 1.20 |
| Ace | 1.40 |

A basic aircraft kill is worth 100 before adjustments.

### Aircraft factor

`Aircraft Factor = 2 / (1 + 10^((AR_Player − AR_AI)/400))`

Where AR_Player is the attacking player's aircraft rating, and AR_AI is the aircraft rating of the AI opponent. Defeating an aircraft rated more capable than one's own yields a higher factor.

Aircraft rating values referenced for six examples:

| Aircraft | AR |
|---|---:|
| Spitfire Mk IX | 417 |
| Messerschmitt Bf 109K-4 | 424 |
| MiG-21bis | 710 |
| F-4E Phantom II | 740 |
| F-16C Block 50 | 875 |
| Su-27 Flanker | 830 |

These are **project model ratings**, not real-world performance statistics.

### Weapon multipliers in the earlier chat (subsequently revised)

| Weapon | Earlier multiplier |
|---|---:|
| Hispano Mk II 20 mm | 1.80 |
| MK 108 30 mm | 1.95 |
| R-60M | 1.15 |
| AIM-7E Sparrow | 1.60 |
| AIM-120C AMRAAM | 0.73 |
| R-27ER Alamo-C | 1.20 |

### Six worked examples — earlier scoring (before revision)

The following tables use the original larger weapon multipliers. Each row is a kill by the first aircraft against an AI flying the second.

| Player aircraft → AI aircraft | Weapon used | Cadet | Rookie | Trained | Veteran | Ace |
|---|---|---:|---:|---:|---:|---:|
| Spitfire IX → Bf 109K-4 | Hispano Mk II | 110 | 147 | 184 | 220 | 257 |
| Bf 109K-4 → Spitfire IX | MK 108 | 115 | 153 | 191 | 229 | 268 |
| MiG-21bis → F-4E | R-60M | 75 | 100 | 125 | 150 | 175 |
| F-4E → MiG-21bis | AIM-7E | 88 | 117 | 146 | 175 | 205 |
| F-16C Block 50 → Su-27 | AIM-120C | 38 | 51 | 64 | 76 | 89 |
| Su-27 → F-16C Block 50 | R-27ER | 81 | 108 | 135 | 163 | 190 |

Numbers above were rounded in the earlier discussion. Note that the scoring difference between early cannon kills and modern missile kills led directly to the follow-up adjustment.

## Decision and next document

The player accepted the illustrative aircraft-factor outcomes as a useful provisional baseline and requested that a specific weapon be assigned to each of the six kills. The next discussion identified that the original weapon multipliers made WWII kills disproportionately rewarding relative to modern kills.

Continue to [DCS Stats and Scoring 2](DCS-Stats-and-Scoring-2.md).
