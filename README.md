# DCS 3.0 — Player vs AI Air-to-Air Scoring

**Current working version: JIRA implementation brief v0.4, 9 November 2026; Weapons Catalogue v1.1.** This repository preserves the project's historical reasoning and a dated snapshot of the current technical specification. The linked Google Docs and Google Sheets are the **authoritative live working copies**, not automatically synchronised with GitHub.

## Current model

**Kill points = ROUND(100 × AI skill factor × aircraft factor × relative weapon factor)**

- **AI skill (of the destroyed AI pilot):** Cadet 0.60, Rookie 0.80, Trained 1.00, Veteran 1.20, Ace 1.40.
- **Aircraft factor:** 2 / (1 + 10^((player aircraft rating − AI aircraft rating) / 400)).
- **Relative weapon factor:** 1 + 0.50 × (catalogue multiplier of the credited player weapon − catalogue multiplier of the reference weapon).
- **Reference weapon, armed AI:** the lowest-multiplier eligible offensive air-to-air weapon **actually carried by the AI target**.
- **Reference weapon, unarmed or defensive-only AI:** the lowest-multiplier eligible offensive air-to-air weapon **actually carried by the attacking player**.
- **Unknown or uncatalogued required weapon:** record for review; do not invent a 1.00 reference.
- **No era bonus:** the same formula applies to World War II, Korea, Cold War and modern combat. Pilot-versus-pilot scoring is out of scope.

The Weapons Catalogue v1.1 **scoring multiplier** already incorporates 50% compression from its raw weapon difficulty. One exact AAM weapon variant retains its universal multiplier on every aircraft. AIM-54A (AAM-076) is **0.84** and AIM-54C (AAM-077) is **0.79**; AIM-9L 1.00, AIM-9M 0.94 and M61 1.20 remain unchanged. The 0.50 in the *relative* factor weights the comparison of **two existing multipliers**; never compress the catalogue factor a second time.

**Presentation standard:** Human-readable tables show factors and weapon multipliers to **no more than two decimal places**; points awarded are whole numbers. Code and calculations must retain the **original source precision** and round only the final points award. The CSV data snapshots deliberately preserve the original catalogue precision.

## Current documents

- [Complete JIRA implementation brief v0.4, Tables 1–38 and Annexes A–F](docs/JIRA-Implementation-Brief-v0.4.md)
- [Archived JIRA implementation brief v0.3, Tables 1–31](docs/JIRA-Implementation-Brief-v0.3.md)
- [Aircraft A2A Ratings V7.0 snapshot — 199 aircraft, including best-weapon mappings](data/aircraft-a2a-ratings-v7.csv)
- [A2A Weapons Catalogue v1.1 snapshot — 101 weapon entries, revised Phoenix](data/weapons-catalogue-v1.1.csv)
- [Archived A2A Weapons Catalogue v1.0 snapshot](data/weapons-catalogue-v1.csv)

### Live working files (authoritative)

- [JIRA scoring specification v0.4 — Google Docs](https://docs.google.com/document/d/1vWaoJLwBNJmNQKArwjg1slQY7EOgR-pvnM7OnytzPtc/edit)
- [Archived JIRA scoring specification v0.3 — Google Docs](https://docs.google.com/document/d/1PzCKKs0nBholQecxu-0vyKEeAWORqKzvBH5lbXl2JsY/edit)
- [Aircraft A2A Ratings V7.0 — Google Sheets](https://docs.google.com/spreadsheets/d/1Whm0B302ITesvfZ9S_Z-5VWFTclyoMdodv5AnfAAb6s/edit)
- [A2A Weapons Catalogue v1.1 — Google Sheets](https://docs.google.com/spreadsheets/d/1k89dwrlw7CEEDyPfPrMp6X3TM9Pu0PX_nT7xDa2PUBc/edit)
- [Archived A2A Weapons Catalogue v1.0 — Google Sheets](https://docs.google.com/spreadsheets/d/1Hf0MztwidHXaa0pqy27kPtgkP_10pxLWLkHhH28gUpE/edit)

## Acceptance test coverage

- **Annex A — World War II:** Spitfire IX vs Bf 109K-4, both directions (Tables 20–21).
- **Annex B — Korea:** F-86F Sabre vs MiG-15bis, both directions; P-51D Mustang vs MiG-15bis, both directions (Tables 22–25).
- **Annex C — Vietnam War (Early Cold War):** F-4E using seven Vietnam-era air-to-air weapons against MiG-21bis AI at all five levels (Tables 26–27). The MiG-21bis is a standardised model opponent, not a claim of exact historical North Vietnamese equipment.
- **Annex D — Late Cold War:** F-14A Tomcat and infrared-only F-16A Block 15 against MiG-29A (Tables 28–31) plus F-14A actual-loadout reference selection (Table 32). F-16A 780 is an earlier project rating, provisionally carried over and pending addition to the V7.0 aircraft catalogue; AIM-54C is a later mid-1980s alternative.
- **Annex E — Early Modern:** Human F/A-18C Lot 20 using M61, AIM-7F, AIM-9L and AIM-120C against Su-27 AI, all five difficulty levels (Tables 33–34). Provisional late-Hornet rating 845 is an annex-only test assumption; Aircraft V7.0 still contains only F/A-18C early 800.
- **Annex F — Modern:** F/A-18C Lot 20 vs Su-27, both human-player directions; weapon tables and five AI levels (Tables 35–38). Hornet rating 845 remains provisional; target reference weapon is loadout-sensitive.

The v0.4 specification includes test cases for unarmed F-111F and defensive-only targets. Actual aircraft and weapon catalogue mappings are **provisional** and may be variant/loadout dependent. The 199-aircraft snapshot currently contains 141 selected catalogue IDs, seven uncatalogued weapons, 36 unarmed aircraft and 15 defensive-only cases.

## Historical documentation — retained for development provenance

- [Chess ELO Scoring Explained](docs/Chess-ELO-Scoring-Explained.md) — original aircraft-rating and pilot-model development.
- [DCS Stats and Scoring 2](docs/DCS-Stats-and-Scoring-2.md) — **superseded** 6 November 2026 proposal to compress weapon multipliers 90% toward neutral.

Those documents are **not** the current implementation specification. Do not implement their historical weapon factors or scores. The active approach is Weapons Catalogue **v1.1** with **50% raw compression**, a **relative opponent-weapon comparison**, and **loadout-sensitive reference selection**. Aircraft ratings V7.0 remain unchanged pending avionics audit; original F-16A IR-only 780 is provisional.

This repository contains specifications and data snapshots, not an executable DCS scoring implementation.
