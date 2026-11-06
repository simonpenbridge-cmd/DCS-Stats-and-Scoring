# DCS Stats and Scoring

Project documentation drawn from two linked ChatGPT conversations:

1. **Chess ELO Scoring Explained** — development history and the player-versus-AI scoring model.
2. **DCS Stats and Scoring 2** — recent weapon-multiplier adjustment and comparison of six example kills.

## Current direction — 6 November 2026

The current scoring model concerns **player versus AI only**. Player-versus-player scoring is set aside for now.

**Kill Points = 100 × AI Skill Factor × Aircraft Factor × Weapon Factor**

- **AI skill:** Cadet 0.60; Rookie 0.80; Trained 1.00; Veteran 1.20; Ace 1.40.
- **Aircraft factor:** `2 / (1 + 10^((AR_Player − AR_AI)/400))`, based on aircraft ratings.
- **Weapon factor:** a *provisional* revision compresses each original multiplier 90% of the way toward 1.0 to reduce the disproportionate rewards for WWII weapon kills.

## Project notes

- [Chess ELO Scoring Explained](docs/Chess-ELO-Scoring-Explained.md)
- [DCS Stats and Scoring 2](docs/DCS-Stats-and-Scoring-2.md)

## Provenance and scope

These documents are **reconstructed project notes**, not complete verbatim chat exports. Some historical details are condensed from prior conversation context. Exact wording is preserved only where explicitly quoted.

No executable program code is included.
